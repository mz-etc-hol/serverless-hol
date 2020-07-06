# Amazon Serverless Hands On - Todo Web App 구성하기

![](images/todo-app-main.png)

이 실습은 101과 201정도의 레벨로 기획되었습니다.

간단한 ToDo web application을 만들어보고 AWS의 서버리스 서비스들을 이용하여 직접 Web Application 구축할 수 있도록 경험하는데 목표를 갖고 제작되었습니다.

다음은 본 Hands-on 과정에서 사용하는 AWS의 서버리스 서비스들입니다.

- Amazon API Gateway
- AWS Lambda
- Amazon DynamoDB
- Amazon S3

위의 4가지 서비스들 각각에는 굉장히 다양한 기능들이 존재하기 때문에 너무 자세한 내용 설명보다는 기본적인 기능 설명 및 실습으로 AWS의 서버리스 서비스가 익숙치 않은 분들도 쉽게 따라하실 수 있도록 구성하였으니 이점 참고 바랍니다.

## 실습 소개

### 실습 목적

1. Amazon S3의 "정적 웹 사이트 호스팅" 기능을 이용하여 웹서버를 직접 구성하지 않아도 손쉽게 홈페이지를 만들 수 있습니다.
1. Amazon API Gateway의 기본적인 구성에 대해 알아보고 AWS Lambda와의 통합 기능에 대한 기본적인 기능을 확인합니다.
1. Amazon DynamoDB 서비를 이용한 NoSQL 서비스에 대한 기본적인 기능에 대해 알아봅니다.
1. Amazon S3의 정적 웹 사이트 호스팅 기능에 대해 알아보고 간단한 웹사이트를 구성해 봅니다.

### 실습 비용

본 Hands-on은 약 1시간 이내에 실습을 마칠 수 있도록 작성되었습니다. AWS 프리 티어 기간에 해당되시는 고객이라면 별도의 추가 비용이 발생하지 않는 범위내에서 실습을 진행하실 수 있습니다. 프리 티어 기간이 지났더라도 큰 비용 부담이 안되는 선에서 실습을 해볼 수 있으니 참고바랍니다.

### 실습 종료 후 리소스 삭제

실습이 종료되고 나면 리소스를 반드시 삭제하시기 바랍니다. AWS Lambda를 제외한 3가지 서비스들은 삭제하지 않을 경우, 일정 비용이 지속해서 발생할 수 있으며, Amazon API Gateway같은 경우, URL 이 노출되면 의도치 않게 많은 비용이 발생할 수 있으니 실습이 끝나면 나면 반드시 생성했던 내용들은 직접 모두 삭제하시기 바랍니다.

## Todo Web application architecture

아래 다이어그램은 여러분이 구축할 Todo Web application의 모습니다. 직접 서버를 만##들지 않는 Amazon의 Serverless 서비스들을 이용하기 때문에 서버의 용량 선정이나 관리 방법에 대해 고민할 필요 없이 사용된 만큼의 비용만 지불됩니다. 이러한 장점은 개발자들에게는 서버의 관리없이, application 개발과 성능에만 집중할 수 있도록 도와줍니다.

![](images/todo-app-architecture.png)

### Todo Web App의 구성

Todo Web App은 여섯 가지 기능으로 구성됩니다.

1. **정적 웹페이지**
   1. Todo Web App의 화면을 처리할 HTML 파일(index.html)과 동적 처리를 담당할 Javascript 파일(todo.js) 2개로 구성됩니다.
   1. 이 2개 파일은 Amazon S3에 업로드되며, 정적 웹 사이트 호스팅 기능을 이용하여 브라우저에서 바로 접근이 가능합니다.
1. **fetch-todo API** - 등록된 Todo 불러오기
   1. 등록된 Todo 데이터를 Amazon API Gateway를 통해 정적 웹페이지에로 불러옵니다.
   1. Amazon API Gateway는 AWS Lambda 함수인 "fetch-todo"를 연결합니다.
   1. "fetch-todo" 함수는 Amazon DynamoDB 테이블인 "TODO"에서 불러와(DynamoDB의 Scan) Todo 데이터를 반환합니다.
