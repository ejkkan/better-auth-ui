---
name: better-auth-ui-react-native
description: Install and configure Better Auth UI components for React Native and Expo. Use for @better-auth-ui/react-native providers, navigation adapters, native plugin screens, theming, and optional native peers.
license: MIT
metadata:
  library: "@better-auth-ui/react-native"
  framework: react-native
---

# Better Auth UI for React Native

Use the installed package exports as the authority for the API version. This skill covers the 1.7 package structure.

This package is an incomplete native port of the HeroUI components. It reuses hooks, queries, and mutations from `@better-auth-ui/react` and view keys from `@better-auth-ui/core`.

Read the package README's feature parity section before promising a capability. Plugin surfaces and form behavior that exist on the web may be absent on native.

## Installation

Configure the Better Auth server and the Expo client integration first. Session storage belongs to the `@better-auth/expo` client, usually with `expo-secure-store`. The UI package does not persist sessions.

```bash
bun add @better-auth-ui/react-native @better-auth-ui/react @better-auth-ui/core better-auth react-native-svg @tanstack/react-query
```

Install optional native peers only for the features in use:

- `expo-image-picker` and `expo-image-manipulator` for avatar and organization logo upload.
- `expo-clipboard` for copy actions.
- `@react-native-community/datetimepicker` for date and time additional fields.
- `@react-native-community/slider` for slider additional fields.

These peers load lazily. The package imports safely without them, and the affected control degrades.

Components style themselves with React Native styles. Do not add NativeWind, Tailwind, a Babel plugin, or Metro configuration for this package.

## Provider and navigation

Import `AuthProvider` from `@better-auth-ui/react-native`, not from `@better-auth-ui/react`. The native provider installs the navigation adapter, binds organization access to the route slug, and mounts the error toaster.

Pass `authClient` from `createAuthClient` in `better-auth/react` with the `expoClient` plugin. Pass the application's `queryClient` when the app already uses React Query.

Pass a `navigation` adapter instead of a `navigate` callback. The provider supplies `navigate` to the shared React provider itself.

- Omit `navigation` to use the in-memory state adapter. Set the first view with `initialView`.
- Use `createExpoRouterNavigation({ router, params })` with `useRouter()` and `useLocalSearchParams()` for expo-router.
- Use `createReactNavigationNavigation({ navigation, route, screens })` with `useNavigation()` and `useRoute()` for React Navigation.

Group React Navigation screen names by section: `screens.auth`, `screens.settings`, and `screens.organization`. The adapter ignores views without a mapped screen, so map every view the app can reach.

Router adapters do not report the current view. Pass `view` to `<Auth />`, `<Settings />`, and `<Organization />` on each route or screen.

Carry the reset-password `token` and the organization `slug` as route params. The adapters read them through `getParam`.

Adapters use the default view paths. They do not follow custom path maps, and the state and name-based adapters cannot resolve arbitrary redirect URLs.

## Views and plugins

Use `<Auth />`, `<Settings />`, `<Organization />`, `<UserButton />`, and `<OrganizationSwitcher />` as the main hosts.

Enable the Better Auth server plugin and matching client plugin before registering the native UI plugin on `AuthProvider`. For example, API keys need `apiKeyClient` from `@better-auth/api-key/client`.

Import UI plugins from `@better-auth-ui/react-native/plugins`. The native package provides `apiKeyPlugin`, `deleteUserPlugin`, `magicLinkPlugin`, `multiSessionPlugin`, `organizationPlugin`, `themePlugin`, and `usernamePlugin`.

Do not import `@better-auth-ui/heroui` plugins or components into a native app. They render DOM elements. Other plugins, such as passkey, two-factor, and email OTP, have no native screens.

Native sign-up submits only name, email, and password. It does not render additional fields, including the username plugin's field. Do not require sign-up fields on the server that the native form cannot send.

Organization queries use the slug from the navigation target or route params. Pass `organizationPlugin({ slug })` for a fixed organization screen.

Without a slug, the provider selects personal access. Do not use session active-organization state or `setActive` to select authorization scope.

Check organization membership and permissions on the server. UI visibility alone does not enforce access.

## Theming, styling, and localization

The theme follows the system color scheme by default. Wrap the tree in `ThemeProvider` and pass `light` or `dark` token overrides to change colors.

`themePlugin` drives a built-in theme store. Pass `useTheme` to the plugin to connect an external theme source.

Components accept `style` and `className`. `className` accepts only the package's own utility subset and ignores unknown classes. Use `style` for anything else.

Pass a locale from `@better-auth-ui/locales/<language-tag>` to `AuthProvider`. Pass partial `localization` overrides for product-specific wording.

Native date and number formatting does not follow `locale.languageTag` yet, and layout does not apply `locale.direction`.

## References

- [React Native package README](https://github.com/better-auth-ui/better-auth-ui/tree/main/packages/react-native#readme)
- [Feature parity assessment](https://github.com/better-auth-ui/better-auth-ui/tree/main/packages/react-native#feature-parity)
- [Better Auth Expo integration](https://www.better-auth.com/docs/integrations/expo)
- [React data APIs](https://better-auth-ui.com/docs/react)
- [Documentation index](https://better-auth-ui.com/llms.txt)

GitHub examples follow the default branch. Check the installed package before using an API from newer documentation.
