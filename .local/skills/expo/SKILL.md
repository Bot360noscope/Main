---
name: expo
description: Guidelines for building mobile applications using Expo, covering UI design, animations, React context patterns, and native device feature permission (Camera, Location, FileUploads)
---

Always follow these guidelines when building a mobile application using Expo:

## Architecture

- iOS 26 exists. Use NativeTabs from expo-router for native tab bars with liquid glass support. Use `isLiquidGlassAvailable()` from expo-glass-effect to check availability and fall back to classic Tabs with BlurView for older iOS/Android.
- Follow modern React Native patterns and best-practices.
- Put as much of the app in the frontend as possible. The backend should only be responsible for data persistence and making API calls.
- Use React context for state that is shared across multiple components.
- Use React Query (@tanstack/react-query) for server state fetching.
- Use useState for very local state.
- Minimize the number of files. Collapse similar components into a single file.
- If the app is complex and requires functionality that can't be done in a single request, it is okay to stub out the backend and implement the frontend first.
- ALWAYS use native device capabilities (camera, location, contacts, etc.) when the app requires them. NEVER use fake/mock data when real device features are available and appropriate.
- Client-Server Communication: The client (Expo app) interacts with the server (Express app) through a RESTful API. The server is responsible for data storage.
- Server-side Logic: The server handles API requests, database interactions, authentication, and any other server-specific logic. It's built with Express and uses TypeScript.

## Routing

This stack uses Expo Router for file-based routing (similar to Next.js Pages Router) for the frontend. The backend uses Express with TypeScript.

- Every file in the `app/` directory becomes a route
- Use `_layout.tsx` files to define shared layouts
- Use `(group)` directories for layout groups without affecting URLs
- Use `[param].tsx` for dynamic routes

Example structure:

```text
app/
  _layout.tsx          # Root layout with providers
  index.tsx            # Home route (/)
  (tabs)/
    _layout.tsx        # Tab layout
    index.tsx          # First tab (/)
    settings.tsx       # Settings tab (/settings)
  profile/
    _layout.tsx        # Profile stack layout
    index.tsx          # /profile
    [id].tsx           # /profile/:id
```

For dynamic parameters: `const { id } = useLocalSearchParams()` from "expo-router"

## State Management

- Use React Query for server state (always use object API)
- Use useState for very local state
- Avoid props drilling - use React context for shared state
- Don't wrap <RootLayoutNav/> in a context hook - wrap at the root layout level
- React Query provider should be the top level provider
- Use AsyncStorage inside context providers for persistent state

## Workflow

- The Expo App runs on port 8081. All web_application_feedback should go through port 8081 as that is where the user's app runs on
- The Express backend runs on port 5000. It serves APIs for the app and a static landing page in server/templates/landing-page.html. Do NOT use port 5000 for web_application_feedback as it only serves the API and a landing page.
- There are two workflows for this stack:
  - `Start Backend`: Restarts (or starts) the Express server. Use `await restartWorkflow({ workflowName: "Start Backend" })` after making any server/backend changes. It is important that you do not restart this workflow if you have only made frontend changes. Restarting this workflow takes time and calling it unnecessarily results in a poor user experience.
  - `Start Frontend`: Restarts (or starts) the Expo dev server. Since the Expo dev server has Hot Module Reloading, it will automatically refresh the app after most code changes. It is important that you do not restart this workflow unless you have updated dependencies or fixed an error. Restarting this workflow takes time and calling it unnecessarily results in a poor user experience.

## React Native Pitfalls

- NEVER use the 'uuid' package - crashes on iOS/Android. Use expo-crypto: `Crypto.randomUUID()`
- ALWAYS check existing template files for correct import paths. useBottomTabBarHeight is from '@react-navigation/bottom-tabs', NOT 'expo-router'
- NOT everything needs to be scrollable. Fixed layouts (timers, dashboards, single-screen UIs) should use View, not ScrollView. FlatList: add scrollEnabled={data.length > 0} to prevent empty bounce. NEVER use contentInsetAdjustmentBehavior="automatic" with transparent/large-title headers
- NEVER sync props to state with useEffect (causes infinite loops). Derive values directly from props. Only call onChange on explicit user actions (onPress, onChangeText), not in useEffect
- Mobile typography: max 48-64pt for time displays, body 14-16pt, headers 20-28pt. Test on 375pt wide screen (iPhone SE)
- Use simple text-based empty states, NOT placeholder images. An icon + descriptive text is sufficient
- NEVER set headerShown dynamically inside screen components - causes remounts. Configure ALL header options in _layout.tsx. Individual screens only set dynamic content (headerLeft/headerRight)
- NEVER use hardcoded top padding - use useSafeAreaInsets() from react-native-safe-area-context. For absolutely positioned headers: header uses insets.top, content below uses paddingTop: insets.top + headerContentHeight
- For streaming, read the mobile-ui skill's expo-fetch.md reference. Use `import { fetch } from 'expo/fetch'` which supports getReader() on all platforms
- NEVER use InputAccessoryView - doesn't work in Expo Go. Use KeyboardAvoidingView wrapping the ENTIRE screen: behavior="padding" (iOS) / "height" (Android), keyboardVerticalOffset ~90, keyboardDismissMode="interactive", keyboardShouldPersistTaps="handled". For chat UIs: use inverted FlatList - NEVER scrollToEnd()
- FlatList boolean props (scrollEnabled, etc.) require actual booleans. ALWAYS coerce with !! when using string variables: `scrollEnabled={!!data}`
- RevenueCat works in Expo Go (Preview API Mode) and on web out of the box. No native build required
- ErrorBoundary: use `reloadAppAsync` from expo to restart on crash. NEVER use `reloadAsync` from expo-updates. No local state in ErrorFallback component
- react-native-maps: ALWAYS pin to exactly version 1.18.0. Do NOT add to plugins array in app.json
- Prefer icons over text for buttons. Apps should look like market-leading apps, no unnecessary titles like "Chatbot"
- Use PanResponder from 'react-native' for gesture handling.