1. **add-todo API** - 새로운 Todo 등록
   1. 새로 등록할 Todo 데이터를 Amazon API Gateway를 통해 Restful API로 전달받습니다.
   1. Amazon API Gateway는 새로 등록할 Todo 데이터를 AWS Lambda 함수인 "add-todo"에 연결합니다.
   1. "add-todo" 함수는 Todo 데이터를 Amazon DynamoDB 테이블인 "TODO"에 저장합니다.
1. **update-todo API** - Todo 수정
   1. 수정된 Todo 데이터를 Amazon API Gateway를 통해 Restful API로 전달받습니다.
   1. Amazon API Gateway는 수정된 Todo 데이터를 AWS Lambda 함수인 "update-todo"에 연결합니다.
   1. "update-todo" 함수는 수정된 Todo 데이터의 고유 ID인 "TODO_ID"와 동일한 데이터를 Amazon DynamoDB 테이블인 "TODO"에서 찾아 갱신합니다.
1. **remove-todo API** - Todo 삭제
   1. 삭제할 Todo 데이터를 Amazon API Gateway를 통해 Restful API로 전달받습니다.
   1. Amazon API Gateway는 삭제할 Todo 데이터를 AWS Lambda 함수인 "remove-todo"에 연결합니다.
   1. "remove-todo" 함수는 삭제할 Todo 데이터의 고유 ID인 "TODO_ID"와 동일한 데이터를 Amazon DynamoDB 테이블인 "TODO"에서 찾아 삭제합니다.
1. **TODO** 테이블
   1. TODO 데이터를 보관하기 위해 Amazon DynamoDB에 "TODO" 라는 테이블을 만들어 사용합니다.

## 실습

실습을 시작하기 전에 먼저 AWS 콘솔 화면으로 접속합니다.
https://console.aws.amazon.com/console
![](images/aws_console_home.png)

### 데이터 설계

TODO 데이터를 관리하기 위해 먼저 Amazon DynamoDB에 테이블을 생성합니다.

생성할 테이블 구조는 다음과 같습니다.

> 테이블 이름: **TODO**

| 속성 이름 (AttributeName) | 속성 유형 (AttributeType) | Primary Key 여부 |
| ----------------------- | :--------------------: | :--------------: |
| TODO_ID                 |          Number         |       Y         |
| TITLE                   |          String         |       -         |
| COMPLETED               |         Boolean         |       -         |

#### 테이블 만들기

1. AWS 콘솔 환경에서 DynamoDB로 이동합니다.
![](images/amazon_dynamodb_1.png)
화면 상단의 `서비스` 메뉴를 선택해서 이동할 수도 있습니다.

1. `테이블 만들기` 버튼을 클릭합니다.
![](images/amazon_dynamodb_2.png)
1. `테이블 이름`과 `기본 키`를 위에서 정의한 이름과 유형으로 선택한여 생성합니다.
![](images/amazon_dynamodb_3.png)
1. 생성된 테이블의 정보를 확인합니다.
![](images/amazon_dynamodb_4.png)


### Lambda 함수 & API Gateway 만들기

TODO 데이터를 처리할 함수 4개와 각 함수를 연결할 API Gateway를 생성합니다.

| 함수 이름       | 역할 |
| :-----------: | --- |
| fetch-todo    | DynamoDB에 저장되어 있는 TODO 목록을 조회 |
| add-todo      | 새로운 Todo 데이터를 DB에 추가 |
| update-todo   | 변경된 Todo 데이터를 DB에 저장(update) |
| remove-todo   | 선택한 Todo 데이터를 DB에서 삭제 |

Lambda 함수는 `Node.js 12.x`로 작성합니다.

함수를 만들기 전에 먼저 AWS Lambda 화면으로 이동합니다.
![](images/aws_lambda_1.png)

