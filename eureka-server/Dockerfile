# Stage 1: Build & Extract layers
FROM maven:3.9-eclipse-temurin-17-alpine AS builder
WORKDIR /app

COPY pom.xml .
COPY checkstyle.xml .
COPY eureka-server/pom.xml eureka-server/
RUN mvn dependency:go-offline -B -pl eureka-server -am

COPY eureka-server/src eureka-server/src
RUN mvn clean package -pl eureka-server -am -DskipTests && \
    java -Djarmode=layertools -jar eureka-server/target/*.jar extract --destination /app/extracted

# Stage 2: Runtime
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app

RUN apk add --no-cache curl && \
    addgroup -S spring && adduser -S spring -G spring
USER spring:spring

COPY --from=builder /app/extracted/dependencies/ ./
COPY --from=builder /app/extracted/spring-boot-loader/ ./
COPY --from=builder /app/extracted/snapshot-dependencies/ ./
COPY --from=builder /app/extracted/application/ ./

EXPOSE 8761

ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS org.springframework.boot.loader.launch.JarLauncher"]