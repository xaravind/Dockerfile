# Use a base image with Java runtime (OpenJDK 21)
FROM openjdk:21-jdk-slim

# Set the working directory in the container
WORKDIR /app

# Copy the built JAR file into the container
COPY target/*.jar /app/app.jar

# Expose the port the Spring Boot app runs on
EXPOSE 8080

# Set environment variables for MySQL connection
ENV SPRING_DATASOURCE_URL=jdbc:mysql://mysql:3306/aja_lib \
    SPRING_DATASOURCE_USERNAME=admin \
    SPRING_DATASOURCE_PASSWORD=Admin@1234!

# Run the Spring Boot application
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
