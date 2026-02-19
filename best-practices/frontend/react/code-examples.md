# React Code Examples

Practical snippets that pair with [overview.md](./overview.md).

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
