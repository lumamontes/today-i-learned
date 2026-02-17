# monorepo structure patterns for full-stack apps

Building tarefitas-monorepo taught me practical patterns for organizing multi-platform applications in a single repository.

## Directory Structure

Organize by platform/technology rather than features:

```
tarefitas-monorepo/
├── backend/          # Kotlin Spring Boot API
│   ├── src/
│   ├── build.gradle.kts
│   └── Dockerfile
├── frontend/         # Web app (React/Vue/etc)
│   ├── src/
│   ├── package.json
│   └── dist/
├── mobile/          # React Native/Flutter app
│   ├── src/
│   ├── package.json
│   └── app.config.js
├── shared/          # Common types, utils, constants
│   └── types/
└── docker-compose.yml
```

## Benefits of This Structure

- **Clear separation of concerns**: Each platform has its own build process
- **Independent deployments**: Backend, frontend, mobile can deploy separately  
- **Shared code**: Common types and utilities in `/shared`
- **Single repository**: Easier coordination between teams

## Package Management

Use workspace features for dependency management:

```json
// package.json (root)
{
  "workspaces": [
    "frontend",
    "mobile",
    "shared"
  ]
}
```

This allows installing all JS dependencies from root: `npm install`

## Docker Compose Orchestration

Define all services in one compose file:

```yaml
services:
  backend:
    build: ./backend
    ports: ["8080:8080"]
  
  frontend:
    build: ./frontend  
    ports: ["3000:3000"]
    
  postgres:
    image: postgres:16
```

## Cross-Platform Development

Keep shared interfaces consistent:

```typescript
// shared/types/task.ts
export interface Task {
  id: string;
  title: string;
  completed: boolean;
}

// Used in backend (Kotlin), frontend (TS), and mobile (TS)
```

This structure scales well as teams and features grow.