#### 1. "fetch-todo" Lambda 함수 만들기
1. Lambda 대시보드 화면에서 `함수 생성` 버튼을 클릭합니다.
![](images/aws_lambda_create_function.png)
1. 생성할 함수 정보를 입력한 후, `함수 생성` 버튼을 클릭합니다.
![](images/aws_lambda_fetch_todo_1.png)
1. `fetch-todo` 함수 생성이 완료되면 Lamdba 함수 편집 화면으로 자동으로 이동됩니다.
![](images/aws_lambda_fetch_todo_2.png)
1. **함수코드** 부분에 아래 코드를 입력합니다.
    ```javascript
    console.log('Loading function');

    var AWS = require("aws-sdk");
    var dynamoDb = new AWS.DynamoDB.DocumentClient();

    exports.handler = async (event, context) => {
        console.log('Received event:\n', JSON.stringify(event, null, 2));

        var scan_params = {
            TableName: "TODO",
        };

        var todos = [];
        todos = await dynamoDb.scan(scan_params)
                                .promise()
                                .then(res => res.Items)
                                .catch(err => err);

        console.log("Todo data: ", todos);

        return todos;
    };
    ```
1. "fetch-todo" Lambda 함수에서 Amazon DynamoDB 사용을 할 수 있도록 권한 정보를 추가합니다.
	1. "fetch-todo" 함수 상단의 "권한" 탭을 선택한 후, 실행 역할 부분의 **역할 이름**의 링크를 클릭합니다.
		![](images/aws_lambda_fetch_todo_4.png)
    1. "IAM(Identity and Access Management(IAM)" 화면이 나오면, 중간의 "정책 연결" 버튼을 클릭합니다.
		![](images/aws_lambda_fetch_todo_5.png)
    1. "정책 필터"에 `DynamoDB`라고 입력하여 나오는 "AmazonDynamoDBFullAccess" 정책을 선택합니다.
		![](images/aws_lambda_fetch_todo_6.png)
    1. 생성된 정책이 잘 연결되었는지 확인합니다.
		![](images/aws_lambda_fetch_todo_7.png)

**"fetch-todo"** Lambda 함수 생성이 완료되었습니다.
![](images/aws_lambda_fetch_todo_3.png)

#### 2. "add-todo" Lambda 함수 만들기

1. Lambda 대시보드 화면에서 `함수 생성` 버튼을 클릭합니다.
![](images/aws_lambda_create_function.png)
1. 생성할 함수 정보를 입력한 후, `함수 생성` 버튼을 클릭합니다.
![](images/aws_lambda_add_todo_1.png)
1. `add-todo` 함수 생성이 완료되면 Lamdba 함수 편집 화면으로 자동으로 이동됩니다.
![](images/aws_lambda_add_todo_2.png)
1. **함수코드** 부분에 아래 코드를 입력합니다.
    ```javascript
    console.log('Loading function');

    var AWS = require("aws-sdk");
    var dynamoDb = new AWS.DynamoDB.DocumentClient();

    exports.handler = async (event, context) => {
        console.log('Received event:\n', JSON.stringify(event, null, 2));

        var put_params = {
            TableName: 'TODO',
            Item: JSON.parse(event.body)
        };

        console.log("Params: ", put_params);

        // Call DynamoDB to add the item to the table
        await dynamoDb.put(put_params)
                        .promise()
                        .then(res => {
                            console.log("Todo data: ", res);
                        })
                        .catch(err => {
                            console.log("Error : ", err);
                        });

        return {
            responseCode: 200,
            message: "Success"
        };
    };
    ```
1. "add-todo" Lambda 함수에서 Amazon DynamoDB 사용을 할 수 있도록 권한 정보를 추가합니다.
    "fetch-todo" Lambda 함수 생성 보여준 DynamoDB 정책 연결 부분을 참고하여 "add-todo" Lambda 함수에도 연결합니다.

**"add-todo"** Lambda 함수 생성이 완료되었습니다.
![](images/aws_lambda_add_todo_3.png)


#### 도전 과제: "update-todo", "remove-todo" 함수 직접 만들기

지금까지 "fetch-todo", "add-todo" 함수를 만들기를 해 보셨다면, Lambda 함수를 생성하는 것은 어렵지 않다는 것을 아실 수 있을 겁니다.
**"update-todo"** 와 **"remove-todo"** 함수는 아래 제공된 코드를 이용해서 직접 한번 만들어 보시기 바랍니다.

