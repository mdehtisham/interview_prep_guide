# Mobile & Cross-Platform

> Expert / Architect Level (5+ years)

---

## Questions

211. What is React Native and how does it differ from React?
212. What is Expo and when should you use it?
213. How do you share code between React and React Native?
214. What is the difference between React Native and Flutter?
215. How do you handle platform-specific code?
216. What is React Native Web?
217. How do you optimize React Native performance?
218. What is the New Architecture in React Native?
219. What is Hermes JavaScript engine?
220. How do you implement native modules in React Native?

---

## Detailed Answers

### 211. What is React Native and how does it differ from React?

<details>
<summary>View Answer</summary>

**React Native** is a framework for building native mobile applications using React and JavaScript. Instead of rendering to the browser DOM, React Native renders to native platform components (iOS/Android). It lets you build mobile apps using the same React paradigms while achieving native performance and UX.

#### What is React Native?

**Definition:**

```javascript
// React (Web) - renders to DOM
function App() {
  return <div>Hello Web</div>;
}
// Renders: <div>Hello Web</div> in browser

// React Native (Mobile) - renders to native components
import { View, Text } from 'react-native';

function App() {
  return <View><Text>Hello Mobile</Text></View>;
}
// Renders: UIView + UILabel (iOS) or android.view.View + TextView (Android)
```

**Key Concept:**
- Same React principles: Components, JSX, hooks, state, props
- Different rendering target: Native platform APIs instead of DOM
- "Learn once, write anywhere" (not "write once, run anywhere")

---

#### Core Differences

**1. Rendering Target**

| Aspect | React (Web) | React Native (Mobile) |
|--------|-------------|----------------------|
| **Renders to** | HTML DOM | Native components |
| **Elements** | `<div>`, `<span>`, `<button>` | `<View>`, `<Text>`, `<TouchableOpacity>` |
| **Styling** | CSS | JavaScript StyleSheet |
| **Platform** | Browser | iOS/Android |
| **Runtime** | Browser JS engine | JavaScriptCore/Hermes |

```jsx
// React Web
import React from 'react';

function Button() {
  return (
    <button className="btn" style={{ padding: 10 }}>
      Click Me
    </button>
  );
}

// React Native
import React from 'react';
import { TouchableOpacity, Text, StyleSheet } from 'react-native';

function Button() {
  return (
    <TouchableOpacity style={styles.btn}>
      <Text>Click Me</Text>
    </TouchableOpacity>
  );
}

const styles = StyleSheet.create({
  btn: {
    padding: 10,
  },
});
```

---

**2. Components**

**React (Web) Components:**

```jsx
import React from 'react';

function WebApp() {
  return (
    <div className="container">
      <h1>Title</h1>
      <p>Paragraph</p>
      <button onClick={() => alert('Clicked')}>Button</button>
      <input type="text" placeholder="Type here" />
      <a href="/page">Link</a>
    </div>
  );
}
```

**React Native Components:**

```jsx
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  TextInput,
  ScrollView,
  Alert,
} from 'react-native';

function MobileApp() {
  return (
    <ScrollView> {/* No div */}
      <Text style={{ fontSize: 24, fontWeight: 'bold' }}>Title</Text> {/* No h1 */}
      <Text>Paragraph</Text> {/* No p */}
      <TouchableOpacity onPress={() => Alert.alert('Clicked')}> {/* No button */}
        <Text>Button</Text>
      </TouchableOpacity>
      <TextInput placeholder="Type here" /> {/* No input */}
      {/* No <a>, use navigation library */}
    </ScrollView>
  );
}
```

**Component Mapping:**

| Web | React Native | Purpose |
|-----|--------------|----------|
| `<div>` | `<View>` | Container |
| `<span>`, `<p>`, `<h1>` | `<Text>` | Text |
| `<button>` | `<TouchableOpacity>` | Touchable |
| `<input>` | `<TextInput>` | Input |
| `<img>` | `<Image>` | Image |
| `<ul>` | `<ScrollView>` or `<FlatList>` | List |
| N/A | `<SafeAreaView>` | Safe area |

---

**3. Styling**

**React (CSS):**

```jsx
// External CSS
import './App.css';

function App() {
  return (
    <div className="container">
      <h1 style={{ color: 'blue' }}>Title</h1>
    </div>
  );
}

/* App.css */
.container {
  display: flex;
  flex-direction: column;
  padding: 20px;
}
```

**React Native (StyleSheet):**

```jsx
import { View, Text, StyleSheet } from 'react-native';

function App() {
  return (
    <View style={styles.container}>
      <Text style={{ color: 'blue' }}>Title</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    flexDirection: 'column',
    padding: 20,
  },
});
```

**Styling Differences:**

| Feature | React (CSS) | React Native |
|---------|-------------|---------------|
| **Format** | CSS strings | JavaScript objects |
| **Properties** | `background-color` | `backgroundColor` (camelCase) |
| **Units** | `px`, `em`, `rem`, `%` | Numbers (logical pixels) |
| **Selectors** | Classes, IDs, pseudo | Inline/StyleSheet only |
| **Cascading** | Yes | No (explicit only) |
| **Flexbox** | One layout option | Default layout |

```jsx
// React: CSS properties with units
style={{ width: '100px', margin: '10px 20px' }}

// React Native: Numeric values, no units
style={{ width: 100, marginVertical: 10, marginHorizontal: 20 }}
```

---

**4. Layout**

**React (Multiple Options):**

```css
/* Block layout */
.container {
  display: block;
}

/* Flexbox */
.flex-container {
  display: flex;
}

/* Grid */
.grid-container {
  display: grid;
}

/* Positioning */
.positioned {
  position: absolute;
  top: 10px;
  left: 20px;
}
```

**React Native (Flexbox by Default):**

```jsx
const styles = StyleSheet.create({
  // ALL containers use Flexbox by default
  container: {
    flex: 1,
    flexDirection: 'column', // Default (vs 'row' in web)
  },
  
  // Positioning also available
  positioned: {
    position: 'absolute',
    top: 10,
    left: 20,
  },
});

// NO Grid layout
// NO Block layout
// Flexbox is the primary (only) layout system
```

---

**5. Navigation**

**React (Browser Routing):**

```jsx
import { BrowserRouter, Routes, Route, Link } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Link to="/profile">Profile</Link>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/profile" element={<Profile />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**React Native (Stack/Tab Navigation):**

```jsx
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { TouchableOpacity, Text } from 'react-native';

const Stack = createNativeStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={Home} />
        <Stack.Screen name="Profile" component={Profile} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}

function Home({ navigation }) {
  return (
    <TouchableOpacity onPress={() => navigation.navigate('Profile')}>
      <Text>Go to Profile</Text>
    </TouchableOpacity>
  );
}
```

---

**6. Events**

**React (DOM Events):**

```jsx
function App() {
  return (
    <div>
      <button onClick={handleClick}>Click</button>
      <input onChange={handleChange} onKeyPress={handleKeyPress} />
      <div onMouseEnter={handleHover}>Hover me</div>
    </div>
  );
}
```

**React Native (Touch Events):**

```jsx
import { TouchableOpacity, TextInput, View } from 'react-native';

function App() {
  return (
    <View>
      <TouchableOpacity onPress={handlePress}>
        <Text>Press</Text>
      </TouchableOpacity>
      <TextInput onChangeText={handleChange} />
      <View onStartShouldSetResponder={() => true}>
        {/* No hover events on touch devices */}
      </View>
    </View>
  );
}
```

---

**7. APIs & Features**

**React (Browser APIs):**

```javascript
// Web APIs
fetch('/api/data');
window.localStorage.setItem('key', 'value');
window.location.href = '/page';
navigator.geolocation.getCurrentPosition();
document.getElementById('el');
```

**React Native (Mobile APIs):**

```javascript
import { AsyncStorage, Linking, Alert, Platform } from 'react-native';
import * as Location from 'expo-location';
import { Camera } from 'expo-camera';

// Storage
import AsyncStorage from '@react-native-async-storage/async-storage';
await AsyncStorage.setItem('key', 'value');

// Linking
await Linking.openURL('https://example.com');

// Geolocation
const location = await Location.getCurrentPositionAsync();

// Camera (not available in web)
const photo = await camera.takePictureAsync();

// Platform detection
if (Platform.OS === 'ios') {
  // iOS-specific code
}
```

---

#### Architecture Comparison

**React (Web):**

```
React Components
       ↓
   React DOM
       ↓
  Browser DOM
       ↓
 Browser Rendering
```

**React Native:**

```
React Components
       ↓
React Native Bridge (JSON serialization)
       ↓
Native Modules (Objective-C/Swift/Java/Kotlin)
       ↓
Native UI (UIKit/Android Views)
```

**Bridge Example:**

```jsx
// JavaScript
import { View, Text } from 'react-native';

function App() {
  return (
    <View style={{ backgroundColor: 'red' }}>
      <Text>Hello</Text>
    </View>
  );
}

// React Native converts to messages:
// {
//   type: 'createView',
//   viewId: 1,
//   viewName: 'RCTView',
//   props: { backgroundColor: 'red' }
// }
// {
//   type: 'createView',
//   viewId: 2,
//   viewName: 'RCTText',
//   props: { text: 'Hello' }
// }

// Native side creates:
// iOS: UIView with red background + UILabel
// Android: View with red background + TextView
```

---

#### Performance Differences

**React (Web):**
- Runs in browser
- Virtual DOM diffing → Browser DOM updates
- Performance bottleneck: DOM manipulation
- Can use Web Workers for heavy computation

**React Native:**
- JavaScript runs in separate thread (JavaScriptCore/Hermes)
- Communication through bridge (async, serialized)
- Native UI thread separate from JS thread
- Performance bottleneck: Bridge communication

```javascript
// React Native Threading Model

[JavaScript Thread]      [Bridge]      [Native/UI Thread]
- React logic       <─ JSON msgs ─>  - Native views
- State updates                       - Gestures
- Business logic                      - Animations

// Bridge can be bottleneck for:
// - Frequent updates (animations)
// - Large data transfers
// - Heavy lists

// Solution: New Architecture (Fabric + TurboModules)
// - JSI (JavaScript Interface): Direct synchronous calls
// - No serialization overhead
```

---

#### Complete Example Comparison

**React (Web) - Todo App:**

```jsx
import React, { useState } from 'react';
import './TodoApp.css';

function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    setTodos([...todos, { id: Date.now(), text: input, done: false }]);
    setInput('');
  };
  
  return (
    <div className="container">
      <h1>Todos</h1>
      <div className="input-group">
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
          placeholder="Add todo"
        />
        <button onClick={addTodo}>Add</button>
      </div>
      <ul className="todo-list">
        {todos.map(todo => (
          <li key={todo.id} className={todo.done ? 'done' : ''}>
            <input
              type="checkbox"
              checked={todo.done}
              onChange={() => {
                setTodos(todos.map(t => 
                  t.id === todo.id ? { ...t, done: !t.done } : t
                ));
              }}
            />
            <span>{todo.text}</span>
          </li>
        ))}
      </ul>
    </div>
  );
}

export default TodoApp;
```

**React Native - Todo App:**

```jsx
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  FlatList,
  StyleSheet,
} from 'react-native';
import { CheckBox } from 'react-native-elements';

function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    setTodos([...todos, { id: Date.now(), text: input, done: false }]);
    setInput('');
  };
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Todos</Text>
      <View style={styles.inputGroup}>
        <TextInput
          style={styles.input}
          value={input}
          onChangeText={setInput}
          onSubmitEditing={addTodo}
          placeholder="Add todo"
        />
        <TouchableOpacity style={styles.button} onPress={addTodo}>
          <Text style={styles.buttonText}>Add</Text>
        </TouchableOpacity>
      </View>
      <FlatList
        data={todos}
        keyExtractor={item => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.todoItem}>
            <CheckBox
              checked={item.done}
              onPress={() => {
                setTodos(todos.map(t => 
                  t.id === item.id ? { ...t, done: !t.done } : t
                ));
              }}
            />
            <Text style={[styles.todoText, item.done && styles.done]}>
              {item.text}
            </Text>
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  inputGroup: {
    flexDirection: 'row',
    marginBottom: 20,
  },
  input: {
    flex: 1,
    borderWidth: 1,
    borderColor: '#ddd',
    padding: 10,
    marginRight: 10,
    borderRadius: 5,
  },
  button: {
    backgroundColor: '#007AFF',
    padding: 10,
    borderRadius: 5,
    justifyContent: 'center',
  },
  buttonText: {
    color: '#fff',
    fontWeight: 'bold',
  },
  todoItem: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingVertical: 10,
    borderBottomWidth: 1,
    borderBottomColor: '#eee',
  },
  todoText: {
    flex: 1,
    fontSize: 16,
  },
  done: {
    textDecorationLine: 'line-through',
    color: '#999',
  },
});

export default TodoApp;
```

---

#### When to Use Each

**Use React (Web):**
- ✅ Building websites/web applications
- ✅ Need SEO (search engine optimization)
- ✅ Desktop/laptop primary target
- ✅ Need browser-specific features
- ✅ Want to avoid app store distribution

**Use React Native:**
- ✅ Building mobile applications
- ✅ Need native performance and UX
- ✅ Access to device features (camera, sensors, etc.)
- ✅ Offline-first applications
- ✅ App store distribution
- ✅ Native look and feel important

**Use Both (React Native Web):**
- ✅ Share code between web and mobile
- ✅ Single codebase for all platforms
- ✅ Consistent UI across platforms

---

#### Similarities

**Both use the same React concepts:**

```jsx
// ✅ Components (function & class)
function MyComponent() { /* ... */ }
class MyComponent extends Component { /* ... */ }

// ✅ JSX
return <View><Text>Hello</Text></View>;

// ✅ State & Props
const [state, setState] = useState(0);
function MyComponent({ prop }) { /* ... */ }

// ✅ Hooks
useState, useEffect, useCallback, useMemo, useRef, etc.

// ✅ Context
const ThemeContext = React.createContext();

// ✅ Lifecycle
componentDidMount, componentDidUpdate, etc.

