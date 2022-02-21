## 

1. Build the Docker images with the following commands:
```sh
./gradlew clean build && docker-compose build
```
2. Start the system landscape in Docker and run the usual tests with the following command:
```sh
./test-em-all.bash start
```

3. From the log output, note the second to last test result, `http://localhost:8080`. That is the output from the test that verifies that the server URL in Swagger UI's OpenAPI specification is correctly rewritten to be the URL of the edge server.