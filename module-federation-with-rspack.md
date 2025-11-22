# module-federation-with-rspack

Module Federation is a powerful feature that allows JavaScript applications to dynamically load code from other applications at runtime. When combined with Rspack (a fast Rust-based web bundler), it enables building microfrontend architectures with excellent performance.

## What is Module Federation?

Module Federation allows a JavaScript application to use modules from another application without bundling them together. This enables:

- **Independent deployments**: Each application can be deployed separately
- **Runtime code sharing**: Share code between applications at runtime
- **Team autonomy**: Different teams can work on different parts independently
- **Smaller bundles**: Only load what you need, when you need it

## What is Rspack?

Rspack is a fast, Rust-based web bundler that's designed to be a drop-in replacement for webpack. It provides:

- **Faster builds**: Significantly faster than webpack due to Rust implementation
- **Webpack compatibility**: Supports most webpack plugins and loaders
- **Better performance**: Optimized for large-scale applications
- **Module Federation support**: Built-in support for Module Federation

## Basic Setup

### Rspack Configuration

```javascript
// rspack.config.js
const { ModuleFederationPlugin } = require('@rspack/core');

module.exports = {
  mode: 'development',
  entry: './src/index.js',
  plugins: [
    new ModuleFederationPlugin({
      name: 'host', // Name of this application
      remotes: {
        // Remote applications we want to consume
        remoteApp: 'remoteApp@http://localhost:3001/remoteEntry.js',
      },
      shared: {
        // Shared dependencies
        react: {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
        'react-dom': {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
      },
    }),
  ],
};
```

### Remote Application Configuration

```javascript
// rspack.config.js (Remote App)
const { ModuleFederationPlugin } = require('@rspack/core');

module.exports = {
  mode: 'development',
  entry: './src/index.js',
  plugins: [
    new ModuleFederationPlugin({
      name: 'remoteApp',
      filename: 'remoteEntry.js', // Entry point for remote
      exposes: {
        // What this app exposes to others
        './Button': './src/components/Button',
        './Card': './src/components/Card',
      },
      shared: {
        react: {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
        'react-dom': {
          singleton: true,
          requiredVersion: '^18.0.0',
        },
      },
    }),
  ],
};
```

## Using Remote Modules

### Dynamic Import

```javascript
// In the host application
import React from 'react';

const RemoteButton = React.lazy(() => import('remoteApp/Button'));

function App() {
  return (
    <div>
      <React.Suspense fallback={<div>Loading...</div>}>
        <RemoteButton />
      </React.Suspense>
    </div>
  );
}
```

### Direct Import (with proper setup)

```javascript
// If configured correctly, you can import directly
import Button from 'remoteApp/Button';

function App() {
  return (
    <div>
      <Button label="Click me" />
    </div>
  );
}
```

## Shared Dependencies

One of the key features is sharing dependencies to avoid loading them multiple times:

```javascript
shared: {
  react: {
    singleton: true, // Only one instance
    requiredVersion: '^18.0.0', // Version requirement
    eager: false, // Load immediately or lazily
  },
  'react-dom': {
    singleton: true,
    requiredVersion: '^18.0.0',
  },
}
```

## Advanced Configuration

### Environment-Specific Remotes

```javascript
const remotes = process.env.NODE_ENV === 'production'
  ? {
      remoteApp: 'remoteApp@https://cdn.example.com/remoteEntry.js',
    }
  : {
      remoteApp: 'remoteApp@http://localhost:3001/remoteEntry.js',
    };

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes,
      // ...
    }),
  ],
};
```

### TypeScript Support

For TypeScript, you'll need to declare remote modules:

```typescript
// types/remotes.d.ts
declare module 'remoteApp/Button' {
  export interface ButtonProps {
    label: string;
    onClick?: () => void;
  }
  const Button: React.FC<ButtonProps>;
  export default Button;
}
```

## Benefits of Rspack + Module Federation

1. **Fast builds**: Rspack's Rust implementation provides significantly faster build times
2. **Better DX**: Faster feedback loop during development
3. **Scalability**: Can handle large codebases efficiently
4. **Compatibility**: Works with existing webpack ecosystem
5. **Performance**: Optimized bundle splitting and code loading

## Common Patterns

### Microfrontend Architecture

```
┌─────────────┐
│   Host App  │
│  (Shell)    │
└──────┬──────┘
       │
       ├───► Remote App 1 (Products)
       ├───► Remote App 2 (Cart)
       └───► Remote App 3 (Checkout)
```

### Shared Component Library

```javascript
// Shared library exposes components
exposes: {
  './Button': './src/components/Button',
  './Input': './src/components/Input',
  './Card': './src/components/Card',
}
```

### Runtime Configuration

```javascript
// Load remotes dynamically at runtime
const loadRemote = (remoteName, moduleName) => {
  return import(`${remoteName}/${moduleName}`);
};
```

## Challenges & Solutions

### Version Conflicts
- Use `singleton: true` for critical dependencies
- Specify `requiredVersion` to enforce compatibility
- Use `eager: true` for dependencies that must load immediately

### Network Issues
- Implement retry logic for remote loading
- Provide fallback UI when remotes fail to load
- Cache remote entries when possible

### Development Workflow
- Run multiple dev servers (one per app)
- Use tools like `concurrently` to manage multiple processes
- Consider using a monorepo tool like Nx or Turborepo

## Best Practices

1. **Version your remotes**: Use semantic versioning for remote modules
2. **Document exposed modules**: Clearly document what each remote exposes
3. **Test integration**: Test how remotes work together
4. **Error boundaries**: Wrap remote components in error boundaries
5. **Loading states**: Always provide loading feedback
6. **Shared dependencies**: Carefully manage shared dependencies to avoid conflicts

Links:

- [Module Federation Documentation](https://module-federation.github.io/)
- [Rspack Documentation](https://rspack.dev/)
- [Webpack Module Federation Guide](https://webpack.js.org/concepts/module-federation/)
- [Micro Frontends (Martin Fowler)](https://martinfowler.com/articles/micro-frontends.html)

