# Build stage
FROM eclipse-temurin:21-jdk-alpine AS simple-api-build

ENV SIMPLE_API_HOME=/opt/simple-api
WORKDIR $SIMPLE_API_HOME

RUN apk add --no-cache maven

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package -DskipTests

# Runtime stage
FROM eclipse-temurin:21-jre-alpine

ENV SIMPLE_API_HOME=/opt/simple-api
WORKDIR $SIMPLE_API_HOME

COPY --from=simple-api-build $SIMPLE_API_HOME/target/*.jar $SIMPLE_API_HOME/simple-api.jar

ENTRYPOINT ["java", "-jar", "simple-api.jar"]