1. **"update-todo"** 함수 코드
    ```javascript
    console.log('Loading function');

    var AWS = require("aws-sdk");
    var docClient = new AWS.DynamoDB.DocumentClient();

    exports.handler = async (event, context) => {
        console.log('Received event:\n', JSON.stringify(event, null, 2));

        var todo = JSON.parse(event.body);
        var update_params = {
            TableName: 'TODO',
            Key: {
                TODO_ID: todo.TODO_ID
            },
            UpdateExpression: 'set #t = :t, #c = :c',
            ExpressionAttributeNames: {
                '#t': 'TITLE',
                '#c': 'COMPLETED'
            },
            ExpressionAttributeValues: {
                ':t': todo.TITLE,
                ':c': todo.COMPLETED
            }
        };

        console.log("Params: ", update_params);

        // Call DynamoDB to add the item to the table
        await docClient.update(update_params)
                        .promise()
                        .then(res => {
                            console.log("Todo data: ", res);
                        })
                        .catch(err => {
                            console.log("Error : ", err);
                        });

        return {
            responseCode: 200,
            message: "Success"
        };
    };
    ```
2. **"remove-todo"** 함수 코드
    ```javascript
    console.log('Loading function');

    var AWS = require("aws-sdk");
    var dynamoDb = new AWS.DynamoDB.DocumentClient();

    exports.handler = async (event, context) => {
        console.log('Received event:\n', JSON.stringify(event, null, 2));

        var todoId = event.queryStringParameters.TODO_ID;
        var delete_params = {
            TableName: "TODO",
            Key: {
                "TODO_ID": Number(todoId)
            }
        };

        console.log("Params: ", delete_params);

        var todos = {
            responseCode: 200,
            message: "Success"
        };
        await dynamoDb.delete(delete_params)
                        .promise()
                        .then(res => {
                            console.log(res);
                            todos.message = "Todo is updated : " + todoId;
                        })
                        .catch(err => {
                            console.log("Error: ", err);
                            todos.responseCode = 400;
                            todos.message = "Error "
                        });

        console.log("Todo data: ", todos);

        return todos;
    };
    ```

### Amazon API Gateway 만들기

위의 AWS Lambda 함수 4가지를 만들었다면, Amazon API Gateway를 만들고 각 함수를 연결시켜 보도록 하겠습니다.

API Gateway 생성을 하기 전에 먼저 API Gateway 관리 화면으로 이동합니다.
![](images/amazon_api_gateway_1.png)
![](images/amazon_api_gateway_2.png)

#### 1. API 생성 하기

위의 Amazon API Gateway의 첫화면에 있듯이 Amazon API Gateway를 통해 생성할 수 있는 API 유형은 4가지가 있습니다.

- **HTTP API**: OIDC, OAuth2연동, CORS 지원이 되는 REST API 구축 가능.
- **REST API**: HTTP API 유형의 이전 세대 기능이며, API 요청/응답의 제어 기능 제공.
- **REST API(Private)**: VPC 안에서만 호출할 수 있는 REST API 기능 제공.
- **WebSocket API**: 실시간 통신을 위해 지속적인 연결이 가능하도록 WebSocket 지원.

이번 실습에서는 **HTTP API** 유형으로 API를 만들어 보도록 하겠습니다.

1. **HTTP API** 부분의 `구축` 버튼을 클릭합니다.
![](images/amazon_api_gateway_3.png)
1. API 생성 화면에서 **API 이름**은 `todo-api`로 입력한 후 `다음` 버튼을 클릭합니다.<br>(*통합* 부분은 API를 먼저 생성한 후에 진행합니다.)
![](images/amazon_api_gateway_todo_api_1.png)
1. 경로 구성 화면에서 `다음` 버튼을 클릭합니다.<br>(이전 "Step 1. API 생성" 화면에서 "통합" 기능을 수행했다면, 경로 구성 화면에서 바로 경로 추가가 가능합니다.)
![](images/amazon_api_gateway_todo_api_2.png)
1. "스테이지 정의" 화면에서는 화면에 표시되는 바와 같이 기본 설정으로 두고 `다음` 버튼을 클릭합니다.
![](images/amazon_api_gateway_todo_api_3.png)
1. "검토 및 생성" 화면에서 현재까지 구성 내용을 확인하고 `생성` 버튼을 클릭합니다.
![](images/amazon_api_gateway_todo_api_4.png)
1. 생성된 API Gateway 정보를 확인합니다.
![](images/amazon_api_gateway_todo_api_5.png)