// ✅ Patterns
HOCs, Render props, Composition
```

---

#### Interview Tips

✅ **Key Points:**
- React Native uses React principles but renders to native components, not DOM
- Different component primitives: `<View>` instead of `<div>`, `<Text>` instead of `<span>`
- Styling with JavaScript objects, not CSS (camelCase properties, numeric values)
- Flexbox is default (and primary) layout system
- Uses native navigation, not browser routing
- Bridge architecture communicates between JS and native threads
- Access to native APIs (camera, sensors, etc.)
- "Learn once, write anywhere" philosophy

✅ **When to Mention:**
- Cross-platform development discussions
- Mobile app development
- Performance comparisons
- Native vs hybrid app questions
- Code sharing strategies

✅ **Common Follow-ups:**
- "Is React Native truly native?"
- "What's the performance compared to native apps?"
- "Can you share code between React and React Native?"
- "What's the bridge and why is it important?"
- "What's the New Architecture?"

✅ **Perfect Answer Structure:**
1. Definition: Framework for building mobile apps using React
2. Key difference: Renders to native components, not DOM
3. Components: Different primitives (`View`, `Text`, etc.)
4. Styling: JavaScript objects, not CSS
5. Architecture: Bridge between JS and native
6. Similarities: Same React principles (hooks, state, props)
7. Example: Show side-by-side React vs React Native code

</details>

---

### 212. What is Expo and when should you use it?

<details>
<summary>View Answer</summary>

**Expo** is a framework and platform built on top of React Native that provides a set of tools, libraries, and services to build, deploy, and iterate on iOS, Android, and web apps. It dramatically simplifies React Native development by handling native configuration, providing pre-built components, and offering over-the-air updates.

#### What is Expo?

**Core Concept:**

```bash
# Traditional React Native
React Native CLI
  ↓
Manual native configuration (Xcode/Android Studio)
  ↓
Custom native modules
  ↓
Complex build process

# Expo
Expo CLI
  ↓
Zero native configuration (managed workflow)
  ↓
Pre-built native modules
  ↓
Simple build process (EAS Build)
```

**Installation:**

```bash
# Create new Expo app
npx create-expo-app my-app

cd my-app

# Start development server
npx expo start

# Scan QR code with Expo Go app
# Or press 'i' for iOS simulator, 'a' for Android emulator
```

---

#### Key Components

**1. Expo SDK**

Collection of libraries providing access to device features:

```javascript
import * as Camera from 'expo-camera';
import * as Location from 'expo-location';
import * as Notifications from 'expo-notifications';
import * as FileSystem from 'expo-file-system';
import * as ImagePicker from 'expo-image-picker';
import * as SecureStore from 'expo-secure-store';
import { Audio } from 'expo-av';
import * as Haptics from 'expo-haptics';
import * as Sensors from 'expo-sensors';

// Example: Camera
function CameraScreen() {
  const [hasPermission, setHasPermission] = useState(null);
  
  useEffect(() => {
    (async () => {
      const { status } = await Camera.requestCameraPermissionsAsync();
      setHasPermission(status === 'granted');
    })();
  }, []);
  
  if (!hasPermission) {
    return <Text>No camera access</Text>;
  }
  
  return <Camera style={{ flex: 1 }} />;
}

// Example: Location
async function getLocation() {
  const { status } = await Location.requestForegroundPermissionsAsync();
  
  if (status !== 'granted') {
    return;
  }
  
  const location = await Location.getCurrentPositionAsync({});
  console.log(location.coords.latitude, location.coords.longitude);
}

// Example: Push Notifications
async function registerForPushNotifications() {
  const { status } = await Notifications.requestPermissionsAsync();
  
  if (status !== 'granted') {
    return;
  }
  
  const token = await Notifications.getExpoPushTokenAsync();
  console.log('Push token:', token);
}
```

---

**2. Expo Go App**

Mobile app for testing during development:

```bash
# Start dev server
npx expo start

# Output:
# Metro waiting on exp://192.168.1.100:8081
# 
# › Scan the QR code above with Expo Go (Android) or the Camera app (iOS)

# Features:
# - Instant reload on code changes
# - No need to build native apps
# - Test on real device immediately
# - Share via QR code
```

---

**3. Expo Application Services (EAS)**

**EAS Build** - Cloud build service:

```bash
# Install EAS CLI
npm install -g eas-cli

# Configure
eas build:configure

# Build for iOS
eas build --platform ios

# Build for Android
eas build --platform android

# Build for both
eas build --platform all

# Creates:
# - iOS: .ipa file (App Store)
# - Android: .apk/.aab file (Play Store)
```

**EAS Submit** - Automated app store submission:

```bash
# Submit to Apple App Store
eas submit --platform ios

# Submit to Google Play Store
eas submit --platform android
```

**EAS Update** - Over-the-air (OTA) updates:

```javascript
// app.json
{
  "expo": {
    "updates": {
      "url": "https://u.expo.dev/[project-id]"
    }
  }
}

// Publish update
// Users get new JS/assets without app store review!
```

```bash
# Publish OTA update
eas update --branch production --message "Fix critical bug"

# Users receive update:
# - Next app launch (or during current session)
# - No app store approval needed
# - Only JS/assets updated (not native code)
```

---

#### Workflows

**1. Managed Workflow (Recommended for most)**

```javascript
// app.json - Complete app configuration
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "ios": {
      "bundleIdentifier": "com.company.myapp",
      "buildNumber": "1.0.0",
      "supportsTablet": true
    },
    "android": {
      "package": "com.company.myapp",
      "versionCode": 1,
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      }
    },
    "plugins": [
      "expo-camera",
      "expo-location"
    ]
  }
}

// No Xcode/Android Studio needed!
// No native code
// Use Expo SDK only
```

**Benefits:**
- ✅ No native configuration
- ✅ Quick setup
- ✅ OTA updates
- ✅ Easier maintenance

**Limitations:**
- ❌ Can't add custom native modules
- ❌ Limited to Expo SDK features
- ❌ Larger app size (includes Expo runtime)

---

**2. Bare Workflow (More control)**

```bash
# Eject to bare workflow
npx expo prebuild

# Creates:
# - ios/ folder (Xcode project)
# - android/ folder (Android Studio project)
# - Can now modify native code
```

```javascript
// Still use Expo SDK
import * as Camera from 'expo-camera';

// But can also add custom native modules
import CustomNativeModule from './native-modules/CustomNativeModule';

// And modify native configuration directly
// ios/MyApp/Info.plist
// android/app/src/main/AndroidManifest.xml
```

**Benefits:**
- ✅ Full native access
- ✅ Custom native modules
- ✅ Fine-grained control
- ✅ Still use Expo SDK

**Trade-offs:**
- ❌ Need Xcode/Android Studio
- ❌ More complex configuration
- ❌ OTA updates limited (JS/assets only, not native)

---

#### Expo vs React Native CLI

| Feature | Expo (Managed) | React Native CLI |
|---------|---------------|------------------|
| **Setup** | `npx create-expo-app` | `npx react-native init` |
| **Native Code** | No access (managed) | Full access |
| **Development** | Expo Go app | Build to device/emulator |
| **Build** | EAS Build (cloud) | Local (Xcode/Android Studio) |
| **OTA Updates** | ✅ Yes (EAS Update) | ❌ No (need CodePush) |
| **Native Modules** | Expo SDK only | Any npm package |
| **App Size** | Larger (~60MB+) | Smaller (~15MB+) |
| **Configuration** | app.json | Multiple config files |
| **Learning Curve** | Easier | Steeper |
| **Custom Native** | ❌ No (use bare workflow) | ✅ Yes |

---

#### When to Use Expo

**✅ Use Expo When:**

**1. Rapid Prototyping:**

```bash
# From idea to app in minutes
npx create-expo-app my-prototype
cd my-prototype
npx expo start

# Immediately test on phone via Expo Go
# No Xcode/Android Studio setup
```

**2. Standard App Features:**

```javascript
// Common features covered by Expo SDK:
import * as Camera from 'expo-camera';           // ✅ Camera
import * as Location from 'expo-location';       // ✅ GPS
import * as ImagePicker from 'expo-image-picker'; // ✅ Image picker
import * as Notifications from 'expo-notifications'; // ✅ Push notifications
import AsyncStorage from '@react-native-async-storage/async-storage'; // ✅ Storage
import { Audio } from 'expo-av';                 // ✅ Audio/Video

// If your app uses these, Expo is perfect!
```

**3. Web + Mobile:**

```json
// app.json
{
  "expo": {
    "platforms": ["ios", "android", "web"]
  }
}
```

```bash
# Run on web
npx expo start --web

# Single codebase, three platforms!
```

**4. OTA Updates Critical:**

```bash
# Fix bug without app store review
eas update --branch production --message "Hotfix: Fix crash on login"

# Users get update within minutes
# No waiting for Apple/Google approval
```

**5. Small Team / Solo Developer:**

```bash
# No need to learn:
# - Xcode
# - Android Studio  
# - Objective-C/Swift
# - Java/Kotlin
# - Native build systems

# Focus on JavaScript/React
```

---

**❌ Don't Use Expo When:**

**1. Custom Native Modules Required:**

```javascript
// Need specific native library not in Expo SDK?
// Example: Custom Bluetooth library
// Example: Proprietary SDK from client
// Solution: Use bare workflow or React Native CLI
```

**2. App Size Critical:**

```bash
# Expo apps larger due to Expo runtime
# Managed workflow: ~60MB+ minimum
# React Native CLI: ~15MB+ minimum

# For apps where size matters (emerging markets, limited storage):
# Consider React Native CLI
```

**3. Bleeding Edge Native Features:**

```javascript
// Want to use latest iOS/Android APIs immediately?
// Example: New iOS 18 feature released yesterday
// Expo SDK takes time to integrate new platform features
// React Native CLI gives immediate access
```

**4. Existing Native Codebase:**

```
Existing iOS/Android App
  ↓
Want to integrate React Native screens
  ↓
Use React Native CLI (not Expo)
  ↓
Can add React Native views to existing native app
```

---

#### Migration Path

**Start with Expo → Eject if Needed:**

```bash
# Phase 1: Start with Expo (managed)
npx create-expo-app my-app
# Rapid development, easy testing

# Phase 2: Need custom native module?
npx expo prebuild
# Eject to bare workflow
# Now have ios/ and android/ folders

# Phase 3: Still benefit from Expo
# - Still use Expo SDK
# - Still use EAS Build/Update
# - But can now add custom native code
```

---

#### Real-World Example

**Building a Social Media App:**

```javascript
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  Image,
  FlatList,
  TouchableOpacity,
  StyleSheet,
} from 'react-native';
import * as ImagePicker from 'expo-image-picker';
import * as Location from 'expo-location';
import * as Notifications from 'expo-notifications';
import { Camera } from 'expo-camera';

function SocialMediaApp() {
  const [posts, setPosts] = useState([]);
  
  // Easy camera access
  const takePhoto = async () => {
    const { status } = await Camera.requestCameraPermissionsAsync();
    if (status !== 'granted') return;
    
    const result = await ImagePicker.launchCameraAsync({
      mediaTypes: ImagePicker.MediaTypeOptions.Images,
      quality: 0.8,
    });
    
    if (!result.canceled) {
      uploadPhoto(result.assets[0].uri);
    }
  };
  
  // Easy location access
  const addLocation = async () => {
    const { status } = await Location.requestForegroundPermissionsAsync();
    if (status !== 'granted') return;
    
    const location = await Location.getCurrentPositionAsync({});
    return location.coords;
  };
  
  // Easy push notifications
  useEffect(() => {
    registerForPushNotifications();
  }, []);
  
  const registerForPushNotifications = async () => {
    const { status } = await Notifications.requestPermissionsAsync();
    if (status !== 'granted') return;
    
    const token = await Notifications.getExpoPushTokenAsync();
    // Send token to server
  };
  
  return (
    <View style={styles.container}>
      <FlatList
        data={posts}
        renderItem={({ item }) => <Post post={item} />}
        keyExtractor={item => item.id}
      />
      <TouchableOpacity style={styles.cameraButton} onPress={takePhoto}>
        <Text>📷</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  cameraButton: {
    position: 'absolute',
    bottom: 20,
    right: 20,
    width: 60,
    height: 60,
    borderRadius: 30,
    backgroundColor: '#007AFF',
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default SocialMediaApp;

// All features work out of the box:
// ✅ Camera
// ✅ Location
// ✅ Push notifications
// ✅ Image picker
// No native configuration needed!
```

---

#### Configuration Example

**app.json (Expo Configuration):**

```json
{
  "expo": {
    "name": "My Social App",
    "slug": "my-social-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "assetBundlePatterns": [
      "**/*"
    ],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.company.mysocialapp",
      "infoPlist": {
        "NSCameraUsageDescription": "Allow access to camera for posting photos",
        "NSPhotoLibraryUsageDescription": "Allow access to photos for posting",
        "NSLocationWhenInUseUsageDescription": "Allow location for nearby posts"
      }
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#ffffff"
      },
      "package": "com.company.mysocialapp",
      "permissions": [
        "CAMERA",
        "READ_EXTERNAL_STORAGE",
        "WRITE_EXTERNAL_STORAGE",
        "ACCESS_FINE_LOCATION"
      ]
    },
    "plugins": [
      "expo-camera",
      "expo-location",
      "expo-notifications",
      [
        "expo-image-picker",
        {
          "photosPermission": "Allow access to photos for posting"
        }
      ]
    ],
    "extra": {
      "eas": {
        "projectId": "your-project-id"
      }
    },
    "updates": {
      "url": "https://u.expo.dev/your-project-id"
    }
  }
}

// Single configuration file for:
// ✅ iOS permissions
// ✅ Android permissions
// ✅ App icons
// ✅ Splash screen
// ✅ Updates
// ✅ Plugin configuration
```

---

#### Interview Tips

✅ **Key Points:**
- Expo is a framework/platform built on React Native that simplifies development
- Provides managed workflow (no native code access) and bare workflow (full access)
- Includes Expo SDK with pre-built modules for camera, location, notifications, etc.
- EAS (Expo Application Services) provides cloud builds and OTA updates
- Perfect for rapid prototyping and standard app features
- Can eject to bare workflow if custom native modules needed
- Larger app size but faster development
- Great for small teams/solo developers

✅ **When to Mention:**
- React Native development discussions
- Rapid prototyping questions
- Mobile app development workflow
- OTA update capabilities
- Cross-platform development

✅ **Common Follow-ups:**
- "What's the difference between Expo and React Native CLI?"
- "Can you add custom native modules to Expo?"
- "What are the downsides of Expo?"
- "What's the app size overhead?"
- "Can you eject from Expo?"

✅ **Perfect Answer Structure:**
1. Definition: Framework built on React Native that simplifies development
2. Key Features: Expo SDK, Expo Go, EAS Build/Update
3. Workflows: Managed (no native code) vs Bare (full access)
4. Benefits: Fast setup, OTA updates, pre-built modules
5. When to use: Rapid prototyping, standard features, small teams
6. When not to use: Custom native modules, app size critical
7. Example: Show how easy it is to use camera/location with Expo

</details>

---

### 213. How do you share code between React and React Native?

<details>
<summary>View Answer</summary>

**Code sharing between React (web) and React Native (mobile)** is achievable because both use React's component model, but requires careful architecture due to platform differences (DOM vs native components, CSS vs StyleSheet, browser APIs vs native APIs). The key is separating business logic from UI and using platform-specific extensions or abstraction layers.

#### Core Strategies

**1. Business Logic Separation**

Share everything except UI:

```javascript
// ✅ Shareable: Business logic, state management, utilities
// ❌ Not shareable: Components (different primitives), styling, platform APIs

