# Angular Best Practices

## 📋 Table of Contents

- [Project Structure & Architecture](#project-structure--architecture)
  - [Example Project Structure](#example-project-structure)
  - [Feature Structure Guidelines](#feature-structure-guidelines)
  - [Key Principles](#key-principles) TODO
  - [Visibility Modifiers](#visibility-modifiers)
  - [File Naming Conventions](#file-naming-conventions)
  - [Smart/Dumb Components Pattern](#smartdumb-components-pattern)
- [Components & Templates](#components--templates)
  - [Component Structure](#component-structure)
  - [Input/Output Best Practices](#inputoutput-best-practices)
  - [Async Pipe vs Manual Subscribe](#async-pipe-vs-manual-subscribe)

---

# Project Structure & Architecture

## Example Project Structure

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

## **Visibility Modifiers**

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

## **File Naming Conventions**

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

## Smart/Dumb Components Pattern

This is a common pattern for component-based programming. An article by Dan Abramov from 2015 can be found [here](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). It was first described for React, but nowadays it's a fundamental principle that has been adopted and adapted by many libraries and frameworks. He introduced a pattern to separate components into two categories:

1. Smart Components (or Container Components)
2. Dumb Components (or Presentational Components).

These two categories separate the view from the application logic.

### Smart Components

These Components care about <strong><em>what</em></strong> data is shown to the user.

- Hold state and business logic
- Inject services and make API calls
- Manage data flow
- Template mostly just passes data to dumb components and handles their events
- Example: `UserListPage` fetches users and handles deletion

```typescript
export class UserListPage {
  private userService = inject(UserService);
  protected users = signal<User[]>([]);

  protected handleDelete(id: string) {
    this.userService.delete(id).subscribe(/*...*/);
  }
}
```

```typescript
@for (user of users(); track user.id) {
  <app-user-card [user]="user" (delete)="handleDelete($event)" />
}
```

### Dumb Components

These Components care about <strong><em>how</em></strong> data is shown to the user.

- Only receive data via input()
- Emit events via output()
- No (or minimal) service injection
- Focus on UI logic only
- Highly reusable and testable
- Example `UserCard` displays a user which is used by the smart component, emits `userClicked` event

```typescript
export class UserCard {
  readonly user = input.required<User>();
  readonly delete = output<string>();

  /**
   * Emits the user ID when the delete button is clicked.
   */
  protected onDeleteClick(): void {
    this.delete.emit(this.user().id);
  }
}
```

# Components & Templates

## **Component Structure**

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
    this.posts().filter((post) => post.published),
  );

  // Example of emitting output
  protected deletePost(id: string): void {
    this.postDeleted.emit(id);
  }
}
```

## Input/Output Best Practices

Sharing data between parent and child components is a common pattern, with Angular you can achieve this by using input() and output() functions. For a more detailed explanation visit the official Guide by Angular for [inputs](https://angular.dev/guide/components/inputs) or [outputs](https://angular.dev/guide/components/outputs).

- Used for clear and type-safe bindings
- Prefer `readonly` for inputs to prevent mutation from child components
- Descriptive names (avoid generic names like `data`)
- Use required inputs for mandatory data `input.required<Type>()`
- Use output for event communication, not data passing back and forth
- Avoid two-way binding, except for simple cases like form controls
- Document input/outputs for better IDE support
- Co-locate input/outputs at the top of the class for visibility
- Emit output only in response to user actions or lifecycle events
- Avoid logic in templates, pass data and handle events in the component class
- If a default value is used, TS can infer the type e.g. `input(0)` => number
- If no default value is set, the value is undefined e.g. `input<number>()` the type is `<number | undefined>` and value `undefined`.

> 💡 **Note**: The `input()` and `output()` functions are a Angular v16+ feature.

**Example:**

```typescript
export class UserCard {
  /**
   * The user object to display in the card.
   * @type {User}
   * @required
   */
  readonly user = input.required<User>();
  /**
   * Emits when the user is deleted. Output is typically the user ID or a message.
   */
  readonly delete = output<string>();

  protected onDeleteClick(): void {
    this.delete.emit(this.user().id);
  }
}

<!-- Example usage in template -->
<button (click)="onDeleteClick()">Delete</button>
```

## Advanced Usage

### Transform

Whenever the value of a bound input changes, the transform function is executed and its result is assigned to the input property.

**Example:**

```typescript
export class UserCard {
  email = input("", { transform: trimString });

  function trimString(value: string | undefined): string {
    return value?.trim() ?? '';
  }
}

<!-- Example usage in template -->
<custom-input [email]="newsletter" />
```

Inside the template, when defining a event listener, event data can be accessed with $event, e.g. `<professional-logger (valueChanged)="logValue($event)" />`

### Two-way binding

Two-way binding keeps the model and view in sync automatically:

- When the model changes, the view updates
- When the view changes (user input), the model updates

Use the `[(ngModel)]` directive for two-way binding:

**Example:**

```html
<!-- Two-way binding with banana-in-a-box syntax -->
<input [(userName)]="userName" />
<p>Hello, {{ userName }}!</p>
```

> 💡 **Note**: The best use-case for two-way bindings is with simple forms or simple inputs, else consider using signals or one-way binding.

### Old decorator @Input and @Output

Although the modern input() and output() functions are recommended, you may still see the older @Input and @Output decorator in legacy Angular code. The way you bind inputs in the template remains the same (see UserCard above), but the class property syntax is different.

@Input supports options (similar to input() options) like required, transform functions, and aliases. You can also use getters and setters, though Angular generally does not recommend this approach.

@Output also supports the alias option.

```typescript
@Component({
  /*...*/
})
export class UserCard {
  @Input() firstName = "Max";
  @Output() delete = new EventEmitter<void>();
}
```

### Inheriting Inputs/Outputs

You can inherit input/output properties from a parent class using the `inputs` or `outputs` array in the `@Component` decorator. This is useful for creating reusable base components.

**Example:**

```typescript
// Base component with an input/output property
export class UserCardBase {
  readonly highlight = input<boolean>(false);
  readonly delete = output<string>();
}

// Child component inherits 'highlight' and 'delete' from UserCardBase
@Component({
  selector: "app-user-card",
  templateUrl: "./user-card.html",
  inputs: ["highlight"],
  outputs: ["delete"],
})
export class UserCard extends UserCardBase {}
```

## Async Pipe vs Manual Subscribe

Instead of manually subscribing, let Angular handle async streams in templates.
