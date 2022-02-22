## DNS 기반 라운드로빈 방식

1. 시스템 환경을 시작하고 다음 명령을 사용하여 일부 테스트 데이터를 삽입합니다.
```
./test-em-all.bash start
``

2. review 마이크로서비스를 두 개의 인스턴스로 확장합니다.
```
docker-compose up -d --scale review=2
```

3. review 서비스에 대해 찾은 IP 주소에 대해 composite-product 서비스에 요청합니다.
```
docker-compose exec product-composite getent host review
```

## 디스커버리 서비스 사용해보기

1. 이미지 빌드
```
./gradlew build && docker-compose build
```

2. 테스트 자동

```
./test-em-all.bash start
```

### 스케일 업

1. Launch two extra review microservice instances:
```
docker-compose up -d --scale review=3
```

브라우저에서

`http://localhost:8761/` 

또는 

```
curl -H "accept:application/json" localhost:8761/eureka/apps -s | jq -r .applications.application[].instance[].instanceId
```

review instance's log records with the following command:
```
docker-compose logs review | grep getReviews
```