#### 2. 경로(Routes) 생성 및 통합(Integerations)하기
Amazon API Gateway에서 말하는 경로(Routes)란 API 요청 수신 정보를 백엔드 리소스로 라우팅하기 위해 정의한 개념이며, "*HTTP Method*" 와 "*Resource path*" 라는 두 부분으로 구성됩니다. (예: `GET /todo`)

통합(Integration)은 경로로 부터 수신된 정보를 백엔드 시스템과 통합하기 위한 기능입니다. HTTP API 유형에서는 AWS Lambda와 통합할 수 있는 기능을 기본으로 제공합니다.

본 실습에서는 총 4개의 경로(Routes)와 각 경로와 연결되는 통합(Integrations)을 생성합니다.

아래는 생성할 경로와 통합 정보입니다.

| Path  | Method   | 통합할 Lambda 함수 |
| ----  | :------: | --------------- |
| /todo | GET      | fetch-todo      |
|       | POST     | add-todo        |
|       | PUT      | update-todo     |
|       | DELETE   | delete-todo     |


1. `[GET] /todo` 경로 생성을 위해 왼쪽의 `경로` 메뉴를 선택한 후, `Create` 버튼을 클릭합니다.
![](images/amazon_api_gateway_todo_api_fetch_todo_1.png)
![](images/amazon_api_gateway_todo_api_fetch_todo_2.png)
1. **경로 및 메서드** 화면에서 Method는 `GET`을 선택하고 경로(Path)는 `/todo`을 입력합니다.
![](images/amazon_api_gateway_todo_api_fetch_todo_3.png)
1. 생성된 경로를 클릭한 후 **"통합 연결"** 버튼을 클릭합니다.
![](images/amazon_api_gateway_todo_api_fetch_todo_4.png)
![](images/amazon_api_gateway_todo_api_fetch_todo_5.png)
1. 통합 화면으로 이동된 것을 확인한 후, **"통합 생성 및 연결"** 버튼을 클릭합니다.
![](images/amazon_api_gateway_todo_api_fetch_todo_6.png)
1. **통합 생성** 화면에서 통합 대상을 `Lambda 함수`로 선택한후, 통합 세부 정보 부분의 Lambda 함수 입력란에 이전에 생성한 "fetch-todo" Lambda 함수를 선택(또는 입력)한 후 `생성` 버튼을 클릭합니다.
![](images/amazon_api_gateway_todo_api_fetch_todo_7.png)
1. 생성된 경로와 통합 정보를 확인합니다.
![](images/amazon_api_gateway_todo_api_fetch_todo_8.png)

#### 3. 도전 과제

현재까지는 API Gateway에 "fetch-todo" Lambda 함수 하나만 통합하였습니다. 나머지 3개 함수도 위의 내용을 참고하여 직접 수행해 보시기 바랍니다.

| Path  | Method   | 통합할 Lambda 함수 |
| ----  | :------: | --------------- |
| /todo | ~~GET~~      | ~~fetch-todo~~      |
|       | POST     | add-todo        |
|       | PUT      | update-todo     |
|       | DELETE   | delete-todo     |

#### 4. CORS 설정

지금까지 설정한 API Gateway를 웹 사이트에서 사용하려면 CORS(Cross-Origin Resource Sharing) 설정이 되어야 합니다.
이 CORS 설정이 되어 있지 않다면, Javascript를 이용하여 AJAX 호출시 오류가 발생하지 주의해 주시기 바랍니다.


본 Hands-on 에서는 전체 허용 설정만 하도록 하겠습니다. 다음 화면을 참고하셔서 설정을 해 주시기 바랍니다.
![](images/amazon_api_gateway_cors_1.png)
![](images/amazon_api_gateway_cors_2.png)
![](images/amazon_api_gateway_cors_3.png)

#### 5. 생성된 API 테스트 해 보기.
현재까지 1개 Lambda에 대해서면 API Gateway 통합이 완료되었습니다. 이 생성된 API Gateway와 Lambda 기능이 정상적으로 동작되는지 확인해보기 위해서 DynamoDB 테이블에 직접 데이터를 추가한 후, Amazon API Gateway에 생성된 API를 테스트 해 보겠습니다.

