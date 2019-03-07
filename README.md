# swagger-all-in-one-docker-container
(Some texts are from upstream - but they mostly are changed here!)
# Overview
This is all-in-one Swagger tool set of dockers using docker-compose.
- Swagger Editor, 
- Swagger UI, 
- Swagger Validator,
- Swagger mock api server(openapi: 3.x), and
- nginx to handle them more easily.
If you want to write swagger spec as `swagger: "2.0"`, use `swagger2.0` branch.
There is a sample swagger spec in this so the Editor, UI and the mock API server will run without any configuration from the start.
All you need to do is edit the swagger spec, save as openapi.json, and restart docker. Voila, UI and the mock API server are updated.

# Run
```
docker-compose up -d
docker ps
```
Here are the list of dockers created by docker-compose (yours might look a bit differently):
```
    Name	            Stack	                            Image	                        Published Ports
-----------------------------------------------------------------------------------------------------------
   mongo-compass-docker	swagger-all-in-one-docker-compose	openkbs/mongo-gui-docker
   restheart-docker	    swagger-all-in-one-docker-compose	softinstigate/restheart	        38080:8080
   restheart-mongodb	swagger-all-in-one-docker-compose	mongo:latest	                37017:27017
   swagger-ui	        swagger-all-in-one-docker-compose	swaggerapi/swagger-ui	        38082:8080
   swagger-api	        swagger-all-in-one-docker-compose	danielgtaylor/apisprout	        38083:8000
   swagger-editor	    swagger-all-in-one-docker-compose	swaggerapi/swagger-editor	    38081:8080
   swagger-validator	swagger-all-in-one-docker-compose	swaggerapi/swagger-validator    38085:8080
   swagger-nginx	    swagger-all-in-one-docker-compose	nginx:mainline-alpine	        38084:8084
```

# Use
1. Edit swagger spec with swagger-editor
2. Save swagger spec as json from swagger-editor File menu
3. Move and save the json file as `swagger/openapi.json`
4. Execute `docker-compose restart` and swagger-ui and swagger-api(mock server) will be updated
5. If you want to read an external openapi.json file, import the file from swagger-editor `File > Import File` menu.
6. Restheart login: 
```
    (default from etc/scurity.yml)
    user1/changeit
    user2/changeit
    admin/changeit
```
7. For Mongodb Compass to connect to Mongodo - Use the following unless you change the default setup:
```
Host: <Host_Ip>
Port: 37017
Authentication: restheart/R3ste4rt!
```

## Troubleshooting / Heads-up
- When the UI is referenced as `http://localhost:8082/`, cache might be used even the changes are made in swagger files. So it may be better to refference as `http://127.0.0.1:8082/`(api, too.)
- When swagger-api failed to run, it's likely that api server failed to run because the openapi.json was not properly read. So use `docker logs` command and get rid of the cause then restart it again.
- If you want to access swagger-api from other domains(CORS), access swagger-api through swagger-nginx.

# swagger-editor
To access, try:
```
http://0.0.0.0:38081/
```
- Can edit swagger spec
- Can export swagger spec as json, yaml and etc. swagger-ui can read the files and they can be beautifly referenced as documentation. apisprout can read the yml and json then it can serve the mock API.

# swagger-ui
To access, try:
```
http://0.0.0.0:38082/
```
- Can referrence the documentation from swagger spec.
- swagger spec can be assigned from json file path or API_URL path.
```
environment:
  SWAGGER_JSON: /openapi.json
  # API_URL: ""
```
- ./swagger/openapi.json is referenced in this repository

# swagger-api (apisprout)
- Internally using [apisprout](https://github.com/danielgtaylor/apisprout) in docker.
- swagger spec is compatible with `openapi: 3.x`.
- ./swagger/openapi.json is also refferenced from api in this repository.
- However, apisprout can not add `Access-Control-Allow-Origin` to Header so I put nginx in front of swagger-api and add it to Header then proxy to swagger-api.(To make it accessable from othe domains. CORS.)

# swagger-nginx (Proxy service for Swagger-API)
- Placed to modify Header.
- Mock API (swagger-api) can be accessed from `38084` port via nginx.
- Of course, you can use the api from curl, etc.
```
firefox http://127.0.0.1:38084/pets
```
Or, command line:
```json
curl -i -X GET http://127.0.0.1:38084/pets/1 -H "accept: application/json"

HTTP/1.1 200 OK
Server: nginx/1.15.7
Date: Sun, 23 Dec 2018 20:02:56 GMT
Content-Type: application/json
Content-Length: 49
Connection: keep-alive
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: POST, GET, PATCH, DELETE, PUT, OPTIONS
Access-Control-Allow-Headers: Origin, Authorization, Accept
Access-Control-Allow-Credentials: true

{
  "id": 1,
  "name": "doggie",
  "tag": "dog"
}
```

# swagger-validator
- REST API validation
* example
```
firefox http://localhost:38085/validator?url=https://petstore.swagger.io/v2/swagger.json
firefox http://0.0.0.0:38085/validator?url=http://generator.swagger.io/api/swagger.json
```
# References
* [Restheart.org](https://restheart.org/)
* [OpenAPI-Specification](https://github.com/OAI/OpenAPI-Specification/)
* [OpenAPI-Specification Example in json](https://github.com/OAI/OpenAPI-Specification/tree/master/examples/v2.0/json)
