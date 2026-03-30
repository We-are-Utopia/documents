# syntax=docker/dockerfile:1.7
FROM maven:3.9.9-eclipse-temurin-21 AS build
WORKDIR /src
COPY pom.xml ./
COPY .mvn .mvn
COPY mvnw ./
RUN --mount=type=cache,target=/root/.m2 (./mvnw -B -q -DskipTests dependency:go-offline || mvn -B -q -DskipTests dependency:go-offline)
COPY src src
RUN --mount=type=cache,target=/root/.m2 (./mvnw -B -DskipTests package || mvn -B -DskipTests package)

FROM eclipse-temurin:21-jre-jammy AS runtime
RUN apt-get update && apt-get install -y --no-install-recommends curl \
    && rm -rf /var/lib/apt/lists/* \
    && useradd -r -u 10001 spring
WORKDIR /app
COPY --from=build /src/target/*.jar app.jar
USER spring
EXPOSE 8080
HEALTHCHECK --interval=30s --timeout=5s --start-period=30s --retries=3 \
  CMD curl -fsS http://127.0.0.1:8080/actuator/health || exit 1
CMD ["java", "-XX:+UseContainerSupport", "-XX:MaxRAMPercentage=75", "-jar", "/app/app.jar"]