1. Amazon DynamoDB 관리 화면으로 이동한 후, `TODO` 테이블을 선택합니다.
![](images/amazon_dynamodb_4.png)
1. TODO 테이블 화면의 상단 탭에서 데이터를 직접 관리할 수 있는 "항목" 메뉴로 이동합니다.
![](images/amazon_api_gateway_testing-dynamodb_1.png)
1. `항목 만들기` 버튼을 눌러 테스트 데이터를 아래와 같이 입력합니다.
![](images/amazon_api_gateway_testing-dynamodb_2.png)
![](images/amazon_api_gateway_testing-dynamodb_3.png)
1. 브라우저를 열어 새로 생성된 API의 URL 정보를 이용하여 호출하여 화면에 아래와 같이 JSON 데이터가 출력되는지 확인합니다.
![](images/amazon_api_gateway_testing-1.png)
![](images/amazon_api_gateway_testing-2.png)

### Amazon S3로 정적 웹 사이트 호스팅하기

Todo Web App의 웹 영역을 처리하는 index.html 파일과 todo.js 파일을 Amazon S3의 버킷에 업로드합니다. S3의 정적 웹 사이트 호스팅 기능을 이용해서 웹서버 기능을 손쉽게 구성할 수 있습니다.

#### 1. Amazon S3 버킷 생성
1. Amazon S3 관리 화면으로 이동합니다.
![](images/amazon_s3_1.png)
1. S3 관리 화면에서 `버킷 만들기` 버튼을 클릭합니다.
![](images/amazon_s3_2.png)
1. 버킷 이름을 **전 세계에서 중복되지 않는 고유한 이름**으로 입력한 후, `생성`버튼을 클릭합니다.
   S3버킷 이름은 정적 웹 사이트 호스팅시 URL로도 사용이 됩니다. 따라서, S3의 버킷 이름은 전 세계에서 유일해야하며 중복된 이름을 입력할 수 없습니다.
   (아래의 *"2020-mzc-serverless-hol"* 이름은 더 이상 사용할 수 없습니다. 😊)
![](images/amazon_s3_3.png)
1. 생성된 버킷이 목록에 나타는 지 확인합니다.
![](images/amazon_s3_4.png)

#### 2. 버킷을 정적 웹 사이트 호스팅 기능 사용하기
1. 버킷을 클릭하여 `속성` 탭으로 이동합니다.
![](images/amazon_s3_5.png)
1. 속성 탭에서 **"정적 웹 사이트 호스팅"** 부분을 클릭합니다.
![](images/amazon_s3_6.png)
1. 정적 웹 사이트 호스팅이 활성화 됐을 때 접속할 수 있는 **엔드포인트 URL**이 표시되는 것을 확인한 후, **"인덱스 문서"** 파일 명에 `"index.html"`로 입력후 `저장` 버튼을 클릭합니다.
![](images/amazon_s3_7.png)
참고로 생성된 S3 버킷의 엔드포인트 주소는 다음과 같습니다.
   - http://2020-mzc-serverless-hol.s3-website.ap-northeast-2.amazonaws.com
2. 정적 웹 사이트 호스팅 항목이 선택되었는지 확인합니다.
![](images/amazon_s3_8.png)

#### 3. 웹 사이트 액세스에 대한 권한 설정
웹 사이트 호스팅 기능을 활성화하더라도 바로 브라우저에서 접속할 수는 없습니다. 이는 버킷의 권한 설정이 기본적으로 "퍼블릭 액세스"가 차단되도록 설정되어 있기 때문입니다.
따라서, 브라우저에서 접속이 가능하게 하려면, 이 기능을 꺼 주셔야 합니다.

1. 권한 탭으로 이동하여 퍼블릭 액세스 차단을 비활성화 합니다.
![](images/amazon_s3_9.png)
![](images/amazon_s3_10.png)
![](images/amazon_s3_11.png)
![](images/amazon_s3_12.png)

