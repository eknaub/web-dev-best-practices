# React Overview

## 📋 Table of Contents

- [⚡ Quick Reference](#-quick-reference)
  - [Golden Rules](#golden-rules)
  - [Common Pitfalls](#common-pitfalls)
  - [Performance Checklist](#performance-checklist)
  - [Security Essentials](#security-essentials)
  - [Quick Decision Trees](#quick-decision-trees)
  - [React 18+ Features & Patterns](#react-18-features--patterns)
  - [React 19 Features](#react-19-features)
  - [Error Handling Essentials](#error-handling-essentials)
  - [Conditional Rendering Patterns](#conditional-rendering-patterns)
  - [Fragment Best Practices](#fragment-best-practices)
  - [Ref Forwarding](#ref-forwarding)
  - [Accessibility Quick Wins](#accessibility-quick-wins)
  - [Form Handling Best Practices](#form-handling-best-practices)
  - [API/Fetch Patterns](#apifetch-patterns)
  - [React.StrictMode (Development Tool)](#reactstrictmode-development-tool)
  - [Event Handling Gotchas](#event-handling-gotchas)
  - [Children Patterns](#children-patterns)
  - [TypeScript Best Practices](#typescript-best-practices)
  - [Testing Quick Tips](#testing-quick-tips)

---

# ⚡ Quick Reference

## Golden Rules

### Hooks Rules (Non-negotiable)
- ✅ Only call hooks at the top level (never in loops, conditions, or nested functions)
- ✅ Only call hooks in React components or custom hooks
- ✅ Custom hooks MUST start with `use` prefix
- ✅ Dependencies arrays must include ALL values used inside the hook

### Component Essentials
- ✅ Always provide stable `key` prop when rendering lists (never use index as key if list can change)
- ✅ Keep components small and focused (single responsibility)
- ✅ Prefer composition over prop drilling for deeply nested data
- ✅ Never mutate state directly - always create new objects/arrays
- ✅ Keep state as close as possible to where it's used (colocation)

### Event Handlers
- ✅ Name handlers `handleXxx` or `onXxx` for clarity
- ✅ Avoid inline arrow functions in JSX for lists (creates new function on every render)
- ✅ For expensive callbacks, wrap in `useCallback` with proper dependencies

## Common Pitfalls

### ❌ State Management Mistakes
```jsx
// ❌ BAD: Direct mutation
const [user, setUser] = useState({ name: 'John' });
user.name = 'Jane'; // NEVER DO THIS

// ✅ GOOD: Create new object
setUser({ ...user, name: 'Jane' });
// OR with functional update
setUser(prev => ({ ...prev, name: 'Jane' }));
```

### ❌ Effect Dependencies
```jsx
// ❌ BAD: Missing dependencies (causes stale closures)
useEffect(() => {
  console.log(count); // Uses count
}, []); // But doesn't list it

// ✅ GOOD: Include all dependencies
useEffect(() => {
  console.log(count);
}, [count]);
```

### ❌ Keys in Lists
```jsx
// ❌ BAD: Index as key (causes bugs when items reorder/delete)
{items.map((item, index) => <Item key={index} {...item} />)}

// ✅ GOOD: Use stable unique identifier
{items.map(item => <Item key={item.id} {...item} />)}
```

### ❌ Derived State
```jsx
// ❌ BAD: Storing derived state
const [items, setItems] = useState([]);
const [count, setCount] = useState(0);
useEffect(() => setCount(items.length), [items]); // Unnecessary!

// ✅ GOOD: Calculate during render
const [items, setItems] = useState([]);
const count = items.length; // Simple and correct
```

### ❌ Unnecessary useEffect
```jsx
// ❌ BAD: Effect for simple transformation
const [firstName, setFirstName] = useState('');
const [lastName, setLastName] = useState('');
const [fullName, setFullName] = useState('');
useEffect(() => {
  setFullName(`${firstName} ${lastName}`);
}, [firstName, lastName]);

// ✅ GOOD: Calculate during render
const fullName = `${firstName} ${lastName}`;
```

## Performance Checklist

### When to Optimize
- ⚠️ **Don't optimize prematurely** - measure first with React DevTools Profiler
- ✅ Optimize when you have actual performance issues (slow renders, lag)
- ✅ Optimize when passing callbacks to heavily re-rendered components

### Quick Wins
```jsx
// ✅ Memoize expensive calculations
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(input);
}, [input]);

// ✅ Memoize callbacks for child components
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);

// ✅ Memoize components that rarely change
const MemoizedChild = memo(ChildComponent);

// ✅ Split context to prevent unnecessary re-renders
// BAD: Single context with all state
// GOOD: Separate contexts for data and actions
```

### Re-render Triggers
A component re-renders when:
1. **State changes** (via `useState`, `useReducer`)
2. **Parent re-renders** (unless wrapped in `memo()`)
3. **Context value changes** (any consumer re-renders)
4. **Props change** (reference or value)

### Avoid Re-renders
- ✅ Move state down (closer to where it's used)
- ✅ Lift content up (children as props pattern)
- ✅ Use `memo()` for expensive pure components
- ✅ Split large contexts into smaller ones
- ✅ Use state management libraries for global state (Zustand, Redux)

## Security Essentials

### XSS Prevention
```jsx
// ✅ SAFE: React escapes by default
<div>{userInput}</div>

// ❌ DANGEROUS: Bypasses escaping
<div dangerouslySetInnerHTML={{ __html: userInput }} />

// ✅ SAFE: Use sanitization library
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

### Other Security Rules
- ❌ Never store sensitive data (tokens, passwords) in state or localStorage without encryption
- ❌ Never trust user input - always validate and sanitize
- ✅ Use HTTPS for all API requests
- ✅ Implement proper CORS on backend
- ✅ Use environment variables for API keys (never hardcode)
- ✅ Sanitize URLs before using in `href` or `src`

### Common Vulnerabilities
```jsx
// ❌ DANGEROUS: Unvalidated redirect
window.location.href = userProvidedUrl;

// ✅ SAFE: Validate against allowlist
const allowedDomains = ['example.com', 'trusted.com'];
const url = new URL(userProvidedUrl);
if (allowedDomains.includes(url.hostname)) {
  window.location.href = userProvidedUrl;
}
```

## Quick Decision Trees

### State Management: Which Tool?
- **Component-local state** → `useState` / `useReducer`
- **Shared between 2-3 components** → Lift state up / composition
- **Shared across many components (same tree)** → Context API
- **Complex global state** → Zustand / Redux Toolkit
- **Server state (API data)** → TanStack Query / SWR
- **Form state** → React Hook Form / Formik
- **URL state** → React Router / Next.js router

### When to Use Each Hook?
- **`useState`** → Simple local state (strings, numbers, booleans, simple objects)
- **`useReducer`** → Complex state with multiple related values or complex update logic
- **`useEffect`** → Side effects (API calls, subscriptions, DOM manipulation)
- **`useLayoutEffect`** → DOM measurements before paint (rare, use only when needed)
- **`useMemo`** → Expensive calculations that slow down renders
- **`useCallback`** → Prevent child re-renders when passing callbacks
- **`useRef`** → DOM elements, mutable values that don't trigger re-renders
- **`useContext`** → Access context values
- **`useId`** → Generate unique IDs for accessibility

### Memoization: When to Use?
```
Should I use useMemo/useCallback/memo?
│
├─ Is it causing measured performance issues? → YES → Use it
├─ Is it a dependency in useEffect/useMemo/useCallback? → YES → Use it
├─ Is it preventing child re-renders? → YES → Use it
└─ Otherwise → NO, don't use it (premature optimization)
```

## React 18+ Features & Patterns

### Automatic Batching
- ✅ React 18+ batches ALL state updates automatically (even in async functions)
- ✅ No need for `unstable_batchedUpdates` anymore
- ✅ Multiple `setState` calls in same function = single re-render

### Transitions (useTransition)
```jsx
const [isPending, startTransition] = useTransition();

// Mark non-urgent updates as transitions
startTransition(() => {
  setSearchResults(filterLargeList(query)); // Non-urgent
});

// Urgent updates happen immediately
setQuery(e.target.value); // User typing - urgent
```

### Suspense for Data Fetching
```jsx
// ✅ Show fallback while data loads
<Suspense fallback={<Spinner />}>
  <UserProfile userId={id} />
</Suspense>

// Works with lazy loading and data fetching libraries
const LazyComponent = lazy(() => import('./Heavy'));
```

## React 19 Features

### Actions & useActionState
```jsx
// ✅ NEW: Built-in form action handling with pending states
import { useActionState } from 'react';

function AddToCart({ productId }) {
  const [message, formAction, isPending] = useActionState(
    async (previousState, formData) => {
      const result = await addToCartAPI(productId, formData);
      return result.message;
    },
    null // initial state
  );
  
  return (
    <form action={formAction}>
      <input name="quantity" type="number" defaultValue="1" />
      <button type="submit" disabled={isPending}>
        {isPending ? 'Adding...' : 'Add to Cart'}
      </button>
      {message && <p>{message}</p>}
    </form>
  );
}

// ✅ Works with Server Actions in Next.js
'use server';
export async function createUser(prevState, formData) {
  // Server-side logic
  return { success: true };
}
```

### useFormStatus
```jsx
// ✅ NEW: Access form status without Context
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending, data, method, action } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}

function MyForm() {
  return (
    <form action={handleSubmit}>
      <input name="email" />
      <SubmitButton /> {/* Automatically knows form state */}
    </form>
  );
}
```

### useOptimistic
```jsx
// ✅ NEW: Optimistic UI updates built-in
import { useOptimistic } from 'react';

function TodoList({ todos }) {
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo) => [...state, { ...newTodo, pending: true }]
  );
  
  async function handleAdd(formData) {
    const newTodo = { id: Date.now(), text: formData.get('text') };
    
    // Show optimistic update immediately
    addOptimisticTodo(newTodo);
    
    // Then do the actual API call
    await createTodo(newTodo);
  }
  
  return (
    <>
      {optimisticTodos.map(todo => (
        <div key={todo.id} style={{ opacity: todo.pending ? 0.5 : 1 }}>
          {todo.text}
        </div>
      ))}
      <form action={handleAdd}>
        <input name="text" />
        <button>Add</button>
      </form>
    </>
  );
}
```

### use() Hook
```jsx
// ✅ NEW: Read promises and context directly in render
import { use } from 'react';

// Read promises
function UserProfile({ userPromise }) {
  const user = use(userPromise); // Suspends until resolved
  return <div>{user.name}</div>;
}

// Read context (alternative to useContext)
function MyComponent() {
  const theme = use(ThemeContext);
  return <div style={{ color: theme.primary }}>Content</div>;
}

// ✅ Can be used conditionally (unlike hooks!)
function Comments({ commentsPromise }) {
  const [showComments, setShowComments] = useState(false);
  
  return (
    <>
      <button onClick={() => setShowComments(true)}>Show Comments</button>
      {showComments && (
        <Suspense fallback={<Loading />}>
          <CommentList commentsPromise={commentsPromise} />
        </Suspense>
      )}
    </>
  );
}

function CommentList({ commentsPromise }) {
  // ✅ This is allowed! use() can be conditional
  const comments = use(commentsPromise);
  return comments.map(c => <Comment key={c.id} {...c} />);
}
```

### ref as Prop (No More forwardRef!)
```jsx
// ❌ OLD: React 18 - Need forwardRef
const Input = forwardRef((props, ref) => {
  return <input ref={ref} {...props} />;
});

// ✅ NEW: React 19 - ref is just a regular prop!
function Input({ ref, ...props }) {
  return <input ref={ref} {...props} />;
}

// Usage is the same
function Form() {
  const inputRef = useRef(null);
  return <Input ref={inputRef} />;
}
```

### Ref Cleanup Functions
```jsx
// ✅ NEW: Refs can return cleanup functions
function VideoPlayer({ src }) {
  return (
    <video
      ref={(node) => {
        if (node) {
          node.play();
          // Return cleanup function
          return () => {
            node.pause();
          };
        }
      }}
      src={src}
    />
  );
}

// Works with useRef too
useEffect(() => {
  const video = videoRef.current;
  if (video) {
    video.play();
  }
}, []);

// Now you can do this in the ref itself!
<video
  ref={(node) => {
    node?.play();
    return () => node?.pause();
  }}
/>
```

### Document Metadata
```jsx
// ✅ NEW: Use <title> and <meta> directly in components
function BlogPost({ post }) {
  return (
    <article>
      <title>{post.title} - My Blog</title>
      <meta name="description" content={post.excerpt} />
      <meta property="og:image" content={post.image} />
      
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}

// React automatically hoists these to <head>
// No need for react-helmet or Next.js Head component!

// ✅ Priority: Last rendered wins
function App() {
  return (
    <>
      <title>Default Title</title>
      <Dashboard /> {/* If Dashboard has <title>, it overrides */}
    </>
  );
}
```

### Context API Improvements
```jsx
// ✅ NEW: Context as a prop (with use hook)
import { use } from 'react';

// Can now pass context as prop
function Child({ context }) {
  const value = use(context);
  return <div>{value}</div>;
}

function Parent() {
  return <Child context={MyContext} />;
}
```

## Error Handling Essentials

### Error Boundaries (Class Component - React 18)
```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    logErrorToService(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }
    return this.props.children;
  }
}

// Usage: Wrap risky components
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

### What Error Boundaries DON'T Catch
- ❌ Event handlers (use try-catch instead)
- ❌ Async code (setTimeout, promises)
- ❌ Server-side rendering
- ❌ Errors in the error boundary itself

### Event Handler Errors
```jsx
// ✅ Use try-catch for event handlers
async function handleSubmit() {
  try {
    await submitForm(data);
  } catch (error) {
    setError(error.message);
    logError(error);
  }
}
```

## Conditional Rendering Patterns

```jsx
// ✅ Short-circuit (most common)
{isVisible && <Component />}

// ✅ Ternary (for if-else)
{isLoggedIn ? <Dashboard /> : <Login />}

// ✅ Early return (cleaner for complex conditions)
if (!user) return <Loading />;
if (error) return <Error />;
return <Dashboard user={user} />;

// ❌ AVOID: Falsy values rendering (0, NaN, "")
{items.length && <List items={items} />} // Renders "0" if empty!

// ✅ CORRECT: Be explicit with booleans
{items.length > 0 && <List items={items} />}
{Boolean(items.length) && <List items={items} />}
```

## Fragment Best Practices

```jsx
// ✅ Short syntax (when no key needed)
<>
  <Header />
  <Main />
</>

// ✅ Full syntax (when you need key prop)
<Fragment key={item.id}>
  <dt>{item.term}</dt>
  <dd>{item.description}</dd>
</Fragment>

// ✅ Fragments don't create extra DOM nodes
return (
  <>
    <h1>Title</h1>
    <p>Content</p>
  </>
);
// Renders: <h1> and <p> as siblings, no wrapper div
```

## Accessibility Quick Wins

```jsx
// ✅ Semantic HTML
<button> not <div onClick={}>
<nav>, <main>, <aside>, <header>, <footer>

// ✅ ARIA labels for screen readers
<button aria-label="Close dialog">×</button>
<img src="..." alt="Description" />

// ✅ Keyboard navigation
onKeyDown={(e) => e.key === 'Enter' && handleClick()}
tabIndex={0} // Make focusable

// ✅ Focus management
useEffect(() => {
  dialogRef.current?.focus();
}, [isOpen]);

// ✅ Use useId for unique IDs
const id = useId();
<label htmlFor={id}>Name</label>
<input id={id} />
```

## Form Handling Best Practices

```jsx
// ✅ Controlled components (React manages state)
function ControlledForm() {
  const [value, setValue] = useState('');
  
  return (
    <input 
      value={value} 
      onChange={(e) => setValue(e.target.value)} 
    />
  );
}

// ✅ Uncontrolled components (DOM manages state)
function UncontrolledForm() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const handleSubmit = () => {
    console.log(inputRef.current?.value);
  };
  
  return <input ref={inputRef} defaultValue="Initial" />;
}

// ✅ For complex forms, use React Hook Form or Formik
import { useForm } from 'react-hook-form';

function MyForm() {
  const { register, handleSubmit, formState: { errors } } = useForm();
  
  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input {...register('email', { required: true })} />
      {errors.email && <span>Required</span>}
    </form>
  );
}
```

## API/Fetch Patterns

```jsx
// ❌ BAD: Fetch in useEffect without cleanup
useEffect(() => {
  fetch('/api/data').then(res => res.json()).then(setData);
}, []);

// ✅ GOOD: Handle cleanup and errors
useEffect(() => {
  let ignore = false;
  
  async function fetchData() {
    try {
      const response = await fetch('/api/data');
      const json = await response.json();
      if (!ignore) {
        setData(json);
      }
    } catch (error) {
      if (!ignore) {
        setError(error);
      }
    }
  }
  
  fetchData();
  return () => { ignore = true; }; // Cleanup
}, []);

// ✅ BEST: Use TanStack Query or SWR for data fetching
import { useQuery } from '@tanstack/react-query';

function Users() {
  const { data, error, isLoading } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetch('/api/users').then(r => r.json())
  });
  
  if (isLoading) return <Loading />;
  if (error) return <Error />;
  return <UserList users={data} />;
}
```

## React.StrictMode (Development Tool)

```jsx
// ✅ Always wrap your app in StrictMode during development
import { StrictMode } from 'react';

root.render(
  <StrictMode>
    <App />
  </StrictMode>
);

// What it does:
// - Highlights potential problems
// - Double-invokes components/effects to catch bugs
// - Warns about deprecated APIs
// - Only runs in development (removed in production build)
```

## Event Handling Gotchas

```jsx
// ❌ BAD: Creating new function on every render
<button onClick={() => handleClick(id)}>Click</button>

// ✅ GOOD: Use useCallback for optimization
const handleClick = useCallback(() => {
  doSomething(id);
}, [id]);
<button onClick={handleClick}>Click</button>

// ✅ ALSO GOOD: For simple cases, inline is fine
<button onClick={() => setCount(count + 1)}>+</button>

// ⚠️ Synthetic Events: Event pooling removed in React 17+
// You can now safely use events in async code
async function handleClick(e) {
  e.preventDefault(); // Works fine now
  await submitForm();
  console.log(e.target); // Also works
}
```

## Children Patterns

```jsx
// ✅ Children as function (render props)
<DataProvider>
  {(data) => <Display data={data} />}
</DataProvider>

// ✅ Children type checking
interface Props {
  children: React.ReactNode; // Any valid JSX
  // OR more specific:
  children: React.ReactElement; // Single React element
  children: React.ReactElement[]; // Array of elements
  children: string; // Text only
}

// ✅ Manipulating children
import { Children, cloneElement } from 'react';

function List({ children }) {
  return Children.map(children, (child, index) => 
    cloneElement(child, { index })
  );
}
```

## TypeScript Best Practices

```jsx
// ✅ Component types
const MyComponent: React.FC<Props> = ({ children }) => { ... };
// OR (preferred - more flexible)
function MyComponent({ children }: Props) { ... }

// ✅ Event types
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => { ... };
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => { ... };
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => { ... };

// ✅ Ref types
const inputRef = useRef<HTMLInputElement>(null);
const divRef = useRef<HTMLDivElement>(null);

// ✅ State with complex types
interface User { name: string; age: number; }
const [user, setUser] = useState<User | null>(null);

// ✅ Generic components
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return <>{items.map(renderItem)}</>;
}
```

## Testing Quick Tips

```jsx
// ✅ Test user behavior, not implementation
// Use @testing-library/react

// ❌ BAD: Testing implementation details
expect(component.state.count).toBe(1);

// ✅ GOOD: Testing user-visible behavior
expect(screen.getByText('Count: 1')).toBeInTheDocument();

// ✅ Query priorities (most to least preferred)
// 1. getByRole (accessible to everyone)
// 2. getByLabelText (forms)
// 3. getByPlaceholderText (fallback for forms)
// 4. getByText (non-interactive elements)
// 5. getByTestId (last resort)

// ✅ Async testing
await waitFor(() => {
  expect(screen.getByText('Loaded')).toBeInTheDocument();
});
```

---