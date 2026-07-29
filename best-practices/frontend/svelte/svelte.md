# Svelte Best Practices

## Reactivity

### State

Use state for reactive values that change over time and directly affect the UI.

- Prefer local state for component-specific data.
- Keep state close to where it is used.
- Use simple primitives first before introducing more complex structures.

```svelte
<script>
  import { writable } from 'svelte/store';

  let count = 0;
</script>
```

### Deep State

Deep state refers to nested objects or arrays that should remain reactive as a whole.

- Update nested values carefully to preserve reactivity.
- Prefer immutable updates for objects and arrays when possible.
- Avoid overusing deeply nested state when a flatter structure is simpler.

```svelte
<script>
  let user = $state({
    name: 'Ada',
    profile: { city: 'London' }
  });
</script>
```

### Derived State

Derived state is computed from other state and should not be duplicated as independent source-of-truth data.

- Compute values from existing state instead of storing them separately.
- Use derived values for formatting, filtering, sorting, or aggregation.
- Keep derived logic simple so it stays predictable and easy to test.

```svelte
<script>
  let todos = $state([]);
  let completedTodos = $derived(todos.filter(todo => todo.done));
</script>
```

### Inspect Rune

Use the Inspect rune to debug reactive values during development.

- It helps you see how state changes over time.
- Use it for tracing updates and understanding reactivity flows.
- Avoid relying on it for business logic or leaving debug logging in production code.

```svelte
<script>
  let count = $state(0);

  $inspect(count, 'count');
</script>
```

### Effect Rune

Use the Effect rune for side effects that should run in response to reactive changes.

- Keep effects focused on one responsibility.
- Clean up subscriptions, timers, and event listeners when the effect ends.
- Prefer derived state over effects when you can compute a value declaratively.

```svelte
<script>
  let count = $state(0);

  $effect(() => {
    console.log('Count changed:', count);
  });
</script>
```

### Props

Use props to pass data from parents to children in a clear and predictable way.

- Keep props focused and minimal; prefer a small interface over a large object.
- Use descriptive prop names and default values where helpful.
- Avoid mutating props inside child components.
- Pass callbacks as props when the child needs to communicate back to the parent.

```svelte
<!-- Parent.svelte -->
<script>
  import Child from './Child.svelte';

  let title = 'Hello';
</script>

<Child {title} />
```

```svelte
<!-- Child.svelte -->
<script>
  let { title } = $props();
</script>

<p>{title}</p>
```

Functions can also be passed as props when the child needs to trigger behavior in the parent.

```svelte
<!-- Parent.svelte -->
<script>
  import Child from './Child.svelte';

  function handleSave() {
    console.log('Saved');
  }
</script>

<Child onSave={handleSave} />
```

```svelte
<!-- Child.svelte -->
<script>
  let { onSave } = $props();
</script>

<button onclick={onSave}>Save</button>
```

You do not need to destructure props. You can access them directly from the props object as well.

```svelte
<!-- Child.svelte -->
<script>
  let props = $props();
</script>

<p>{props.title}</p>
```

Event handlers can also be spread through props when you want to forward several callbacks at once.

```svelte
<!-- Parent.svelte -->
<script>
  import Child from './Child.svelte';

  function handleSave() {
    console.log('Saved');
  }

  function handleCancel() {
    console.log('Canceled');
  }

  let handlers = {
    onclick: handleSave,
    oncancel: handleCancel
  };
</script>

<Child {...handlers} />
```

#### Spread Props

Use spread props when you want to forward a set of attributes or props to a child component without listing each one manually.

```svelte
<!-- Parent.svelte -->
<script>
  import Child from './Child.svelte';

  let props = {
    id: 'user-card',
    class: 'card',
    title: 'Profile'
  };
</script>

<Child {...props} />
```

#### Default Props

Use default values for props when a child component can still render sensibly without every prop being provided.