// Shared folder structure
src/
  shared/              // Code for both platforms
    hooks/
      useAuth.js       // ✅ Share hooks
      useApi.js        // ✅ Share API calls
    store/
      userSlice.js     // ✅ Share Redux/Zustand
    utils/
      validation.js    // ✅ Share utilities
      formatting.js    // ✅ Share helpers
    types/
      user.ts          // ✅ Share TypeScript types
  web/                 // Web-specific
    components/
      Button.web.tsx   // <button>
  mobile/              // Mobile-specific
    components/
      Button.native.tsx // <TouchableOpacity>
```

---

#### Method 1: Platform Extensions

**File Naming Convention:**

```bash
# Platform-specific extensions
Button.web.tsx      # Web version
Button.native.tsx   # React Native version
Button.ios.tsx      # iOS only
Button.android.tsx  # Android only

# React Native auto-selects:
# - Web build uses .web.tsx
# - iOS uses .ios.tsx (or .native.tsx)
# - Android uses .android.tsx (or .native.tsx)
```

**Example:**

```typescript
// components/Button.web.tsx (Web)
import React from 'react';
import './Button.css';

interface ButtonProps {
  onPress: () => void;
  title: string;
}

export const Button: React.FC<ButtonProps> = ({ onPress, title }) => {
  return (
    <button className="btn" onClick={onPress}>
      {title}
    </button>
  );
};

// components/Button.native.tsx (React Native)
import React from 'react';
import { TouchableOpacity, Text, StyleSheet } from 'react-native';

interface ButtonProps {
  onPress: () => void;
  title: string;
}

export const Button: React.FC<ButtonProps> = ({ onPress, title }) => {
  return (
    <TouchableOpacity style={styles.btn} onPress={onPress}>
      <Text style={styles.text}>{title}</Text>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  btn: {
    padding: 10,
    backgroundColor: '#007AFF',
  },
  text: {
    color: '#fff',
  },
});

// Usage in shared code (both platforms)
import { Button } from './components/Button';
// React Native automatically picks correct file!

function App() {
  return <Button onPress={() => alert('Clicked')} title="Click Me" />;
}
```

---

#### Method 2: React Native Web

**Single Codebase for All Platforms:**

```bash
npm install react-native-web
```

```javascript
// Write React Native code, run on web!
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';

function App() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello</Text>
      <TouchableOpacity style={styles.button} onPress={() => alert('Pressed')}>
        <Text>Press Me</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
  },
  button: {
    padding: 10,
    backgroundColor: '#007AFF',
  },
});

export default App;

// This code runs on:
// ✅ iOS (via React Native)
// ✅ Android (via React Native)
// ✅ Web (via React Native Web)
// Single codebase!
```

**Webpack Configuration for Web:**

```javascript
// webpack.config.js
module.exports = {
  resolve: {
    alias: {
      'react-native$': 'react-native-web',
    },
    extensions: ['.web.js', '.js', '.web.tsx', '.tsx'],
  },
  // ...
};
```

---

#### Method 3: Shared Business Logic

**Custom Hooks (100% Shareable):**

```typescript
// shared/hooks/useAuth.ts
import { useState, useEffect } from 'react';
import AsyncStorage from '@react-native-async-storage/async-storage';

export function useAuth() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    loadUser();
  }, []);
  
  const loadUser = async () => {
    try {
      const token = await AsyncStorage.getItem('token');
      if (token) {
        const response = await fetch('/api/me', {
          headers: { Authorization: `Bearer ${token}` },
        });
        const userData = await response.json();
        setUser(userData);
      }
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
    }
  };
  
  const login = async (email: string, password: string) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    });
    const { token, user } = await response.json();
    await AsyncStorage.setItem('token', token);
    setUser(user);
  };
  
  const logout = async () => {
    await AsyncStorage.removeItem('token');
    setUser(null);
  };
  
  return { user, loading, login, logout };
}

// Use in both platforms:
// Web: LoginScreen.web.tsx
import { useAuth } from '../shared/hooks/useAuth';

function LoginScreen() {
  const { login, loading } = useAuth();
  // ... UI with <div>, <input>, etc.
}

// Mobile: LoginScreen.native.tsx
import { useAuth } from '../shared/hooks/useAuth';
import { View, TextInput } from 'react-native';

function LoginScreen() {
  const { login, loading } = useAuth();
  // ... UI with <View>, <TextInput>, etc.
}
```

---

**State Management (100% Shareable):**

```typescript
// shared/store/userSlice.ts (Redux Toolkit)
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

interface UserState {
  user: User | null;
  loading: boolean;
}

const initialState: UserState = {
  user: null,
  loading: false,
};

export const fetchUser = createAsyncThunk(
  'user/fetch',
  async (userId: string) => {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }
);

const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {
    setUser: (state, action) => {
      state.user = action.payload;
    },
    clearUser: (state) => {
      state.user = null;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUser.fulfilled, (state, action) => {
        state.loading = false;
        state.user = action.payload;
      });
  },
});

export const { setUser, clearUser } = userSlice.actions;
export default userSlice.reducer;

// Use in both platforms:
// Just import and use with Redux hooks
import { useSelector, useDispatch } from 'react-redux';
import { fetchUser } from '../shared/store/userSlice';

function Profile() {
  const user = useSelector(state => state.user.user);
  const dispatch = useDispatch();
  
  useEffect(() => {
    dispatch(fetchUser('123'));
  }, []);
  
  // ... platform-specific UI
}
```

---

**Utilities (100% Shareable):**

```typescript
// shared/utils/validation.ts
export function validateEmail(email: string): boolean {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return re.test(email);
}

export function validatePassword(password: string): {
  valid: boolean;
  errors: string[];
} {
  const errors: string[] = [];
  
  if (password.length < 8) {
    errors.push('Password must be at least 8 characters');
  }
  if (!/[A-Z]/.test(password)) {
    errors.push('Password must contain uppercase letter');
  }
  if (!/[0-9]/.test(password)) {
    errors.push('Password must contain number');
  }
  
  return {
    valid: errors.length === 0,
    errors,
  };
}

// shared/utils/formatting.ts
export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount);
}

export function formatDate(date: Date): string {
  return new Intl.DateTimeFormat('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  }).format(date);
}

// Use in both platforms
import { validateEmail, formatCurrency } from '../shared/utils';
```

---

#### Method 4: Platform Abstraction Layer

**Abstract Platform Differences:**

```typescript
// shared/platform/storage.ts
import AsyncStorage from '@react-native-async-storage/async-storage';

export interface Storage {
  getItem(key: string): Promise<string | null>;
  setItem(key: string, value: string): Promise<void>;
  removeItem(key: string): Promise<void>;
}

// Web implementation
export const webStorage: Storage = {
  async getItem(key: string) {
    return localStorage.getItem(key);
  },
  async setItem(key: string, value: string) {
    localStorage.setItem(key, value);
  },
  async removeItem(key: string) {
    localStorage.removeItem(key);
  },
};

// Mobile implementation
export const mobileStorage: Storage = {
  async getItem(key: string) {
    return AsyncStorage.getItem(key);
  },
  async setItem(key: string, value: string) {
    return AsyncStorage.setItem(key, value);
  },
  async removeItem(key: string) {
    return AsyncStorage.removeItem(key);
  },
};

// Platform selector
import { Platform } from 'react-native';

export const storage: Storage = Platform.OS === 'web' 
  ? webStorage 
  : mobileStorage;

// Shared code uses abstraction
import { storage } from '../shared/platform/storage';

async function saveToken(token: string) {
  await storage.setItem('token', token);
  // Works on web (localStorage) and mobile (AsyncStorage)
}
```

---

#### Real-World Example: E-commerce App

**Project Structure:**

```
project/
  packages/
    shared/                    # Shared code (70%)
      api/
        products.ts            # API calls
        cart.ts
        auth.ts
      hooks/
        useProducts.ts         # Business logic
        useCart.ts
        useAuth.ts
      store/
        productsSlice.ts       # Redux state
        cartSlice.ts
        authSlice.ts
      utils/
        validation.ts
        formatting.ts
      types/
        Product.ts             # TypeScript types
        User.ts
    
    web/                       # Web-specific (15%)
      components/
        Button.tsx             # <button>
        ProductCard.tsx        # <div>
        Navigation.tsx         # React Router
      pages/
        HomePage.tsx
        ProductPage.tsx
      styles/
        global.css
    
    mobile/                    # Mobile-specific (15%)
      components/
        Button.tsx             # <TouchableOpacity>
        ProductCard.tsx        # <View>
        Navigation.tsx         # React Navigation
      screens/
        HomeScreen.tsx
        ProductScreen.tsx
```

**Shared API Layer:**

```typescript
// packages/shared/api/products.ts
import { Product } from '../types/Product';

const API_BASE = 'https://api.example.com';

export async function fetchProducts(): Promise<Product[]> {
  const response = await fetch(`${API_BASE}/products`);
  return response.json();
}

export async function fetchProduct(id: string): Promise<Product> {
  const response = await fetch(`${API_BASE}/products/${id}`);
  return response.json();
}

export async function searchProducts(query: string): Promise<Product[]> {
  const response = await fetch(
    `${API_BASE}/products/search?q=${encodeURIComponent(query)}`
  );
  return response.json();
}

// Used by both platforms
```

**Shared Hook:**

```typescript
// packages/shared/hooks/useProducts.ts
import { useState, useEffect } from 'react';
import { fetchProducts } from '../api/products';
import { Product } from '../types/Product';

export function useProducts() {
  const [products, setProducts] = useState<Product[]>([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  useEffect(() => {
    loadProducts();
  }, []);
  
  const loadProducts = async () => {
    try {
      setLoading(true);
      const data = await fetchProducts();
      setProducts(data);
    } catch (err) {
      setError(err as Error);
    } finally {
      setLoading(false);
    }
  };
  
  return { products, loading, error, refetch: loadProducts };
}

// Used identically in both platforms
```

**Platform-Specific UI (Web):**

```tsx
// packages/web/pages/HomePage.tsx
import React from 'react';
import { useProducts } from '@shared/hooks/useProducts';
import { ProductCard } from '../components/ProductCard';
import './HomePage.css';

function HomePage() {
  const { products, loading } = useProducts();
  
  if (loading) return <div className="loading">Loading...</div>;
  
  return (
    <div className="home-page">
      <h1>Products</h1>
      <div className="product-grid">
        {products.map(product => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
    </div>
  );
}

export default HomePage;
```

**Platform-Specific UI (Mobile):**

```tsx
// packages/mobile/screens/HomeScreen.tsx
import React from 'react';
import { View, Text, FlatList, StyleSheet } from 'react-native';
import { useProducts } from '@shared/hooks/useProducts';
import { ProductCard } from '../components/ProductCard';

function HomeScreen() {
  const { products, loading } = useProducts();
  
  if (loading) {
    return (
      <View style={styles.loading}>
        <Text>Loading...</Text>
      </View>
    );
  }
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Products</Text>
      <FlatList
        data={products}
        keyExtractor={item => item.id}
        renderItem={({ item }) => <ProductCard product={item} />}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    padding: 20,
  },
  title: {
    fontSize: 32,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  loading: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
});

export default HomeScreen;
```

---

#### Monorepo Setup

**Using Yarn Workspaces:**

```json
// package.json (root)
{
  "name": "my-app",
  "private": true,
  "workspaces": [
    "packages/*"
  ],
  "scripts": {
    "web": "cd packages/web && npm start",
    "mobile": "cd packages/mobile && npx expo start",
    "build:web": "cd packages/web && npm run build",
    "build:mobile": "cd packages/mobile && eas build"
  }
}

// packages/shared/package.json
{
  "name": "@myapp/shared",
  "version": "1.0.0",
  "main": "index.ts"
}

// packages/web/package.json
{
  "name": "@myapp/web",
  "dependencies": {
    "@myapp/shared": "*",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  }
}

// packages/mobile/package.json
{
  "name": "@myapp/mobile",
  "dependencies": {
    "@myapp/shared": "*",
    "react": "^18.2.0",
    "react-native": "^0.73.0"
  }
}
```

---

#### What Can Be Shared (Summary)

| Category | Shareable? | Notes |
|----------|-----------|-------|
| **Hooks** | ✅ 100% | Business logic, no UI |
| **State Management** | ✅ 100% | Redux, Zustand, etc. |
| **API Calls** | ✅ 100% | fetch, axios |
| **Utilities** | ✅ 100% | Pure functions |
| **Types** | ✅ 100% | TypeScript interfaces |
| **Constants** | ✅ 100% | Config, enums |
| **Components** | ❌ / ⚠️ | Need platform versions |
| **Styling** | ❌ | CSS vs StyleSheet |
| **Navigation** | ❌ | Different libraries |
| **Platform APIs** | ❌ / ⚠️ | Need abstraction layer |

---

#### Best Practices

**1. Start with Mobile-First (React Native):**

```typescript
// Write React Native code
import { View, Text } from 'react-native';

// Then add React Native Web for web support
// Easier than going web → mobile
```

**2. Separate Concerns:**

```typescript
// ✅ Good: Logic separate from UI
const useProductLogic = () => {
  const [products, setProducts] = useState([]);
  // ... logic
  return { products, loadProducts };
};

// Platform-specific rendering
function ProductList() {
  const { products } = useProductLogic();
  return <PlatformSpecificUI products={products} />;
}

// ❌ Bad: Mixed logic and UI
function ProductList() {
  const [products, setProducts] = useState([]);
  // ... logic mixed with JSX
  return <div>...</div>; // Hard to share
}
```

**3. Use TypeScript:**

```typescript
// Shared types ensure consistency
interface Product {
  id: string;
  name: string;
  price: number;
}

// Both platforms use same types
// Catch errors at compile time
```

**4. Abstract Platform APIs:**

```typescript
// Don't use platform APIs directly in shared code
// Create abstraction layer
import { storage } from './platform/storage';
// Works on both platforms
```

---

#### Interview Tips

✅ **Key Points:**
- Separate business logic (hooks, state, utilities) from UI components
- Use platform extensions (.web.tsx, .native.tsx) for platform-specific code
- React Native Web allows single codebase for web+mobile
- State management (Redux/Zustand) is 100% shareable
- API calls and utilities are 100% shareable
- Components need platform-specific versions (different primitives)
- Monorepo structure (Yarn workspaces) helps organize shared code
- Abstract platform APIs (storage, navigation) for cleaner sharing

✅ **When to Mention:**
- Cross-platform development questions
- Code reusability discussions
- Monorepo architecture
- React Native Web
- Team efficiency strategies

✅ **Common Follow-ups:**
- "What percentage of code can typically be shared?"
- "How do you handle styling differences?"
- "What about navigation?"
- "Is React Native Web production-ready?"
- "How do you structure a monorepo?"

✅ **Perfect Answer Structure:**
1. Strategy: Separate business logic from UI
2. Platform extensions: .web.tsx vs .native.tsx
3. Shareable: Hooks, state management, API calls, utilities
4. Not shareable: Components, styling, navigation
5. React Native Web: Single codebase option
6. Abstraction layer: For platform APIs
7. Example: Show shared hook with platform-specific UI

</details>

---

### 214. What is the difference between React Native and Flutter?

<details>
<summary>View Answer</summary>

**React Native** (by Meta) and **Flutter** (by Google) are both frameworks for building cross-platform mobile apps, but they differ fundamentally in language (JavaScript vs Dart), architecture (bridge vs compiled), rendering (native components vs custom engine), and ecosystem maturity. Both achieve "write once, run anywhere," but with different trade-offs.

#### High-Level Comparison

| Aspect | React Native | Flutter |
|--------|-------------|----------|
| **Company** | Meta (Facebook) | Google |
| **Released** | 2015 | 2017 |
| **Language** | JavaScript/TypeScript | Dart |
| **Rendering** | Native components | Custom Skia engine |
| **Architecture** | Bridge (async) | Compiled to native (direct) |
| **UI** | Platform-specific | Consistent across platforms |
| **Performance** | Near-native | Native-like (60 FPS) |
| **Hot Reload** | ✅ Yes | ✅ Yes |
| **Learning Curve** | Easier (JS knowledge) | Steeper (learn Dart) |
| **Community** | Larger, more mature | Growing fast |
| **Ecosystem** | Vast (npm) | Growing |
| **Web Support** | Via React Native Web | Official |
| **Desktop** | Experimental | Official (Windows, macOS, Linux) |

---

#### Language

**React Native - JavaScript/TypeScript:**

```javascript
// JavaScript (familiar to millions of developers)
import React, { useState } from 'react';
import { View, Text, TouchableOpacity } from 'react-native';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <View>
      <Text>Count: {count}</Text>
      <TouchableOpacity onPress={() => setCount(count + 1)}>
        <Text>Increment</Text>
      </TouchableOpacity>
    </View>
  );
}

// ✅ Pros:
// - Huge existing JS developer pool
// - Share code with React web apps
// - Familiar syntax
// - TypeScript support

// ❌ Cons:
// - Single-threaded (can be bottleneck)
// - Not as performant as compiled languages
```

**Flutter - Dart:**

```dart
// Dart (Google's language, optimized for UI)
import 'package:flutter/material.dart';

class Counter extends StatefulWidget {
  @override
  _CounterState createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int count = 0;
  
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $count'),
        ElevatedButton(
          onPressed: () {
            setState(() {
              count++;
            });
          },
          child: Text('Increment'),
        ),
      ],
    );
  }
}

