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

## 노출한 API 확인

1. docker-compose 명령을 사용하여 서비스에서 노출되는 포트를 확인합니다.
```
docker-compose ps gateway eureka product-composite product recommendation review
```

2. 에지 서버가 설정한 경로를 확인

```bash
curl localhost:8080/actuator/gateway/routes -s | jq '.[] | {"\(.route_id)": "\(.uri)"}' | grep -v '{\|}'
```

## ### 라우팅 규칙 테스트



