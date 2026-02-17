# docker multi-stage builds for kotlin gradle projects

Working on the tarefitas-monorepo, I learned how to set up efficient Docker builds for Kotlin backend services with Gradle.

## Multi-Stage Dockerfile Structure

Use separate stages for building and running to reduce image size:

```dockerfile
# Build stage
FROM gradle:8.11-jdk21 AS build
WORKDIR /app
COPY . .
RUN gradle build --no-daemon

# Runtime stage  
FROM openjdk:21-jre-slim
WORKDIR /app
COPY --from=build /app/build/libs/*.jar app.jar
EXPOSE 8080
CMD ["java", "-jar", "app.jar"]
```

## Key Benefits

- **Smaller images**: Runtime stage only contains JRE and built JAR
- **Build caching**: Gradle dependencies cached in build layer
- **Security**: No build tools in production image

## Docker Compose Integration

Combine with Postgres and health checks:

```yaml
version: '3.8'
services:
  backend:
    build: .
    ports:
      - "8080:8080"
    depends_on:
      postgres:
        condition: service_healthy
    environment:
      - DATABASE_URL=jdbc:postgresql://postgres:5432/tarefitas

  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: tarefitas
      POSTGRES_PASSWORD: password
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

## Gradle Build Optimization

Speed up builds by copying dependency info first:

```dockerfile
# Copy gradle files for dependency resolution
COPY build.gradle.kts settings.gradle.kts ./
COPY gradle gradle
RUN gradle dependencies --no-daemon

# Then copy source and build
COPY src src  
RUN gradle build --no-daemon
```

This leverages Docker layer caching - dependencies only rebuild when gradle files change.