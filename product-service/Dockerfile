# Stage 1: Build the application
FROM openjdk:17.0.1-jdk-slim AS builder

WORKDIR /app
COPY . .
RUN chmod +x ./mvnw
RUN ./mvnw clean package -DskipTests

# Stage 2: Run the application
FROM openjdk:17.0.1-jdk-slim

WORKDIR /app
COPY --from=builder /app/target/product-service-0.0.1-SNAPSHOT.war /app/product-service.war

EXPOSE 5861
ENTRYPOINT exec java -jar /app/product-service.war --spring.config.location=/app/src/main/resources/application.properties
