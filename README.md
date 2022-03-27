# gorm-tutorial

#### To run the application
```shell
go run main.go
```
## Architecture
### Layer Architecture
![Layer Architecture](https://camo.githubusercontent.com/d9b21eb50ef70dcaebf5a874559608f475e22c799bc66fcf99fb01f08576540f/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a4a4459546c4b3030796730496c556a5a392d737037512e706e67)

### Layer Architecture with standard features: config, health check, logging, middleware log tracing
![Layer Architecture with standard features: config, health check, logging, middleware log tracing](https://camo.githubusercontent.com/c3fcfa0a473ae4a86397e7624a0d1a5ca5a0c814119982ea3cf8694e10df1f6b/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6e6b4673633275796f4c437a4b716c66474d4e6b6f772e706e67)

### Architecture of Tutorial
Based on the layer architecture, we simplify the architecture for this tutorial
![Architecture of tutorial](https://camo.githubusercontent.com/3067c81fbbfc7c32430e1a31dec0c932c92d152bb9414cdf6d69b6a2b686aff3/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a564646353531774d2d6d33713036676a41386d4368412e706e67)

## API Design
### Common HTTP methods
- GET: retrieve a representation of the resource
- POST: create a new resource
- PUT: update the resource
- PATCH: perform a partial update of a resource
- DELETE: delete a resource

## API design for health check
To check if the service is available.
#### *Request:* GET /health
#### *Response:*
```json
{
    "status": "UP",
    "details": {
        "sql": {
            "status": "UP"
        }
    }
}
```

## API design for users
#### *Resource:* users

### Get all users
#### *Request:* GET /users
#### *Response:*
```json
[
    {
        "id": "spiderman",
        "username": "peter.parker",
        "email": "peter.parker@gmail.com",
        "phone": "0987654321",
        "dateOfBirth": "1962-08-25T16:59:59.999Z"
    },
    {
        "id": "wolverine",
        "username": "james.howlett",
        "email": "james.howlett@gmail.com",
        "phone": "0987654321",
        "dateOfBirth": "1974-11-16T16:59:59.999Z"
    }
]
```

### Get one user by id
#### *Request:* GET /users/:id
```shell
GET /users/wolverine
```
#### *Response:*
```json
{
    "id": "wolverine",
    "username": "james.howlett",
    "email": "james.howlett@gmail.com",
    "phone": "0987654321",
    "dateOfBirth": "1974-11-16T16:59:59.999Z"
}
```

### Create a new user
#### *Request:* POST /users 
```json
{
    "id": "wolverine",
    "username": "james.howlett",
    "email": "james.howlett@gmail.com",
    "phone": "0987654321",
    "dateOfBirth": "1974-11-16T16:59:59.999Z"
}
```
#### *Response:* 1: success, 0: duplicate key, -1: error
```json
1
```

### Update one user by id
#### *Request:* PUT /users/:id
```shell
PUT /users/wolverine
```
```json
{
    "username": "james.howlett",
    "email": "james.howlett@gmail.com",
    "phone": "0987654321",
    "dateOfBirth": "1974-11-16T16:59:59.999Z"
}
```
#### *Response:* 1: success, 0: not found, -1: error
```json
1
```

### Delete a new user by id
#### *Request:* DELETE /users/:id
```shell
DELETE /users/wolverine
```
#### *Response:* 1: success, 0: not found, -1: error
```json
1
```

## Common libraries
- [core-go/health](https://github.com/core-go/health): include HealthHandler, HealthChecker, SqlHealthChecker
- [core-go/config](https://github.com/core-go/config): to load the config file, and merge with other environments (SIT, UAT, ENV)
- [core-go/log](https://github.com/core-go/log): log and log middleware

### core-go/health
To check if the service is available, refer to [core-go/health](https://github.com/core-go/health)
#### *Request:* GET /health
#### *Response:*
```json
{
    "status": "UP",
    "details": {
        "sql": {
            "status": "UP"
        }
    }
}
```
To create health checker, and health handler
```go
    db, err := sql.Open(conf.Driver, conf.DataSourceName)
    if err != nil {
        return nil, err
    }

    sqlChecker := s.NewSqlHealthChecker(db)
    healthHandler := health.NewHealthHandler(sqlChecker)
```

To handler routing
```go
    r := mux.NewRouter()
    r.HandleFunc("/health", healthHandler.Check).Methods("GET")
```

### core-go/config
To load the config from "config.yml", in "configs" folder
```go
package main

import "github.com/core-go/config"

type Root struct {
    DB DatabaseConfig `mapstructure:"db"`
}

type DatabaseConfig struct {
    Driver         string `mapstructure:"driver"`
    DataSourceName string `mapstructure:"data_source_name"`
}

func main() {
    var conf Root
    err := config.Load(&conf, "configs/config")
    if err != nil {
        panic(err)
    }
}
```

### core-go/log *&* core-go/middleware
```go
import (
    "github.com/core-go/config"
    "github.com/core-go/log"
    m "github.com/core-go/middleware"
    "github.com/gorilla/mux"
)

func main() {
    var conf app.Root
    config.Load(&conf, "configs/config")

    r := mux.NewRouter()

    log.Initialize(conf.Log)
    r.Use(m.BuildContext)
    logger := m.NewLogger()
    r.Use(m.Logger(conf.MiddleWare, log.InfoFields, logger))
    r.Use(m.Recover(log.ErrorMsg))
}
```
To configure to ignore the health check, use "skips":
```yaml
middleware:
  skips: /health
```
