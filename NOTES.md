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

## Service Discovery & Consul

[Consul docs](https://developer.hashicorp.com/consul/docs/architecture)

## Class 30 The context
- Useful for communication between microservices: know the callee machine (DNS, IP);
- Useful for managing scaling callee systems
- Questions:
  - which machineto call?
  - on which port?
  - need to know IP addr?
  - how to know healthiness?
  - how to check permissions?
- Service discovery: 
  - discover available machine from a specific service
  - segmentation of machines for security
  - DNS resolution
  - health checking
  - permissions management

## Class 31 Consul overview
- Service discovery
- Service segmentation
- Load balancer on layer 7
- Key/Value configuration
- Open-source

## Class 32 Service registry
- Uses a centralized service registry to all applications plugged in
- Agent/Server architecture
- Health check is running on the agent locally (closer to the app)
- Gossip protocol
- Consul has a DNS server to call the underlying application instances

## Class 33 Health check and multicloud
- Healthcheck -> Consul agent: local configuration to monitor the application locally. Informs the consul server if the app stops responding.
- Multicloud: consul's agents may work across different data centers (even different cloud providers)

## Class 34 Agent, Client and Server
- Agent: process (daemon) runing on every cluster node. Client mode or Server mode
- Client: register services locally (with config file), health check, forward information and configs from the services to the server
- Server: Keep the cluster state, register the services, membership (who is client and who is server), query returns (DNS, API), information exchange between datacenters, etc.

## Class 35 Starting a consul agent
- Create docker-compose.yml
- Run `docker compose up -d`
- Enter on the container: `docker exec -it consul01 sh`
- Inside the container, specify mode of consul execution: `consul agent -dev` for agent (server)
- Run `consul members` to check members inside the mesh
- Accessing consul through API: `curl localhost:8500/v1/catalog/nodes`
- Use consul's datacenter concept to create the mesh. Communication across datacenters is also possible.
- Accessing consul through DNS: Install `apk -u add bind-tools` & Run `dig @localhost -p 8600 consul01.node.consul` optional +short

## Class 36 Create consul cluster
- Create a cluster with 3 servers
- Modify docker-compose.yml to create 3 services. 
- Run `docker compose up -d`
- Objective is make all 3 agents communicate with each other
- Run the agent in server mode: 
  - enter container: `docker exec -it consulserver01 sh`
  - Check ip `ifconfig`
  - Run `mkdir /etc/consul.d` & `mkdir /var/lib/consul`
  - Run `consul agent -server -bootstrap-expect=3 -node=consulserver01 -bind=172.19.0.2 -data-dir=/var/lib/consul -config-dir=/etc/consul.d`
  - Repeat for the other 2 server agents
- Now we have 3 consuls running separatedly. Run `consul join 172.19.0.2` (ip from the other servers) to connect them
- Rerun `consul members` to see them connected.

## Class 37 Creating the first client
- Create folder /clients/consul01
- Adjust docker-compose.yml to add consulclient01;
- Start docker compose
- run `docker exec -it consulclient01 sh`
  - inside the container: `ifconfig` & `consul agent -bind=172.19.0.4 -data-dir=/var/lib/consul -config-dir=/etc/consul.d` & `consul join 172.19.0.2`
  - now the client is inside the consul cluster

## Class 38 Register services
- Create file `services.json` under `/clients/consul01` to register a new service (nginx)
- Registering a service is different from creating a service
- Inside `consulclient01` container, run `consul reload` to register the service
- The servers already know about the new registered client: `apk -U add bind-tools` -> `dig @localhost -p 8600 nginx.service.consul` -> see the service. Go to one server and run the same commands. Check the same result.
- We can create another nginx service and resolve itself with the IPs
- We can also check the service through the API: `curl localhost:8500/v1/catalog/services`
- Find which consul node the nginx service is registered: `consul catalog nodes -service nginx`

## Class 39 Register 2nd service with retry join
- Add a consulclient02 to docker compose file;
- Rerun `docker compose up -d` & `docker exec it consulclient02 sh`
- Improve previous command with -retry-join: `consul agent -bind=172.19.0.6 -data-dir=/var/lib/consul -config-dir=/etc/consul.d -retry-join=172.19.0.2 --retry-join=172.19.0.3`
- Run `apk -U add bind-tools` -> `dig @localhost -p 8600 nginx.service.consul` and check the 2 services available.
- Note how the command points to localhost. It accesses the local registry of the client to access the information about the services connected to the network

## Class 40 Implementing Checks
- Add checks to verify if the service is really up and running, removing them from the service (machine IP) discovery if not.
- Checks: script, http, tcp, TTL, Docker, gRPC, etc;
- Add http check on `services.json` on `/clients/consul01`,
- Run `consul reload` to apply the change;
- Notice that the check is returning a critical status for the check and the IP discovery doesn't return the IP of the faulty service anymore; (`dig @localhost -p 8600 nginx.service.consul`);
- Install and run nginx: `apk add ngnix` & `nginx`;
- Notice how the check is now synced and the IP is found on IP discovery service from consul

## Class 41 Syncronizing servers with file
- Create `servers/server01` and create `server.json` to facitilate server creation;
- Adjust `docker-compose` to use the file through the volume and rerun `docker compose up -d`;
- Enter the server container and run `consul agent -config-dir=/etc/consul.d` to see all synced;
- Replicate config to other 2 servers and rerun previous command to other 2 servers;

## Class 42 Consul Encryption
- Adjust `server.json` file to include encryption with `encrypt` param
- Generate encryption key: `consul keygen` and populate the param on all servers to properly communicate
- Redeploy consul servers `docker compose up -d` & `consult agent -config-dir=/etc/consul.d`
- If the server config doesn't have the encryption config, it will fail to join the consul datacenter.
- Check the data being transferred encrypted on the container: `apk add tcpdump` & `tcpdump -i eth0 -an port 8301 -A`

## Class 43 Consul UI
- Enable consul UI through CLI command: `consult agent -config-dir=/etc/consul.d -ui -client_addr=0.0.0.0`
- Or edit server config file with:
```
  "client_addr": "0.0.0.0",
  "ui_config":{
    "enabled":true
  }
```
- Expose port 8500 from server01 on docker-compose.yml file
- Redeploy agent: `consult agent -config-dir=/etc/consul.d`
- access: `localhost:8500` to see UI
- Service Discovery, Service Mesh and ACL available to inspect on consul UI
- For production: 3 servers; encryption to join the DC; and mTLS between agents