// ✅ Pros:
// - Compiled to native (ARM/x86)
// - AOT (Ahead-of-Time) compilation for performance
// - JIT (Just-in-Time) for development (hot reload)
// - Optimized for UI (async/await built-in)

// ❌ Cons:
// - Smaller developer pool (learn new language)
// - Less reusable (Dart not common in web)
```

---

#### Architecture

**React Native - Bridge Architecture:**

```
[JavaScript Thread]    [Bridge]    [Native Thread]
- React components  <─ JSON ─>  - iOS/Android views
- Business logic    serialized   - UI rendering
- State management               - Gestures

Flow:
1. JavaScript code runs in JS engine
2. UI updates serialized to JSON
3. Sent across bridge (async)
4. Native modules create/update views
5. User interactions sent back to JS

Bottleneck: Bridge communication (async, serialization overhead)
```

**Example:**

```javascript
// React Native
function App() {
  return (
    <View style={{ backgroundColor: 'red' }}>
      <Text>Hello</Text>
    </View>
  );
}

// Under the hood:
// 1. JS: Create View with red background
// 2. Bridge: { type: 'createView', viewId: 1, props: { backgroundColor: 'red' } }
// 3. Native: Create UIView (iOS) or android.view.View (Android) with red background
// 4. Result: Platform-specific native component
```

**Flutter - Compiled Architecture:**

```
[Dart Code]
    ↓
[Compiled to Native ARM/x86]
    ↓
[Skia Graphics Engine]
    ↓
[Canvas APIs (OpenGL/Metal/Vulkan)]
    ↓
[Screen pixels]

Flow:
1. Dart code compiled directly to native
2. Flutter draws every pixel using Skia
3. No bridge, direct communication
4. Renders custom widgets (not native components)

Advantage: No bridge overhead, direct rendering
```

**Example:**

```dart
// Flutter
class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Container(
      color: Colors.red,
      child: Text('Hello'),
    );
  }
}

// Under the hood:
// 1. Dart code compiled to native
// 2. Flutter draws container and text using Skia
// 3. Result: Custom-rendered UI (looks same on iOS/Android)
```

---

#### Rendering

**React Native - Native Components:**

```javascript
import { View, Text, Button } from 'react-native';

function App() {
  return (
    <View>
      <Text>Hello</Text>
      <Button title="Click" onPress={() => {}} />
    </View>
  );
}

// iOS:
// <View> → UIView
// <Text> → UILabel
// <Button> → UIButton

// Android:
// <View> → android.view.View
// <Text> → TextView
// <Button> → AppCompatButton

// ✅ Pros:
// - Native look and feel
// - Automatic platform updates (iOS 17 → new button style)
// - Feels native because it IS native

// ❌ Cons:
// - Inconsistent across platforms
// - Need to test on both platforms
// - Platform-specific bugs
```

**Flutter - Custom Rendering:**

```dart
import 'package:flutter/material.dart';

class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Hello'),
        ElevatedButton(
          onPressed: () {},
          child: Text('Click'),
        ),
      ],
    );
  }
}

// iOS & Android:
// Everything rendered by Flutter's Skia engine
// Draws every pixel (button, text, colors)
// Looks IDENTICAL on both platforms

// ✅ Pros:
// - Consistent UI across platforms
// - Pixel-perfect control
// - Easy to implement Material Design or custom themes
// - Single set of tests

// ❌ Cons:
// - Doesn't look "native" by default
// - Need to manually follow platform guidelines
// - Larger app size (includes rendering engine)
```

---

#### Performance

**React Native:**

```javascript
// Bridge bottleneck example
function AnimatedComponent() {
  const translateX = useRef(new Animated.Value(0)).current;
  
  useEffect(() => {
    // Animation loop
    Animated.timing(translateX, {
      toValue: 100,
      duration: 1000,
      useNativeDriver: true, // ⚠️ Required for smooth animations
    }).start();
  }, []);
  
  return (
    <Animated.View style={{ transform: [{ translateX }] }}>
      <Text>Animated</Text>
    </Animated.View>
  );
}

// Without useNativeDriver: 30-40 FPS (bridge bottleneck)
// With useNativeDriver: 60 FPS (native side handles animation)

// Performance:
// ✅ Near-native for most apps
// ⚠️ Bridge can be bottleneck for:
//    - Complex animations (without native driver)
//    - Large lists (use FlatList optimization)
//    - Frequent updates

// New Architecture (Fabric + TurboModules):
// - JSI (JavaScript Interface) bypasses bridge
// - Synchronous calls
// - ~50% faster
```

**Flutter:**

```dart
// Direct rendering - no bridge
class AnimatedComponent extends StatefulWidget {
  @override
  _AnimatedComponentState createState() => _AnimatedComponentState();
}

class _AnimatedComponentState extends State<AnimatedComponent>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(seconds: 1),
      vsync: this,
    )..forward();
  }
  
  @override
  Widget build(BuildContext context) {
    return AnimatedBuilder(
      animation: _controller,
      builder: (context, child) {
        return Transform.translate(
          offset: Offset(_controller.value * 100, 0),
          child: Text('Animated'),
        );
      },
    );
  }
}

// Compiled to native, direct rendering
// Consistent 60 FPS (120 FPS on supported devices)
// No bridge overhead

// Performance:
// ✅ Excellent out of the box
// ✅ Smooth animations (Skia optimized)
// ✅ Handles complex UIs
// ⚠️ Startup time slightly slower (larger engine)
```

---

#### UI Components

**React Native - Platform-Aware:**

```javascript
import { Platform, View, Text } from 'react-native';

function App() {
  return (
    <View>
      <Text style={{
        fontSize: Platform.OS === 'ios' ? 17 : 16,
        fontFamily: Platform.OS === 'ios' ? 'San Francisco' : 'Roboto',
      }}>
        Platform-specific text
      </Text>
      
      {Platform.OS === 'ios' ? (
        <IOSSpecificComponent />
      ) : (
        <AndroidSpecificComponent />
      )}
    </View>
  );
}

// Libraries provide platform-specific look:
import { NavigationContainer } from '@react-navigation/native';
// Automatically uses iOS-style navigation on iOS
// and Material Design on Android
```

**Flutter - Material and Cupertino:**

```dart
import 'package:flutter/material.dart';
import 'package:flutter/cupertino.dart';

class App extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Option 1: Material Design (Android-like on all platforms)
    return MaterialApp(
      home: Scaffold(
        body: Text('Material Design'),
        floatingActionButton: FloatingActionButton(
          onPressed: () {},
          child: Icon(Icons.add),
        ),
      ),
    );
    
    // Option 2: Cupertino (iOS-like on all platforms)
    return CupertinoApp(
      home: CupertinoPageScaffold(
        child: Text('iOS Design'),
        navigationBar: CupertinoNavigationBar(
          middle: Text('Title'),
        ),
      ),
    );
    
    // Option 3: Platform-adaptive
    return Platform.isIOS
      ? CupertinoButton(child: Text('iOS'), onPressed: () {})
      : ElevatedButton(child: Text('Android'), onPressed: () {});
  }
}

// Flutter provides both design systems
// But needs manual platform detection for adaptive UI
```

---

#### Developer Experience

**React Native:**

```bash
# Setup (requires Node.js, Xcode, Android Studio)
npx react-native init MyApp
cd MyApp

# Run on iOS
npx react-native run-ios

# Run on Android
npx react-native run-android

# Hot reload: ⌘R (iOS) / Double R (Android)
# Fast Refresh: Automatic on save

# Debug:
# - Chrome DevTools (JS debugging)
# - Flipper (native debugging, network, layout)
# - React DevTools

# Learning:
# ✅ Easy if you know React/JavaScript
# ✅ Huge community, many tutorials
# ✅ Can hire React developers
```

**Flutter:**

```bash
# Setup (requires Flutter SDK)
flutter create my_app
cd my_app

# Run on iOS
flutter run -d ios

# Run on Android
flutter run -d android

# Hot reload: Press 'r' in terminal
# Hot restart: Press 'R' in terminal
# Very fast hot reload (stateful)

# Debug:
# - Dart DevTools (debugging, profiling)
# - Flutter Inspector (widget tree, layout)
# - Performance overlay (FPS, memory)

# Learning:
# ⚠️ Need to learn Dart
# ✅ Excellent documentation
# ✅ Growing community
# ⚠️ Smaller talent pool
```

---

#### Ecosystem

**React Native:**

```javascript
// npm ecosystem - HUGE
import axios from 'axios';                    // HTTP
import { Provider } from 'react-redux';       // State
import AsyncStorage from '@react-native-async-storage/async-storage'; // Storage
import { NavigationContainer } from '@react-navigation/native'; // Navigation
import { Camera } from 'expo-camera';         // Camera

// Mature libraries for everything
// Many third-party packages
// Can use most web libraries (if no DOM dependencies)

// ✅ Pros: Vast ecosystem, mature packages
// ⚠️ Cons: Quality varies, maintenance issues
```

**Flutter:**

```dart
// pub.dev ecosystem - growing fast
import 'package:http/http.dart' as http;     // HTTP
import 'package:provider/provider.dart';     // State
import 'package:shared_preferences/shared_preferences.dart'; // Storage
import 'package:camera/camera.dart';         // Camera

// Official packages well-maintained
// Growing third-party ecosystem
// Google provides many official packages

// ✅ Pros: High-quality official packages
// ⚠️ Cons: Smaller ecosystem, fewer options
```

---

#### Code Comparison: Todo App

**React Native:**

```javascript
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  FlatList,
  StyleSheet,
} from 'react-native';

function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, { id: Date.now(), text: input }]);
      setInput('');
    }
  };
  
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Todos</Text>
      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          value={input}
          onChangeText={setInput}
          placeholder="Add todo"
        />
        <TouchableOpacity style={styles.button} onPress={addTodo}>
          <Text style={styles.buttonText}>Add</Text>
        </TouchableOpacity>
      </View>
      <FlatList
        data={todos}
        keyExtractor={item => item.id.toString()}
        renderItem={({ item }) => (
          <View style={styles.todoItem}>
            <Text>{item.text}</Text>
          </View>
        )}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: { flex: 1, padding: 20 },
  title: { fontSize: 24, fontWeight: 'bold', marginBottom: 20 },
  inputContainer: { flexDirection: 'row', marginBottom: 20 },
  input: { flex: 1, borderWidth: 1, padding: 10, marginRight: 10 },
  button: { backgroundColor: '#007AFF', padding: 10, borderRadius: 5 },
  buttonText: { color: '#fff' },
  todoItem: { padding: 15, borderBottomWidth: 1, borderColor: '#eee' },
});

export default TodoApp;
```

**Flutter:**

```dart
import 'package:flutter/material.dart';

class TodoApp extends StatefulWidget {
  @override
  _TodoAppState createState() => _TodoAppState();
}

class _TodoAppState extends State<TodoApp> {
  List<Map<String, dynamic>> todos = [];
  TextEditingController controller = TextEditingController();
  
