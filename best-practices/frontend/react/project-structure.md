## 📋 Table of Contents

- [Project Structure & Architecture](#project-structure--architecture)
  - [Example Project Structure](#example-project-structure)
  - [Naming Conventions](#naming-conventions)

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