#### 4. 버킷 정책 추가
마지막으로 버킷안의 파일을 공개적으로 읽기 가능하도록 만들기 위해 버킷 정책을 추가합니다.
![](images/amazon_s3_13.png)

버킷 정책 내용은 아래 내용을 사용합니다.
```json
 {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::버킷이름/*"
            ]
        }
    ]
}
```
단! 내용을 입력하실 때 `Resource` 부분의 내용중에서 `버킷이름` 부분은 여러분들이 만드신 버킷 이름으로 변경하시기 바랍니다.

버킷 정책이 올바르게 적용되었다면, 아래와 같이 "퍼블릭 액세스 권한" 안내 메세지가 출력됩니다.
![](images/amazon_s3_14.png)

#### 5. 접속 테스트
지금까지 설정한 내용이 잘 적용되었는지 확인해 봅니다.

`index.html` 파일과 `todo.js` 파일 2개를 S3 에 업로드합니다.

> **index.html**

```html
<!DOCTYPE html>
<html>

<head>
  <meta http-equiv="content-type" content="text/html; charset=UTF-8">
  <title>Serverless Todo App</title>
  <meta http-equiv="content-type" content="text/html; charset=UTF-8">
  <meta name="robots" content="noindex, nofollow">
  <meta name="googlebot" content="noindex, nofollow">
  <meta name="viewport" content="width=device-width, initial-scale=1">

  <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
  <script type="text/javascript" src="https://unpkg.com/vue@latest/dist/vue.js"></script>
  <link rel="stylesheet" type="text/css" href="https://unpkg.com/todomvc-app-css@2.2.0/index.css">

  <style id="compiled-css" type="text/css">
    [v-cloak] {
      display: none;
    }

    /* EOS */
  </style>

</head>

<body>

  <section class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input class="new-todo" autofocus autocomplete="off" placeholder="What needs to be done?" v-model="newTodo"
        @keyup.enter="addTodo">
    </header>
    <section class="main" v-show="todos.length" v-cloak>
      <ul class="todo-list">
        <li v-for="todo in filteredTodos" class="todo" :key="todo.TODO_ID"
          :class="{ completed: todo.COMPLETED, editing: todo == editedTodo }">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.COMPLETED" @change="changeCompleted(todo)">
            <label @dblclick="editTodo(todo)">{{ todo.TITLE }}</label>
            <button class="destroy" @click="removeTodo(todo)"></button>
          </div>
          <input class="edit" type="text" v-model="todo.TITLE" v-todo-focus="todo == editedTodo" @blur="doneEdit(todo)"
            @keyup.enter="doneEdit(todo)" @keyup.esc="cancelEdit(todo)">
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="todos.length" v-cloak>
      <span class="todo-count">
        <strong>{{ remaining }}</strong> {{ remaining | pluralize }} left
      </span>
      <ul class="filters">
        <li><a href="#/all" :class="{ selected: visibility == 'all' }">All</a></li>
        <li><a href="#/active" :class="{ selected: visibility == 'active' }">Active</a></li>
        <li><a href="#/completed" :class="{ selected: visibility == 'COMPLETED' }">Completed</a></li>
      </ul>
    </footer>
  </section>
  <footer class="info">
    <p>Double-click to edit a todo</p>
    <p>Written by <a href="http://evanyou.me">Evan You</a></p>
    <p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
  </footer>

  <script type="text/javascript" src="todo.js"></script>
</body>

</html>
```
<br>

> **todo.js**

```diff
- 아래 내용 중에서 `serverlessUrl` 부분의 주소는 위에서 생성한 여러부들의 API Gateway의 주소로 변경하시기 바랍니다.
```