  void addTodo() {
    if (controller.text.trim().isNotEmpty) {
      setState(() {
        todos.add({
          'id': DateTime.now().millisecondsSinceEpoch,
          'text': controller.text,
        });
        controller.clear();
      });
    }
  }
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Padding(
        padding: EdgeInsets.all(20),
        child: Column(
          children: [
            Text(
              'Todos',
              style: TextStyle(fontSize: 24, fontWeight: FontWeight.bold),
            ),
            SizedBox(height: 20),
            Row(
              children: [
                Expanded(
                  child: TextField(
                    controller: controller,
                    decoration: InputDecoration(
                      hintText: 'Add todo',
                      border: OutlineInputBorder(),
                    ),
                  ),
                ),
                SizedBox(width: 10),
                ElevatedButton(
                  onPressed: addTodo,
                  child: Text('Add'),
                ),
              ],
            ),
            SizedBox(height: 20),
            Expanded(
              child: ListView.builder(
                itemCount: todos.length,
                itemBuilder: (context, index) {
                  return ListTile(
                    title: Text(todos[index]['text']),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}

void main() => runApp(MaterialApp(home: TodoApp()));
```

---

#### When to Choose

**Choose React Native if:**

✅ Your team knows JavaScript/React
✅ Want to share code with React web app
✅ Need truly native look and feel
✅ Want access to vast npm ecosystem
✅ Need mature community and resources
✅ Hiring JavaScript developers is easier
✅ Integrating with existing native app

**Choose Flutter if:**

✅ Want consistent UI across platforms
✅ Performance is critical (animations, complex UI)
✅ Building from scratch (no existing React code)
✅ Want official web and desktop support
✅ Team willing to learn Dart
✅ Need pixel-perfect custom designs
✅ Want faster development (hot reload is excellent)

---

#### Interview Tips

✅ **Key Points:**
- React Native uses JavaScript, renders to native components, uses bridge architecture
- Flutter uses Dart, renders with custom Skia engine, compiles to native
- React Native: Platform-specific native UI, larger ecosystem, easier for React devs
- Flutter: Consistent UI, better performance, official multi-platform support
- React Native: Bridge can be bottleneck (New Architecture improves this)
- Flutter: Compiled code, no bridge, smooth 60 FPS animations
- React Native: Easier hiring (JavaScript), larger community
- Flutter: Steeper learning curve (Dart), but excellent documentation

✅ **When to Mention:**
- Cross-platform framework comparisons
- Mobile development strategy discussions
- Performance requirements
- Team composition questions
- Technology stack decisions

✅ **Common Follow-ups:**
- "Which is more performant?"
- "Which has better developer experience?"
- "Can you use native modules in both?"
- "Which should startups choose?"
- "What about React Native's New Architecture?"

✅ **Perfect Answer Structure:**
1. Overview: Both cross-platform, different approaches
2. Language: JavaScript (familiar) vs Dart (learn new)
3. Architecture: Bridge (async) vs Compiled (direct)
4. Rendering: Native components vs Custom engine
5. Performance: React Native near-native, Flutter native-like
6. When to choose: Team skills, requirements, ecosystem needs
7. Example: Show side-by-side code comparison

</details>

---

### 215. How do you handle platform-specific code?

<details>
<summary>View Answer</summary>

**Platform-specific code** in React Native allows you to write different implementations for iOS, Android, and web while maintaining a shared codebase. React Native provides several mechanisms: the `Platform` module for runtime checks, file extensions for separate files, and `Platform.select()` for cleaner conditional code.

#### Method 1: Platform Module

**Runtime Detection:**

```javascript
import { Platform, StyleSheet } from 'react-native';

// Simple check
function MyComponent() {
  return (
    <View>
      <Text>
        {Platform.OS === 'ios' ? 'iOS' : 'Android'}
      </Text>
    </View>
  );
}

// Platform.OS values:
// - 'ios'
// - 'android'
// - 'web' (with React Native Web)
// - 'windows' (with React Native Windows)
// - 'macos' (with React Native macOS)

// Check iOS version
if (Platform.OS === 'ios' && parseInt(Platform.Version, 10) >= 14) {
  console.log('iOS 14 or higher');
}

// Check Android API level
if (Platform.OS === 'android' && Platform.Version >= 29) {
  console.log('Android 10 (API 29) or higher');
}
```

**Styling Differences:**

```javascript
const styles = StyleSheet.create({
  container: {
    flex: 1,
    paddingTop: Platform.OS === 'ios' ? 20 : 0, // iOS status bar
    backgroundColor: Platform.OS === 'ios' ? '#fff' : '#f5f5f5',
  },
  text: {
    fontSize: Platform.OS === 'ios' ? 17 : 16,
    fontFamily: Platform.OS === 'ios' ? 'System' : 'Roboto',
  },
  shadow: Platform.select({
    ios: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.25,
      shadowRadius: 3.84,
    },
    android: {
      elevation: 5,
    },
  }),
});
```

#### Method 2: Platform.select()

**Cleaner Conditional Values:**

```javascript
const fontFamily = Platform.select({
  ios: 'System',
  android: 'Roboto',
  web: 'Arial',
  default: 'System',
});
```

#### Method 3: Platform-Specific File Extensions

**Automatic File Selection:**

```bash
components/
  Button.js           # Shared (fallback)
  Button.ios.js       # iOS-specific
  Button.android.js   # Android-specific
  Button.web.js       # Web-specific
  Button.native.js    # iOS + Android (not web)
```

**Interview Tips:**
- Platform module for runtime checks
- File extensions for separate implementations
- Common differences: shadows, status bar, touch feedback
- Minimize platform-specific code
- Always test on both platforms

</details>

---

### 216. What is React Native Web?

<details>
<summary>View Answer</summary>

**React Native Web** is a library that enables React Native components and APIs to run on the web. It provides web implementations of React Native primitives (View, Text, Image, etc.), allowing you to write code once using React Native and deploy to iOS, Android, and web from a single codebase. Used by Twitter, Uber, and many others.

#### What is React Native Web?

**Core Concept:**

```javascript
// Write React Native code
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

function App() {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Hello World</Text>
    </View>
  );
}

// This SAME code runs on:
// ✅ iOS (native UIView)
// ✅ Android (native View)
// ✅ Web (React Native Web converts to <div>, <span>)
```

**Component Mapping:**

| React Native | React Native Web (HTML) |
|--------------|------------------------|
| `<View>` | `<div>` |
| `<Text>` | `<span>` |
| `<Image>` | `<img>` |
| `<TouchableOpacity>` | `<div>` with `onClick` |
| `<ScrollView>` | `<div>` with `overflow: auto` |

#### Setup

```bash
npm install react-native-web react-dom
```

```javascript
// webpack.config.js
module.exports = {
  resolve: {
    alias: {
      'react-native$': 'react-native-web',
    },
  },
};
```

#### Real-World Example: Twitter

```javascript
// Shared component for iOS, Android, Web
function Tweet({ tweet }) {
  return (
    <View style={styles.container}>
      <Image source={{ uri: tweet.user.avatar }} style={styles.avatar} />
      <Text style={styles.name}>{tweet.user.name}</Text>
      <Text style={styles.text}>{tweet.text}</Text>
    </View>
  );
}

// Works on iOS app, Android app, and twitter.com!
```

#### Benefits

1. **Code Reuse:** 70-90% code sharing typical
2. **Consistent UI:** Same component design on all platforms
3. **Single Team:** No separate web developers needed
4. **Developer Experience:** Same tools and workflow

#### Limitations

1. **Bundle Size:** ~35KB gzipped overhead
2. **SEO:** Needs SSR setup
3. **Native Modules:** Mobile-only features don't work on web
4. **Performance:** Complex UIs may need web-specific optimization

#### When to Use

**✅ Use when:**
- Building mobile-first app with web version
- Team knows React Native
- Want consistent UI across platforms
- High code reuse important

**❌ Don't use when:**
- Web-only application (use regular React)
- SEO critical (Next.js easier)
- Bundle size critical
- Heavy web-specific interactions

**Interview Tips:**
- Enables React Native code to run on web
- Converts RN components to HTML/CSS
- Used by Twitter, Uber in production
- Great for mobile-first apps needing web version
- Trade-off: Bundle size vs code reuse

</details>

---

### 217. How do you optimize React Native performance?

<details>
<summary>View Answer</summary>

**React Native performance optimization** involves minimizing bridge communication, optimizing rendering, reducing JavaScript thread workload, and leveraging native performance features. The key bottlenecks are the asynchronous bridge, JavaScript single-threading, and unnecessary re-renders.

#### Performance Bottlenecks

**1. Bridge Communication:**

```javascript
// Problem: Bridge is async and serializes data
[JavaScript Thread] <--JSON--> [Native Thread]

// Every UI update crosses bridge:
// 1. JS creates view description
// 2. Serializes to JSON
// 3. Sends across bridge
// 4. Native deserializes
// 5. Updates native view

// Bottleneck for:
// - Frequent updates (animations)
// - Large data transfers
// - Complex lists
```

**2. JavaScript Thread:**

```javascript
// Single-threaded JavaScript
// Everything competes for CPU time:
// - React rendering
// - Business logic
// - API calls
// - State updates

// Heavy computation blocks UI:
function heavyCalculation() {
  let result = 0;
  for (let i = 0; i < 100000000; i++) {
    result += i;
  }
  return result; // Blocks UI for ~1 second!
}
```

---

#### Optimization 1: FlatList Optimization

**Problem: ScrollView renders all items:**

```javascript
// ❌ Bad: Renders 10,000 items immediately
import { ScrollView, View, Text } from 'react-native';

function ProductList({ products }) {
  return (
    <ScrollView>
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </ScrollView>
  );
}

// 10,000 items × 50ms render = 500 seconds!
// Memory: All items kept in memory
// Result: Slow scroll, high memory usage
```

**Solution: FlatList with virtualization:**

```javascript
// ✅ Good: Renders only visible items
import { FlatList } from 'react-native';

function ProductList({ products }) {
  return (
    <FlatList
      data={products}
      renderItem={({ item }) => <ProductCard product={item} />}
      keyExtractor={item => item.id}
      
      // Optimization props
      removeClippedSubviews={true} // Unmount off-screen items
      maxToRenderPerBatch={10} // Render 10 items per batch
      updateCellsBatchingPeriod={50} // Batch every 50ms
      initialNumToRender={10} // Render 10 initially
      windowSize={5} // Keep 5 screens of items
      
      // Performance monitoring
      onEndReachedThreshold={0.5} // Load more at 50% scroll
      getItemLayout={(data, index) => (
        { length: 100, offset: 100 * index, index }
      )} // Skip measurement if fixed height
    />
  );
}

// Only renders ~20 visible items
// Memory: Recycles item views
// Result: Smooth 60 FPS scroll
```

---

#### Optimization 2: Memoization

**React.memo:**

```javascript
// ❌ Bad: Re-renders on every parent update
function ProductCard({ product }) {
  console.log('Rendering', product.id);
  return (
    <View>
      <Text>{product.name}</Text>
      <Text>${product.price}</Text>
    </View>
  );
}

function ProductList() {
  const [filter, setFilter] = useState('');
  // Every filter change re-renders ALL cards
  return products.map(p => <ProductCard product={p} />);
}

// ✅ Good: Only re-renders if props change
const ProductCard = React.memo(({ product }) => {
  console.log('Rendering', product.id);
  return (
    <View>
      <Text>{product.name}</Text>
      <Text>${product.price}</Text>
    </View>
  );
});

// Shallow comparison of props
// Re-renders only if product object changes
```

**useMemo and useCallback:**

```javascript
import { useMemo, useCallback } from 'react';

function ProductList({ products }) {
  const [sortOrder, setSortOrder] = useState('asc');
  
  // ❌ Bad: Sorts on every render
  const sortedProducts = products.sort((a, b) => 
    sortOrder === 'asc' ? a.price - b.price : b.price - a.price
  );
  
  // ✅ Good: Only sorts when products or sortOrder changes
  const sortedProducts = useMemo(() => {
    console.log('Sorting...');
    return products.sort((a, b) => 
      sortOrder === 'asc' ? a.price - b.price : b.price - a.price
    );
  }, [products, sortOrder]);
  
  // ❌ Bad: Creates new function on every render
  const handlePress = (id) => {
    console.log('Pressed', id);
  };
  
  // ✅ Good: Memoizes function
  const handlePress = useCallback((id) => {
    console.log('Pressed', id);
  }, []);
  
  return (
    <FlatList
      data={sortedProducts}
      renderItem={({ item }) => (
        <ProductCard product={item} onPress={handlePress} />
      )}
    />
  );
}
```

---

#### Optimization 3: Native Driver for Animations

**Problem: JS thread animations drop frames:**

```javascript
import { Animated } from 'react-native';

function FadeIn() {
  const opacity = useRef(new Animated.Value(0)).current;
  
  useEffect(() => {
    // ❌ Bad: JS-driven animation
    Animated.timing(opacity, {
      toValue: 1,
      duration: 1000,
      useNativeDriver: false, // JS thread
    }).start();
    
    // Every frame:
    // 1. JS calculates new opacity
    // 2. Sends to native via bridge
    // 3. Native updates view
    // Result: 30-40 FPS (bridge bottleneck)
  }, []);
  
  return <Animated.View style={{ opacity }} />;
}
```

**Solution: Native driver:**

```javascript
function FadeIn() {
  const opacity = useRef(new Animated.Value(0)).current;
  
  useEffect(() => {
    // ✅ Good: Native-driven animation
    Animated.timing(opacity, {
      toValue: 1,
      duration: 1000,
      useNativeDriver: true, // Native thread!
    }).start();
    
    // Animation runs entirely on native side:
    // 1. JS sends animation config once
    // 2. Native runs animation at 60 FPS
    // 3. No bridge communication per frame
    // Result: Smooth 60 FPS
  }, []);
  
  return <Animated.View style={{ opacity }} />;
}

// useNativeDriver works for:
// ✅ opacity
// ✅ transform (translate, rotate, scale)
// ❌ layout properties (width, height, flex)
// ❌ colors (backgroundColor, etc.)
```

---

#### Optimization 4: Image Optimization

**Efficient Image Loading:**

```javascript
import FastImage from 'react-native-fast-image';

function ProductImage({ url }) {
  return (
    // ❌ Bad: Default Image component
    // - No caching
    // - Loads on UI thread
    // - No priority
    <Image source={{ uri: url }} style={{ width: 200, height: 200 }} />
    
    // ✅ Good: FastImage with optimizations
    <FastImage
      source={{
        uri: url,
        priority: FastImage.priority.normal,
        cache: FastImage.cacheControl.immutable,
      }}
      style={{ width: 200, height: 200 }}
      resizeMode={FastImage.resizeMode.cover}
    />
    // - Caches images
    // - Background loading
    // - Priority control
  );
}

// Also specify dimensions to avoid layout shifts
const styles = StyleSheet.create({
  image: {
    width: 200,
    height: 200, // ✅ Fixed dimensions = no reflow
  },
});
```

---

#### Optimization 5: Avoid Inline Functions

**Problem: New function on every render:**

```javascript
function ProductList({ products }) {
  return (
    <FlatList
      data={products}
      renderItem={({ item }) => (
        // ❌ Bad: New function every render
        <TouchableOpacity onPress={() => console.log(item.id)}>
          <Text>{item.name}</Text>
        </TouchableOpacity>
      )}
    />
  );
}

// Every render creates new arrow function
// PureComponent/memo can't optimize (props always different)
```

**Solution: Extract and memoize:**

```javascript
function ProductList({ products }) {
  const handlePress = useCallback((id) => {
    console.log(id);
  }, []);
  
  const renderItem = useCallback(({ item }) => (
    <ProductCard product={item} onPress={handlePress} />
  ), [handlePress]);
  
  return (
    <FlatList
      data={products}
      renderItem={renderItem} // ✅ Stable reference
    />
  );
}

const ProductCard = React.memo(({ product, onPress }) => (
  <TouchableOpacity onPress={() => onPress(product.id)}>
    <Text>{product.name}</Text>
  </TouchableOpacity>
));
```

---

#### Optimization 6: Reduce Bridge Traffic

**Batch Updates:**

```javascript
import { InteractionManager } from 'react-native';

// ❌ Bad: Multiple state updates
function loadData() {
  fetch('/api/user').then(user => setUser(user)); // Update 1
  fetch('/api/posts').then(posts => setPosts(posts)); // Update 2
  fetch('/api/friends').then(friends => setFriends(friends)); // Update 3
  // 3 separate renders + bridge crossings
}

// ✅ Good: Batch updates
function loadData() {
  Promise.all([
    fetch('/api/user'),
    fetch('/api/posts'),
    fetch('/api/friends'),
  ]).then(([user, posts, friends]) => {
    // Single state update
    setState({ user, posts, friends });
    // 1 render + 1 bridge crossing
  });
}

// Defer expensive operations
function handlePress() {
  // Run after animations complete
  InteractionManager.runAfterInteractions(() => {
    // Heavy computation here
    expensiveOperation();
  });
}
```

---

#### Optimization 7: Hermes JavaScript Engine

**Enable Hermes (React Native 0.70+):**

```javascript
// android/gradle.properties
hermesEnabled=true

// ios/Podfile
:hermes_enabled => true

// Benefits:
// ✅ 50% faster app start time
// ✅ Reduced memory usage (~30-50%)
// ✅ Smaller app size
// ✅ Bytecode precompilation
// ✅ Better garbage collection
```

---

#### Optimization 8: Profiling

**React DevTools Profiler:**

```javascript
import { Profiler } from 'react';

function App() {
  return (
    <Profiler id="ProductList" onRender={onRenderCallback}>
      <ProductList />
    </Profiler>
  );
}

function onRenderCallback(
  id, // "ProductList"
  phase, // "mount" or "update"
  actualDuration, // Time spent rendering
  baseDuration, // Estimated time without memoization
  startTime,
  commitTime
) {
  console.log(`${id} took ${actualDuration}ms to render`);
}
```

**Performance Monitor:**

```javascript
import { PerformanceObserver } from 'react-native-performance';

const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    console.log(`${entry.name}: ${entry.duration}ms`);
  });
});

observer.observe({ entryTypes: ['measure'] });

// Measure operations
performance.mark('data-load-start');
await fetchData();
performance.mark('data-load-end');
performance.measure('data-load', 'data-load-start', 'data-load-end');
```

**Flipper Integration:**

```bash
# Open Flipper
npx react-native run-android

# Plugins:
# - Network: Monitor API calls
# - Images: View image cache
# - Layout: Inspect view hierarchy
# - Performance: FPS, memory, CPU
```

---

#### Quick Wins Checklist

**List Rendering:**
- ✅ Use `FlatList` instead of `ScrollView.map()`
- ✅ Add `keyExtractor`
- ✅ Enable `removeClippedSubviews`
- ✅ Set `getItemLayout` if fixed height
- ✅ Use `initialNumToRender` and `windowSize`

**Animations:**
- ✅ Enable `useNativeDriver: true`
- ✅ Animate transform/opacity (not layout)
- ✅ Use `Animated.event` for gestures

**Images:**
- ✅ Use `FastImage` library
- ✅ Specify width/height
- ✅ Use appropriate `resizeMode`
- ✅ Enable caching

**Components:**
- ✅ Wrap with `React.memo`
- ✅ Use `useMemo` for expensive calculations
- ✅ Use `useCallback` for event handlers
- ✅ Avoid inline functions in render

**General:**
- ✅ Enable Hermes
- ✅ Use Release build for testing
- ✅ Profile with Flipper
- ✅ Monitor with Performance Monitor
- ✅ Keep JavaScript bundle small

---

#### Interview Tips

✅ **Key Points:**
- Main bottleneck: Bridge between JS and native threads
- FlatList virtualizes large lists (only renders visible items)
- useNativeDriver offloads animations to native thread (60 FPS)
- React.memo/useMemo/useCallback prevent unnecessary re-renders
- Hermes engine improves startup time and memory usage
- Profile with React DevTools and Flipper
- Avoid inline functions in render (creates new references)
- Batch state updates to reduce bridge crossings

✅ **When to Mention:**
- React Native performance discussions
- Scrolling performance issues
- Animation smoothness
- Memory optimization
- App startup time

✅ **Common Follow-ups:**
- "How does the bridge affect performance?"
- "What's useNativeDriver?"
- "When should you use FlatList vs ScrollView?"
- "What is Hermes?"
- "How do you profile React Native apps?"

✅ **Perfect Answer Structure:**
1. Bottlenecks: Bridge, JS thread, unnecessary renders
2. FlatList: Virtualization for large lists
3. Native animations: useNativeDriver for 60 FPS
4. Memoization: React.memo, useMemo, useCallback
5. Hermes: Better startup and memory
6. Profiling: DevTools, Flipper, Performance Monitor
7. Example: Show FlatList optimization or animation with native driver

</details>

---

### 218. What is the New Architecture in React Native?

<details>
<summary>View Answer</summary>

**The New Architecture** (React Native 0.68+) is a complete redesign of React Native's internals, replacing the asynchronous bridge with synchronous communication (JSI), introducing a new rendering engine (Fabric), and modernizing native module system (TurboModules). It dramatically improves performance, enables concurrent rendering, and simplifies native integration.

#### Old Architecture Problems

**Bridge Bottleneck:**

```javascript
// Old Architecture (pre-0.68)

[JavaScript Thread]    [Bridge (Async)]    [Native Thread]
- React rendering   <-- JSON messages -->  - Native views
- Business logic       (Serialization)     - UI rendering

Flow:
1. JS: Create/update component
2. JS: Serialize to JSON
3. Bridge: Queue message (async)
4. Native: Deserialize JSON
5. Native: Update view

Problems:
❌ Asynchronous (can't get return values)
❌ JSON serialization overhead
❌ Can't share memory
❌ Batching required
❌ Bridge is single-threaded bottleneck

Example:
const width = NativeModules.UIManager.measureView(viewRef);
// ❌ Can't do this! Bridge is async
// Must use callback: measureView(viewRef, (width) => {})
```

---

#### New Architecture Components

**1. JSI (JavaScript Interface)**

**Direct JS ↔ Native Communication:**

```javascript
// New Architecture with JSI

[JavaScript]  <-- JSI (Sync) -->  [Native (C++)]
- Direct calls                    - Direct calls
- Shared memory                   - Shared memory
- No serialization                - No serialization

JSI allows:
✅ Synchronous calls
✅ Direct function invocation
✅ Shared memory (ArrayBuffer)
✅ No JSON serialization
✅ Type safety

Example:
// Can now do synchronous calls!
const width = NativeModules.UIManager.measureView(viewRef);
console.log(width); // Immediate result!

// Under the hood:
// 1. JS calls C++ function directly (via JSI)
// 2. C++ executes native code
// 3. Returns value immediately
// No bridge, no serialization!
```

**JSI Benefits:**

```javascript
// Old: Async callback hell
NativeModules.Camera.takePicture((error, photo) => {
  if (error) {
    console.error(error);
  } else {
    NativeModules.Storage.saveImage(photo, (error, path) => {
      if (error) {
        console.error(error);
      } else {
        console.log('Saved:', path);
      }
    });
  }
});

// New: Synchronous (or async/await)
const photo = await NativeModules.Camera.takePicture();
const path = await NativeModules.Storage.saveImage(photo);
console.log('Saved:', path);

// Performance comparison:
// Old: 3-5ms per bridge crossing
// New: ~0.1ms direct call (30-50x faster!)
```

---

**2. Fabric (New Renderer)**

**Direct UI Operations:**

```javascript
// Old Architecture: Bridge-based rendering

JS: Update state
  ↓
React: Calculate diff
  ↓
Bridge: Send JSON commands
  ↓
Native: Parse JSON
  ↓
Native: Update views

Total: 16-50ms (causes jank at 60 FPS)

// New Architecture: Fabric renderer

JS: Update state
  ↓
React: Calculate diff
  ↓
JSI: Direct C++ calls
  ↓
Native: Update views

Total: 2-8ms (smooth 60 FPS)
```

**Fabric Features:**

```javascript
// 1. Synchronous Layout
// Old: Can't get layout info synchronously
componentDidMount() {
  // ❌ Requires callback
  this.view.measure((x, y, width, height) => {
    console.log('Layout:', width, height);
  });
}

// New: Synchronous layout
componentDidMount() {
  // ✅ Immediate result
  const { width, height } = this.view.getBoundingClientRect();
  console.log('Layout:', width, height);
}

// 2. Concurrent Rendering
// Fabric enables React 18 features:
import { startTransition } from 'react';

function App() {
  const [query, setQuery] = useState('');
  
  const handleChange = (text) => {
    // High priority: Update input immediately
    setQuery(text);
    
    // Low priority: Update search results
    startTransition(() => {
      setSearchResults(search(text));
    });
  };
  
  // Input stays responsive even during heavy search!
}

// 3. Improved Animations
// Fabric + JSI enable smooth 120 FPS on ProMotion displays
Animated.timing(value, {
  toValue: 100,
  duration: 1000,
  useNativeDriver: true, // Even better with Fabric!
}).start();
```

---

**3. TurboModules (New Native Modules)**

**Lazy Loading & Type Safety:**

```javascript
// Old: All native modules loaded at startup
// ❌ App startup: 2-3 seconds
// Even if you never use the module!

// New: TurboModules load on-demand
// ✅ App startup: 0.5-1 second
// Modules loaded when first used

import { TurboModuleRegistry } from 'react-native';

// Loads only when called
const CameraModule = TurboModuleRegistry.get('Camera');

// Type-safe (TypeScript/Flow)
interface CameraModule {
  takePicture(): Promise<string>;
  hasPermission(): boolean;
}

const camera: CameraModule = TurboModuleRegistry.get('Camera');
const photo = await camera.takePicture(); // Type-checked!
```

**Creating a TurboModule:**

```typescript
// NativeCamera.ts (JavaScript side)
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  takePicture(quality: number): Promise<string>;
  hasPermission(): boolean;
}

export default TurboModuleRegistry.get<Spec>('Camera');
```

```cpp
// NativeCamera.cpp (C++ implementation)
#include <jsi/jsi.h>

namespace facebook::react {

class NativeCamera : public NativeCameraSpec {
public:
  std::string takePicture(double quality) override {
    // Direct C++ implementation
    return captureCameraImage(quality);
  }
  
