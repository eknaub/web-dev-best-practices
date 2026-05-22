# React Code Examples

Practical snippets that pair with [overview.md](./overview.md).

## Navigation Index

- [SOLID Principles](#solid-principles)
  - [Single Responsibility (SRP)](#single-responsibility-srp)
  - [Open/Closed (OCP)](#openclosed-ocp)
  - [Liskov Substitution (LSP)](#liskov-substitution-lsp)
  - [Interface Segregation (ISP)](#interface-segregation-isp)
  - [Dependency Inversion (DIP)](#dependency-inversion-dip)
- [State & Effects](#state--effects)
  - [Functional state updates](#functional-state-updates)
  - [Lazy initialization with useState](#lazy-initialization-with-usestate)
  - [Avoid derived state in effects](#avoid-derived-state-in-effects)
  - [Effect cleanup with AbortController](#effect-cleanup-with-abortcontroller)
  - [Stable keys for list rendering](#stable-keys-for-list-rendering)
  - [useRef for DOM and mutable values](#useref-for-dom-and-mutable-values)
  - [First render flag with custom hook](#first-render-flag-with-custom-hook-and-ref)
- [Components & Composition](#components--composition)
  - [Good vs bad keys](#good-vs-bad-keys)
  - [Explicit props API + selective prop forwarding](#explicit-props-api--selective-prop-forwarding)
  - [Default props with function parameter defaults](#default-props-with-function-parameter-defaults)
  - [Grouped into logical props](#grouped-into-logical-props)
  - [Discriminated union props for variants](#discriminated-union-props-for-variants)
  - [Children as structure, data as behavior](#children-as-structure-data-as-behavior)
  - [Render props](#render-props)
  - [Compound components with shared context](#compound-components-with-shared-context)
  - [Smart/container + dumb/presentational split](#smartcontainer--dumbpresentational-split)
  - [Prop drilling vs composition vs context](#prop-drilling-vs-composition-vs-context)
- [Headless component pattern](#headless-component-pattern)
  - [Headless compound with context](#headless-compound-with-context)
  - [Controlled/uncontrolled state hook](#controlleduncontrolled-state-hook)
  - [Slot pattern for flexible rendering](#slot-pattern-for-flexible-rendering)
- [Forms & API](#forms--api)
  - [Controlled form submit](#controlled-form-submit)
  - [Request state pattern](#request-state-pattern)
  - [React 19 form action with pending state](#react-19-form-action-with-pending-state)
- [Routing](#routing)
  - [React Router basics](#react-router-basics)
  - [useOutletContext for nested routes](#useoutletcontext-for-nested-routes)
- [API/Fetch Patterns](#apifetch-patterns)
  - [Centralized request helper](#centralized-request-helper)
  - [Cancellation for stale requests](#cancellation-for-stale-requests)
  - [Treat non-2xx as explicit errors](#treat-non-2xx-as-explicit-errors)
  - [Transport vs domain errors](#transport-vs-domain-errors)
  - [Bounded retry with backoff](#bounded-retry-with-backoff)
  - [Mirage JS setup](#mirage-js-setup)
- [Performance](#performance)
  - [Memoized components with stable props](#memoized-components-with-stable-props)
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
- [Styling & Layout](#styling--layout)
  - [Centering an element](#centering-an-element)
    - [Center horizontally + vertically (flex/grid)](#center-horizontally--vertically-flexgrid)
    - [Center a block horizontally (margin auto)](#center-a-block-horizontally-margin-auto)
    - [Center inline/text content](#center-inlinetext-content)
  - [Margin/padding shorthand](#marginpadding-shorthand)

---

## SOLID Principles

### Single Responsibility (SRP)

```tsx
type User = { id: string; name: string };

function useUsers() {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    fetch("/api/users")
      .then((response) => response.json())
      .then((data: User[]) => setUsers(data));
  }, []);

  return users;
}

function UserList() {
  const users = useUsers();
  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### Open/Closed (OCP)

```tsx
function Card({ children }: { children: React.ReactNode }) {
  return <section className="card">{children}</section>;
}

function ProductCard({
  product,
}: {
  product: { name: string; price: number };
}) {
  return (
    <Card>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
    </Card>
  );
}
```

### Liskov Substitution (LSP)

```tsx
type PrimaryButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement>;

function PrimaryButton({ className, ...rest }: PrimaryButtonProps) {
  return <button {...rest} className={`btn btn-primary ${className ?? ""}`} />;
}
```

### Interface Segregation (ISP)

```tsx
type AvatarProps = { name: string; imageUrl: string };
type UserMetaProps = { name: string; email: string };

function Avatar({ name, imageUrl }: AvatarProps) {
  return <img src={imageUrl} alt={name} />;
}

function UserMeta({ name, email }: UserMetaProps) {
  return (
    <p>
      {name} ({email})
    </p>
  );
}
```

### Dependency Inversion (DIP)

```tsx
type User = { id: string; name: string };
type UsersApi = { list: () => Promise<User[]> };

function useUsers(api: UsersApi) {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    api.list().then(setUsers);
  }, [api]);

  return users;
}
```

## State & Effects

### Functional state updates

```jsx
const [count, setCount] = useState(0);

function incrementTwice() {
  setCount((prev) => prev + 1);
  setCount((prev) => prev + 1);
}
```

### Lazy initialization with useState

```tsx
type CartItem = { id: string; quantity: number };

const [cartItems, setCartItems] = useState<CartItem[]>(() => {
  const storedCart = localStorage.getItem("cart-items");
  if (!storedCart) return [];

  try {
    return JSON.parse(storedCart) as CartItem[];
  } catch {
    return [];
  }
});
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

### useRef for DOM and mutable values

Note: `useRef` stores a mutable value in `ref.current`, and changing it does **not** notify React to re-render.
React re-renders when state/props/context change, not when a ref value changes.
If `renderCount` were tracked with `useState`, calling `setRenderCount` in an effect that runs after every render would schedule another render each time and create an infinite loop.

```jsx
function Main() {
  const [on, setOn] = React.useState(false);
  const renderCount = React.useRef(0);

  function forceRender() {
    setOn((prevOn) => !prevOn);
  }

  function incrementRenderCount() {
    renderCount.current++;
  }

  React.useEffect(() => {
    renderCount.current++;
  });

  return (
    <>
      <h3>Understanding refs</h3>
      <button onClick={forceRender}>Force re-render w/ state change</button>
      <button onClick={incrementRenderCount}>Increment Ref Count</button>
      <h4>Render count: {renderCount.current}</h4>
    </>
  );
}
```

### First render flag with custom hook and ref

```jsx
function useIsFirstRender() {
  const isFirstRenderRef = React.useRef(true);

  React.useEffect(() => {
    isFirstRenderRef.current = false;
  }, []);

  return isFirstRenderRef.current;
}
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

```tsx
interface ButtonProps {
  onClick: () => void;

  // Appearance
  appearance?: {
    variant?: "primary" | "secondary";
    size?: "sm" | "md" | "lg";
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

### Render props

Note: You can also use a regular function prop (for example, `render`) instead of function-as-children.

```tsx
function Toggle({ children }: { children: (on: boolean) => React.ReactNode }) {
  const [on, setOn] = useState(false);
  return <button onClick={() => setOn(!on)}>{children(on)}</button>;
}

<Toggle>{(on) => (on ? "On" : "Off")}</Toggle>;
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
      <Menu.Button>Sports</Menu.Button>
      <Menu.Dropdown>
        {sports.map((sport) => (
          <Menu.Item key={sport}>{sport}</Menu.Item>
        ))}
      </Menu.Dropdown>
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

## Headless component pattern

### Headless compound with context

```tsx
const ToggleContext = createContext();

function Toggle({ children }) {
  const [on, setOn] = useState(false);
  function toggle() {
    setOn((prevOn) => !prevOn);
  }
  return (
    <ToggleContext.Provider value={{ on, toggle }}>
      {children}
    </ToggleContext.Provider>
  );
}

function ToggleButton({ children }) {
  const { toggle } = useContext(ToggleContext);
  return <div onClick={toggle}>{children}</div>;
}

function ToggleOff({ children }) {
  const { on } = useContext(ToggleContext);
  return on ? null : children;
}

function ToggleOn({ children }) {
  const { on } = useContext(ToggleContext);
  return on ? children : null;
}

Toggle.Button = ToggleButton;
Toggle.On = ToggleOn;
Toggle.Off = ToggleOff;

//usage
function App() {
  return (
    <Toggle>
      <Toggle.Button>
        <Toggle.On>
          <BsStarFill className="star filled" />
        </Toggle.On>
        <Toggle.Off>
          <BsStar className="star" />
        </Toggle.Off>
      </Toggle.Button>
    </Toggle>
  );
}
```

### Controlled/uncontrolled state hook

```tsx
type UseControllableStateParams<T> = {
  value?: T;
  defaultValue: T;
  onChange?: (nextValue: T) => void;
};

function useControllableState<T>({
  value,
  defaultValue,
  onChange,
}: UseControllableStateParams<T>) {
  const [internalValue, setInternalValue] = useState<T>(defaultValue);
  const isControlled = value !== undefined;
  const currentValue = isControlled ? (value as T) : internalValue;

  const setValue = (nextValue: T) => {
    if (!isControlled) setInternalValue(nextValue);
    onChange?.(nextValue);
  };

  return [currentValue, setValue] as const;
}
```

### Slot pattern for flexible rendering

```tsx
type CardProps = {
  title: React.ReactNode;
  media?: React.ReactNode;
  actions?: React.ReactNode;
  children: React.ReactNode;
};

function Card({ title, media, actions, children }: CardProps) {
  return (
    <article>
      {media}
      <h3>{title}</h3>
      <div>{children}</div>
      {actions ? <footer>{actions}</footer> : null}
    </article>
  );
}

// usage
<Card
  title="Pro plan"
  media={<img alt="Plan preview" src="/preview.png" />}
  actions={<button type="button">Upgrade</button>}
>
  Includes analytics, alerts, and team permissions.
</Card>;
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

## Routing

### React Router basics

```tsx
import {
  BrowserRouter,
  Link,
  NavLink,
  Outlet,
  Route,
  Routes,
  useParams,
} from "react-router-dom";

function AppLayout() {
  return (
    <>
      <nav>
        <NavLink to="/">Home</NavLink>
        {" | "}
        <Link to="/products/42">Product 42</Link>
      </nav>
      <Outlet />
    </>
  );
}

function ProductDetails() {
  const { productId } = useParams<{ productId: string }>();
  return <h2>Product: {productId}</h2>;
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route element={<AppLayout />}>
          <Route index element={<p>Home</p>} />
          <Route path="products/:productId" element={<ProductDetails />} />
        </Route>
      </Routes>
    </BrowserRouter>
  );
}
```

### useOutletContext for nested routes

```tsx
import {
  BrowserRouter,
  NavLink,
  Outlet,
  Route,
  Routes,
  useOutletContext,
} from "react-router-dom";

type DashboardContext = {
  currentUserName: string;
  canManageBilling: boolean;
};

function DashboardLayout() {
  const contextValue: DashboardContext = {
    currentUserName: "Ada",
    canManageBilling: true,
  };

  return (
    <>
      <nav>
        <NavLink to="/dashboard">Overview</NavLink>
        {" | "}
        <NavLink to="/dashboard/billing">Billing</NavLink>
      </nav>
      <Outlet context={contextValue} />
    </>
  );
}

function DashboardOverview() {
  const { currentUserName } = useOutletContext<DashboardContext>();
  return <h2>Welcome back, {currentUserName}</h2>;
}

function BillingPage() {
  const { canManageBilling } = useOutletContext<DashboardContext>();

  return canManageBilling ? (
    <button type="button">Open billing portal</button>
  ) : (
    <p>You do not have billing access.</p>
  );
}

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/dashboard" element={<DashboardLayout />}>
          <Route index element={<DashboardOverview />} />
          <Route path="billing" element={<BillingPage />} />
        </Route>
      </Routes>
    </BrowserRouter>
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

### Mirage JS setup

```ts
// src/mocks/server.ts
import { createServer, Model, Response } from "miragejs";

export function makeServer({ environment = "development" } = {}) {
  return createServer({
    environment,
    models: {
      user: Model,
    },
    seeds(server) {
      server.create("user", { id: "1", name: "Ada" });
      server.create("user", { id: "2", name: "Grace" });
    },
    routes() {
      this.namespace = "api";

      this.get("/users", (schema) => {
        return schema.user.all();
      });

      this.get("/users/:id", (schema, request) => {
        const user = schema.user.find(request.params.id);
        if (!user) {
          return new Response(404, {}, { message: "User not found" });
        }
        return user;
      });
    },
  });
}
```

```ts
// src/main.tsx (or src/index.tsx)
if (import.meta.env.DEV) {
  const { makeServer } = await import("./mocks/server");
  makeServer();
}
```

```tsx
// Example fetch usage with Mirage-backed endpoints
import { useEffect, useState } from "react";

type User = { id: string; name: string };

function UsersPanel() {
  const [users, setUsers] = useState<User[]>([]);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    fetch("/api/users")
      .then(async (response) => {
        if (!response.ok) {
          throw new Error(`Request failed with status ${response.status}`);
        }

        const payload = await response.json();
        // Mirage collection shape: { users: User[] }
        setUsers(payload.users ?? []);
      })
      .catch((requestError) => {
        setError((requestError as Error).message);
      });
  }, []);

  if (error) return <p role="alert">{error}</p>;
  return <pre>{JSON.stringify(users, null, 2)}</pre>;
}
```

## Performance

### Memoized components with stable props

```jsx
function UserRow({ user, onDelete }) {
  console.log("render row", user.id);

  return (
    <li>
      <span>{user.name}</span>
      <button onClick={() => onDelete(user.id)}>Delete</button>
    </li>
  );
}

const MemoizedUserRow = memo(UserRow);

const handleDelete = useCallback((id) => {
  setUsers((prev) => prev.filter((use r) => user.id !== id));
}, []);

return (
  <ul>
    {users.map((user) => (
      <MemoizedUserRow key={user.id} user={user} onDelete={handleDelete} />
    ))}
  </ul>
);
```

### Memoize expensive derived values

Note: When passing an object (or array) as a prop to a memoized component, use `useMemo` to maintain referential equality. Without it, a new object is created on every render, causing the memoized child to re-render even if the logical value hasn't changed.

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

## Styling & Layout

### Centering an element

Pick the method based on what you need to center.

#### Center horizontally + vertically (flex/grid)

```html
<div class="parent">
  <div class="child">Centered</div>
</div>
```

```css
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}

/* Alternative */
.parent-grid {
  display: grid;
  place-items: center;
  min-height: 100vh;
}
```

#### Center a block horizontally (margin auto)

```css
.child {
  display: block;
  width: 300px;
  margin: 0 auto;
}
```

#### Center inline/text content

```html
<div class="text-parent">
  <span class="badge">Centered inline content</span>
</div>
```

```css
.text-parent {
  text-align: center;
}
```

### Margin/padding shorthand

```css
/* 1 value: all sides */
.box {
  margin: 16px;
  padding: 12px;
}

/* 2 values: vertical | horizontal */
.box {
  margin: 12px 24px;
  padding: 8px 16px;
}

/* 3 values: top | horizontal | bottom */
.box {
  margin: 8px 20px 12px;
  padding: 4px 12px 16px;
}

/* 4 values (clockwise): top | right | bottom | left */
.box {
  margin: 8px 16px 12px 20px;
  padding: 6px 10px 14px 18px;
}
```

```css
/* Axis-based logical properties */
.box {
  margin-block: 12px; /* top + bottom */
  margin-inline: 24px; /* left + right in LTR */
  padding-block: 8px;
  padding-inline: 16px;
}
```
