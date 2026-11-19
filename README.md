<div align="center">

# 🚀 SpringBoot Init

**A production-ready Spring Boot 2.7 backend template — clone it, replace a few configs, and your API is live in about a minute.**

**English** | [简体中文](README.zh-CN.md)

![Java](https://img.shields.io/badge/Java-8-orange?logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.7.2-6DB33F?logo=springboot&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-5.7%2B-4479A1?logo=mysql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-session%20store-DC382D?logo=redis&logoColor=white)
![Elasticsearch](https://img.shields.io/badge/Elasticsearch-7.x-06B8D4?logo=elasticsearch&logoColor=white)
![Knife4j](https://img.shields.io/badge/API%20Docs-Knife4j-3.0.3-blue)
![GitHub stars](https://img.shields.io/github/stars/Andrewlv/springboot-init?style=social)

</div>

---

## ✨ Features

**Everything you'd otherwise rebuild from scratch, already wired up:**

- 🔐 **Auth out of the box** — register / login / logout with Spring Session + Redis distributed sessions, a `@AuthCheck` role annotation (`user` / `admin` / `ban`) enforced by AOP, and WeChat Open Platform login
- 📋 **Typical business CRUD** — a complete posts domain (create / update / delete / search) with likes (👍) and favorites (⭐), ready to copy for your own entities
- 🔎 **Elasticsearch sync** — full and incremental scheduled jobs that mirror MySQL data into ES, plus flexible search endpoints
- 📄 **Excel import/export** — EasyExcel integration with a working example
- 📦 **File upload** — per-business uploads (validated by a `FileUploadBizEnum` whitelist) straight to Tencent Cloud COS
- 💬 **WeChat Official Account** — subscribe events, message routing, and menu config via `wx-java`
- 🛡️ **Engineering guardrails** — global exception handler, unified `BaseResponse` wrapper, custom error codes, global CORS, request logging AOP, and `Long → String` JSON serialization to prevent JS precision loss
- 📖 **Live API docs** — Swagger 3 + Knife4j UI, debug every endpoint without a frontend
- 🧪 **JUnit 5 test samples** — mapper, service, and manager examples showing how to test each layer
- 🏷️ **`todo` markers everywhere** — every value you need to replace is tagged `todo` in `application.yml`, so onboarding is a guided search instead of a guessing game

## 🧱 Tech Stack

| Category | Technology |
| --- | --- |
| Framework | Spring Boot 2.7.2 · Spring MVC · Spring AOP · Spring Scheduler · Spring TX |
| Data Access | MyBatis 2.2.2 · MyBatis-Plus 3.5.2 (pagination, logical delete) |
| Database | MySQL |
| Session / Cache | Redis · Spring Session Data Redis |
| Search | Spring Data Elasticsearch |
| Object Storage | Tencent Cloud COS (`cos_api` 5.6.x) |
| API Docs | SpringDoc (Swagger 3) · Knife4j 3.0.3 |
| WeChat | `wx-java-mp-spring-boot-starter` 4.4.0 (MP + Open Platform) |
| Utilities | Hutool 5.8.8 · EasyExcel 3.1.1 · Gson 2.9.1 · Commons Lang3 · Lombok |

## 📂 Project Structure

```
springboot-init
├── sql/                                  # create_table.sql · post_es_mapping.json
├── Dockerfile                            # Multi-stage Maven build (prod profile)
└── src/main
    ├── java/com/yupi/springbootinit
    │   ├── MainApplication.java          # Entry point
    │   ├── annotation/                   # @AuthCheck — role guard
    │   ├── aop/                          # AuthInterceptor · LogInterceptor
    │   ├── common/                       # BaseResponse · ErrorCode · ResultUtils · PageRequest
    │   ├── config/                       # CORS · Knife4j · MyBatis-Plus · COS · JSON · WeChat
    │   ├── constant/                     # Common · File · User constants
    │   ├── controller/                   # User · Post · PostThumb · PostFavour · File · WxMp
    │   ├── esdao/                        # PostEsDao — Elasticsearch data access
    │   ├── exception/                    # GlobalExceptionHandler · BusinessException · ThrowUtils
    │   ├── job/
    │   │   ├── once/FullSyncPostToEs     # One-shot full MySQL → ES sync
    │   │   └── cycle/IncSyncPostToEs     # Scheduled incremental sync
    │   ├── manager/                      # CosManager — file upload to COS
    │   ├── mapper/                       # MyBatis mappers
    │   ├── model/                        # DTOs · entities · enums · VOs
    │   ├── service/                      # User · Post · PostThumb · PostFavour services
    │   ├── utils/                        # Net · Sql · SpringContext utils
    │   └── wxmp/                         # WeChat MP message router + handlers
    └── resources
        ├── application.yml               # Shared config (active profile: dev)
        ├── application-{dev,test,prod}.yml
        └── mapper/                       # MyBatis XML mappers
```

## 🚀 Quick Start

### Prerequisites

| Requirement | Version | Notes |
| --- | --- | --- |
| JDK | 8+ | `pom.xml` sets `java.version = 1.8` |
| Maven | 3.6+ | Wrapper included — `./mvnw` works standalone |
| MySQL | 5.7+ | Required |
| Redis | 6+ | Optional — enables distributed login |
| Elasticsearch | 7.x | Optional — enables post search |

### 1. Create the database

The script creates the `my_db` database and four tables — `user`, `post`, `post_thumb`, `post_favour`:

```bash
mysql -u root -p < sql/create_table.sql
```

### 2. Point the app at your database

In `src/main/resources/application.yml`, replace the values marked `todo`:

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/my_db
    username: root
    password: 123456
```

### 3. Run the app

```bash
# Maven wrapper (no local Maven needed)
./mvnw spring-boot:run

# — or build a jar
./mvnw package -DskipTests
java -jar target/springboot-init-0.0.1-SNAPSHOT.jar
```

The service starts on **port 8101** with the base path `/api`.

### 4. Open the API docs

Head to <http://localhost:8101/api/doc.html> — you get a full interactive Swagger UI and can debug every endpoint without writing a frontend:

![Knife4j API docs](doc/swagger.png)

That's the whole happy path: database in, service up, docs live. ✅

## 🧩 Optional Modules

<details>
<summary><b>Redis — distributed login</b></summary>

1. Add your Redis settings to `application.yml` (uncomment the `spring.redis` block):

    ```yml
    spring:
      redis:
        database: 1
        host: localhost
        port: 6379
        timeout: 5000
        password: 123456
    ```

2. Enable Redis-backed sessions:

    ```yml
    spring:
      session:
        store-type: redis
    ```

3. Remove the `RedisAutoConfiguration` exclude in `MainApplication.java`:

    ```java
    // Before
    @SpringBootApplication(exclude = {RedisAutoConfiguration.class})
    // After
    @SpringBootApplication
    ```

</details>

<details>
<summary><b>Elasticsearch — post search</b></summary>

1. Configure the connection in `application.yml` (uncomment `spring.elasticsearch`):

    ```yml
    spring:
      elasticsearch:
        uris: http://localhost:9200
        username: root
        password: 123456
    ```

2. Create the `post_v1` index — paste the contents of `sql/post_es_mapping.json` into Kibana Dev Tools:

    ```
    PUT post_v1
    {
      // contents of sql/post_es_mapping.json
    }
    ```

3. Enable the sync jobs by uncommenting `@Component` in `job/once/FullSyncPostToEs.java` and `job/cycle/IncSyncPostToEs.java`, then restart the app:

    ```java
    // todo uncomment to enable the job
    @Component
    ```

</details>

<details>
<summary><b>WeChat — Official Account & Open Platform</b></summary>

Fill in the `wx.mp` (Official Account) and `wx.open` (Open Platform) blocks in `application.yml` with your app's `appId`, `secret`, `token`, and `aesKey`. Message routing and subscribe-event handling are already implemented under `wxmp/`.

</details>

<details>
<summary><b>Tencent COS — file upload</b></summary>

Fill in the `cos.client` block in `application.yml` with your `accessKey`, `secretKey`, `region`, and `bucket`. Uploads are validated against `FileUploadBizEnum` and served by `FileController` / `CosManager`.

</details>

## 🐳 Docker Deployment

The included multi-stage `Dockerfile` builds with Maven and runs the `prod` profile — configure `application-prod.yml` first:

```bash
docker build -t springboot-init .
docker run -d -p 8101:8101 springboot-init
```

## 🧪 Tests

```bash
./mvnw test
```

`src/test` contains JUnit 5 examples for every layer: mappers, services, the COS manager, the ES dao, and EasyExcel import/export.

## 🤝 Acknowledgments

This template is based on [程序员鱼皮 (liyupi)](https://github.com/liyupi)'s [springboot-init](https://github.com/liyupi/springboot-init) from the [编程导航知识星球](https://yupi.icu). Many thanks for open-sourcing the original scaffolding! 🙏

## ⭐ Support

If this template saved you setup time, consider giving the repo a star — it helps other developers find it.
