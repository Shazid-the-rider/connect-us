# Step-by-Step Guide: Installing NativeWind in React Native (Expo)

This guide will walk you through installing and configuring NativeWind (Tailwind CSS for React Native) in your Expo project.

## Prerequisites

- An existing Expo project (created with `create-expo-app`)
- Node.js (v16 or higher)
- npm or yarn

---

## Step 1: Install NativeWind and Tailwind CSS

Install NativeWind and Tailwind CSS as dependencies:

```bash
npm install nativewind tailwindcss
```

**Note:** For NativeWind v4, you need both `nativewind` and `tailwindcss` packages.

---

## Step 2: Create Tailwind Configuration File

Create a `tailwind.config.js` file in the root of your project:

```bash
touch tailwind.config.js
```

Add the following configuration:

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./App.{js,jsx,ts,tsx}",
    "./src/**/*.{js,jsx,ts,tsx}"
  ],
  presets: [require("nativewind/preset")],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

**Explanation:**
- `content`: Specifies which files NativeWind should scan for class names
- `presets`: Includes NativeWind's preset configuration
- `theme.extend`: Allows you to customize Tailwind's default theme
- `plugins`: For additional Tailwind plugins (if needed)

**Important:** Make sure to include all file paths where you'll use Tailwind classes in the `content` array.

---

## Step 3: Configure Babel

Update your `babel.config.js` file to include the NativeWind Babel preset.

If you don't have a `babel.config.js` file, create it:

```bash
touch babel.config.js
```

Add or update the configuration:

```javascript
module.exports = function(api) {
  api.cache(true);
  return {
    presets: [
      "babel-preset-expo",
      "nativewind/babel",
    ],
  };
};
```

**Explanation:**
- `babel-preset-expo`: Required for Expo projects
- `nativewind/babel`: Enables NativeWind's Babel plugin to transform Tailwind classes

**Note:** The order matters - `nativewind/babel` should come after `babel-preset-expo`.

---

## Step 4: Configure Metro Bundler

Create or update `metro.config.js` in the root of your project:

```bash
touch metro.config.js
```

Add the following configuration:

```javascript
const { getDefaultConfig } = require("expo/metro-config");
const { withNativeWind } = require('nativewind/metro');

const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, { input: './global.css' });
```

**Explanation:**
- `getDefaultConfig`: Gets Expo's default Metro configuration
- `withNativeWind`: Wraps the config with NativeWind's Metro plugin
- `input: './global.css'`: Points to your global CSS file (created in next step)

---

## Step 5: Create Global CSS File

Create a `global.css` file in the root of your project:

```bash
touch global.css
```

Add the Tailwind directives:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

**Explanation:**
- These directives inject Tailwind's base styles, component classes, and utility classes
- This file is processed by Metro bundler and included in your app bundle

---

## Step 6: Import Global CSS in Your App

Import the `global.css` file in your main `App.tsx` file:

```typescript
import "./global.css";
```

**Example App.tsx:**

```typescript
import React from "react";
import { View, Text } from "react-native";
import "./global.css";

export default function App() {
  return (
    <View className="flex-1 justify-center items-center bg-white">
      <Text className="text-2xl font-bold text-gray-900">
        Hello NativeWind!
      </Text>
    </View>
  );
}
```

**Important:** The import must be at the top level of your app entry point, not inside a component.

---

## Step 7: Configure TypeScript (For TypeScript Projects)

If you're using TypeScript, create a `nativewind-env.d.ts` file in the root:

```bash
touch nativewind-env.d.ts
```

Add the following:

```typescript
/// <reference types="nativewind/types" />
```

**Note:** This file provides TypeScript type definitions for NativeWind's `className` prop.

---

## Step 8: Update TypeScript Configuration (Optional)

If you have a `tsconfig.json`, make sure to include the `nativewind-env.d.ts` file:

```json
{
  "compilerOptions": {
    // ... your existing options
  },
  "include": [
    "**/*.ts",
    "**/*.tsx",
    "nativewind-env.d.ts"
  ]
}
```

---

## Step 9: Clear Cache and Restart

After completing all the configuration steps, clear your cache and restart the development server:

```bash
# Clear Expo cache
npx expo start --clear

# Or if using npm start
npm start -- --clear
```

**Why?** Metro bundler caches files, and you need to clear it to pick up the new configuration.

---

## Step 10: Verify Installation

Test that NativeWind is working by using Tailwind classes in a component:

```typescript
import React from "react";
import { View, Text, TouchableOpacity } from "react-native";
import "./global.css";

export default function TestScreen() {
  return (
    <View className="flex-1 justify-center items-center bg-blue-500">
      <Text className="text-white text-2xl font-bold mb-4">
        NativeWind is Working! 🎉
      </Text>
      <TouchableOpacity className="bg-white px-6 py-3 rounded-lg">
        <Text className="text-blue-500 font-semibold">Click Me</Text>
      </TouchableOpacity>
    </View>
  );
}
```

If you see the styles applied correctly, NativeWind is successfully installed!

---

## Common Usage Examples

### Basic Styling

```typescript
<View className="flex-1 bg-gray-100 p-4">
  <Text className="text-2xl font-bold text-gray-900 mb-2">
    Title
  </Text>
  <Text className="text-gray-600">
    Description text
  </Text>
</View>
```

### Buttons

```typescript
<TouchableOpacity className="bg-blue-500 px-6 py-3 rounded-lg">
  <Text className="text-white font-semibold text-center">
    Button
  </Text>
</TouchableOpacity>
```

### Input Fields

```typescript
<TextInput
  className="border border-gray-300 rounded-lg px-4 py-3 bg-white"
  placeholder="Enter text"
/>
```

### Flexbox Layouts

```typescript
<View className="flex-row items-center justify-between p-4">
  <Text className="text-lg">Left</Text>
  <Text className="text-lg">Right</Text>
</View>
```

---

## Troubleshooting

### Issue: Styles not applying

**Solutions:**
1. Make sure `global.css` is imported in your `App.tsx`
2. Clear the cache: `npx expo start --clear`
3. Verify `metro.config.js` is correctly configured
4. Check that `babel.config.js` includes `nativewind/babel`
5. Ensure all file paths in `tailwind.config.js` content array are correct

### Issue: TypeScript errors with className

**Solutions:**
1. Make sure `nativewind-env.d.ts` exists and contains the reference
2. Restart your TypeScript server in your IDE
3. Verify `tsconfig.json` includes the `nativewind-env.d.ts` file

### Issue: Metro bundler errors

**Solutions:**
1. Delete `node_modules` and reinstall: `rm -rf node_modules && npm install`
2. Clear Metro cache: `npx expo start --clear`
3. Verify `metro.config.js` syntax is correct
4. Check that you're using the correct NativeWind version for your Expo SDK

### Issue: Classes not found in content

**Solutions:**
1. Add the file path to the `content` array in `tailwind.config.js`
2. Restart the development server after updating `tailwind.config.js`
3. Use the full file path pattern (e.g., `./src/**/*.{js,jsx,ts,tsx}`)

### Issue: Build fails on production

**Solutions:**
1. Ensure `global.css` is imported in your entry point
2. Verify all configuration files are committed to version control
3. Check that `nativewind` and `tailwindcss` are in `dependencies`, not `devDependencies`

---

## File Structure Summary

After installation, your project should have these files:

```
your-project/
├── App.tsx                 # Import global.css here
├── babel.config.js         # Includes nativewind/babel preset
├── metro.config.js         # Configured with withNativeWind
├── tailwind.config.js      # Tailwind configuration
├── global.css              # Tailwind directives
├── nativewind-env.d.ts    # TypeScript definitions (if using TS)
└── package.json            # Contains nativewind and tailwindcss
```

---

## Version Compatibility

- **NativeWind v4**: Works with Expo SDK 50+
- **React Native**: 0.72+
- **Expo**: ~54.0.0+

Make sure your versions are compatible. Check NativeWind's documentation for the latest compatibility information.

---

## Additional Resources

- [NativeWind Documentation](https://www.nativewind.dev/)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Expo Documentation](https://docs.expo.dev/)

---

## Quick Reference Checklist

- [ ] Install `nativewind` and `tailwindcss` packages
- [ ] Create `tailwind.config.js` with correct content paths
- [ ] Update `babel.config.js` to include `nativewind/babel`
- [ ] Create/update `metro.config.js` with `withNativeWind`
- [ ] Create `global.css` with Tailwind directives
- [ ] Import `global.css` in `App.tsx`
- [ ] Create `nativewind-env.d.ts` (for TypeScript projects)
- [ ] Clear cache and restart development server
- [ ] Test with a simple component using Tailwind classes

---

## Next Steps

Once NativeWind is installed, you can:

1. **Customize your theme** in `tailwind.config.js`
2. **Add custom colors, fonts, and spacing** in the `theme.extend` section
3. **Use Tailwind utilities** throughout your app
4. **Create reusable component styles** using Tailwind's `@apply` directive
5. **Enable dark mode** (if supported in your NativeWind version)

Happy styling! 🎨
