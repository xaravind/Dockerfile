# Stage 1: Build with Maven (Eclipse Temurin JDK 24 on Alpine)
FROM maven:3.9-eclipse-temurin-24-alpine AS build

WORKDIR /app

COPY . .

RUN mvn package -DskipTests


# Stage 2: Runtime with OpenJDK 17 JRE slim
FROM openjdk:25-slim-bullseye

WORKDIR /app

# Create a non-root user and group named ajabackend
RUN groupadd -r ajabackend && useradd -r -g ajabackend ajabackend

COPY --from=build /app/target/*.jar /app/app.jar

# Change ownership of app folder to the new user
RUN chown -R ajabackend:ajabackend /app

# Switch to non-root user
USER ajabackend

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app/app.jar"]
