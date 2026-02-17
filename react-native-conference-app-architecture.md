# react native conference app architecture

Building the pupunha-conf app taught me patterns for structuring event/conference mobile apps with React Native and Expo.

## Core Features Architecture

A conference app typically needs these core modules:

```
/app
  /(tabs)
    /feed       - Social posts and announcements  
    /schedule   - Sessions and speaker lineup
    /bookmarks  - Saved sessions for attendees
  /modal        - Overlays for details and creation
  /speakers     - Speaker profiles and bios
```

## Data Structure Patterns

Organize event data with clear relationships:

```typescript
interface Conference {
  id: string;
  name: string;
  year: number;
  sessions: Session[];
  speakers: Speaker[];
}

interface Session {
  id: string;
  title: string;
  speakerId: string;
  timeSlot: string;
  isBookmarked: boolean;
}
```

## Session Bookmark Management

Use `useFocusEffect` to refresh bookmarked data when users navigate back:

```typescript
import { useFocusEffect } from '@react-navigation/native';

const BookmarkedScreen = () => {
  const { refetch } = useQuery(['bookmarked-sessions']);
  
  useFocusEffect(
    useCallback(() => {
      refetch(); // Refresh when screen comes into focus
    }, [refetch])
  );
}
```

## Event Data Organization

Structure multi-year events with clear separation:

```typescript
// events/pupunha-code-2025.ts
export const pupunhaCode2025 = {
  id: 'pupunha-2025',
  sessions: [...],
  speakers: [...]
};

// events/pupunha-code-2026.ts  
export const pupunhaCode2026 = {
  id: 'pupunha-2026', 
  sessions: [...],
  speakers: [...]
};
```

## Feed and Social Features

Implement post creation with image handling:

```typescript
const CreatePostModal = () => {
  const [images, setImages] = useState([]);
  
  const pickImages = async () => {
    const result = await ImagePicker.launchImageLibrary({
      mediaType: ['photo'], // Use array for media types
      multiple: true
    });
  };
}
```

This architecture scales well for multi-day conferences with complex scheduling needs.