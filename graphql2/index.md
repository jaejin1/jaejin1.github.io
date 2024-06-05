# Explore GraphQL2


서버리스 GraphQL워크샵 내용을 참조했습니다. 

[앞의 글](https://jaejin1.github.io/graphql1/)은 개념만 다뤘다면 이번 글에서는 실질적으로 사용을 함에 있어 필요한 내용들을 다룬다.

<!--more-->

## 소개

GraphQL은 페이스북에서 만든 어플리케이션 레이어 쿼리 언어이다. 기존의 웹 혹은 모바일의 어플리케이션의 API를 구현 할 때는 보통 REST API를 사용한다. 기존의 REST API를 사용하게 되면, 클라이언트에서 기능이 필요할때마다 그때 그때 필요한 API를 만들어 줘야 했다.

예를들어 어플리케이션에 Account라는 모델이 있고,
/accounts라는 endpoint가 있따고 하면

* GET /accounts
* GET /accounts/{id}
* ...

~~~json
{
  "account": {
    "id": "1",
    "email": "jaejin@gmail.com",
    "friends": [
      "2",
      "3"
    ],
    "first_name": "Jaejin",
    "last_name": "Lee"
  }
}
~~~

이렇게 가져올텐데 만일 friends목록을 가져오려면 또 다른 API를 만들어야 할 것이다.

`GET /accounts/1/?include_friend=username`

or

`GET /include_friend/1`

이렇게 계속 API를 만들다 보면 수 많은 endpoint가 생길 것이다.

### 쿼리

~~~graghql
query {
    account(id: "1") {
        username
        email
        firstName
        lastName
        friends {
            firstName
            username
        }
    }
}
~~~

### 결과

~~~json
{
  "data": {
    "account": {
      "username": "velopert",
      "email": "public.velopert@gmail.com",
      "firstName": "Minjun",
      "lastName": "Kim",
      "friends": [
        {
          "firstName": "Jayna",
          "username": "jn4kim"
        },
        {
          "firstName": "Abet",
          "username": "abet"
        }
      ]
    }
  }
}
~~~

GraphQL은 다음과 같이 query로 만들어 서버에 전달하면 원하는 데이터를 가져올 수 있는 것이다.

## 문제?

GraphQL은 REST API의 새로운 관점의 스펙이다. 따라서 어느 언어를 사용하든 GraphQL의 스펙에 맞는 코드를 작성해야한다.

* [javascript](https://github.com/graphql/graphql-js)
* [go](https://github.com/graphql-go/graphql)
* [python](https://github.com/graphql-python)
* ...

기존 REST로 구현된 API는 `GET /accounts`, `POST /accounts`, `DELETE /accounts/:accountsID`등 과 같은 방식으로 개발이 된다. 이 구조를 GraphQL로 바꿀 수 있지만 GraphQL은 `단일 endpoint`를 권장한다.  
따라서 모든 요청을 `/graghql`에서 처리 해야한다. 왜냐하면 GraphQL은 url이 아니라 Query를 통해 표현하기 때문에.  
기존 REST API와 같이 사용하려면 기존 REST API들은 놔두고 새로운 endpoint `/graphql`을 작성하면 된다.

### Content-Type: "application/graphql"

GraphQL은 기본적으로 `Content-Type: "application/json"`을 사용한다. 물론 `Content-Type: "application/graphql"`도  사용가능하다.  
하지만 두 요청은 차이점이 있다.

* `Content-Type: "application/json"`

request body는 다음 형식을 따라야한다.

~~~json
{
  "query": "...",
  "operationName": "...",
  "variables": { "myVariable": "someValue", ... }
}
~~~

* `Content-Type: "application/graphql"`

Query요청을 위해 축약형 body를 사용할 수 있게 해준다. Content-Type: "application/json" 의 query필드 내부에 있는 값을 그대로 사용하는 것이다.

~~~query
{
    account(id: "1") {
        id
        email
        ...
    }
}
~~~

이러한 형식 차이 때문에 `Content-Type: "application/graphql"` 을 사용하기 보다는 `Content-Type: "application/json"`로 통일 하여 사용 할 것이다. 

## Example

go로 간단한 GraphQL 서버를 만들어보자.

### Code

* `main.go`

~~~go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"net/http"

	"github.com/graphql-go/graphql"
)

type user struct {
	ID   string `json:"id"`
	Name string `json:"name"`
}

var data map[string]user

/*
   Create User object type with fields "id" and "name" by using GraphQLObjectTypeConfig:
       - Name: name of object type
       - Fields: a map of fields by using GraphQLFields
   Setup type of field use GraphQLFieldConfig
*/
var userType = graphql.NewObject(
	graphql.ObjectConfig{
		Name: "User",
		Fields: graphql.Fields{
			"id": &graphql.Field{
				Type: graphql.String,
			},
			"name": &graphql.Field{
				Type: graphql.String,
			},
		},
	},
)

/*
   Create Query object type with fields "user" has type [userType] by using GraphQLObjectTypeConfig:
       - Name: name of object type
       - Fields: a map of fields by using GraphQLFields
   Setup type of field use GraphQLFieldConfig to define:
       - Type: type of field
       - Args: arguments to query with current field
       - Resolve: function to query data using params from [Args] and return value with current type
*/
var queryType = graphql.NewObject(
	graphql.ObjectConfig{
		Name: "Query",
		Fields: graphql.Fields{
			"user": &graphql.Field{
				Type: userType,
				Args: graphql.FieldConfigArgument{
					"id": &graphql.ArgumentConfig{
						Type: graphql.String,
					},
				},
				Resolve: func(p graphql.ResolveParams) (interface{}, error) {
					idQuery, isOK := p.Args["id"].(string)
					if isOK {
						return data[idQuery], nil
					}
					return nil, nil
				},
			},
		},
	})

var schema, _ = graphql.NewSchema(
	graphql.SchemaConfig{
		Query: queryType,
	},
)

func executeQuery(query string, schema graphql.Schema) *graphql.Result {
	result := graphql.Do(graphql.Params{
		Schema:        schema,
		RequestString: query,
	})
	if len(result.Errors) > 0 {
		fmt.Printf("wrong result, unexpected errors: %v", result.Errors)
	}
	return result
}

func main() {
	_ = importJSONDataFromFile("data.json", &data)

	http.HandleFunc("/graphql", func(w http.ResponseWriter, r *http.Request) {
		result := executeQuery(r.URL.Query().Get("query"), schema)
		json.NewEncoder(w).Encode(result)
	})

	fmt.Println("Now server is running on port 8080")
	fmt.Println("Test with Get      : curl -g 'http://localhost:8080/graphql?query={user(id:\"1\"){name}}'")
	http.ListenAndServe(":8080", nil)
}

//Helper function to import json from file to map
func importJSONDataFromFile(fileName string, result interface{}) (isOK bool) {
	isOK = true
	content, err := ioutil.ReadFile(fileName)
	if err != nil {
		fmt.Print("Error:", err)
		isOK = false
	}
	err = json.Unmarshal(content, result)
	if err != nil {
		isOK = false
		fmt.Print("Error:", err)
	}
	return
}
~~~

* `data.json`

~~~json
{
    "1": {
      "id": "1",
      "name": "Dan"
    },
    "2": {
      "id": "2",
      "name": "Lee"
    },
    "3": {
      "id": "3",
      "name": "Nick"
    }
}
~~~


### Run

~~~bash
$ go run main.go


Now server is running on port 8080
Test with Get      : curl -g 'http://localhost:8080/graphql?query={user(id:"1"){name}}'
~~~

~~~bash
$ curl -g 'http://localhost:8080/graphql?query={user(id:"1"){name}}'


{"data":{"user":{"name":"Dan"}}}
~~~