  bool hasPermission() override {
    return checkCameraPermission();
  }
};

} // namespace facebook::react
```

---

#### Performance Improvements

**Benchmarks:**

```
Metric                  Old Arch    New Arch    Improvement
─────────────────────────────────────────────────────────────
App Startup             2.5s        0.8s        3x faster
Bridge Calls            5ms         0.1ms       50x faster
List Scrolling          45 FPS      60 FPS      Smooth
Animations              40 FPS      120 FPS     ProMotion
Memory Usage            150MB       100MB       33% less
Bundle Size             No change   No change   Same
```

**Real-World Example:**

```javascript
// Smooth 60 FPS list with images
function ProductList({ products }) {
  return (
    <FlatList
      data={products}
      renderItem={({ item }) => (
        <View>
          <Image source={{ uri: item.image }} />
          <Text>{item.name}</Text>
          <Text>${item.price}</Text>
        </View>
      )}
      // Old: Drops to 30-40 FPS when scrolling fast
      // New: Solid 60 FPS even with complex items
    />
  );
}
```

---

#### Enabling New Architecture

**React Native 0.70+:**

```bash
# Create new app with New Architecture
npx react-native init MyApp --template react-native@latest
```

**Android (android/gradle.properties):**

```properties
# Enable New Architecture
newArchEnabled=true

# Enable Hermes (recommended with New Architecture)
hermesEnabled=true
```

**iOS (ios/Podfile):**

```ruby
use_react_native!(
  :path => config[:reactNativePath],
  :fabric_enabled => true,          # Enable Fabric
  :new_arch_enabled => true,        # Enable New Architecture
  :hermes_enabled => true           # Enable Hermes
)
```

**Rebuild:**

```bash
# iOS
cd ios
pod install
cd ..
npx react-native run-ios

# Android
npx react-native run-android
```

---

#### Migration Considerations

**Breaking Changes:**

```javascript
// 1. NativeModules become TurboModules
// Old:
import { NativeModules } from 'react-native';
const { MyModule } = NativeModules;

// New: Still works, but migrate to TurboModules
import { TurboModuleRegistry } from 'react-native';
const MyModule = TurboModuleRegistry.get('MyModule');

// 2. Native components need Fabric compatibility
// ViewManager classes need updates

// 3. Some community libraries not yet compatible
// Check compatibility: https://reactnative.directory/
```

**Compatibility:**

```javascript
// New Architecture is backward compatible
// Old bridge still available during transition

// Interop layer allows mixing:
// ✅ TurboModules + Old native modules
// ✅ Fabric components + Old components
// ✅ Gradual migration

// Eventually:
// Bridge will be removed in future versions
// All modules must migrate to New Architecture
```

---

#### Architecture Comparison

**Old Architecture:**

```
┌─────────────────┐
│  JavaScript     │
│  (React)        │
└────────┬────────┘
         │
    JSON over
   async bridge
         │
┌────────▼────────┐
│  Native         │
│  (iOS/Android)  │
└─────────────────┘

Limitations:
❌ Async only
❌ Serialization overhead
❌ No concurrent rendering
❌ All modules loaded at startup
```

**New Architecture:**

```
┌─────────────────┐
│  JavaScript     │
│  (React)        │
└────────┬────────┘
         │
    JSI (sync/async)
    Direct C++ calls
         │
┌────────▼────────┐
│  C++ Layer      │
│  (Fabric/Turbo) │
└────────┬────────┘
         │
┌────────▼────────┐
│  Native         │
│  (iOS/Android)  │
└─────────────────┘

Benefits:
✅ Sync & async
✅ Direct calls (no serialization)
✅ Concurrent rendering
✅ Lazy module loading
✅ Type safety
✅ Shared memory
```

---

#### Future Benefits

**Concurrent React Features:**

```javascript
import { useTransition, useDeferredValue } from 'react';

// Enabled by Fabric renderer
function SearchApp() {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();
  
  const handleChange = (text) => {
    setQuery(text);
    startTransition(() => {
      // Low priority update
      fetchResults(text);
    });
  };
  
  return (
    <View>
      <TextInput
        value={query}
        onChangeText={handleChange}
        // Stays responsive!
      />
      {isPending && <ActivityIndicator />}
      <Results />
    </View>
  );
}
```

**Server Components (Experimental):**

```javascript
// Future: React Server Components on mobile
// Components render on server, stream to mobile
// Reduces bundle size, faster initial load
```

---

#### Interview Tips

✅ **Key Points:**
- New Architecture replaces async bridge with synchronous JSI
- JSI enables direct JS ↔ Native communication (50x faster)
- Fabric is new renderer enabling concurrent React features
- TurboModules load lazily (faster app startup)
- Performance: 3x faster startup, 60-120 FPS animations
- Backward compatible with gradual migration
- Enabled by default in React Native 0.70+
- Breaking changes: Native modules need updates

✅ **When to Mention:**
- React Native architecture discussions
- Performance optimization questions
- Bridge bottleneck problems
- Concurrent React features
- React Native roadmap

✅ **Common Follow-ups:**
- "What is JSI?"
- "How does it improve performance?"
- "What is Fabric?"
- "What are TurboModules?"
- "Is it production-ready?"
- "How do I migrate?"

✅ **Perfect Answer Structure:**
1. Problem: Old bridge was async bottleneck
2. JSI: Direct synchronous JS ↔ Native calls
3. Fabric: New renderer for concurrent features
4. TurboModules: Lazy loading native modules
5. Performance: 3x startup, 50x faster calls, 60-120 FPS
6. Migration: Backward compatible, gradual adoption
7. Example: Show sync call or concurrent rendering

</details>

---

### 219. What is Hermes JavaScript engine?

<details>
<summary>View Answer</summary>

**Hermes** is an open-source JavaScript engine optimized for React Native on Android and iOS. Developed by Meta (Facebook), it dramatically improves app startup time, reduces memory usage, and decreases app size through ahead-of-time (AOT) compilation and bytecode optimization. It's the default engine in React Native 0.70+.

#### What is Hermes?

**JavaScript Engine Comparison:**

```javascript
// JavaScript engines execute your React Native code

// Traditional engines (JavaScriptCore, V8):
// 1. Parse JavaScript source code
// 2. Compile to bytecode (JIT - Just In Time)
// 3. Execute bytecode
// 4. Optimize hot code paths at runtime

// Hermes:
// 1. Compile to bytecode ahead of time (AOT)
// 2. Ship bytecode in app bundle
// 3. Execute bytecode immediately (no parse/compile)
// 4. Smaller footprint, faster startup

| Engine          | Used By              | Compilation | Optimization |
|-----------------|---------------------|-------------|---------------|
| JavaScriptCore  | iOS Safari, old RN  | JIT         | Runtime |
| V8              | Chrome, Node.js     | JIT         | Runtime |
| Hermes          | React Native        | AOT         | Build time |
```

---

#### Key Benefits

**1. Faster App Startup:**

```bash
# Without Hermes:
# App launch → Parse JS → Compile → Execute
# Time: 2-4 seconds

# With Hermes:
# App launch → Execute bytecode directly
# Time: 0.5-1 second

# Performance gains:
# - 50-70% faster startup on Android
# - 30-50% faster startup on iOS
# - Especially noticeable on low-end devices
```

**Real Numbers:**

```
Device              Without Hermes    With Hermes    Improvement
────────────────────────────────────────────────────────────────
Pixel 4            3.2s              0.9s           3.5x faster
iPhone 12          2.1s              1.0s           2.1x faster
Low-end Android    5.8s              1.5s           3.9x faster
```

---

**2. Reduced Memory Usage:**

```javascript
// Hermes uses less memory than JSC/V8

// Memory footprint:
// JavaScriptCore: ~150MB baseline
// V8: ~180MB baseline
// Hermes: ~80MB baseline

// 40-50% less memory usage!
// Critical for:
// - Low-end devices
// - Apps with many screens
// - Memory-intensive operations

// Garbage Collection:
// Hermes: Generational GC (like V8)
// - Young generation: Frequent, fast
// - Old generation: Infrequent, thorough
// - Less GC pauses
```

---

**3. Smaller App Size:**

```bash
# APK/IPA size comparison:

# Without Hermes:
# - Includes JavaScript engine
# - Ships .js bundle
# - Size: ~8-10MB for engine + JS

# With Hermes:
# - Hermes engine smaller
# - Ships .hbc (bytecode) - smaller than .js
# - Size: ~4-6MB for engine + bytecode

# App size reduction:
# Android: 20-40% smaller APK
# iOS: 10-20% smaller IPA

# Example:
# Without Hermes: 45MB APK
# With Hermes: 32MB APK (29% smaller)
```

---

#### How Hermes Works

**AOT Compilation:**

```bash
# Build process with Hermes:

1. Write React Native code:
   App.js, components, etc.

2. Metro bundler creates JavaScript bundle:
   index.bundle.js (all code combined)

3. Hermes compiler (hermesc) compiles to bytecode:
   hermesc -emit-binary -out index.hbc index.bundle.js

4. Bytecode shipped in app:
   - Android: assets/index.android.bundle.hbc
   - iOS: main.jsbundle.hbc

5. At runtime:
   - Hermes VM loads bytecode
   - Executes immediately (no parsing/compilation)
   - Fast startup!
```

**Bytecode Format:**

```javascript
// Source JavaScript:
function add(a, b) {
  return a + b;
}

const result = add(5, 3);

// Hermes bytecode (conceptual):
// Function 'add' at offset 0:
//   LoadParam 0 -> r0   // Load 'a'
//   LoadParam 1 -> r1   // Load 'b'
//   Add r0, r1 -> r2    // a + b
//   Ret r2              // Return result
//
// Main:
//   LoadConst 5 -> r0
//   LoadConst 3 -> r1
//   Call 'add', r0, r1 -> r2
//   StoreGlobal 'result', r2

// Bytecode is:
// - Pre-compiled (no runtime parsing)
// - Optimized (constant folding, dead code elimination)
// - Compact (smaller than source)
```

---

#### Enabling Hermes

**React Native 0.70+ (Default):**

```bash
# Hermes enabled by default!
npx react-native init MyApp
# Already using Hermes
```

**Android (android/gradle.properties):**

```properties
# Enable Hermes
hermesEnabled=true
```

**iOS (ios/Podfile):**

```ruby
use_react_native!(
  :path => config[:reactNativePath],
  :hermes_enabled => true
)
```

**Rebuild:**

```bash
# Android
cd android
./gradlew clean
cd ..
npx react-native run-android

# iOS
cd ios
pod install
cd ..
npx react-native run-ios
```

**Verify Hermes is Running:**

```javascript
import { Platform } from 'react-native';

// Check if Hermes is enabled
const isHermes = () => {
  return !!global.HermesInternal;
};

console.log('Hermes enabled:', isHermes());
// true = Hermes
// false = JavaScriptCore or other engine

// Or check global object:
if (global.HermesInternal) {
  console.log('Running on Hermes!');
  console.log('Version:', global.HermesInternal.getRuntimeProperties?.()?.['OSS Release Version']);
}
```

---

#### Hermes Features

**1. Bytecode Caching:**

```javascript
// Hermes caches compiled bytecode
// Second app launch even faster:
// - First launch: 1.0s
// - Subsequent: 0.5s

// No need to re-compile
```

**2. Debugging Support:**

```bash
# Chrome DevTools (via Flipper or standalone)
# - Breakpoints
# - Step through code
# - Inspect variables
# - Console logging

# Source maps work correctly:
# Bytecode → Source map → Original JavaScript
```

**3. Profiling:**

```javascript
// Hermes has built-in profiler
// Records:
// - Function calls
// - Execution time
// - Memory allocation

// Enable profiling:
// - In app: global.HermesInternal.enableSamplingProfiler();
// - Via CLI flags

// Export profile:
const profile = global.HermesInternal.dumpSamplingProfiler();
// Analyze in Chrome DevTools
```

**4. ES6+ Support:**

```javascript
// Hermes supports modern JavaScript:

// ✅ Arrow functions
const add = (a, b) => a + b;

// ✅ Classes
class MyComponent extends Component {}

// ✅ Template literals
const message = `Hello ${name}`;

// ✅ Destructuring
const { id, name } = user;

// ✅ Spread operator
const newArray = [...oldArray, newItem];

// ✅ Async/await
async function fetchData() {
  const response = await fetch(url);
}

// ✅ Optional chaining
const userName = user?.profile?.name;

// ✅ Nullish coalescing
const value = input ?? defaultValue;

// Most ES6+ features supported!
```

---

#### Limitations

**1. No eval() or Function():**

```javascript
// ❌ Not supported: Dynamic code generation
eval('console.log("Hello")'); // Throws error!

const dynamicFunction = new Function('a', 'b', 'return a + b');
// Throws error!

// Why: Bytecode is pre-compiled
// Can't compile new code at runtime

// Workaround: Avoid eval/Function
// Use proper JavaScript patterns instead
```

**2. Proxies (Limited):**

```javascript
// Basic Proxy support exists
// But some advanced features missing

const handler = {
  get(target, prop) {
    return target[prop];
  },
};

const proxy = new Proxy({ name: 'John' }, handler);
console.log(proxy.name); // Works

// Some advanced Proxy features may not work
// Check Hermes docs for current support
```

**3. WeakMap/WeakSet (Full support in newer versions):**

```javascript
// Early Hermes versions had limited WeakMap/WeakSet
// Now fully supported in Hermes 0.12+

const weakMap = new WeakMap();
const obj = {};
weakMap.set(obj, 'value'); // Works!
```

---

#### Hermes vs Other Engines

**Performance Comparison:**

```
Metric              JavaScriptCore  V8      Hermes
────────────────────────────────────────────────────────
Startup Time        2.5s            2.8s    0.9s
Memory Usage        150MB           180MB   80MB
App Size            45MB            48MB    32MB
Runtime Speed       Fast            Fastest Medium
GC Pauses           Medium          Short   Short
Low-end Devices     Slow            Slower  Fast
Battery Impact      Medium          Higher  Lower
```

**When to Use:**

```javascript
// ✅ Use Hermes (default choice):
// - Most React Native apps
// - Startup time important
// - Low-end device support
// - Memory constraints
// - Smaller app size needed

// ⚠️ Consider JSC/V8:
// - Need eval() or Function()
// - Heavy computation (math/crypto)
// - Advanced Proxy usage
// - Specific engine features needed

// For 99% of apps: Hermes is the best choice!
```

---

#### Real-World Example

**Before Hermes (JSC):**

```bash
# App: E-commerce with 50 screens
# Device: Mid-range Android

Metrics:
- Cold start: 3.5 seconds
- Memory: 180MB baseline
- APK size: 52MB
- Smooth scrolling: Sometimes janky
- Low-end devices: Very slow (6-8s startup)
```

**After Hermes:**

```bash
# Same app, same device

Metrics:
- Cold start: 1.2 seconds (66% faster!)
- Memory: 95MB baseline (47% less!)
- APK size: 38MB (27% smaller!)
- Smooth scrolling: Always smooth
- Low-end devices: 2-3s startup (acceptable!)

# User experience improvement:
# - Users more likely to use app (faster)
# - Less uninstalls due to size
# - Better reviews (smoother experience)
```

---

#### Debugging with Hermes

**Chrome DevTools:**

```bash
# Method 1: Flipper (recommended)
npx react-native run-android
# Open Flipper
# Hermes Debugger plugin available

# Method 2: Direct Chrome debugging
# 1. Enable debugging in dev menu
# 2. Open chrome://inspect
# 3. Find Hermes target
# 4. Click "inspect"

# Features:
# ✅ Breakpoints work
# ✅ Step through code
# ✅ Console logging
# ✅ Network inspection
# ✅ Performance profiling
```

**Profiling:**

```javascript
// Enable sampling profiler
if (global.HermesInternal) {
  global.HermesInternal.enableSamplingProfiler();
  
  // Do work...
  heavyOperation();
  
  // Dump profile
  const profile = global.HermesInternal.dumpSamplingProfiler();
  
  // Save to file
  import RNFS from 'react-native-fs';
  const path = `${RNFS.DocumentDirectoryPath}/profile.json`;
  await RNFS.writeFile(path, JSON.stringify(profile));
  
  // Analyze in Chrome DevTools:
  // 1. Open chrome://inspect
  // 2. Performance tab
  // 3. Load profile.json
}
```

---

#### Migration Tips

**Enabling Hermes:**

```bash
# 1. Update React Native to 0.70+
npm install react-native@latest

# 2. Enable Hermes (if not default)
# android/gradle.properties
hermesEnabled=true

# ios/Podfile
:hermes_enabled => true

# 3. Rebuild
cd android && ./gradlew clean && cd ..
cd ios && pod install && cd ..

# 4. Test thoroughly
# - Check all features work
# - Monitor crash reports
# - Test on various devices
```

**Common Issues:**

```javascript
// Issue 1: eval() usage
// Solution: Refactor to avoid eval()

// ❌ Bad:
const result = eval(`return ${userInput}`);

// ✅ Good:
const result = JSON.parse(userInput);

// Issue 2: Large bytecode
// Solution: Code splitting, lazy loading

// Issue 3: Third-party libraries using eval
// Solution: Find alternative library or wait for update
```

---

#### Interview Tips

✅ **Key Points:**
- Hermes is JavaScript engine optimized for React Native
- AOT compilation: Compiles to bytecode at build time (not runtime)
- 50-70% faster startup, 40-50% less memory, 20-40% smaller apps
- Default in React Native 0.70+
- Limitations: No eval(), limited Proxy support initially
- Excellent for low-end devices and performance-critical apps
- Supports modern JavaScript (ES6+, async/await, optional chaining)
- Built-in profiling and debugging support

✅ **When to Mention:**
- React Native performance discussions
- App startup time optimization
- Memory usage reduction
- JavaScript engine comparisons
- Low-end device support

✅ **Common Follow-ups:**
- "How does Hermes improve startup time?"
- "What's the difference between AOT and JIT?"
- "What are Hermes limitations?"
- "Should I always use Hermes?"
- "How do I enable Hermes?"

✅ **Perfect Answer Structure:**
1. Definition: JS engine optimized for React Native with AOT compilation
2. Benefits: 50-70% faster startup, 40-50% less memory, smaller apps
3. How it works: Compiles to bytecode at build time, executes directly
4. Limitations: No eval(), some advanced features initially limited
5. When to use: Most React Native apps (default choice)
6. Example: Show startup time improvement numbers

</details>

---

### 220. How do you implement native modules in React Native?

<details>
<summary>View Answer</summary>

**Native modules** allow React Native apps to access platform-specific functionality not available in JavaScript (Bluetooth, biometrics, custom UI, etc.). You write native code in Swift/Objective-C (iOS) or Java/Kotlin (Android) and expose it to JavaScript. With the New Architecture, TurboModules provide better performance and type safety.

#### Why Native Modules?

**Use Cases:**

```javascript
// Scenarios requiring native modules:

// 1. Platform-specific APIs
// - Bluetooth Low Energy
// - NFC
// - Biometric authentication
// - HealthKit (iOS) / Google Fit (Android)
// - Custom camera features

// 2. Performance-critical operations
// - Image/video processing
// - Encryption/decryption
// - Complex calculations

// 3. Existing native libraries
// - Third-party SDKs (payment, analytics)
// - Proprietary native code

// 4. Custom native UI
// - Complex custom views
// - Native animations
```

---

#### Creating Native Module (Old Architecture)

**iOS (Swift/Objective-C)**

**Step 1: Create Swift Module:**

```swift
// ios/CalendarModule.swift
import Foundation

@objc(CalendarModule)
class CalendarModule: NSObject {
  