```svelte
<!-- Child.svelte -->
<script>
  let { title = 'Untitled' } = $props();
</script>

<p>{title}</p>
```

### Control Flow

Use control-flow blocks to keep templates clear and expressive.

#### If, Else, and Else If

Use conditional blocks when UI should change based on a boolean or a small branching condition.

```svelte
<script>
  let isAdmin = $state(false);
</script>

{#if isAdmin}
  <p>Admin panel</p>
{:else if !isAdmin}
  <p>Guest view</p>
{:else}
  <p>Unknown role</p>
{/if}
```

#### Each

Use each blocks for rendering lists. Keep list rendering simple and avoid complex logic inside the template.

```svelte
<script>
  let todos = $state(['Learn Svelte', 'Write docs']);
</script>

{#each todos as todo}
  <li>{todo}</li>
{/each}
```

#### Unique Key in Each

Use a stable unique key when rendering dynamic lists so Svelte can preserve DOM identity and avoid unnecessary re-renders.

```svelte
<script>
  let todos = $state([
    { id: 1, text: 'Learn Svelte' },
    { id: 2, text: 'Write docs' }
  ]);
</script>

{#each todos as todo (todo.id)}
  <li>{todo.text}</li>
{/each}
```

#### Await, Then, and Catch

Use await blocks for asynchronous values such as promises or data loading. Handle loading and error states explicitly.

```svelte
<script>
  async function loadUser() {
    return { name: 'Ada' };
  }

  let userPromise = loadUser();
</script>

{#await userPromise}
  <p>Loading...</p>
{:then user}
  <p>Hello {user.name}</p>
{:catch error}
  <p>Failed: {error.message}</p>
{/await}
```

### DOM Events and Inline Handlers

Use DOM event handlers to respond to user interactions in a simple and readable way.

- Prefer named functions for complex logic instead of long inline expressions.
- Keep inline handlers short and focused.
- Use event modifiers such as `preventDefault` and `stopPropagation` when appropriate.

```svelte
<script>
  let count = $state(0);

  function increment() {
    count += 1;
  }
</script>

<button onclick={increment}>Increment</button>
<button onclick={() => (count += 1)}>Inline</button>
```

#### Capturing Events

Use event capturing when you want to react to an event before it reaches the element that triggered it. With capture, the outer element runs first and then the button. Without capture, the event bubbles up from the button to the outer element.

```svelte
<script>
  function handleCapture(event) {
    console.log('Captured:', event.type);
  }
</script>

<div oncapture:click={handleCapture}>
  <button>Click me</button>
</div>
```

### Bindings

Use bindings to keep form elements and component state in sync with less boilerplate.

#### bind:value

`bind:value` creates two-way binding between form fields and state.

- Great for text inputs, textareas, and selects.
- For `type="number"` and `type="range"`, Svelte keeps the value as a numeric type for you.

```svelte
<script>
  let name = $state('');
  let age = $state(18);
</script>

<input bind:value={name} placeholder="Name" />
<input type="number" bind:value={age} min="0" />

<p>{name} is {age} years old.</p>
```

#### Checkbox

Use `bind:checked` for a single boolean checkbox.

```svelte
<script>
  let accepted = $state(false);
</script>

<label>
  <input type="checkbox" bind:checked={accepted} />
  Accept terms
</label>

<p>Accepted: {accepted ? 'yes' : 'no'}</p>
```

#### Checkbox Group

Use `bind:group` with checkboxes to collect multiple selected values in an array.

```svelte
<script>
  let selectedTags = $state([]);
</script>

<label><input type="checkbox" bind:group={selectedTags} value="html" /> HTML</label>
<label><input type="checkbox" bind:group={selectedTags} value="css" /> CSS</label>
<label><input type="checkbox" bind:group={selectedTags} value="svelte" /> Svelte</label>

<p>Selected: {selectedTags.join(', ') || 'none'}</p>
```

#### Radio Group

Use `bind:group` with radios to keep one selected value.

