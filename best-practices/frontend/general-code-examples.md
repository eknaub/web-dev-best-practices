# Frontend General Code Examples

Practical snippets that pair with [general.md](./general.md).

## Navigation Index

- [SOLID Principles](#solid-principles)
  - [Single Responsibility (SRP)](#single-responsibility-srp)
  - [Open/Closed (OCP)](#openclosed-ocp)
  - [Liskov Substitution (LSP)](#liskov-substitution-lsp)
  - [Interface Segregation (ISP)](#interface-segregation-isp)
  - [Dependency Inversion (DIP)](#dependency-inversion-dip)
- [Forms & API](#forms--api)
  - [Controlled form submit](#controlled-form-submit)
  - [Request state pattern](#request-state-pattern)
  - [React 19 form action with pending state](#react-19-form-action-with-pending-state)
- [Error Handling & Rendering](#error-handling--rendering)
  - [Error boundary for UI failures](#error-boundary-for-ui-failures)
  - [Early-return conditional rendering](#early-return-conditional-rendering)
  - [Safe async event handler with try/catch](#safe-async-event-handler-with-trycatch)
- [API/Fetch Patterns](#apifetch-patterns)
  - [Centralized request helper](#centralized-request-helper)
  - [Cancellation for stale requests](#cancellation-for-stale-requests)
  - [Treat non-2xx as explicit errors](#treat-non-2xx-as-explicit-errors)
  - [Transport vs domain errors](#transport-vs-domain-errors)
  - [Bounded retry with backoff](#bounded-retry-with-backoff)
  - [Mirage JS setup](#mirage-js-setup)
- [Accessibility & TypeScript](#accessibility--typescript)
  - [Text color contrast](#text-color-contrast)
  - [Alt text for informative and decorative images](#alt-text-for-informative-and-decorative-images)
  - [ARIA live region for async status updates](#aria-live-region-for-async-status-updates)
  - [Hiding content accessibly](#hiding-content-accessibly)
  - [Descriptive links with visible underline](#descriptive-links-with-visible-underline)
  - [Input labels and placeholder as example text](#input-labels-and-placeholder-as-example-text)
  - [Fieldset and legend for grouped controls](#fieldset-and-legend-for-grouped-controls)
  - [Landmark regions page shell](#landmark-regions-page-shell)
  - [Use rem for font sizing](#use-rem-for-font-sizing)
  - [Heading hierarchy (one h1, no skipped levels)](#heading-hierarchy-one-h1-no-skipped-levels)
  - [Label/input association and error messaging](#labelinput-association-and-error-messaging)
  - [Typed props with explicit interfaces](#typed-props-with-explicit-interfaces)
- [Styling & Layout](#styling--layout)
  - [Centering an element](#centering-an-element)
    - [Center horizontally + vertically (flex/grid)](#center-horizontally--vertically-flexgrid)
    - [Center a block horizontally (margin auto)](#center-a-block-horizontally-margin-auto)
    - [Center inline/text content](#center-inlinetext-content)
    - [Block, inline, and inline-block](#block-inline-and-inline-block)
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

## Accessibility & TypeScript

### Text color contrast

```css
:root {
  --text-primary: #1b1f24;
  --text-muted: #44515f;
  --surface: #ffffff;
  --focus-ring: #0b63ce;
}

body {
  color: var(--text-primary); /* strong contrast on white */
  background-color: var(--surface);
}

.help-text {
  color: var(--text-muted); /* still readable, not too faint */
}

a:focus-visible,
button:focus-visible,
input:focus-visible {
  outline: 2px solid var(--focus-ring);
  outline-offset: 2px;
}
```

### Alt text for informative and decorative images

```jsx
function ProductMedia() {
  return (
    <section>
      {/* Informative image: alt describes purpose */}
      <img
        src="/images/wireless-headphones.jpg"
        alt="Wireless headphones with noise cancellation"
      />

      {/* Decorative image: empty alt hides it from screen readers */}
      <img src="/images/divider-wave.svg" alt="" role="presentation" />
    </section>
  );
}
```

### ARIA live region for async status updates

```jsx
function SaveProfileButton() {
  const [statusMessage, setStatusMessage] = useState("");

  async function handleSave() {
    setStatusMessage("Saving profile...");

    try {
      await saveProfile();
      setStatusMessage("Profile saved successfully.");
    } catch {
      setStatusMessage("Save failed. Please try again.");
    }
  }

  return (
    <div>
      <button type="button" onClick={handleSave}>
        Save profile
      </button>

      {/* Polite live region for non-blocking updates */}
      <p role="status" aria-live="polite" aria-atomic="true">
        {statusMessage}
      </p>
    </div>
  );
}
```

### Hiding content accessibly

```css
/* Visible only to screen readers */
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

```jsx
function CheckoutSummary({ cartCount }: { cartCount: number }) {
	return (
		<section>
			{/* Hidden from everyone when not needed */}
			<p hidden={cartCount === 0}>You have items ready to checkout.</p>

			{/* Hidden visually, but announced by screen readers */}
			<p className="sr-only">Cart has {cartCount} items.</p>

			{/* Decorative icon only */}
			<span aria-hidden="true">Cart</span>
		</section>
	);
}
```

### Descriptive links with visible underline

```jsx
<p>
  Read the <a href="/docs/password-policy">password policy requirements</a>
  before creating an account.
</p>
```

```css
a {
  text-decoration: underline;
  text-underline-offset: 2px;
}

a:hover,
a:focus-visible {
  text-decoration-thickness: 2px;
}
```

### Input labels and placeholder as example text

```jsx
function ContactForm() {
  return (
    <form>
      <label htmlFor="phone">Phone number</label>
      <input id="phone" name="phone" placeholder="Example: +1 555 123 4567" />

      <label htmlFor="company">Company website</label>
      <input
        id="company"
        name="company"
        placeholder="Example: https://acme.com"
      />
    </form>
  );
}
```

### Fieldset and legend for grouped controls

```jsx
function ContactPreference() {
  return (
    <fieldset>
      <legend>Preferred contact method</legend>

      <label>
        <input type="radio" name="contact" value="email" /> Email
      </label>
      <label>
        <input type="radio" name="contact" value="phone" /> Phone
      </label>
    </fieldset>
  );
}
```

### Landmark regions page shell

```jsx
function PageLayout() {
  return (
    <>
      <header>
        <h1>Account settings</h1>
      </header>

      <nav aria-label="Primary">
        <a href="/settings/profile">Profile</a>
        <a href="/settings/security">Security</a>
      </nav>

      <main>
        <section aria-labelledby="security-title">
          <h2 id="security-title">Security settings</h2>
          <p>Manage your password, 2FA, and active sessions.</p>
        </section>
      </main>

      <footer>
        <small>Last updated: June 2026</small>
      </footer>
    </>
  );
}
```

### Use rem for font sizing

```css
html {
  font-size: 100%; /* usually 16px */
}

body {
  font-size: 1rem;
  line-height: 1.5;
}

h1 {
  font-size: 2rem; /* scales with user preference */
}

.small-note {
  font-size: 0.875rem;
}
```

### Heading hierarchy (one h1, no skipped levels)

```jsx
function ArticlePage() {
  return (
    <main>
      <h1>Checkout guide</h1>
      <h2>Shipping information</h2>
      <h3>International delivery windows</h3>

      <h2>Payment options</h2>
      <h3>Saved cards</h3>
    </main>
  );
}
```

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

#### Block, inline, and inline-block

```html
<div class="layout-demo">
  <span class="demo-inline">Inline</span>
  <span class="demo-inline-block">Inline-block</span>
  <div class="demo-block">Block</div>
</div>
```

```css
.demo-inline {
  display: inline;
}

.demo-inline-block {
  display: inline-block;
  width: 140px;
  margin-top: 12px;
  margin-bottom: 12px;
  padding: 8px 12px;
  overflow: hidden;
}

.demo-block {
  display: block;
  width: 140px;
  margin-top: 12px;
  padding: 8px 12px;
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
