# Start [OpenAPITools][]

[OpenAPITools]: https://github.com/OpenAPITools

## Quick start

Generate Go API client for Petstore server (in Docker):

```bash
docker run --rm -v "${PWD}:/local" openapitools/openapi-generator-cli generate \
-i https://raw.githubusercontent.com/openapitools/openapi-generator/master/modules/openapi-generator/src/test/resources/3_0/petstore.yaml \
-g go \
-o /local/out/go
```

The generated code will be located under `./out/go` in the current directory.

<!--
docker pull swaggerapi/swagger-editor
docker run -d -p 80:8080 swaggerapi/swagger-editor

docker pull swaggerapi/swagger-ui
docker run -p 80:8080 swaggerapi/swagger-ui
-->

Open [Swagger Editor](https://editor.swagger.io/) `File > Import URL` to see `petstore.yaml` API:

![](img/petstore.yaml.png)

See `openapi-generator-cli` usage:

```bash
docker run --rm -it \
-v "${PWD}:/local" \
--entrypoint /bin/bash \
openapitools/openapi-generator-cli

ln -s /usr/local/bin/docker-entrypoint.sh /usr/local/bin/openapi-generator
openapi-generator help
openapi-generator help generate
```

## Let's practice

### Design RESTful API

Open [Swagger Editor](https://editor.swagger.io/) design our api:

- /albums
  - GET - Get all albums
  - POST - Create a new album
- /albums/:id
  - GET - Get an album by its id
  - PUT - Update an album by its id
  - DELETE - Delete an album by its id

We could see [spec/api.yaml](spec/api.yaml), it mainly consists of three parts:

1. header: api information

    ```yaml
    openapi: 3.0.0
    info:
      title: Start OpenAPITools
      description: Let's practice designing our api.
      version: 0.1.0
      license:
        name: MIT
        url: https://spdx.org/licenses/MIT.html
    servers:
      - url: https://github.com/ikuokuo/start-openapitools
    ```

2. body: `paths` and their operations (`get`, `post`, etc.)

    ```yaml
    paths:
      /albums/{id}:
        get:
          tags:
            - album
          summary: Get an album by its id
          operationId: getAlbum
          parameters:
            - $ref: '#/components/parameters/AlbumId'
          responses:
            200:
              description: Get success, return the album
              content:
                application/json:
                  schema:
                    $ref: '#/components/schemas/Album'
            404:
              description: Album not found
    ```

3. footer: reusable `components` that could be referenced in doc

    ```yaml
    components:
      parameters:
        AlbumId:
          name: id
          in: path
          description: Album id
          required: true
          schema:
            type: string
      schemas:
        Album:
          title: Album
          type: object
          required:
            - title
            - artist
            - price
          properties:
            id:
              type: string
              format: uuid
            title:
              type: string
              maxLength: 200
              minLength: 1
            artist:
              type: string
              maxLength: 100
              minLength: 1
            price:
              type: number
              format: double
              minimum: 0.0
            created_at:
              type: string
              format: date-time
            updated_at:
              type: string
              format: date-time
    ```

Read [OpenAPI Specification](https://spec.openapis.org/oas/latest.html) to learn more.

### Generate Online

We could generate codes easily using online service:

- latest stable version: https://api.openapi-generator.tech

Below we will practice generating codes manually.

### Generate Server Stub

Generate Go [Gin](https://github.com/gin-gonic/gin) stub:

```bash
docker run --rm -it \
-v "${PWD}:/local" \
--entrypoint /bin/bash \
openapitools/openapi-generator-cli

ln -s /usr/local/bin/docker-entrypoint.sh /usr/local/bin/openapi-generator

# Config Options for go-gin-server
#  https://openapi-generator.tech/docs/generators/go-gin-server
openapi-generator config-help -g go-gin-server

openapi-generator generate \
-g go-gin-server \
-i /local/spec/api.yaml \
-o /local/out/gin-server \
--additional-properties=packageName=startapi
```

Output contents:

```bash
❯ tree out/gin-server -aF --dirsfirst
out/gin-server
├── .openapi-generator/
├── api/
│   └── openapi.yaml
├── go/
│   ├── README.md
│   ├── api_album.go
│   ├── api_albums.go
│   ├── model_album.go
│   └── routers.go
├── .openapi-generator-ignore
├── Dockerfile
└── main.go
```

Implement `GetAlbum()` in `go/api_album.go`:

```go
// GetAlbum - Get an album by its id
func GetAlbum(c *gin.Context) {
	c.JSON(http.StatusOK, Album{
		Id:        "3fa85f64-5717-4562-b3fc-2c963f66afa6",
		Title:     "Start OpenAPITools",
		Artist:    "GoCoding",
		Price:     0.99,
		CreatedAt: time.Now(),
		UpdatedAt: time.Now(),
	})
}
```

Run the server:

```bash
cd out/gin-server/
go mod init github.com/ikuokuo/start-openapitools/gin-server
go mod tidy

# change import path in `main.go`
#  sw "github.com/ikuokuo/start-openapitools/gin-server/go"
go mod edit -replace github.com/ikuokuo/start-openapitools/gin-server/go=./go
```

```bash
❯ go run .
2021/11/05 18:20:00 Server started
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:   export GIN_MODE=release
 - using code:  gin.SetMode(gin.ReleaseMode)

[GIN-debug] GET    /ikuokuo/start-openapitools/ --> github.com/ikuokuo/start-openapitools/gin-server/go.Index (3 handlers)
[GIN-debug] DELETE /ikuokuo/start-openapitools/albums/:id --> github.com/ikuokuo/start-openapitools/gin-server/go.DeleteAlbum (3 handlers)
[GIN-debug] GET    /ikuokuo/start-openapitools/albums/:id --> github.com/ikuokuo/start-openapitools/gin-server/go.GetAlbum (3 handlers)
[GIN-debug] PUT    /ikuokuo/start-openapitools/albums/:id --> github.com/ikuokuo/start-openapitools/gin-server/go.PutAlbum (3 handlers)
[GIN-debug] GET    /ikuokuo/start-openapitools/albums --> github.com/ikuokuo/start-openapitools/gin-server/go.GetAlbums (3 handlers)
[GIN-debug] POST   /ikuokuo/start-openapitools/albums --> github.com/ikuokuo/start-openapitools/gin-server/go.PostAlbums (3 handlers)
[GIN-debug] Listening and serving HTTP on :8080

❯ curl http://localhost:8080/ikuokuo/start-openapitools/
Hello World!
```

### Generate Client SDK

Generate Python SDK:

```bash
docker run --rm -it \
-v "${PWD}:/local" \
--entrypoint /bin/bash \
openapitools/openapi-generator-cli

ln -s /usr/local/bin/docker-entrypoint.sh /usr/local/bin/openapi-generator

# Config Options for python
#  https://openapi-generator.tech/docs/generators/python
openapi-generator config-help -g python

openapi-generator generate \
-g python \
-i /local/spec/api.yaml \
-o /local/out/py-sdk \
--additional-properties=packageName=startapi \
--additional-properties=library=urllib3
```

Output contents:

```bash
❯ tree out/py-sdk -aF --dirsfirst
out/py-sdk
├── .openapi-generator/
├── docs/
├── startapi/
│   ├── api/
│   │   ├── __init__.py
│   │   ├── album_api.py
│   │   └── albums_api.py
│   ├── apis/
│   │   └── __init__.py
│   ├── model/
│   │   ├── __init__.py
│   │   └── album.py
│   ├── models/
│   │   └── __init__.py
│   ├── __init__.py
│   ├── api_client.py
│   ├── configuration.py
│   ├── exceptions.py
│   ├── model_utils.py
│   └── rest.py
├── test/
├── .gitignore
├── .gitlab-ci.yml
├── .openapi-generator-ignore
├── .travis.yml
├── README.md
├── git_push.sh
├── requirements.txt
├── setup.cfg
├── setup.py
├── test-requirements.txt
└── tox.ini
```

Test the SDK:

```bash
❯ cd out/py-sdk/
❯ python - <<EOF
from startapi import ApiClient, Configuration, apis

config = Configuration(host="http://localhost:8080/ikuokuo/start-openapitools")
with ApiClient(configuration=config) as client:
  api = apis.AlbumApi(client)
  album = api.get_album("xxxxx")
  print(album)
EOF
{'artist': 'GoCoding',
 'created_at': datetime.datetime(2021, 11, 5, 18, 30, 0, 545305, tzinfo=tzoffset(None, 28800)),
 'id': '3fa85f64-5717-4562-b3fc-2c963f66afa6',
 'price': 0.99,
 'title': 'Start OpenAPITools',
 'updated_at': datetime.datetime(2021, 11, 5, 18, 30, 0, 545305, tzinfo=tzoffset(None, 28800))}
```

## References

- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [OpenAPI Generator](https://github.com/OpenAPITools/openapi-generator)
  - [OpenAPI Generator vs Swagger Codegen](https://openapi-generator.tech/docs/faq/#what-is-the-difference-between-swagger-codegen-and-openapi-generator)
