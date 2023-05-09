# Notes

## REST
## Class 1: REST and maturity levels
- Representational state of transfer
- Stateless
- Simple
- Cacheable

Maturity levels (Richardson Maturity Model):
- level 0: No standardization
- level 1: using resources (e.g /products)
- level 2: using verbs (i.e. GET POST PUT DELETE)
- level 3: HATEOAS: Hypermedia as the Engine of Application State. Provide links to further operations possible over the resource

## Class 2: Method and Content Negotiation
- Single URIs for each resource
- Use all verbs to perform the operations 
- Aditional links showing what may be done further

REST patterns: HAL, Collection + JSON and Siren

- JSON doesn't provide a hypermedia standard
- HAL: Hypermedia Application Language

Media type: application/hal+json: Fields to show relations and links of the resource:
- `_links`
- `_embedded`

_HTTP Method Negoatiation_:
- Method `OPTIONS`: Response should provide which verbs are available for the endpoint. If user sends a different verb, 405 Method Not Allowe status code is returned.

_Content Negotiation_:
- `Accept` header: specifies which is the the expected response format from the server, e.g. `application/json`. Response from server may be the data or a 406 Not Acceptable status code.
- `Content-Type` header: specifies which is the data type the client is sending on the body, e.g. `application/json`. If the server cannot process it, response should be 415 Unsupported Media Type status code.

## Class 3: Installing Laminas API Tools
Use remote container extension on visual studio code to work with php inside a container. Docs from Laminas API Tools: https://api-tools.getlaminas.org/download.
Alternatively, clone https://github.com/codeedu/api-tools-skeleton, then run `docker build -t api-tools-test .` & `docker run -p 8000:80 -v $(pwd):/var/www api-tools-test`

## Class 4: Practicing content negotiation
Playing around on localhost:8000 with Laminas API Tools.


## gRPC
## Class 5: gRPC - initial concepts
Facilitate the communication between systems. Lightweight and performatic.

When to use it?
- Microservices
- Mobile, Browsers, Backend
- Lib generation automatically
- bidirectional streaming with http/2

Supported languages:
- Golang
- Java
- C
    - C++
    - Python
    - Ruby
    - PHP
    - C#
    - Node.js
    - Dart

## Class 6: gRPC, HTTP/2 and Protocol Buffers
Client/Server arquitecture. Remote call to a service in another machine.

gRPC's Data protocol: Protocol Buffers: language-neutral, platform-neutral, extensible mechanism for serializing structured data.

Protocol Buffers vs JSON
- Binary files < JSON
- Serialization process is more lightweight
- Less network resources
- Faster process

Example:
```
syntax = "proto3";

message SearchRequest {
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
}
```

HTTP/2:
- binary data traffic instead of text data
- Reuse TCP connection
- Server push
- Compressed headers
- Consume less network resources

## Class 7: gRPC communication formats
API unary: request/response beween client and server
API server streaming: client sends request and server may reply with multiple responses
API client streaming: client sends multiple requests and server reply with one response
API bidirectional streaming: combination of server and client streaming

## Class 8: REST vs gRPC
REST:
- Text/JSON
- Unidirectional
- High latency
- No contract well defined
- No streaming support
- Predefined design (CRUD)

gRPC:
- protocol buffers
- birectional and async
- low latency
- well defined contract
- streaming support
- free design
- code generation

## Class 9: gRPC vs Protocol Buffers
To undertand gRPC, it's necessary to understand protocol buffers first.

Protocol buffers guide: https://protobuf.dev/
gRPC guide: https://grpc.io/

## Class 10, 11, 12, 13, 14, 15, 16
- Install protocol buffers binaries (`https://grpc.io/docs/languages/go/quickstart/`)
- Create .proto files and generate go code with `protoc --go_out=. --go-grpc_out=. proto/course_category.proto`
- Create grpc server (main.go)
- Install evans grpc client: `go install github.com/ktr0731/evans@latest` and run it `evans -r repl`
- Interact with server: `call createCategory`
- Create `ListCategories` and `GetCategory` messages

## Class 17 Streams with gRPC
- Create stream on protofile
- Implement `CreateCategoryStream` to receive any data and process all at once
- Start evans `evans -r repl` and `call CreateCategoryStream`
- Press ctrl + d to close the stream

## Class 18 Bidirectional streams
- Create bidirectional stream on protofile
- Very similar to regular streams


## GraphQL
## Class 19 Graphql
- Reference: `https://gqlgen.com/`
- Run `go mod init github.com/luisfelipe998/graphql`
- Run `printf '// +build tools\npackage tools\nimport (_ "github.com/99designs/gqlgen"\n _ "github.com/99designs/gqlgen/graphql/introspection")' | gofmt > tools.go`
- Run `go run github.com/99designs/gqlgen init`
- Start server with `go run ./server.go`

## Class 20 Graphql schemas
- Create schema on `schema.graphqls` file
- Create data types, data inputs, queries and mutations

## Class 21 Create skeleton of the app
- Run graphql gen code with `go run github.com/99designs/gqlgen generate` to create resolvers

## Class 22, 23 Add code to resolvers with a mutation
- Create package database and add dependency on Resolver struct from graphql
- Inject on server.go the new dependency on the resolver
- Start the server and open `localhost:8080`
- On UI, type:
```
mutation createCategory {
    createCategory(input: { name: "foo", description: "bar" }) {
        id
        name description
    }
}
```
- Create sqlite3 table: `sqlite3 data.db` > `create table category (id string, name string, description string);`
- rerun mutation to see execution on graphql

## Class 24 Add a query

- Create Find all method
- On UI, type:
```
query queryCategories {
  categories {
    id
    name
    description
  }
}
```

## Class 25, 26, 27 Add query and mutation for courses
- Equal to the categories

## Class 28 Handling nested data
- Open `gqlgen.yml` and add:
```
models:
  Category:
    model:
      - "github.com/luisfelipe998/graphql/graph/model.Category"
  Course:
    model:
      - "github.com/luisfelipe998/graphql/graph/model.Course"
```
- Create a `category.go` and `course.go` file under `graph/model` package
- Rerun `go run github.com/99designs/gqlgen generate`
- Now we have a method `Courses` to retrieve the nested references of courses under for each category (see `graph/schema.resolvers.go` file). Implement this method.
- Restart the server (`go run cmd/server/server.go`) and run the following query:
```
query queryCategories {
  categories {
    id
    name
    description
    courses {
      id
      name
    }
  }
}
```
- Notice the nested information being retrieved.

## Class 29 Bidirectional relationship between models 
This is to enable retrieval of categories with courses (previous class) AND courses with categories (this class)

- Adjust the Course model (remove the category_id pointer field)
- Rerun `go run github.com/99designs/gqlgen generate`
- Implement course resolver to get category (on `graph/schema.resolvers.go`)
- Restart the server (`go run cmd/server/server.go`) and run the following query:
```
query queryCourses {
  courses {
    id
    name
    description
    category {
      id
      name
      description
    }
  }
}
```
- Notice how the category was retrieved within the course