  // Expose to JavaScript
  @objc
  func createEvent(
    _ name: String,
    location: String,
    date: NSNumber,
    callback: @escaping RCTResponseSenderBlock
  ) {
    // Native implementation
    let eventId = UUID().uuidString
    
    // Simulate creating calendar event
    print("Creating event: \(name) at \(location) on \(date)")
    
    // Return to JavaScript
    callback([NSNull(), eventId])
    // callback([error, result])
  }
  
  // Synchronous method
  @objc
  func getName() -> String {
    return "CalendarModule"
  }
  
  // Promise-based method
  @objc
  func createEventAsync(
    _ name: String,
    location: String,
    date: NSNumber,
    resolve: @escaping RCTPromiseResolveBlock,
    reject: @escaping RCTPromiseRejectBlock
  ) {
    let eventId = UUID().uuidString
    
    if name.isEmpty {
      reject("ERROR", "Event name cannot be empty", nil)
    } else {
      resolve(eventId)
    }
  }
  
  // Constants exported to JS
  @objc
  func constantsToExport() -> [String: Any] {
    return [
      "DEFAULT_EVENT_NAME": "New Event",
      "MAX_ATTENDEES": 100
    ]
  }
  
  // Required for RCTBridgeModule
  @objc
  static func requiresMainQueueSetup() -> Bool {
    return false // true if needs to run on main thread
  }
}
```

**Step 2: Objective-C Bridge Header:**

```objc
// ios/CalendarModule.m
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(CalendarModule, NSObject)

// Expose methods to JavaScript
RCT_EXTERN_METHOD(
  createEvent:(NSString *)name
  location:(NSString *)location
  date:(nonnull NSNumber *)date
  callback:(RCTResponseSenderBlock)callback
)

RCT_EXTERN_METHOD(
  getName
)

RCT_EXTERN_METHOD(
  createEventAsync:(NSString *)name
  location:(NSString *)location
  date:(nonnull NSNumber *)date
  resolve:(RCTPromiseResolveBlock)resolve
  reject:(RCTPromiseRejectBlock)reject
)

@end
```

---

**Android (Kotlin/Java)**

**Step 1: Create Kotlin Module:**

```kotlin
// android/app/src/main/java/com/myapp/CalendarModule.kt
package com.myapp

import com.facebook.react.bridge.*
import java.util.UUID

class CalendarModule(reactContext: ReactApplicationContext) :
  ReactContextBaseJavaModule(reactContext) {
  
  // Module name exposed to JavaScript
  override fun getName(): String {
    return "CalendarModule"
  }
  
  // Callback-based method
  @ReactMethod
  fun createEvent(
    name: String,
    location: String,
    date: Double,
    callback: Callback
  ) {
    val eventId = UUID.randomUUID().toString()
    
    // Simulate creating event
    println("Creating event: $name at $location on $date")
    
    // Return to JavaScript: callback(error, result)
    callback.invoke(null, eventId)
  }
  
  // Promise-based method
  @ReactMethod
  fun createEventAsync(
    name: String,
    location: String,
    date: Double,
    promise: Promise
  ) {
    try {
      if (name.isEmpty()) {
        promise.reject("ERROR", "Event name cannot be empty")
        return
      }
      
      val eventId = UUID.randomUUID().toString()
      promise.resolve(eventId)
    } catch (e: Exception) {
      promise.reject("ERROR", e.message, e)
    }
  }
  
  // Export constants to JS
  override fun getConstants(): Map<String, Any> {
    return mapOf(
      "DEFAULT_EVENT_NAME" to "New Event",
      "MAX_ATTENDEES" to 100
    )
  }
}
```

**Step 2: Register Module:**

```kotlin
// android/app/src/main/java/com/myapp/CalendarPackage.kt
package com.myapp

import com.facebook.react.ReactPackage
import com.facebook.react.bridge.NativeModule
import com.facebook.react.bridge.ReactApplicationContext
import com.facebook.react.uimanager.ViewManager

class CalendarPackage : ReactPackage {
  override fun createNativeModules(
    reactContext: ReactApplicationContext
  ): List<NativeModule> {
    return listOf(CalendarModule(reactContext))
  }
  
