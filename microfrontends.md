# microfrontends

Microfrontends are an architectural approach where a frontend application is composed of smaller, independently deployable frontend applications. Each microfrontend is owned by a different team and can be developed, tested, and deployed independently.

## What are Microfrontends?

Similar to microservices for the backend, microfrontends break down a monolithic frontend into smaller, independent pieces. Each piece:

- **Owns a feature or domain**: Like "product catalog", "shopping cart", "user profile"
- **Can be developed independently**: Different teams can work on different parts
- **Can be deployed independently**: Changes to one part don't require redeploying everything
- **Uses its own tech stack** (optional): Teams can choose their own frameworks

## Why Microfrontends?

### Benefits

1. **Team Autonomy**: Different teams can work independently without blocking each other
2. **Technology Diversity**: Teams can use different frameworks (React, Vue, Angular, etc.)
3. **Independent Deployments**: Deploy changes to one feature without affecting others
4. **Scalability**: Easier to scale teams and codebases
5. **Faster Development**: Smaller codebases are easier to understand and modify

### Challenges

1. **Complexity**: More moving parts to manage
2. **Bundle Size**: Risk of loading duplicate dependencies
3. **Consistency**: Harder to maintain UI/UX consistency across teams
4. **Integration**: Need to coordinate between different applications
5. **Testing**: More complex integration testing

## Implementation Approaches

### 1. Build-Time Integration

Compose microfrontends at build time:

```
┌─────────────────┐
│  Main App       │
│  (Build Time)   │
├─────────────────┤
│  - Products App │
│  - Cart App     │
│  - Checkout App │
└─────────────────┘
```

**Pros**: Simple, single bundle
**Cons**: Requires rebuild for any change, no independent deployment

### 2. Runtime Integration (Module Federation)

Load microfrontends at runtime using Module Federation:

```javascript
// Host application
const RemoteProduct = React.lazy(() => import('products/ProductList'));
const RemoteCart = React.lazy(() => import('cart/CartView'));

function App() {
  return (
    <div>
      <React.Suspense fallback={<div>Loading...</div>}>
        <RemoteProduct />
        <RemoteCart />
      </React.Suspense>
    </div>
  );
}
```

**Pros**: True independent deployment, runtime composition
**Cons**: More complex setup, network dependencies

### 3. iframe Integration

Embed microfrontends using iframes:

```html
<iframe src="https://products.example.com" />
<iframe src="https://cart.example.com" />
```

**Pros**: Complete isolation, easy to implement
**Cons**: Communication challenges, styling issues, performance overhead

### 4. Web Components

Use Web Components as the integration layer:

```javascript
// Microfrontend exposes as web component
class ProductList extends HTMLElement {
  connectedCallback() {
    this.innerHTML = '<div>Product List</div>';
  }
}

customElements.define('product-list', ProductList);
```

```html
<!-- Host uses web component -->
<product-list></product-list>
```

**Pros**: Framework agnostic, native browser support
**Cons**: Limited browser support for some features, less developer experience

## Architecture Patterns

### Shell Pattern

A shell application that orchestrates microfrontends:

```
┌─────────────────────────┐
│      Shell App          │
│  (Navigation, Layout)   │
├─────────────────────────┤
│  ┌──────────┐ ┌───────┐│
│  │ Products │ │ Cart  ││
│  │   App    │ │  App  ││
│  └──────────┘ └───────┘│
└─────────────────────────┘
```

### Self-Contained Apps

Each microfrontend is a complete application:

```
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│ Products    │  │ Cart        │  │ Checkout    │
│ App         │  │ App         │  │ App         │
│ (Full App)  │  │ (Full App)  │  │ (Full App)  │
└─────────────┘  └─────────────┘  └─────────────┘
```

## Communication Patterns

### 1. Props/Events (Parent-Child)

```javascript
// Host passes data down
<RemoteProduct userId={userId} />

// Microfrontend emits events
window.dispatchEvent(new CustomEvent('product-selected', { 
  detail: { productId: 123 } 
}));
```

### 2. Shared State (Global Store)

```javascript
// Shared state management
const store = {
  user: null,
  cart: [],
  setUser: (user) => { /* ... */ },
  addToCart: (item) => { /* ... */ },
};

// All microfrontends access the same store
window.sharedStore = store;
```

### 3. Custom Events

```javascript
// Microfrontend 1 publishes
window.dispatchEvent(new CustomEvent('cart-updated', {
  detail: { itemCount: 5 }
}));

// Microfrontend 2 subscribes
window.addEventListener('cart-updated', (event) => {
  console.log('Cart updated:', event.detail);
});
```

### 4. URL/Route Based

```javascript
// Share state through URL parameters
// /products?category=electronics&sort=price

// Microfrontends read from URL
const params = new URLSearchParams(window.location.search);
const category = params.get('category');
```

## Technology Stack Options

### Module Federation (Webpack/Rspack)

- **Best for**: React, Vue, Angular applications
- **Pros**: Framework support, code splitting, shared dependencies
- **Cons**: Build tool coupling

### Single-SPA

- **Best for**: Multi-framework environments
- **Pros**: Framework agnostic, routing integration
- **Cons**: Additional abstraction layer

### Qiankun (Alibaba)

- **Best for**: Large-scale applications
- **Pros**: Production-tested, feature-rich
- **Cons**: Primarily Chinese documentation

### Web Components

- **Best for**: Framework-agnostic solutions
- **Pros**: Native browser support, no build tools needed
- **Cons**: Less developer experience, limited features

## Best Practices

### 1. Design System

Create a shared design system to maintain consistency:

```javascript
// Shared design tokens
export const colors = {
  primary: '#007bff',
  secondary: '#6c757d',
  // ...
};

// Shared components
export { Button, Input, Card } from './components';
```

### 2. API Contracts

Define clear contracts between microfrontends:

```typescript
// Shared types
export interface Product {
  id: string;
  name: string;
  price: number;
}

export interface CartItem {
  product: Product;
  quantity: number;
}
```

### 3. Versioning

Version your microfrontends and APIs:

```
/products/v1/ProductList
/products/v2/ProductList
```

### 4. Error Boundaries

Wrap each microfrontend in an error boundary:

```javascript
class MicrofrontendErrorBoundary extends React.Component {
  componentDidCatch(error, errorInfo) {
    // Log error, show fallback UI
  }
  
  render() {
    if (this.state.hasError) {
      return <FallbackUI />;
    }
    return this.props.children;
  }
}
```

### 5. Testing Strategy

- **Unit tests**: Within each microfrontend
- **Integration tests**: Test how microfrontends work together
- **E2E tests**: Test the complete user journey

### 6. Performance

- **Lazy loading**: Load microfrontends on demand
- **Code splitting**: Split code within each microfrontend
- **Shared dependencies**: Avoid loading duplicate libraries
- **Caching**: Cache remote entries and assets

## When to Use Microfrontends

### Good Fit

- Large organizations with multiple frontend teams
- Need for independent deployments
- Different teams want different tech stacks
- Large, complex applications that are hard to maintain as monoliths

### Not a Good Fit

- Small teams or applications
- Tight coupling between features
- Performance is critical (overhead of runtime integration)
- Limited resources for managing complexity

Links:

- [Micro Frontends (Martin Fowler)](https://martinfowler.com/articles/micro-frontends.html)
- [Module Federation Documentation](https://module-federation.github.io/)
- [Single-SPA Documentation](https://single-spa.js.org/)
- [Web Components Guide](https://developer.mozilla.org/en-US/docs/Web/Web_Components)

