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

#### 3. 생성된 API 테스트 해 보기.
현재까지 1개 Lambda에 대해서면 API Gateway 통합이 완료되었습니다. 이 생성된 API Gateway와 Lambda 기능이 정상적으로 동작되는지 확인해보기 위해서 DynamoDB 테이블에 직접 데이터를 추가한 후, Amazon API Gateway에 생성된 API를 테스트 해 보겠습니다.

1. Amazon DynamoDB 관리 화면으로 이동한 후, `TODO` 테이블을 선택합니다.
![](images/amazon_dynamodb_4.png)
1. TODO 테이블 화면의 상단 탭에서 데이터를 직접 관리할 수 있는 "항목" 메뉴로 이동합니다.
![](images/amazon_api_gateway_testing-dynamodb_1.png)
1. `항목 만들기` 버튼을 눌러 테스트 데이터를 아래와 같이 입력합니다.
![](images/amazon_api_gateway_testing-dynamodb_2.png)
![](images/amazon_api_gateway_testing-dynamodb_3.png)
1. 브라우저를 열어 새로 생성된 API의 URL 정보를 이용하여 호출해 봅니다.
![](images/amazon_api_gateway_testing-1.png)
![](images/amazon_api_gateway_testing-2.png)

### Amazon S3 정적 웹 호스팅 만들기

Todo Web App의 웹 영역을 처리하는 index.html 파일과 todo.js 파일을 Amazon S3의 버킷에 업로드합니다.

#### 1. S3 버킷 생성

#### 1. sldkfjsldfkj
