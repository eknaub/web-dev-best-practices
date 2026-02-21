# React Code Examples

Practical snippets that pair with [overview.md](./overview.md).

## Navigation Index

- [State & Effects](#state--effects)
  - [Functional state updates](#functional-state-updates)
  - [Avoid derived state in effects](#avoid-derived-state-in-effects)
  - [Effect cleanup with AbortController](#effect-cleanup-with-abortcontroller)
  - [Stable keys for list rendering](#stable-keys-for-list-rendering)
- [Components & Composition](#components--composition)
  - [Good vs bad keys](#good-vs-bad-keys)
  - [Explicit props API + selective prop forwarding](#explicit-props-api--selective-prop-forwarding)
  - [Default props with function parameter defaults](#default-props-with-function-parameter-defaults)
  - [Grouped into logical props](#grouped-into-logical-props)
  - [Discriminated union props for variants](#discriminated-union-props-for-variants)
  - [Children as structure, data as behavior](#children-as-structure-data-as-behavior)
  - [Compound components with shared context](#compound-components-with-shared-context)
  - [Smart/container + dumb/presentational split](#smartcontainer--dumbpresentational-split)
  - [Prop drilling vs composition vs context](#prop-drilling-vs-composition-vs-context)
- [Forms & API](#forms--api)
  - [Controlled form submit](#controlled-form-submit)
  - [Request state pattern](#request-state-pattern)
  - [React 19 form action with pending state](#react-19-form-action-with-pending-state)
- [API/Fetch Patterns](#apifetch-patterns)
  - [Centralized request helper](#centralized-request-helper)
  - [Cancellation for stale requests](#cancellation-for-stale-requests)
  - [Treat non-2xx as explicit errors](#treat-non-2xx-as-explicit-errors)
  - [Transport vs domain errors](#transport-vs-domain-errors)
  - [Bounded retry with backoff](#bounded-retry-with-backoff)
- [Performance](#performance)
  - [Memoize expensive derived values](#memoize-expensive-derived-values)
  - [Stable callbacks for memoized children](#stable-callbacks-for-memoized-children)
  - [Lazy-load heavy modules](#lazy-load-heavy-modules)
- [Accessibility & TypeScript](#accessibility--typescript)
  - [Label/input association and error messaging](#labelinput-association-and-error-messaging)
  - [Typed props with explicit interfaces](#typed-props-with-explicit-interfaces)
- [Error Handling & Rendering](#error-handling--rendering)
  - [Error boundary for UI failures](#error-boundary-for-ui-failures)
  - [Early-return conditional rendering](#early-return-conditional-rendering)
  - [Safe async event handler with try/catch](#safe-async-event-handler-with-trycatch)

---

## State & Effects

### Functional state updates

```jsx
const [count, setCount] = useState(0);

function incrementTwice() {
  setCount((prev) => prev + 1);
  setCount((prev) => prev + 1);
}
```

### Avoid derived state in effects

```jsx
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");

const fullName = `${firstName} ${lastName}`.trim();
```

### Effect cleanup with AbortController

```jsx
useEffect(() => {
  const controller = new AbortController();

  async function loadUser() {
    const response = await fetch(`/api/users/${userId}`, {
      signal: controller.signal,
    });
    const data = await response.json();
    setUser(data);
  }

  loadUser().catch((error) => {
    if (error.name !== "AbortError") {
      setError(error);
    }
  });

  return () => controller.abort();
}, [userId]);
```

### Stable keys for list rendering

```jsx
return (
  <ul>
    {items.map((item) => (
      <ItemRow key={item.id} item={item} />
    ))}
  </ul>
);
```

## Components & Composition

### Good vs bad keys

```jsx
// ✅ Good: stable key from data
users.map((user) => <UserCard key={user.id} user={user} />);

// ❌ Avoid for dynamic lists
users.map((user, index) => <UserCard key={index} user={user} />);
```

### Explicit props API + selective prop forwarding

```tsx
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary";
}

function Button({
  variant = "primary",
  type = "button",
  ...rest
}: Readonly<ButtonProps>) {
  return (
    <button type={type} data-variant={variant} {...rest}>
      {rest.children}
    </button>
  );
}

// User can pass any attribute (HTML props and custom props)
<Button variant="primary" onClick={handleClick} disabled aria-label="Submit" />;
```

### Default props with function parameter defaults

```tsx
interface BadgeProps {
  label: string;
  tone?: "neutral" | "success" | "danger";
  rounded?: boolean;
}

function Badge({
  label,
  tone = "neutral",
  rounded = false,
}: Readonly<BadgeProps>) {
  return (
    <span data-tone={tone} data-rounded={rounded}>
      {label}
    </span>
  );
}
```

### Grouped into logical props

```jsx
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

### Discriminated union props for variants

```tsx
type ActionButtonProps =
  | { variant: "submit"; label: string; onClick: () => void }
  | { variant: "link"; label: string; href: string }
  | { variant: "icon"; icon: React.ReactNode; onClick: () => void };

function ActionButton(props: Readonly<ActionButtonProps>) {
  switch (props.variant) {
    case "submit":
      return <button onClick={props.onClick}>{props.label}</button>;
    case "link":
      return <a href={props.href}>{props.label}</a>;
    case "icon":
      return <button onClick={props.onClick}>{props.icon}</button>;
  }
}
```

### Children as structure, data as behavior

```tsx
interface CardProps {
  title: string;
  children: React.ReactNode;
}

function Card({ title, children }: Readonly<CardProps>) {
  return (
    <section>
      <h2>{title}</h2>
      <div>{children}</div>
    </section>
  );
}

<Card title="Profile">
  <ProfileDetails />
</Card>;
```

### Compound components with shared context

```jsx
const MenuContext = createContext(null);

function Menu({ children }) {
  const [open, setOpen] = useState(false);
  const toggle = () => setOpen((prev) => !prev);
  return (
    <MenuContext.Provider value={{ open, toggle }}>
      {children}
    </MenuContext.Provider>
  );
}

function MenuButton({ children }) {
  const { toggle } = useContext(MenuContext);
  return <button onClick={toggle}>{children}</button>;
}

function MenuDropdown({ children }) {
  const { open } = useContext(MenuContext);
  return open ? <div>{children}</div> : null;
}

function MenuItem({ children }) {
  return <div className="menu-item">{children}</div>;
}

// Dot Syntax (in explicit file = single import)
// Attach sub-components using dot notation
Menu.Button = MenuButton;
Menu.Dropdown = MenuDropdown;
Menu.Item = MenuItem;

// Usage
function App() {
  const sports = ["Tennis", "Pickleball", "Racquetball", "Squash"];

  return (
    <Menu>
      <Menu.Button>Sports</MenuButton>
      <Menu.Dropdown>
        {sports.map((sport) => (
          <Menu.Item key={sport}>{sport}</MenuItem>
        ))}
      </MenuDropdown>
    </Menu>
  );
}
```

### Smart/container + dumb/presentational split

```tsx
type User = { id: string; name: string; email: string };

function UserListContainer() {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    fetch("/api/users")
      .then((res) => res.json())
      .then((data: User[]) => setUsers(data));
  }, []);

  const handleDelete = (id: string) => {
    setUsers((prev) => prev.filter((user) => user.id !== id));
  };

  return <UserList users={users} onDelete={handleDelete} />;
}

function UserList({
  users,
  onDelete,
}: {
  users: User[];
  onDelete: (id: string) => void;
}) {
  return users.map((user) => (
    <button key={user.id} onClick={() => onDelete(user.id)}>
      {user.name}
    </button>
  ));
}
```

### Prop drilling vs composition vs context

```jsx
// Props (good for shallow trees)
function Profile({ user }) {
  return <UserName user={user} />;
}

// Composition (skip intermediates)
function App({ user }) {
  return <UserList>{user && <UserProfile user={user} />}</UserList>;
}

// Context (many distant consumers)
const UserContext = createContext(null);
function UserNameFromContext() {
  const user = useContext(UserContext);
  return <span>{user?.name}</span>;
}
```

## Forms & API

### Controlled form submit

```jsx
function ProfileForm() {
  const [name, setName] = useState("");
  const [isSaving, setIsSaving] = useState(false);

  async function handleSubmit(event) {
    event.preventDefault();
    setIsSaving(true);
    try {
      await saveProfile({ name });
    } finally {
      setIsSaving(false);
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={(event) => setName(event.target.value)} />
      <button type="submit" disabled={isSaving || !name.trim()}>
        {isSaving ? "Saving..." : "Save"}
      </button>
    </form>
  );
}
```

### Request state pattern

```jsx
const [status, setStatus] = useState("idle"); // idle | loading | success | error
const [data, setData] = useState(null);
const [error, setError] = useState(null);

async function fetchProducts() {
  setStatus("loading");
  setError(null);
  try {
    const response = await fetch("/api/products");
    const result = await response.json();
    setData(result);
    setStatus("success");
  } catch (requestError) {
    setError(requestError);
    setStatus("error");
  }
}
```

### React 19 form action with pending state

```jsx
import { useActionState } from "react";

async function createUser(previousState, formData) {
  const response = await fetch("/api/users", {
    method: "POST",
    body: JSON.stringify({ name: formData.get("name") }),
    headers: { "Content-Type": "application/json" },
  });

  if (!response.ok) {
    return { ok: false, message: "Could not create user" };
  }

  return { ok: true, message: "User created" };
}

function CreateUserForm() {
  const [state, formAction, isPending] = useActionState(createUser, {
    ok: true,
    message: "",
  });

  return (
    <form action={formAction}>
      <input name="name" required />
      <button type="submit" disabled={isPending}>
        {isPending ? "Creating..." : "Create"}
      </button>
      {state.message ? <p>{state.message}</p> : null}
    </form>
  );
}
```

## API/Fetch Patterns

### Centralized request helper

```ts
class ApiError extends Error {
  status: number;
  details?: unknown;

  constructor(message: string, status: number, details?: unknown) {
    super(message);
    this.name = "ApiError";
    this.status = status;
    this.details = details;
  }
}

async function apiRequest<T>(
  input: RequestInfo,
  init: RequestInit = {},
): Promise<T> {
  const response = await fetch(input, {
    headers: {
      "Content-Type": "application/json",
      ...init.headers,
    },
    ...init,
  });

  if (!response.ok) {
    const details = await response.json().catch(() => null);
    throw new ApiError("Request failed", response.status, details);
  }

  return response.json() as Promise<T>;
}
```

### Cancellation for stale requests

```tsx
function ProductSearch({ query }: { query: string }) {
  const [products, setProducts] = useState<{ id: string; name: string }[]>([]);

  useEffect(() => {
    if (!query.trim()) {
      setProducts([]);
      return;
    }

    const controller = new AbortController();

    apiRequest<{ id: string; name: string }[]>(
      `/api/products?query=${encodeURIComponent(query)}`,
      { signal: controller.signal },
    )
      .then(setProducts)
      .catch((error) => {
        if ((error as Error).name !== "AbortError") {
          console.error(error);
        }
      });

    return () => controller.abort();
  }, [query]);

  return <ProductList items={products} />;
}
```

### Treat non-2xx as explicit errors

```ts
async function getProfile() {
  const response = await fetch("/api/profile");

  if (!response.ok) {
    throw new Error(`Profile request failed with status ${response.status}`);
  }

  return response.json();
}
```

### Transport vs domain errors

```ts
type ServiceResult<T> =
  | { ok: true; data: T }
  | { ok: false; kind: "domain"; message: string };

async function createOrder(payload: { items: string[] }) {
  try {
    const data = await apiRequest<{ orderId?: string; message?: string }>(
      "/api/orders",
      {
        method: "POST",
        body: JSON.stringify(payload),
      },
    );

    if (!data.orderId) {
      return {
        ok: false,
        kind: "domain",
        message: data.message ?? "Order could not be created",
      } as ServiceResult<never>;
    }

    return { ok: true, data: { orderId: data.orderId } } as ServiceResult<{
      orderId: string;
    }>;
  } catch (error) {
    if (error instanceof TypeError) {
      throw new Error("Network error. Check your connection and try again.");
    }
    throw error;
  }
}
```

### Bounded retry with backoff

```ts
const sleep = (ms: number) => new Promise((resolve) => setTimeout(resolve, ms));

async function fetchWithRetry<T>(
  input: RequestInfo,
  init: RequestInit = {},
  maxAttempts = 3,
): Promise<T> {
  for (let attempt = 1; attempt <= maxAttempts; attempt += 1) {
    try {
      return await apiRequest<T>(input, init);
    } catch (error) {
      const isLastAttempt = attempt === maxAttempts;
      if (isLastAttempt) throw error;

      const backoffMs = 300 * 2 ** (attempt - 1);
      await sleep(backoffMs);
    }
  }

  throw new Error("Unexpected retry flow");
}
```

## Performance

### Memoize expensive derived values

```jsx
const filteredUsers = useMemo(() => {
  return users.filter((user) =>
    user.name.toLowerCase().includes(query.toLowerCase()),
  );
}, [users, query]);
```

### Stable callbacks for memoized children

```jsx
const handleDelete = useCallback(
  (id) => {
    setItems((prev) => prev.filter((item) => item.id !== id));
  },
  [setItems],
);

return <UserList items={items} onDelete={handleDelete} />;
```

### Lazy-load heavy modules

```jsx
const ReportsPage = lazy(() => import("./ReportsPage"));

return (
  <Suspense fallback={<Spinner />}>
    <ReportsPage />
  </Suspense>
);
```

## Accessibility & TypeScript

### Label/input association and error messaging

```jsx
const inputId = useId();
const errorId = `${inputId}-error`;

return (
  <div>
    <label htmlFor={inputId}>Email</label>
    <input
      id={inputId}
      aria-invalid={Boolean(error)}
      aria-describedby={error ? errorId : undefined}
    />
    {error ? (
      <p id={errorId} role="alert">
        {error}
      </p>
    ) : null}
  </div>
);
```

### Typed props with explicit interfaces

```tsx
interface UserCardProps {
  id: string;
  name: string;
  onSelect: (id: string) => void;
}

export function UserCard({ id, name, onSelect }: UserCardProps) {
  return <button onClick={() => onSelect(id)}>{name}</button>;
}
```

## Error Handling & Rendering

### Error boundary for UI failures

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError() {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    reportError(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <p>Something went wrong. Please refresh.</p>;
    }
    return this.props.children;
  }
}
```

### Early-return conditional rendering

```jsx
function ProductPanel({ status, error, products }) {
  if (status === "loading") return <Spinner />;
  if (status === "error") return <ErrorMessage message={error.message} />;
  if (!products.length) return <EmptyState title="No products" />;

  return <ProductList items={products} />;
}
```

### Safe async event handler with try/catch

```jsx
async function handleSave() {
  try {
    await saveSettings();
    toast.success("Saved");
  } catch (error) {
    toast.error("Save failed");
    console.error(error);
  }
}
```
