# react native modal navigation patterns

Working on the pupunha-conf app, I learned some key patterns for implementing modal navigation in React Native with Expo Router.

## Modal Layout Structure

Instead of handling modals as separate screens, create a dedicated modal layout:

```typescript
// app/(tabs)/_layout.tsx
<Stack.Screen 
  name="modal" 
  options={{ presentation: 'modal' }} 
/>
```

This allows you to present multiple types of modals (create post, image viewer, speaker details) under a single modal route.

## Form Sheet Presentation

For iOS, use `formSheet` presentation style for better UX:

```typescript
// Modal screen options
{
  presentation: 'formSheet', // Better than 'modal' on iOS
  headerShown: false
}
```

Form sheets provide a more native iOS feel and handle edge swipe gestures better.

## Direct Modal Navigation

Instead of nested routes like `/dashboard/modal/speaker`, simplify to direct modal paths:

```typescript
// Before: complex nested routing
router.push('/dashboard/modal/speaker/123')

// After: direct modal routing  
router.push('/modal/speaker/123')
```

This reduces navigation complexity and makes modal handling more predictable.

## Modal State Management

When navigating to modals, consider the parent screen state:

```typescript
// Use safeArea="none" when the modal should handle its own safe areas
<Screen safeArea="none">
  {/* Modal content */}
</Screen>
```

This prevents double safe area padding when the modal has its own header/navigation.