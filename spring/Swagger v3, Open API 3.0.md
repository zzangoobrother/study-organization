## Swagger v3 + Open API 3.0

````shell
implementation 'org.springdoc:springdoc-openapi-ui:1.7.0'
````

### 사용이유
**springfox vs. springdoc**

springfox는 마지막 업데이트가 2020년이며 github issues에도 springdoc을 권고하고 있습니다. 
springdoc은 업데이트가 지속되기 때문에 최근에는 springdoc을 많이 사용하고 있습니다.

Controller
- @Tag : 하나의 api 그룹으로 묶는 역할
- @Schema : Dto에 대한 정보를 작성하는 것
- @Operation : Controller method 단에서 작성. api 하나를 가리킴
- @ApiResponse : 응답 결과에 따른 구조 작성
- @Parameter : 컨트롤러 메서드의 매개변수에 대한 설명 작성

#### 참조
https://blog.jiniworld.me/91
https://jeonyoungho.github.io/posts/Open-API-3.0-Swagger-v3-%EC%83%81%EC%84%B8%EC%84%A4%EC%A0%95/
