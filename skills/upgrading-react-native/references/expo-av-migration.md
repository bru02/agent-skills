---
title: Expo AV to Expo Video Migration
impact: HIGH
tags: expo, expo-av, expo-video, migration, sdk-55
---

# Skill: Expo AV to Expo Video Migration

`expo-av` video playback is removed in SDK 55. Migrate to `expo-video` before upgrading. For audio, migrate to `expo-audio` (separate concern, not covered here).

## When to Apply

- Project imports from `expo-av` for video playback (`Video`, `AVPlaybackStatus`)
- Upgrading to Expo SDK 55+ (where `expo-av` is removed)

## Quick Commands

```bash
# Install replacement
npx expo install expo-video

# After all code is migrated, remove expo-av
npx expo uninstall expo-av
```

## Architecture Change

`expo-av` combined player and view in one `<Video>` component controlled via ref.
`expo-video` separates them: `useVideoPlayer()` hook creates a player instance, `<VideoView>` renders it.

This separation enables pre-buffering, sharing a player across views, and Picture-in-Picture.

## Migration Pattern

### Before (expo-av)

```tsx
import { Video, AVPlaybackStatus } from 'expo-av';

function Player() {
  const videoRef = useRef<Video>(null);
  const [status, setStatus] = useState<AVPlaybackStatus>({});

  return (
    <Video
      ref={videoRef}
      source={{ uri: 'https://example.com/video.mp4' }}
      shouldPlay
      isLooping
      volume={1.0}
      onPlaybackStatusUpdate={status => setStatus(status)}
      useNativeControls
      style={{ width: 300, height: 200 }}
    />
  );
}

// Imperative control via ref
videoRef.current.playAsync();
videoRef.current.pauseAsync();
videoRef.current.setPositionAsync(5000); // milliseconds
videoRef.current.setStatusAsync({ shouldPlay: true, volume: 0.5 });
```

### After (expo-video)

```tsx
import { useVideoPlayer, VideoView } from 'expo-video';
import { useEvent } from 'expo';

function Player() {
  const player = useVideoPlayer('https://example.com/video.mp4', player => {
    player.loop = true;
    player.play();
  });

  const { status } = useEvent(player, 'statusChange', {
    status: player.status,
  });

  return (
    <VideoView
      player={player}
      nativeControls
      style={{ width: 300, height: 200 }}
    />
  );
}

// Direct control via player instance
player.play();
player.pause();
player.currentTime = 5; // seconds (not milliseconds)
player.volume = 0.5;
```

## Property Mapping

| expo-av (status/prop) | expo-video (player property/method) |
|---|---|
| `source={{ uri }}` | `useVideoPlayer(uri)` or `player.replaceAsync(uri)` |
| `shouldPlay` | `player.play()` in setup function |
| `isLooping` | `player.loop = true` |
| `volume` | `player.volume` |
| `isMuted` | `player.muted` |
| `rate` | `player.playbackRate` |
| `positionMillis` | `player.currentTime` (in **seconds**) |
| `durationMillis` | `player.duration` (in **seconds**) |
| `useNativeControls` | `<VideoView nativeControls />` |
| `resizeMode` | `<VideoView contentFit="cover" />` |
| `ref.playAsync()` | `player.play()` |
| `ref.pauseAsync()` | `player.pause()` |
| `ref.setPositionAsync(ms)` | `player.currentTime = seconds` |
| `ref.loadAsync(source)` | `player.replaceAsync(source)` |
| `onPlaybackStatusUpdate` | `useEvent(player, 'statusChange')` |

## Critical Differences

1. **Time units**: `expo-av` uses milliseconds, `expo-video` uses **seconds**. Divide by 1000 when migrating.
2. **No ref pattern**: Replace all `videoRef.current.methodAsync()` calls with direct `player.method()` calls.
3. **Status tracking**: Replace `onPlaybackStatusUpdate` callback with `useEvent` hook for reactive state or `useEventListener` for side effects.
4. **Source format**: `useVideoPlayer` accepts a string URL, `require()` asset, or `VideoSource` object directly (no `{ uri }` wrapper needed for plain URLs).
5. **Setup function**: The second argument to `useVideoPlayer` runs once on creation — set initial properties there, not in effects.
6. **Lifecycle**: `useVideoPlayer` auto-cleans up on unmount. For manual control, use `createVideoPlayer()` and call `player.release()`.

## Event Handling

```tsx
import { useEvent, useEventListener } from 'expo';

// Reactive state (re-renders on change)
const { isPlaying } = useEvent(player, 'playingChange', {
  isPlaying: player.playing,
});

// Side effects (no re-render)
useEventListener(player, 'statusChange', ({ status, error }) => {
  if (status === 'error') console.error(error);
});
```

## VideoView Props

| Prop | Type | Description |
|------|------|-------------|
| `player` | `VideoPlayer` | Required. Player instance from `useVideoPlayer` |
| `nativeControls` | `boolean` | Show platform native playback controls |
| `contentFit` | `string` | Video scaling: `"contain"`, `"cover"`, `"fill"` |
| `allowsPictureInPicture` | `boolean` | Enable PiP mode |
| `startsPictureInPictureAutomatically` | `boolean` | Auto-enter PiP on background |

## New Capabilities (not in expo-av)

- **Pre-buffering**: Create player before attaching to view for instant playback
- **Picture-in-Picture**: Native PiP support on iOS and Android
- **DRM**: Digital rights management support
- **Persistent caching**: LRU video cache that works offline
- **Subtitles**: Advanced subtitle/caption management

## Related Skills

- [expo-sdk-upgrade.md](expo-sdk-upgrade.md) - Expo SDK upgrade workflow
- [upgrading-dependencies.md](upgrading-dependencies.md) - Dependency migration planning
- [upgrade-verification.md](upgrade-verification.md) - Post-upgrade validation
