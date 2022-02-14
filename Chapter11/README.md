# All

### 자동 테스트를 빌드하고 실행

To build and run automated tests, we perform the following steps:

1. 도커 이미지를 빌드합니다
```
./gradlew build && docker-compose build
```
2. 도커로 시스템을 시작하고 테스트를 실행합니다
```
./test-em-all.bash start
```
네거티브 테스트 끝에서는 인증이 안된 경우 `401 Unauthorized` 코드를 받으며 권한이 없는 경우는 `403 Forbidden` 코드를 얻는지 확인합니다.


## ### 액세스 토큰 얻기

**클라이언트 자격증명 방식**

writer에 대한 액세스 토큰을 얻어 옵니다:

```sh
curl -k https://writer:secret@localhost:8443/oauth2/token -d grant_type=client_credentials -s | jq .
```

응답은
```
{
  "access_token": "eyJraWQiOiI3NjAyYTFiNS0wYjRkLTQyZWEtOGQ5MC05OTY1NmVmZmEzYWUiLCJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJ3cml0ZXIiLCJhdWQiOiJ3cml0ZXI...",
  "scope": "product:write openid product:read",
  "token_type": "Bearer",
  "expires_in": "3599"
}
```
- 액세스 토큰
- 토큰에 부여된 스코프. The writer client is granted both the `product:write` and `product:read` scope. It is also granted the `openid` scope, allowing access to information regarding the user's ID, such as an email address.
- 토큰 타입은 `Bearer`; 이 토큰의 베어러는 토큰에게 부여된 스코프에 따라 액세스를 갖는다는 의미
- 유효기간 3599초

이번에는, reader를 위한 액세스 토큰을 받아옵니다.

```sh
curl -k https://reader:secret@localhost:8443/oauth2/token -d grant_type=client_credentials -s | jq .
```
응답은
```
{
  "access_token": "eyJraWQiOiI3NjAyYTFiNS0wYjRkLTQyZWEtOGQ5MC05OTY1NmVmZmEzYWUiLCJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdWIiOiJyZWFkZXIiLCJhdWQiOiJyZWFkZXIiL...",
  "scope": "openid product:read",
  "token_type": "Bearer",
  "expires_in": "3599"
}
```

### 액세스 토큰으로 APIs 액세스

`OAuth 2.0`  액세스 토큰은 앞에 `Bearer`를 붙여 표준 HTTP authorization header로 보내집니다.

Run the following commands to call the protected APIs:

1. First, call an API to retrieve a `composite product` without a valid access token:
```
ACCESS_TOKEN=an-invalid-token
curl https://localhost:8443/product-composite/1 -k -H "Authorization: Bearer $ACCESS_TOKEN" -i  
```
It should return the following response:

```
HTTP/1.1 401 Unauthorized
```

The error message clearly states that the access token is invalid!

2. Next, try using the API to retrieve a composite product using one of the access tokens acquired for the reader client from the previous section:
```
ACCESS_TOKEN={a-reader-access-token}
curl https://localhost:8443/product-composite/1 -k -H "Authorization: Bearer $ACCESS_TOKEN" -i 
```
Now we will get the 200 OK status code and the expected response body will be returned:

```
HTTP/1.1 200 OK
```
Figure 11.12: Valid access token results in a 200 OK response

3. If we try to access an updating API, for example, the delete API, with an access token acquired for the `reader` client, the call will fail:
```
READER_ACCESS_TOKEN={a-reader-access-token}
curl https://localhost:8443/product-composite/999 -k -H "Authorization: Bearer $READER_ACCESS_TOKEN" -X DELETE -i  
```

It will fail with a response similar to the following:

```
HTTP/1.1 403 Forbidden
```

From the error response, it is clear that we are forbidden to call the API since the request requires higher privileges than what our access token is granted.

4. If we repeat the call to the delete API, but with an access token acquired for the writer client, the call will succeed with 200 OK in the response.

> The delete operation should return 200 even if the product with the specified product ID does not exist in the underlying database, since the delete operation is idempotent, as described in Chapter 6, Adding Persistence. Refer to the Adding new APIs section.

If you look into the log output using the `docker-compose logs -f product-composite` command, you should be able to find authorization information such as the following:


Figure 11.14: Authorization info in the log output

This information was extracted in the `product-composite` service from the JWT-encoded access token; the `product-composite` service did not need to communicate with the authorization server to get this information!


## 인증서버로 Auth0

export TENANT=dev-wemeetplace.us.auth0.com
export WRITER_CLIENT_ID=0C2IJVeswSuspJA6fbwBh5YyCnzgzmLz
export WRITER_CLIENT_SECRET=VuDoeZ_0HMYqACznyqRoig2yqE2jvHWwEFXToK4HP7qnNx8L4VkHlqLqflrZsOlX
export READER_CLIENT_ID=OorSAjMrLPlWWOQV4AWHEiYD1TpeKtZk
export READER_CLIENT_SECRET=dUbLg0ZZLefQPGce_u0PaNO5EjeWc9uyarscfFAYSd--2yTfrC_Tu3TmrnUgewHn