```javascript
var todoStorage = {
    serverlessUrl: "https://여러분의_API_GATEWAY_주소",
    fetch: function () {
        axios.get(this.serverlessUrl + "/todo")
            .then(response => {
                var todos = response.data;
                console.log("Received todos: ", todos)
                app.todos = todos
            })
            .catch(error => {
                console.error("Uh oh? Something wrong!!")
            })
    },
    add: function (todo) {
        axios.post(this.serverlessUrl + "/todo", todo)
            .then(response => {
                console.log("Added todo: " + todo)
            })
            .catch(error => {
                console.error("Uh oh? Something wrong!!")
            })
    },
    update: function (todo) {
        axios.put(this.serverlessUrl + "/todo", todo)
            .then(response => {
                console.log("Updated todo: " + todo)
            })
            .catch(error => {
                console.error("Uh oh? Something wrong!!")
            })
    },
    remove: function (todo) {
        axios.delete(this.serverlessUrl + "/todo", {
                params: {
                    TODO_ID: todo.TODO_ID
                }
            })
            .then(response => {
                console.log("Updated todo: " + todo)
            })
            .catch(error => {
                console.error("Uh oh? Something wrong!!")
            })
    }
}

// visibility filters
var filters = {
    all: function (todos) {
        return todos
    },
    active: function (todos) {
        return todos.filter(function (todo) {
            return !todo.COMPLETED
        })
    },
    completed: function (todos) {
        return todos.filter(function (todo) {
            return todo.COMPLETED
        })
    }
}

// app Vue instance
var app = new Vue({
    // app initial state
    data: {
        todos: [],
        newTodo: '',
        editedTodo: null,
        visibility: 'all'
    },

    // computed properties
    // http://vuejs.org/guide/computed.html
    computed: {
        filteredTodos: function () {
            return filters[this.visibility](this.todos)
        },
        remaining: function () {
            return filters.active(this.todos).length
        },
        allDone: {
            get: function () {
                return this.remaining === 0
            },
            set: function (value) {
                this.todos.forEach(function (todo) {
                    todo.COMPLETED = value
                })
            }
        }
    },

    filters: {
        pluralize: function (n) {
            return n === 1 ? 'item' : 'items'
        }
    },

    // methods that implement data logic.
    // note there's no DOM manipulation here at all.
    methods: {
        addTodo: function () {
            var value = this.newTodo && this.newTodo.trim()
            if (!value) {
                return
            }
            var _newTodo = {
                //id: todoStorage.uid++,
                TODO_ID: Date.now(),
                TITLE: value,
                COMPLETED: false
            };
            this.todos.push(_newTodo)
            todoStorage.add(_newTodo)
            this.newTodo = ''
        },

        removeTodo: function (todo) {
            this.todos.splice(this.todos.indexOf(todo), 1)
            todoStorage.remove(todo)
        },

        editTodo: function (todo) {
            this.beforeEditCache = todo.TITLE
            this.editedTodo = todo
        },

        changeCompleted: function (todo) {
            todoStorage.update(todo)
        },

        doneEdit: function (todo) {
            if (!this.editedTodo) {
                return
            }
            this.editedTodo = null
            todo.TITLE = todo.TITLE.trim()
            if (!todo.TITLE) {
                this.removeTodo(todo)
            } else {
                todoStorage.update(todo)
            }
        },

        cancelEdit: function (todo) {
            this.editedTodo = null
            todo.TITLE = this.beforeEditCache
        },

        removeCompleted: function () {
            this.todos = filters.active(this.todos)
        }
    },

    created: function () {
        todoStorage.fetch();
    },

    // a custom directive to wait for the DOM to be updated
    // before focusing on the input field.
    // http://vuejs.org/guide/custom-directive.html
    directives: {
        'todo-focus': function (el, binding) {
            if (binding.value) {
                el.focus()
            }
        }
    }
})

// handle routing
function onHashChange() {
    var visibility = window.location.hash.replace(/#\/?/, '')
    if (filters[visibility]) {
        app.visibility = visibility
    } else {
        window.location.hash = ''
        app.visibility = 'all'
    }
}

window.addEventListener('hashchange', onHashChange)
onHashChange()

// mount
app.$mount('.todoapp')
```

1. S3 버킷에서 2개 파일을 업로드 합니다.
![](images/amazon_s3_15.png)
![](images/amazon_s3_16.png)
![](images/amazon_s3_17.png)
1. 업로드가 완료되면 2개 파일이 목록에 표시됩니다.
![](images/amazon_s3_18.png)
1. 접속해 봅니다.
![](images/amazon_s3_19.png)

## 리소스 삭제

실습을 완료하였다면, 지금까지 생성한 리소스를 모두 지워서 불필요한 지출이 발생하지 않도록 하시기를 당부 드립니다.

감사합니다.