```svelte
<script>
  let plan = $state('starter');
</script>

<label><input type="radio" bind:group={plan} value="starter" /> Starter</label>
<label><input type="radio" bind:group={plan} value="pro" /> Pro</label>
<label><input type="radio" bind:group={plan} value="team" /> Team</label>

<p>Plan: {plan}</p>
```

#### Select and Select Multiple

Use `bind:value` for single select and an array state for `<select multiple>`.

```svelte
<script>
  let country = $state('pt');
  let frameworks = $state(['svelte']);
</script>

<select bind:value={country}>
  <option value="pt">Portugal</option>
  <option value="br">Brazil</option>
  <option value="jp">Japan</option>
</select>

<select bind:value={frameworks} multiple>
  <option value="svelte">Svelte</option>
  <option value="react">React</option>
  <option value="vue">Vue</option>
</select>

<p>Country: {country}</p>
<p>Frameworks: {frameworks.join(', ')}</p>
```

### Class and Style

Use Svelte class and style features to express UI states clearly without verbose string concatenation.

#### class Shortform

Svelte accepts strings, arrays, and objects in the `class` attribute and normalizes them similarly to `clsx`.

- String values are added directly.
- Arrays combine multiple class values.
- Object keys are included when their value is truthy.

```svelte
<script>
  let isActive = $state(true);
  let isDisabled = $state(false);
  let size = $state('lg');

  const base = 'btn';
</script>

<button
  class={[
    base,
    `btn-${size}`,
    { 'is-active': isActive, 'is-disabled': isDisabled }
  ]}
>
  Save
</button>
```

#### style Attribute

Use inline `style` when values are dynamic and scoped to one element.

