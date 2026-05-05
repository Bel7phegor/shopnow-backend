# Stage 1: Build the application
FROM openjdk:17.0.1-jdk-slim AS builder

WORKDIR /app
COPY . .
RUN chmod +x ./mvnw
RUN ./mvnw clean package -DskipTests

# Stage 2: Run the application
FROM openjdk:17.0.1-jdk-slim

WORKDIR /app
COPY --from=builder /app/target/config-server-0.0.1-SNAPSHOT.war /app/config-server.war

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar api-gateway.war"]