  override fun createViewManagers(
    reactContext: ReactApplicationContext
  ): List<ViewManager<*, *>> {
    return emptyList()
  }
}
```

**Step 3: Add to Application:**

```kotlin
// android/app/src/main/java/com/myapp/MainApplication.kt
package com.myapp

import android.app.Application
import com.facebook.react.PackageList
import com.facebook.react.ReactApplication
import com.facebook.react.ReactNativeHost
import com.facebook.react.ReactPackage

class MainApplication : Application(), ReactApplication {
  
  override val reactNativeHost: ReactNativeHost =
    object : ReactNativeHost(this) {
      override fun getPackages(): List<ReactPackage> {
        val packages = PackageList(this).packages.toMutableList()
        // Add custom package
        packages.add(CalendarPackage())
        return packages
      }
    }
}
```

---

#### Using Native Module in JavaScript

```javascript
// JavaScript usage
import { NativeModules } from 'react-native';

const { CalendarModule } = NativeModules;

// 1. Callback-based
function createEvent() {
  CalendarModule.createEvent(
    'Birthday Party',
    'My House',
    Date.now(),
    (error, eventId) => {
      if (error) {
        console.error(error);
      } else {
        console.log('Event created:', eventId);
      }
    }
  );
}

// 2. Promise-based (better!)
async function createEventAsync() {
  try {
    const eventId = await CalendarModule.createEventAsync(
      'Birthday Party',
      'My House',
      Date.now()
    );
    console.log('Event created:', eventId);
  } catch (error) {
    console.error('Failed to create event:', error);
  }
}

// 3. Access constants
const { DEFAULT_EVENT_NAME, MAX_ATTENDEES } = CalendarModule;
console.log('Default name:', DEFAULT_EVENT_NAME);
console.log('Max attendees:', MAX_ATTENDEES);
```

---

#### TurboModules (New Architecture)

**Better Performance & Type Safety:**

```typescript
// 1. Define TypeScript interface
// NativeCalendar.ts
import type { TurboModule } from 'react-native';
import { TurboModuleRegistry } from 'react-native';

export interface Spec extends TurboModule {
  createEvent(
    name: string,
    location: string,
    date: number
  ): Promise<string>;
  
  getConstants(): {
    DEFAULT_EVENT_NAME: string;
    MAX_ATTENDEES: number;
  };
}

export default TurboModuleRegistry.get<Spec>('CalendarModule');
```

**Benefits:**

```javascript
import CalendarModule from './NativeCalendar';

// ✅ Type-safe calls
const eventId = await CalendarModule.createEvent(
  'Party', // TypeScript ensures this is string
  'Home',
  Date.now() // TypeScript ensures this is number
);

// ❌ TypeScript error if wrong types
await CalendarModule.createEvent(123, true, 'invalid');
// Error: Expected string, got number

// ✅ Lazy loading (loaded on first use)
// Faster app startup!

// ✅ Direct C++ calls via JSI
// 50x faster than old bridge
```

---

#### Sending Events to JavaScript

**iOS (Event Emitter):**

```swift
// ios/CalendarModule.swift
import Foundation

@objc(CalendarModule)
class CalendarModule: RCTEventEmitter {
  
  override func supportedEvents() -> [String] {
    return ["EventCreated", "EventDeleted"]
  }
  
  @objc
  func createEvent(
    _ name: String,
    location: String,
    date: NSNumber
  ) {
    let eventId = UUID().uuidString
    
    // Send event to JavaScript
    sendEvent(
      withName: "EventCreated",
      body: [
        "eventId": eventId,
        "name": name,
        "location": location
      ]
    )
  }
  
  override static func requiresMainQueueSetup() -> Bool {
    return false
  }
}
```

**Android (Event Emitter):**

```kotlin
// android/app/src/main/java/com/myapp/CalendarModule.kt
class CalendarModule(reactContext: ReactApplicationContext) :
  ReactContextBaseJavaModule(reactContext) {
  
  override fun getName() = "CalendarModule"
  
  private fun sendEvent(eventName: String, params: WritableMap?) {
    reactApplicationContext
      .getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter::class.java)
      .emit(eventName, params)
  }
  
  @ReactMethod
  fun createEvent(name: String, location: String, date: Double) {
    val eventId = UUID.randomUUID().toString()
    
    // Send event to JavaScript
    val params = Arguments.createMap().apply {
      putString("eventId", eventId)
      putString("name", name)
      putString("location", location)
    }
    
    sendEvent("EventCreated", params)
  }
}
```

**JavaScript (Listen to Events):**

```javascript
import { NativeModules, NativeEventEmitter } from 'react-native';

const { CalendarModule } = NativeModules;
const calendarEmitter = new NativeEventEmitter(CalendarModule);

function MyComponent() {
  useEffect(() => {
    // Subscribe to event
    const subscription = calendarEmitter.addListener(
      'EventCreated',
      (event) => {
        console.log('Event created:', event.eventId);
        console.log('Name:', event.name);
      }
    );
    
    // Cleanup
    return () => subscription.remove();
  }, []);
  
  return <View>...</View>;
}
```

---

#### Threading

**iOS:**

```swift
@objc(CalendarModule)
class CalendarModule: NSObject {
  
  // Run on background thread (default)
  @objc
  static func requiresMainQueueSetup() -> Bool {
    return false
  }
  
  // Force main thread
  @objc
  func updateUI() {
    DispatchQueue.main.async {
      // UI updates here
    }
  }
}
```

**Android:**

```kotlin
class CalendarModule(reactContext: ReactApplicationContext) :
  ReactContextBaseJavaModule(reactContext) {
  
  @ReactMethod
  fun heavyOperation(promise: Promise) {
    // Runs on background thread automatically
    Thread.sleep(1000)
    promise.resolve("Done")
  }
  
  @ReactMethod(isBlockingSynchronousMethod = true)
  fun synchronousMethod(): String {
    // Runs synchronously (blocks JS thread)
    return "Result"
  }
}
```

---

#### Complete Example: Bluetooth Module

**iOS Implementation:**

```swift
// ios/BluetoothModule.swift
import CoreBluetooth

@objc(BluetoothModule)
class BluetoothModule: RCTEventEmitter, CBCentralManagerDelegate {
  
  private var centralManager: CBCentralManager!
  
  override init() {
    super.init()
    centralManager = CBCentralManager(delegate: self, queue: nil)
  }
  
  override func supportedEvents() -> [String] {
    return ["DeviceFound", "DeviceConnected"]
  }
  
  @objc
  func startScanning(_ resolve: @escaping RCTPromiseResolveBlock,
                     reject: @escaping RCTPromiseRejectBlock) {
    if centralManager.state == .poweredOn {
      centralManager.scanForPeripherals(withServices: nil)
      resolve(true)
    } else {
      reject("BLUETOOTH_ERROR", "Bluetooth not available", nil)
    }
  }
  
  @objc
  func stopScanning() {
    centralManager.stopScan()
  }
  
  // CBCentralManagerDelegate
  func centralManager(
    _ central: CBCentralManager,
    didDiscover peripheral: CBPeripheral,
    advertisementData: [String: Any],
    rssi RSSI: NSNumber
  ) {
    sendEvent(
      withName: "DeviceFound",
      body: [
        "id": peripheral.identifier.uuidString,
        "name": peripheral.name ?? "Unknown",
        "rssi": RSSI
      ]
    )
  }
  
  func centralManagerDidUpdateState(_ central: CBCentralManager) {
    // Handle state changes
  }
  
  override static func requiresMainQueueSetup() -> Bool {
    return true // Bluetooth needs main thread
  }
}
```

**JavaScript Usage:**

```javascript
import { NativeModules, NativeEventEmitter } from 'react-native';

const { BluetoothModule } = NativeModules;
const bluetoothEmitter = new NativeEventEmitter(BluetoothModule);

function BluetoothScanner() {
  const [devices, setDevices] = useState([]);
  
  useEffect(() => {
    const subscription = bluetoothEmitter.addListener(
      'DeviceFound',
      (device) => {
        setDevices(prev => [...prev, device]);
      }
    );
    
    return () => subscription.remove();
  }, []);
  
  const startScan = async () => {
    try {
      await BluetoothModule.startScanning();
      console.log('Scanning started');
    } catch (error) {
      console.error('Failed to start scanning:', error);
    }
  };
  
  const stopScan = () => {
    BluetoothModule.stopScanning();
  };
  
  return (
    <View>
      <Button title="Start Scan" onPress={startScan} />
      <Button title="Stop Scan" onPress={stopScan} />
      <FlatList
        data={devices}
        renderItem={({ item }) => (
          <Text>{item.name} (RSSI: {item.rssi})</Text>
        )}
      />
    </View>
  );
}
```

---

#### Interview Tips

✅ **Key Points:**
- Native modules expose platform-specific functionality to JavaScript
- Write in Swift/Objective-C (iOS) or Java/Kotlin (Android)
- Methods can use callbacks, promises, or be synchronous
- Event emitters send data from native to JavaScript
- TurboModules (New Architecture) provide type safety and better performance
- Use for: Bluetooth, biometrics, custom native libraries
- Threading: Background by default, can specify main thread
- Constants exported once at initialization

✅ **When to Mention:**
- Platform-specific feature discussions
- Third-party SDK integration
- Performance-critical operations
- Accessing native APIs
- Custom native UI components

✅ **Common Follow-ups:**
- "When do you need a native module?"
- "What's the difference between callbacks and promises?"
- "How do you send events from native to JS?"
- "What are TurboModules?"
- "How do you handle threading?"

✅ **Perfect Answer Structure:**
1. Purpose: Access platform-specific functionality from JavaScript
2. Implementation: Write native code, expose to JS
3. Methods: Callbacks, promises, synchronous, events
4. iOS: Swift/Objective-C with RCT bridge macros
5. Android: Kotlin/Java with @ReactMethod annotations
6. TurboModules: Type-safe, lazy-loaded, faster
7. Example: Show simple native module with promise-based method

</details>
