---
title: "Hello World - Java (Quarkus)"
linkTitle: "Java (Quarkus)"
weight: 1
type: "docs"
---

A simple [JAX-RS REST API](https://github.com/jax-rs) application that is
written in Java and uses [Quarkus](https://quarkus.io/).

This samples uses Docker to build locally. The app reads in a `TARGET` env
variable and then prints "Hello World: \${TARGET}!". If a value for `TARGET` is
not specified, the "NOT SPECIFIED" default value is used.

## Before you begin

You must meet the following requirements to run this sample:

- An installed version of the following tools:
  - [Java SE 8 or later JDK](https://www.eclipse.org/openj9/)
  - [Maven](https://maven.apache.org/download.cgi)

## Getting the code

You can either clone a working copy of the sample code from the repository, or
following the steps in the
[Recreating the sample code](#recreating-the-sample-code) to walk through the
steps of updating all the files.

### Recreating the sample code

Use the following steps to obtain an incomplete copy of the sample code for
which you update and create the necessary build and configuration files:

1. From the console, create a new empty web project using the Maven archetype
   commands:

   ```shell
   mvn io.quarkus:quarkus-maven-plugin:0.13.3:create \
    -DprojectGroupId=com.redhat.developer.demos \
    -DprojectArtifactId=helloworld-java-quarkus \
    -DclassName="com.redhat.developer.demos.GreetingResource" \
    -Dpath="/"
   ```

1. Update the `GreetingResource` class in
   `src/main/java/com/redhat/developer/demos/GreetingResource.java` to handle
   the "/" mapping and also add a `@ConfigProperty` field to provide the TARGET
   environment variable:

   ```java
   package com.redhat.developer.demos;

   import javax.ws.rs.GET;
   import javax.ws.rs.Path;
   import javax.ws.rs.Produces;
   import javax.ws.rs.core.MediaType;
   import org.eclipse.microprofile.config.inject.ConfigProperty;

   @Path("/")
   public class GreeterResource {
     @ConfigProperty(name = "TARGET", defaultValue="World")
     String target;

     @GET
     @Produces(MediaType.TEXT_PLAIN)
     public String greet() {
       return "Hello " + target + "!";
     }
   }
   ```

1. Update `src/main/resources/application.properties` to configuration the
   application to default to port 8080, but allow the port to be overriden by
   the `PORT` environmental variable:

   ```
   # Configuration file
   # key = value

   quarkus.http.port=${PORT:8080}
   ```

1. Update `src/test/java/com/redhat/developer/demos/GreetingResourceTest.java`
   test to reflect the change:

   ```java
   package com.redhat.developer.demos;

   import io.quarkus.test.junit.QuarkusTest;
   import org.junit.jupiter.api.Test;

   import static io.restassured.RestAssured.given;
   import static org.hamcrest.CoreMatchers.is;

   @QuarkusTest
   public class GreetingResourceTest {

     @Test
     public void testHelloEndpoint() {
         given()
           .when().get("/")
           .then()
             .statusCode(200)
             .body(is("Hello World!"));
     }
   }

   ```

1. Remove `src/main/resources/META-INF/resources/index.html` file since it's
   unncessary for this example.

   ```shell
   rm src/main/resources/META-INF/resources/index.html
   ```

1. Remove `.dockerignore` file since it's unncessary for this example.

   ```shell
   rm .dockerignore
   ```

1. In your project directory, create a file named `Dockerfile` and copy the code
   block below into it.

   ```docker
   FROM quay.io/rhdevelopers/quarkus-java-builder:graal-1.0.0-rc15 as builder
   COPY . /project
   WORKDIR /project
   # uncomment this to set the MAVEN_MIRROR_URL of your choice, to make faster builds
   # ARG MAVEN_MIRROR_URL=<your-maven-mirror-url>
   # e.g.
   #ARG MAVEN_MIRROR_URL=http://192.168.64.1:8081/nexus/content/groups/public

   RUN /usr/local/bin/entrypoint-run.sh mvn -DskipTests clean package

   FROM fabric8/java-jboss-openjdk8-jdk:1.5.4
   USER jboss
   ENV JAVA_APP_DIR=/deployments

   COPY --from=builder /project/target/lib/* /deployments/lib/
   COPY --from=builder /project/target/*-runner.jar /deployments/app.jar

   ENTRYPOINT [ "/deployments/run-java.sh" ]
   ```

   If you want to build Quarkus native image, then copy the following code block
   in to file called `Dockerfile.native`

   ```docker
   FROM quay.io/rhdevelopers/quarkus-java-builder:graal-1.0.0-rc15 as builder
   COPY . /project
   # uncomment this to set the MAVEN_MIRROR_URL of your choice, to make faster builds
   # ARG MAVEN_MIRROR_URL=<your-maven-mirror-url>
   # e.g.
   # ARG MAVEN_MIRROR_URL=http://192.168.64.1:8081/nexus/content/groups/public

   RUN /usr/local/bin/entrypoint-run.sh mvn -DskipTests clean package -Pnative

   FROM registry.fedoraproject.org/fedora-minimal

   COPY --from=builder /project/target/helloworld-java-quarkus-runner /app

   ENTRYPOINT [ "/app" ]
   ```

## Locally testing your sample

1. Run the application locally:

   ```shell
   ./mvnw compile quarkus:dev
   ```

   Go to `http://localhost:8080/` to see your `Hello World!` message.
