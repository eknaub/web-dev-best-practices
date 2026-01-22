# React Best Practices

## 📋 Table of Contents

- [Project Structure & Architecture](#project-structure--architecture)
  - [Example Project Structure](#example-project-structure)
  - [Naming Conventions](#naming-conventions)
- [Components & Composition](#components--composition)
  - [Key Principles](#key-principles)
  - [List Rendering & Keys](#list-rendering--keys)

---

# Project Structure & Architecture

## Example Project Structure

```
src/app/
├── main.tsx                 # Main entry point for React
├── App.tsx                  # App component (entry point)
├── provider.tsx             # Wrapper for App with different providers
├── router.tsx               # Routing config
│
├── assets/                  # Static assets (images, fonts, etc.)
│
├── shared/                  # Reusable across features
│   ├── components/          # Dumb/presentational components
│   ├── services/            # API requests, utilities, external service integrations
│   ├── hooks/               # Global Custom hooks
│   ├── utils/               # Utility functions, helpers, and constants (camelCase)
│   ├── types/               # TypeScript types (if using TS)
│   └── constants/           # Reused consts
│
│
├── features/                # Feature modules
│   ├── auth/
│   │   ├── components/            # Feature-specific presentational components
│   │   ├── hooks/                 # Feature-specific hooks
│   │   ├── stores/                # Feature-specific stores
│   │   ├── utils/                 # Feature-specific utility functions, helpers, and constants
│   │   ├── view/                  # Feature-specific view components (PascalCase)
│   │   │   ├── AuthLayout.tsx     # Layout wrapper component
│   │   │   └── Auth.tsx           # Authentication page component
│   │   ├── tests/                 # Feature-specific tests
│   │   └── index.tsx              # Feature Main Component
│   │
│   └── dashboard/
│       ├── features/              # Sub-features
│       │   ├── analytics/
│       │   └── settings/
│       └── services/
│
├── api/                     # API Client configurations
│
├── stores/                  # State management (Redux, Zustand, Context API)
│
├── styles/                  # Global styles (CSS, SASS, Styled Components)
│
├── config/                  # Environment variables and configuration files
│
├── lib/                     # Third-party libraries
│
└── layouts/                 # Layout components
    ├── header/
    ├── footer/
    └── sidebar/
```

## Naming Conventions

### Files & Folders

- **Components**: PascalCase (`Button.tsx`, `UserProfile.tsx`)
- **Utilities/Hooks**: camelCase (`formatDate.ts`, `useAuth.ts`)
- **Custom hooks**: Always start with `use`, as defined by the React Team.
- **Folders**: lowercase with dashes (`user-profile/`, `auth-service/`)
- **Tests**: Same as source file + `.test` or `.spec` (`Button.test.tsx`)
- **Types**: PascalCase (`User.types.ts`, `ApiResponse.types.ts`)
- **Constants**: UPPER_SNAKE_CASE or camelCase file (`API_ENDPOINTS` in `apiEndpoints.ts`)
- **Enums**: Enum values with UPPER_SNAKE_CASE, enum names with PascalCase

### Special Files

- **Index files**: `index.ts` or `index.tsx` for barrel exports
- **Config files**: lowercase with dots (`vite.config.ts`, `.env.local`)

### Variables

- **State - booleans**: Always use prefix `is`, `has` or `should` (`isActive`, `hasError`, `shouldRender`)
- **Event handlers**: Start with `handle` (or `on`) (`handleButtonClick`)
- **CSS classes**: lowercase with dashes (`main-container`)

> 💡 **Info**: Always use `descriptive` and `meaningful` names.

# Components & Composition

## **Key Principles**

1. ✅ Components should always be named (also helps with debugging)
1. ✅ Keep files (components, functions, ...) as close together as possible (e.g. feature-specific components)
1. ✅ Use **functional components** (no class components, except for ErrorBoundary, as of React v19 it is not possible)
1. ✅ Keep components **small and focused** (around 300 lines), refactor if needed
1. ✅ Rendering lists with `map` can be moved into separate components for better readability
1. ✅ **Separate constants and helper functions** into different files, keeps the component small and organized
1. ✅ **Avoid nested rendering functions**, refactor to separate component if needed
1. ✅ **Avoid a large amount of props** a component can take (complex and harder to read and maintain)
1. ✅ **Prever passing objects as props**, instead of primitive values
1. ✅ Conditionally render like so `{isVisible && <div>Im visible!</div>}`
1. ✅ Use Fragments to group elements without a wrapper (`<Fragment>{elems}</Fragment>` or `<>{elems}</>`)

### List Rendering & Keys

If you have a collection of data, your code to display it would probably look something like this:

```jsx
{
  users.map((user) => <UserCard user={user} />);
}
```

With this, if you open the console, you will encounter a warning, telling you that each child in a list needs a unique key prop. Whenever you're rendering an array of elements, each one needs a unique key prop.

Keys help match the correct array items faster, if something has been changed. With good unique keys, react knows exactly which item changed and can update accordingly. If there were no keys, how would react know which one exactly to change? Might as well re-render the whole array.

So you might be wondering, what's a good key? Or bad key?

### The Good vs The Bad

**Good**

Data from a database usually comes with a unique key/id already, which is perfect for us to use.

```jsx
{
  users.map((user) => <UserCard key={user.id} user={user} />);
}
```

Locally generated data can use a randomly generated (e.g. a generated UUID) and locally persisted key.

**Bad**

Probably the quickest "fix" to the warning is just adding the index as key, but this is a bad practice. Since data can change and our list might get a reorder, with index keys our items might receive new keys. This does not help performance and can produce bugs (lost input focus, incorrect component state, animation glitches).

> 💡 **Info**: If no key is specified, React falls back to using the index.

```jsx
{
  users.map((user, index) => <UserCard key={index} user={user} />);
}
```

**Exception**: Index can be used as a key, if the list is static and never changes (no reordering, adding, or removing items).

It's important to note that the keys should not change (DO NOT generate them while rendering). If they change on every render, React can never match them up between renders and the list will always be recreated.

> 💡 **Info**: Components do _not_ receive `key` as a prop, its only used internally by React.