```svelte
<script>
  let progress = $state(65);
</script>

<div
  style={`width:${progress}%; background:linear-gradient(90deg, #0ea5e9, #22c55e); height:12px; border-radius:999px;`}
></div>
```

#### style Directive

Use `style:property={value}` for better readability when setting one or a few dynamic style properties.

```svelte
<script>
  let x = $state(24);
  let y = $state(12);
</script>

<div
  style:transform={`translate(${x}px, ${y}px)`}
  style:opacity={0.9}
>
  Floating panel
</div>
```

#### Component Styles with CSS Variables

Use CSS custom properties to theme a child component from its parent without tightly coupling styles.

```svelte
<!-- Parent.svelte -->
<script>
  import Badge from './Badge.svelte';

  let brandColor = $state('#f59e0b');
</script>

<Badge style={`--color:${brandColor}`}>Pro Plan</Badge>
```

```svelte
<!-- Badge.svelte -->
<span class="badge"><slot /></span>

<style>
  .badge {
    background-color: var(--color, #64748b);
    color: white;
    padding: 0.25rem 0.5rem;
    border-radius: 999px;
  }
</style>
```

### Attachments

Use attachments to run setup and cleanup logic when an element mounts, updates, and unmounts.

#### attach Tag

Use `{@attach ...}` to attach behavior to an element. The attachment can return a cleanup function.

```svelte
<script>
  function logMount(element) {
    console.log('Mounted:', element.nodeName);

    return () => {
      console.log('Unmounted:', element.nodeName);
    };
  }
</script>

<div {@attach logMount}>Tracked element</div>
```

#### Attachment Factory

Use an attachment factory when behavior depends on reactive input. When factory inputs change, Svelte re-runs the attachment.

```svelte
<script>
  let message = $state('Hello!');

  function tooltip(content) {
    return (element) => {
      element.setAttribute('title', content);

      return () => {
        element.removeAttribute('title');
      };
    };
  }
</script>

<input bind:value={message} />

<button {@attach tooltip(message)}>
  Hover me
</button>
```

### Transitions

Use transitions to animate elements entering and leaving the DOM in a declarative way.

#### transition Directive

Use `transition:name` when the same effect should run for both intro and outro.

```svelte
<script>
  import { fade } from 'svelte/transition';

  let visible = $state(true);
</script>

<button onclick={() => (visible = !visible)}>Toggle</button>

{#if visible}
  <p transition:fade>Fades in and out</p>
{/if}
```

You can pass parameters to configure the transition.

```svelte
<script>
  import { fly } from 'svelte/transition';

  let open = $state(false);
</script>

<button onclick={() => (open = !open)}>Toggle panel</button>

{#if open}
  <aside transition:fly={{ x: 24, duration: 250 }}>
    Sliding panel
  </aside>
{/if}
```

#### in and out Transitions

Use `in:` and `out:` when intro and outro effects should differ.

```svelte
<script>
  import { fade, fly } from 'svelte/transition';

  let show = $state(false);
</script>

<button onclick={() => (show = !show)}>Toggle card</button>

{#if show}
  <section in:fly={{ y: 16, duration: 220 }} out:fade={{ duration: 140 }}>
    Intro and outro are different
  </section>
{/if}
```

#### Custom CSS Transitions

Return a `css` function for transitions that can be expressed with interpolated CSS.

```svelte
<script>
  function pop(node, { duration = 220 } = {}) {
    return {
      duration,
      css: (t) => `
        transform: scale(${0.9 + 0.1 * t});
        opacity: ${t};
      `
    };
  }

  let active = $state(false);
</script>

<button onclick={() => (active = !active)}>Toggle badge</button>

{#if active}
  <span transition:pop>New</span>
{/if}
```

#### Custom JS Transitions

Use `tick` for transitions that need imperative updates every frame.

```svelte
<script>
  function draw(node, { duration = 350 } = {}) {
    return {
      duration,
      tick: (t) => {
        node.style.opacity = String(t);
        node.style.clipPath = `inset(${(1 - t) * 100}% 0 0 0)`;
      }
    };
  }

  let reveal = $state(false);
</script>

<button onclick={() => (reveal = !reveal)}>Toggle reveal</button>

{#if reveal}
  <div transition:draw>
    Revealed by JS transition
  </div>
{/if}
```

#### Transition Events

Use transition events to react to lifecycle moments.

```svelte
<script>
  import { fade } from 'svelte/transition';

  let show = $state(false);
  let status = $state('idle');
</script>

<button onclick={() => (show = !show)}>Toggle</button>

{#if show}
  <div
    transition:fade
    onintrostart={() => (status = 'intro started')}
    onintroend={() => (status = 'intro ended')}
    onoutrostart={() => (status = 'outro started')}
    onoutroend={() => (status = 'outro ended')}
  >
    Watch console and status
  </div>
{/if}

<p>Status: {status}</p>
```

#### Global Transitions

Use `|global` when transitions should also run for parent block changes, not only local insertion/removal.

```svelte
<script>
  import { fade } from 'svelte/transition';

  let outer = $state(true);
  let inner = $state(true);
</script>

<button onclick={() => (outer = !outer)}>Toggle outer</button>
<button onclick={() => (inner = !inner)}>Toggle inner</button>

{#if outer}
  {#if inner}
    <p transition:fade|global>Also transitions when outer block toggles</p>
  {/if}
{/if}
```

#### Key Blocks

Use key blocks to force an element subtree to be recreated when a key changes, so transitions replay cleanly.

```svelte
<script>
  import { fade } from 'svelte/transition';

  let count = $state(0);
</script>

<button onclick={() => (count += 1)}>Next</button>

{#key count}
  <h2 transition:fade>Step {count}</h2>
{/key}
```

### Practical Guidelines

- Keep state minimal and focused on the current feature.
- Use derived state for computed values to avoid synchronization bugs.
- Prefer clear ownership: local component state for local concerns, stores for shared app-wide state.
- If state becomes hard to reason about, split it into smaller pieces instead of nesting everything deeply.
- Use runes outside components for shared reactivity when multiple components need to coordinate around the same reactive state.
