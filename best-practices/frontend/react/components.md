## 📋 Table of Contents

- [Components & Composition](#components--composition)
  - [Key Principles](#key-principles)
  - [List Rendering & Keys](#list-rendering--keys)
  - [Props Best Practices](#props-best-practices)
  - [Composition pattern](#composition-pattern)
  - [Default props pattern](#default-props-pattern)
  - [Prop drilling vs composition vs context](#prop-drilling-vs-composition-vs-context)

# Components & Composition

## **Key Principles**

1. ✅ Components should always be **named** (also helps with debugging in React DevTools)
1. ✅ Keep files (components, functions, ...) **as close together as possible** (e.g. feature-specific components in feature folders)
1. ✅ Use **functional components** (no class components, except for ErrorBoundary until React v19+ provides an alternative)
1. ✅ Keep components **small and focused** (aim for under 300 lines), refactor into smaller components if needed
1. ✅ **Extract complex list rendering** with `map` into separate components for better readability and performance
1. ✅ **Separate constants and helper functions** into different files, keeps the component small and organized
1. ✅ **Avoid nested rendering functions**, refactor to separate component if needed
1. ✅ **Avoid a large amount of props** a component can take (makes components complex, harder to read and maintain)
1. ✅ **Prefer passing objects as props** instead of many primitive values (reduces prop drilling)
1. ✅ **Conditionally render** using short-circuit evaluation: `{isVisible && <div>Im visible!</div>}`
1. ✅ Use **Fragments** to group elements without a wrapper DOM node (`<Fragment>{elems}</Fragment>` or `<>{elems}</>`)
1. ✅ **One component per file** for better organization and easier imports (exceptions: small, tightly coupled components)
1. ✅ **Avoid inline object/array creation** in JSX props (can cause unnecessary re-renders)
1. ✅ **Be explicit with component APIs** - avoid using `any` types or excessive prop spreading that hides what a component accepts
1. ✅ **Use composition over prop drilling** - restructure your component tree to avoid passing props through components that don't need them
1. ✅ **Provide sensible defaults** for optional props using destructuring with default values in the function signature
1. ✅ **Group related props into objects** instead of having many individual primitive props (reduces prop count and improves maintainability)
1. ✅ **Start with props, refactor to composition, use Context only when necessary** - avoid premature optimization with Context API
1. ✅ **Design component APIs to be discoverable** - use clear prop names, TypeScript interfaces, and avoid magic behaviors

## List Rendering & Keys

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

## Props Best Practices

### Props Spreading

Props spreading (`...props`) is a powerful feature but can introduce hidden behaviors and make components harder to understand. Here's when to use it and when to avoid it.

**✅ Good Use Cases:**

Forwarding props to HTML elements (good for flexibility):

```jsx
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary';
}

function Button({ variant = 'primary', ...rest }: ButtonProps) {
  return (
    <button className={`btn btn-${variant}`} {...rest}>
      {rest.children}
    </button>
  );
}

// User can pass any button attribute
<Button variant="primary" onClick={handleClick} disabled aria-label="Submit" />
```

**❌ Bad Use Cases:**

Spreading all props without explicit declarations (unclear API):

```jsx
// ❌ Bad: What props does Button actually accept?
function Button(props: any) {
  return <button {...props}>{props.children}</button>;
}
```

Spreading to hide props from documentation:

```jsx
// ❌ Bad: Props are unclear and hard to discover
function Card(props: Record<string, any>) {
  const { title, ...rest } = props;
  return <div {...rest}><h1>{title}</h1></div>;
}
```

Spreading when you should be explicit:

```jsx
// ❌ Bad: Makes it unclear what props this component accepts
interface DialogProps {
  [key: string]: any;
}

function Dialog({ isOpen, ...rest }: DialogProps) {
  return isOpen ? <div {...rest} /> : null;
}

// ✅ Good: Explicit and clear
interface DialogProps {
  isOpen: boolean;
  title: string;
  onClose: () => void;
  children: React.ReactNode;
}

function Dialog({ isOpen, title, onClose, children }: DialogProps) {
  return isOpen ? (
    <div>
      <h2>{title}</h2>
      {children}
      <button onClick={onClose}>Close</button>
    </div>
  ) : null;
}
```

### Component API Design

A well-designed component API is explicit, predictable, and easy to use.

**Principles:**

1. **Be Explicit**: Don't use `any` or spreading to hide what your component accepts
2. **Use Meaningful Names**: Props should describe their purpose clearly
3. **Provide Defaults**: Use sensible defaults for optional props
4. **Group Related Props**: Use object props for related values instead of many individual props

```jsx
// ❌ Bad: Too many unrelated props
interface ButtonProps {
  onClick: () => void;
  disabled: boolean;
  loading: boolean;
  size: 'sm' | 'md' | 'lg';
  variant: 'primary' | 'secondary';
  fullWidth: boolean;
  rounded: boolean;
  shadow: boolean;
}

// ✅ Good: Grouped into logical props
interface ButtonProps {
  onClick: () => void;
  
  // Appearance
  appearance?: {
    variant?: 'primary' | 'secondary';
    size?: 'sm' | 'md' | 'lg';
  };
  
  // State
  state?: {
    disabled?: boolean;
    loading?: boolean;
  };
  
  // Layout
  fullWidth?: boolean;
}
```

**Avoid Prop Overload:**

```jsx
// ❌ Bad: Component is doing too much, props are unclear
function Card({ 
  title, 
  description, 
  image, 
  onClick, 
  onHover, 
  onFocus,
  isSelected,
  isDisabled,
  isLoading,
  variant,
  size,
  padding,
  margin,
  borderRadius,
  // ... 15+ more props
}: CardProps) { ... }

// ✅ Good: Use composition instead
function Card({ title, children, variant = 'default' }: CardProps) {
  return (
    <div className={`card card-${variant}`}>
      <h3>{title}</h3>
      {children}
    </div>
  );
}

// Complex card - use composition
function ProductCard({ product, onSelect }: { product: Product; onSelect: () => void }) {
  return (
    <Card title={product.name} variant="product">
      <img src={product.image} alt={product.name} />
      <p>{product.description}</p>
      <button onClick={onSelect}>Select</button>
    </Card>
  );
}
```

### Props Interface Design

TypeScript props interfaces should be clear and type-safe.

**Best Practices:**

```jsx
// ✅ Good: Clear, type-safe props interface
interface ButtonProps {
  // Required props
  label: string;
  onClick: () => void;
  
  // Optional props with defaults
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  
  // Callback props
  onHover?: () => void;
  
  // Children
  children?: React.ReactNode;
}

function Button({ 
  label, 
  onClick, 
  variant = 'primary',
  size = 'md',
  disabled = false,
  onHover,
  children
}: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant} btn-${size}`}
      disabled={disabled}
      onClick={onClick}
      onMouseEnter={onHover}
    >
      {label}
      {children}
    </button>
  );
}
```

**Use `Readonly` for Immutability:**

```jsx
interface CardProps {
  title: string;
  children: React.ReactNode;
}

// ✅ Good: Prevents accidental mutations
function Card(props: Readonly<CardProps>) {
  return <div>{props.title}{props.children}</div>;
}
```

**Extend HTML Attributes for DOM Components:**

```jsx
// ✅ Good: Allows standard HTML props on custom components
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label?: string;
  error?: string;
}

function Input({ label, error, ...rest }: InputProps) {
  return (
    <div>
      {label && <label>{label}</label>}
      <input {...rest} />
      {error && <span className="error">{error}</span>}
    </div>
  );
}

// Users can pass any input attributes
<Input type="email" placeholder="Enter email" required />
```

**Avoid Union Types for Control Flow:**

```jsx
// ❌ Bad: Unclear which props are needed for each variant
interface ButtonProps {
  variant: 'submit' | 'link' | 'icon';
  href?: string;
  icon?: React.ReactNode;
  label?: string;
}

// ✅ Good: Discriminated unions for type safety
type ButtonProps = { variant: 'submit'; label: string; onClick: () => void }
  | { variant: 'link'; href: string; label: string }
  | { variant: 'icon'; icon: React.ReactNode; onClick: () => void };

function Button(props: ButtonProps) {
  switch (props.variant) {
    case 'submit':
      return <button onClick={props.onClick}>{props.label}</button>;
    case 'link':
      return <a href={props.href}>{props.label}</a>;
    case 'icon':
      return <button onClick={props.onClick}>{props.icon}</button>;
  }
}
```

> 💡 **Info**: Props spreading is useful for flexibility, but explicit interfaces make components more maintainable and discoverable. Use spreading sparingly for HTML attributes and HOCs, but be explicit for component-specific props.

## Composition pattern

Composition pattern helps a lot with reusability and flexibility of components. Since the components are built more modular, it's easier to maintain and read.

> 💡 **Info**: Composition with children defines structure, composition with data props defines behavior.

### Children as Props

Children as Props is best used for layout or wrapper components, when you are unsure of the exact structure since with this approach, the component doesnt know (and care) about its children. The following example shows a structural `Card` component, which defines the layout of the component and accepts any children. This approach makes the Card component highly reusable and maintainable.

```jsx
interface CardProps {
  children: React.ReactNode;
}

function Card(props: Readonly<CardProps>) {
  return <div className="card">{props.children}</div>;
}

function App() {
  return (
    <Card>
      <h1>Title</h1>
      <p>Content</p>
    </Card>
  );
}
```

With composition you can also achieve specialization, for example a base component `Dialog` can be used by a special component `DeleteDialog`.

### Data as Props

Passing data as props is a very common pattern to pass down data from a parent to a child component. Props cannot be modified within the child. Using the Card example from above, we could also pass data instead of children. 

```jsx
interface CardProps {
  title: string;
  content: string;
}

function Card(props: Readonly<CardProps>) {
  return (
    <div className="card">
      <h1>{props.title}</h1>
      <p>{props.content}</p>
    </div>
  );
}

function App() {
  return (
    <Card title="Title" content="Content" />
  );
}
```
> 💡 **Important**: Keep the anti-pattern Prop Drilling in mind, if you start passing data through multiple components (which in the worse case don't need them), it's time to think about a better solution (Context, State, HOCs).

### Combination - Children and Data

Both children and data can be combined and passed to a component. Using the same `Card` example, the title could be a data prop and the body children.

```jsx
interface CardProps {
  title: string;
  children: React.ReactNode;
}

function Card(props: Readonly<CardProps>) {
  return (
    <div className="card">
      <h1>{props.title}</h1>
      <div className="card-body">{props.children}</div>
    </div>
  );
}

function App() {
  return (
    <Card title="Title">
      <p>Content</p>
    </Card>
  );
}
```

## Default props pattern

Default props allow you to specify fallback values for props that are not provided by the parent component. This is useful for providing sensible defaults and making components more flexible.

**Modern Approach: Destructuring with Defaults**

The recommended way is to use destructuring with default values in the function signature:

```jsx
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary';
  disabled?: boolean;
  onClick?: () => void;
}

function Button({ 
  label, 
  variant = 'primary', 
  disabled = false, 
  onClick = () => {} 
}: ButtonProps) {
  return (
    <button 
      className={`btn btn-${variant}`} 
      disabled={disabled}
      onClick={onClick}
    >
      {label}
    </button>
  );
}

// Usage - defaults are applied automatically
<Button label="Click me" />
<Button label="Delete" variant="secondary" disabled={true} />
```

## Prop drilling vs composition vs context

Passing data through your component tree is a fundamental part of React. However, there are different approaches with different trade-offs. Here's how to choose:

### Prop Drilling

Passing data through multiple intermediate components that don't need it themselves. While straightforward for small trees, it becomes problematic as your app grows.

**When to use:**
- Simple component trees (1-2 levels deep)
- Data that naturally flows downward
- You want to keep components isolated and testable

**When to avoid:**
- Data passes through 3+ levels of components
- Many components ignore the props (anti-pattern indicator)

```jsx
// ❌ Bad: data passes through UserList which doesn't need it
function App() {
  const [user, setUser] = useState(null);
  return <UserList user={user} />;
}

function UserList({ user }) {
  return <UserProfile user={user} />;
}

function UserProfile({ user }) {
  return <div>{user.name}</div>;
}
```

### Composition

Restructure your component tree to avoid passing unused props. Move components closer together so data doesn't need to travel as far.

**When to use:**
- You have multiple levels of prop drilling
- Components in between don't need the data
- You want to keep components simple and focused

**How it works:**
Instead of passing data down, pass the already-composed component tree down.

```jsx
// ✅ Good: Composition approach
function App() {
  const [user, setUser] = useState(null);
  return <UserList>{user && <UserProfile user={user} />}</UserList>;
}

function UserList({ children }) {
  return <div className="user-list">{children}</div>;
}

function UserProfile({ user }) {
  return <div>{user.name}</div>;
}
```

### Context API

Use Context for global state that many components need (theme, auth, language, etc.). Context avoids prop drilling but adds some complexity.

**When to use:**
- Data is needed by many components at different levels
- Global state (auth, theme, language settings)
- You want to avoid prop drilling entirely

**When to avoid:**
- Frequently changing data (causes unnecessary re-renders)
- Local component state (use `useState` instead)
- Simple parent-child communication

```jsx
// Create context
const UserContext = React.createContext<User | null>(null);

// Provider (usually at app root)
function App() {
  const [user, setUser] = useState(null);
  
  return (
    <UserContext.Provider value={user}>
      <UserList />
    </UserContext.Provider>
  );
}

// Consume anywhere without prop drilling
function UserProfile() {
  const user = useContext(UserContext);
  return <div>{user?.name}</div>;
}

function UserList() {
  return <UserProfile />; // No need to pass user prop
}
```

### Decision Tree

1. **Is the data needed by 1-2 adjacent components?** → Use **props**
2. **Does data need to skip multiple intermediate components?** → Try **composition** first
3. **Is data needed by many unrelated components?** → Use **Context** or **state management** (Redux, Zustand)
4. **Does data change frequently?** → Consider **state management library** instead of Context

> 💡 **Info**: Start with props, refactor to composition, use Context only when necessary. Avoid premature optimization.
