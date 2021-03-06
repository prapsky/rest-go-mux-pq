# REST API in Go example

[![Build Status](https://travis-ci.org/AmundsenJunior/rest-go-mux-pq.svg?branch=master)](https://travis-ci.org/AmundsenJunior/rest-go-mux-pq)

* gorilla/mux for routing
* lib/pq PostgreSQL driver
* docker/lib/postgres as database

___Source: https://semaphoreci.com/community/tutorials/building-and-testing-a-rest-api-in-go-with-gorilla-mux-and-postgresql___

## Project structure

* `app.go` - App type to provide REST web service endpoints
* `handlers.go` - Handle HTTP requests and execute model function calls
* `main.go` - main package to run service
* `model.go` - defines products database model and executes CRUD operations
  against database

## CRUD of products

* create a new product: POST to /product
* delete an existing product: DELETE to /product/{id}
* update an existing product: PUT to /product/{id}
* fetch an existing product: GET to /product/{id}
* fetch a list of all existing products: GET to /products

## Development Environment

### Create project workspace

```
$ git clone https://github.com/amundsenjunior/rest-go-mux-pq.git
```

### Get project dependencies

You can pull the two dependencies directly, via:

```
$ go get github.com/gorilla/mux github.com/lib/pq
```

Or, if using Go 1.11+, use `go mod` and the present `go.mod` and `go.sum` files:

```
$ go mod download 
```

### Start PostgreSQL instance and create database & table
* https://hub.docker.com/_/postgres/
* https://www.postgresql.org/docs/9.2/static/app-psql.html
* https://www.tutorialspoint.com/postgresql/postgresql_create_database.htm

Either create a development database with Docker and the below SQL:

```
$ docker run --name rgmp -d -e POSTGRES_PASSWORD=restgomuxpq -p 5432:5432 postgres:alpine
$ docker run -it --rm --link rgmp:postgres postgres:alpine psql -h postgres -U postgres -W
  password: restgomuxpq
=# CREATE TABLE IF NOT EXISTS products (
    id SERIAL,
    name TEXT NOT NULL,
    price NUMERIC(10,2) NOT NULL DEFAULT 0.00,
    CONSTRAINT products_pkey PRIMARY KEY (id)
  );
=# \dt
=# \q
```

Or, build and run the database Docker image from `docker/postgres`, which
contains an `init-table.sql` script of the above SQL to run on container
startup:

```
$ docker build -t postgres:rgmp ./docker/postgres
$ docker run -d -e POSTGRES_PASSWORD=restgomuxpq -p 5432:5432 postgres:rgmp
```

## Running the Application

### Build and run the application from local source

Using the above-created Postgres Docker DB container, build the RGMP executable
and run with environment variables passed in for connecting to Postgres:

```
$ go build
$ export DB_USERNAME=postgres DB_PASSWORD=restgomuxpq DB_NAME=postgres DB_HOST=localhost DB_PORT=5432 DB_SSLMODE=disable; ./rest-go-mux-pq
```

### Build and run with `docker-compose`

Get a running application via `docker-compose` from the `docker-compose.yaml`
configuration file with the following command:

```
$ docker-compose up -d
```

This instance contains both the RGMP application in a Docker container and the 
Postgres container, with the two services linked and initialized for taking HTTP
requests. The application is up and available at `http://localhost:8000`.

### Execute HTTP requests against the application

```
$ curl -X POST -H "Content-Type: application/json" -d '{"name": "toy gorilla", "price": 29.99}' http://localhost:8000/product
$ curl -X GET http://localhost:8000/products
```

## Testing

* TestHealthStatus - expected is an OK and JSON health content
* TestEmptyTable - expected is a 200 code, but response returns 404
* TestGetNonExistentProduct - a GET request for a product by id should return
  error
* TestCreateProduct - an OK response code should come from a POST request of a
  new product
* TestGetProduct - an OK response code should come from a GET request on an
  existing product
* TestGetAllProducts - an OK response code and correct length of response body
  should come from a GET request on all existing products
* TestUpdateProduct - an OK response code should come from a PUT request on an
  existing product to update it
* TestDeleteProduct - an OK response code should come from a DELETE request on
  removing an existing product


Execute TestMain with env vars:

```
$ export TEST_DB_USERNAME=postgres TEST_DB_PASSWORD=restgomuxpq TEST_DB_NAME=rgmp TEST_DB_HOST=localhost TEST_DB_PORT=5432 TEST_DB_SSLMODE=disable; go test -v
```

Alternatively, run `test_exec.sh` to start a Docker test database, execute the
tests, and cleanup:

```
$ bash ./test_exec.sh
```

## GoDoc
* [database/sql](https://golang.org/pkg/database/sql/)
* [orilla/mux](http://www.gorillatoolkit.org/pkg/mux)
* [gorilla/handlers](http://www.gorillatoolkit.org/pkg/handlers)
* [testing](https://golang.org/pkg/testing/)
* [net/http](https://godoc.org/net/http)
* [log](https://golang.org/pkg/log)

## References
* [Original SemaphoreCI tutorial and codebase](https://semaphoreci.com/community/tutorials/building-and-testing-a-rest-api-in-go-with-gorilla-mux-and-postgresql)
* [database/sql tutorial](http://go-database-sql.org/)
* [JSON encoding/decoding](https://blog.golang.org/json-and-go)
* When running a db unencrypted, [include 'sslmode=disable' in connection string](https://stackoverflow.com/questions/21959148/ssl-is-not-enabled-on-the-server)
* [Don't do local/relative imports](https://stackoverflow.com/questions/30885098/go-local-import-in-non-local-package)
* When using 'go run' of multiple-file package, [name all files in command](https://stackoverflow.com/questions/28153203/golang-undefined-function-declared-in-another-file)

