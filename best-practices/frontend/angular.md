### Angular Best Practices

## 📋 Table of Contents

- [Project Structure & Architecture](#project-structure--architecture)
  - [Feature Structure Guidelines](#feature-structure-guidelines)
- [Key Principles](#key-principles)
- [Visibility Modifiers](#visibility-modifiers)
- [File Naming Conventions](#file-naming-conventions)
- [Component Structure](#component-structure)
- [Components & Templates](#components--templates)
- [Services & Dependency Injection](#services--dependency-injection)
- [RxJS & State Management](#rxjs--state-management)
- [Signals](#signals)
- [Reusability](#reusability)
- [Code Style & Conventions](#code-style--conventions)
- [Testing](#testing)
- [Performance](#performance)
- [Security](#security)
- [Documentation](#documentation)
- [Dos & Don'ts](#dos--donts)
- [Routing & Navigation](#routing--navigation)
- [HTTP & API Integration](#http--api-integration)
- [Error Handling & Logging](#error-handling--logging)
- [Lifecycle Hooks](#lifecycle-hooks)
- [Styling & UI](#styling--ui)
- [Forms](#forms)
- [Authentication & Authorization](#authentication--authorization)
- [Advanced Testing](#advanced-testing)
- [Packaging](#packaging)
- [Accessibility (A11y)](#accessibility-a11y)
- [Tooling](#tooling)
- [Clean Code](#clean-code)

---

# Project Structure & Architecture

```
src/app/
├── app.ts                   # Root component
├── app.config.ts            # Application configuration
├── app.routes.ts            # Route definitions
│
├── core/                    # Singleton services & app-wide logic
│   ├── api/                 # Generated API client code
|   |   ├── models/           # Generated API models
|   |   └── services/         # Generated API services
│   ├── guards/              # Route guards
│   ├── interceptors/        # HTTP interceptors (if app-specific)
│   └── services/            # Core singleton services
│
├── shared/                  # Reusable across features
│   ├── components/          # Dumb/presentational components
│   ├── directives/          # Custom directives
│   ├── pipes/               # Custom pipes
│   ├── models/              # TypeScript interfaces/types and shared DTOs
│   ├── services/            # Shared services
│   ├── utils/               # Helper functions
│   ├── validators/          # Form validators
│   └── interceptors/        # Shared HTTP interceptors
│
├── features/                # Feature modules (lazy-loaded)
│   ├── auth/
│   │   ├── pages/                 # Smart/container components
│   │   │   ├── login/
│   │   │   │   ├── login.ts           # Component logic
│   │   │   │   ├── login.html         # Template
│   │   │   │   ├── login.css          # Styles
│   │   │   │   └── login.spec.ts      # Tests
│   │   │   └── register/
│   │   ├── components/            # Feature-specific presentational components
│   │   ├── directives/            # Feature-specific directives
│   │   ├── pipes/                 # Feature-specific pipes
│   │   ├── dialogs/               # Feature-specific dialogs/modals
│   │   ├── stores/                # Feature-specific state (NgRx/Signals)
│   │   ├── models/                # Feature-specific models/interfaces
│   │   └── services/              # Feature-specific services
│   │
│   ├── blog/                      # Flat structure (Angular style guide)
│   │   ├── post-list.ts           # List component
│   │   ├── post-list.html
│   │   ├── post-list.css
│   │   ├── post-detail.ts         # Detail component
│   │   ├── post-detail.html
│   │   ├── post-detail.css
│   │   ├── post-editor.ts         # Editor component
│   │   ├── post-editor.html
│   │   ├── post-editor.css
│   │   ├── post-card.ts           # Reusable card component
│   │   ├── post-card.html
│   │   ├── post.service.ts        # Feature service
│   │   ├── post.model.ts          # Feature models
│   │   ├── post.store.ts          # Feature state store
│   │   ├── time-ago.pipe.ts       # Feature-specific pipe
│   │   └── highlight.directive.ts # Feature-specific directive
│   │
│   └── dashboard/
│       ├── features/              # Sub-features
│       │   ├── analytics/
│       │   └── settings/
│       └── services/
│
└── layouts/                 # Layout components
    ├── header/
    ├── footer/
    └── sidebar/
```

## Feature Structure Guidelines

### **Option 1: Pragmatic Grouping (auth example above)**

- Group by type folders (`components/`, `services/`, `pipes/`, etc.)
- **Use when**: Feature has 20+ files or complex organization
- **Pros**: Clear organization, easier navigation in large features
- **Cons**: Violates Angular style guide, more nesting

### **Option 2: Flat Structure (blog example above)**

- All files at feature root, no type-based subdirectories
- **Use when**: Feature has <10-15 files (Angular style guide recommendation)
- **Pros**: Less navigation, follows official guidelines, simpler
- **Cons**: Can become cluttered with many files

> 💡 **Recommendation**: Start flat, introduce type folders only when a feature grows beyond ~15 files. Consistency within a feature is most important.
> You could also continue using the flat structure style, but if a directory becomes hard to read or navigate, consider splitting into sub-directories.

## **Key Principles**

1. ✅ Enable TypeScripts **strict mode**
1. ✅ Use **standalone components** (no NgModules)
1. ✅ Use **signals** for state management
1. ✅ Use **OnPush change detection** everywhere
1. ✅ Use **inject()** function over constructor injection
1. ✅ Use **control flow** syntax (`@if`, `@for`) over structural directives
1. ✅ Keep components **small and focused**
1. ✅ Co-locate files (`.ts`, `.html`, `.css`, `.spec.ts`)
1. ✅ Follow the given style convention, prioritize maintaining the conistency withing a file instead of mixing.
1. ✅ Components should generally relate to UI shown on screen, prefer refactoring code that makes sense on its own like form validation rules.
1. ✅ Avoid complex code in templates, refactor to TypeScript Code if so (e.g. with computed)
1. ✅ Use appropriate access modifiers to control visibility and improve encapsulation
1. ✅ Use readonly for properties that shouldn't change
1. ✅ Prefer class and style over ngClass and ngStyle
1. ✅ Name event handlers for what they do, not for the triggering event (`handleClick()` => `saveUserData()`)

# **Visibility Modifiers**

Use appropriate access modifiers to control visibility and improve encapsulation:

- **`private`** - Only accessible within the class. **Not accessible in template**. Use for internal implementation details.

  ```typescript
  private readonly postService = inject(PostService); // Service, not used in template
  ```

- **`protected`** - Accessible within the class, in the component's template, and by subclasses. **Preferred for template-bound members**.

  ```typescript
  protected readonly posts = signal<Post[]>([]); // Used in template
  protected deletePost(id: string): void { } // Called from template
  ```

- **`public`** - Accessible everywhere. Use sparingly, typically only for inputs/outputs or APIs intended for external use.
  ```typescript
  readonly postId = input.required<string>(); // Public by convention (inputs/outputs)
  ```

> 💡 **Best Practice**: Use `protected` instead of `public` for template-accessible members. This signals that the member is for internal use (class + template), not part of the component's external API.

# **File Naming Conventions**

- **kebab-case** for file names, **CamelCase** for class names (e.g., `BlogPost` class → `blog-post.ts` file)
- **Co-locate related files** with the same base name: `blog-post.ts`, `blog-post.html`, `blog-post.css`, `blog-post.spec.ts`
- **Multiple style files** use descriptive suffixes: `blog-post.css`, `blog-post.theme.css`, `blog-post.print.css`
- **Descriptive names** over generic ones: avoid `utils.ts`, `common.ts`, `helpers.ts` without context

**Type-specific suffixes:**

- Components: `blog-post.ts` (not `.component.ts` in Angular 21+)
- Services: `post.service.ts`
- Stores: `post.store.ts`
- Guards: `auth.guard.ts`
- Pipes: `date-format.pipe.ts`
- Directives: `highlight.directive.ts`
- Models: `post.model.ts` or `post.ts` (for interfaces/types)
- Routes: `blog.routes.ts`

# **Component Structure**

```typescript
// Modern Angular 21+ component with signals
import {
  ChangeDetectionStrategy,
  Component,
  signal,
  computed,
  inject,
  input,
  output,
} from "@angular/core";

@Component({
  selector: "app-blog-post", //should be a custom element name, prefixes like 'blog' can be used for more clarity
  templateUrl: "./blog-post.html",
  styleUrl: "./blog-post.css",
  changeDetection: ChangeDetectionStrategy.OnPush,
  imports: [
    /* standalone imports */
  ],
})
export class BlogPost {
  // Use input() for component inputs (signal-based)
  readonly postId = input.required<string>(); // Required input
  readonly showComments = input(true); // Optional input with default

  // Use output() for component outputs (signal-based)
  readonly postDeleted = output<string>(); // Emits post ID
  readonly commentAdded = output<{ postId: string; text: string }>();

  // Use inject() instead of constructor injection
  private readonly postService = inject(PostService);

  /*
    Inject has several advantages over constructor injection: 
      - inject is generally more readable, especially when a class injects many dependencies.
      - It's more syntactically straightforward to add comments to injected dependencies
      - inject offers better type inference.
  */

  // Use signals for reactive state
  protected readonly posts = signal<Post[]>([]);
  protected readonly loading = signal(false);

  // Use computed for derived state
  protected readonly publishedPosts = computed(() =>
    this.posts().filter((post) => post.published)
  );

  // Example of emitting output
  protected deletePost(id: string): void {
    this.postDeleted.emit(id);
  }
}
```

# Components & Templates

# Services & Dependency Injection

# RxJS & State Management

# Signals

# Reusability

# Code Style & Conventions

# Testing

# Performance

# Security

# Documentation

# Dos & Don'ts

# Routing & Navigation

# HTTP & API Integration

# Error Handling & Logging

# Lifecycle Hooks

# Styling & UI

# Forms

# Authentication & Authorization

# Advanced Testing

# Packaging

# Accessibility (A11y)

# Tooling

# Clean Code