## Safe Area View

When to use SafeAreaView:

1. Built-in tabs or header: Don't add SafeAreaView - they handle insets automatically
2. Custom header: Add SafeAreaView to the header component
3. Removed header: Add SafeAreaView inside a View with background color (not just white space)
4. Pages inside stacks: Don't add SafeAreaView if parent _layout.tsx has header enabled

Games and absolute positioning:

- Account for safe area insets in positioning calculations
- Use useSafeAreaInsets() hook to get inset values
- Apply insets to positioning calculations in game physics
- Avoid using SafeAreaView in game screens - factor insets into game loop instead

## Library Compatibility

- ONLY use libraries from the Expo Go compatible list
- Do not install additional native libraries unless they're JS only
- Pre-installed: expo-router, @expo/vector-icons, @tanstack/react-query, react-native-reanimated
- Error boundary: Custom ErrorBoundary component (components/ErrorBoundary.tsx)

## Insets

Web-only insets for web edge cases (always implement for every app):

- Apply on web only (Platform.OS === "web")
- Always add at least a 67px top inset to top-level header content for the status bar. Additional vertical padding will likely still be required.
- Always add a 34px bottom insets
  - If using a bottom tab bar, instead modify the tab bar height to be 84px (50px + 34px). Do not add paddingBottom to the tab bar.
- Do not add these insets on iOS/Android; native safe areas already handle them
- Before finishing any screen, verify that web platform insets handling is implemented and are correct.

## Web Compatibility

Polyfilled (work on web): expo-secure-store, expo-haptics, react-native-maps, Alert, RefreshControl.
Partial web support: expo-camera (no switch/record), expo-clipboard, expo-file-system, expo-image, expo-notifications, expo-video, react-native-reanimated (no layout animations/native driver on web).
No web support (use Platform.OS checks): expo-battery, expo-brightness, expo-contacts, expo-device, expo-local-authentication, expo-location (use web geolocation API), expo-media-library, expo-sensors, expo-sharing.

## Payments

If prompted to add payments (i.e. subscriptions, in-app purchases, etc.), always use RevenueCat.
Do not use Stripe unless the user explicitly requests it.

## Testing

- Add testID to interactive elements for testing
- Use mobile screen sizes for automated testing
- Recommended dimensions: height: 720, width: 400

## Replit Environment

- User can scan QR code from Replit's URL bar menu to test on their physical device via Expo Go
- The dev server requires specific env vars for Replit's proxy: EXPO_PACKAGER_PROXY_URL and REACT_NATIVE_PACKAGER_HOSTNAME
- Hot module reloading (HMR) is enabled - no need to restart the dev server for code changes

## Forbidden Changes

- NEVER edit package.json directly. Use the packager_install_tool to install packages.
- NEVER change bundle identifiers after initial setup unless user explicitly requests it.
- NEVER downgrade the version of React Native or Expo that is declared in package.json.
- NEVER create app.config.ts or app.config.js. The project MUST use a static app.json for Expo configuration. Dynamic config files (app.config.ts/js) break the Expo Launch build process. If you need to modify Expo settings, edit app.json directly.

## References

Before writing code, identify whether any reference below applies to the task. If it does, read it first.

- `references/react_context.md` - Use this reference when creating or modifying shared state with React context, provider composition, or context-based hooks.
- `references/design_and_aesthetics.md` - Use this reference when designing or restyling UI, selecting iconography, or implementing animations and visual polish.
- `references/device_features_and_permissions.md` - Use this reference when implementing camera, location, notifications, contacts, file uploads, or any permission request/denial flow.
