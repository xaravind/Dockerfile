# Use a base image with Java runtime (OpenJDK 21)
FROM openjdk:21-jdk-slim

# Set the working directory in the container
WORKDIR /app

# Copy the built JAR file into the container
COPY target/*.jar /app/

# Expose the port the Spring Boot app will run on
EXPOSE 8080

# Install MySQL client tools to run MySQL commands
RUN apt-get update && \
    apt-get install -y default-mysql-client && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
# Set environment variables for MySQL connection
ENV SPRING_DATASOURCE_URL=jdbc:mysql://13.217.197.65:3306/aja_lib \
    SPRING_DATASOURCE_USERNAME=admin \
    SPRING_DATASOURCE_PASSWORD=Admin@1234!

# Run the Spring Boot application
ENTRYPOINT ["sh", "-c", "java -jar /app/*.jar"]