# shopify flashlist for react native performance

While building the pupunha-conf app, I replaced FlatList with Shopify's FlashList for better performance with large datasets.

## Why FlashList?

FlashList provides better performance than FlatList for large lists by:
- More efficient recycling of list items
- Better memory management
- Smoother scrolling with less jank

## Basic Implementation

```typescript
import { FlashList } from "@shopify/flash-list";

<FlashList
  data={sessions}
  renderItem={({ item }) => <SessionCard session={item} />}
  keyExtractor={(item) => item.id}
  estimatedItemSize={100} // Important for performance
/>
```

## Key Differences from FlatList

The `estimatedItemSize` prop is crucial - it helps FlashList pre-calculate layout:

```typescript
// FlashList - requires estimated size
<FlashList
  estimatedItemSize={120} // Height estimate for each item
  data={items}
  renderItem={renderItem}
/>

// vs FlatList - no size estimation needed
<FlatList
  data={items}
  renderItem={renderItem}
/>
```

## Loading States

FlashList works great with loading indicators in empty states:

```typescript
<FlashList
  data={isLoading ? [] : sessions}
  ListEmptyComponent={() => 
    isLoading ? <ActivityIndicator /> : <EmptyState />
  }
  renderItem={renderSessionCard}
  estimatedItemSize={150}
/>
```

## Performance Tips

- Always provide accurate `estimatedItemSize`
- Use `keyExtractor` for stable keys
- Avoid inline functions in `renderItem` - extract to separate components
- Consider `getItemType` for lists with different item heights