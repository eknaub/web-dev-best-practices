# Svelte Best Practices

## Navigation Index

- [Reactivity](#reactivity)
- [Props and Composition](#props-and-composition)
- [Control Flow](#control-flow)
- [DOM Events](#dom-events)
- [Bindings](#bindings)
- [Class and Style](#class-and-style)
- [Attachments](#attachments)
- [Transitions](#transitions)
- [Practical Checklist](#practical-checklist)
- [Advanced Svelte Next](./advanced-svelte.md)

## Reactivity

Svelte reactivity is compile-time driven. Think "state-first template updates" without hook dependency arrays.

### State and Derived State

```svelte
<script>
  let count = $state(0);
  let todos = $state([{ id: 1, text: 'Learn Svelte', done: false }]);

  let doneCount = $derived(todos.filter((t) => t.done).length);

  function toggleTodo(id) {
    todos = todos.map((t) => (t.id === id ? { ...t, done: !t.done } : t));
  }
</script>

<button onclick={() => (count += 1)}>Count: {count}</button>
<p>Completed: {doneCount}</p>
```

### Effects and Inspect

```svelte
<script>
  let query = $state('');

  $effect(() => {
    if (query.length > 2) {
      console.log('searching for', query);
    }
  });

  $inspect(query, 'query');
</script>
```

## Props and Composition

### Props

```svelte
<!-- Parent.svelte -->
<script>
  import UserCard from './UserCard.svelte';

  let title = 'Profile';
  function onSave() {
    console.log('saved');
  }
</script>

<UserCard {title} onSave={onSave} />
```

```svelte
<!-- UserCard.svelte -->
<script>
  let { title = 'Untitled', onSave } = $props();
</script>

<h2>{title}</h2>
<button onclick={onSave}>Save</button>
```

### Spread Props

```svelte
<!-- Parent.svelte -->
<script>
  import Panel from './Panel.svelte';

  let panelProps = { id: 'main-panel', class: 'panel', role: 'region' };
</script>

<Panel {...panelProps} />
```

## Control Flow

### If and Each

```svelte
<script>
  let isAdmin = $state(false);
  let items = $state([
    { id: 1, name: 'A' },
    { id: 2, name: 'B' }
  ]);
</script>

{#if isAdmin}
  <p>Admin panel</p>
{:else}
  <p>Guest view</p>
{/if}

{#each items as item (item.id)}
  <li>{item.name}</li>
{/each}
```

### Await

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

## DOM Events

```svelte
<script>
  let count = $state(0);
  function increment() {
    count += 1;
  }
</script>

<button onclick={increment}>Increment</button>
<form onsubmit|preventDefault={() => console.log('submit')}>
  <button>Submit</button>
</form>
```

## Bindings

### bind:value (including numeric)

```svelte
<script>
  let name = $state('');
  let age = $state(18); // number input stays numeric
</script>

<input bind:value={name} placeholder="Name" />
<input type="number" bind:value={age} min="0" />
<input type="range" bind:value={age} min="0" max="100" />
```

### Checkboxes, Radios, and Select Multiple

```svelte
<script>
  let accepted = $state(false);
  let tags = $state([]);
  let plan = $state('starter');
  let country = $state('pt');
  let frameworks = $state(['svelte']);
</script>

<label><input type="checkbox" bind:checked={accepted} /> Accept terms</label>

<label><input type="checkbox" bind:group={tags} value="html" /> HTML</label>
<label><input type="checkbox" bind:group={tags} value="css" /> CSS</label>

<label><input type="radio" bind:group={plan} value="starter" /> Starter</label>
<label><input type="radio" bind:group={plan} value="pro" /> Pro</label>

<select bind:value={country}>
  <option value="pt">Portugal</option>
  <option value="br">Brazil</option>
</select>

<select bind:value={frameworks} multiple>
  <option value="svelte">Svelte</option>
  <option value="react">React</option>
  <option value="vue">Vue</option>
</select>
```

## Class and Style

### class shortform (string, array, object)

Svelte normalizes class values similarly to clsx.

```svelte
<script>
  let isActive = $state(true);
  let size = $state('lg');
  const base = 'btn';
</script>

<button class={[base, `btn-${size}`, { 'is-active': isActive }]}>Save</button>
```

### style attribute and style directive

```svelte
<script>
  let progress = $state(65);
  let x = $state(24);
</script>

<div style={`width:${progress}%; height:8px; background:#0ea5e9;`}></div>
<div style:transform={`translateX(${x}px)`} style:opacity={0.9}>Panel</div>
```

### Component styles via CSS variable

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

## Attachments

Attachments are reactive mount/update/unmount behavior.

### attach tag

```svelte
<script>
  function logMount(node) {
    console.log('Mounted', node.nodeName);
    return () => console.log('Unmounted', node.nodeName);
  }
</script>

<div {@attach logMount}>Tracked element</div>
```

### Attachment factory

```svelte
<script>
  let message = $state('Hello!');

  function tooltip(content) {
    return (node) => {
      node.setAttribute('title', content);
      return () => node.removeAttribute('title');
    };
  }
</script>

<input bind:value={message} />
<button {@attach tooltip(message)}>Hover me</button>
```

## Transitions

### transition directive (with and without params)

```svelte
<script>
  import { fade, fly } from 'svelte/transition';
  let visible = $state(true);
</script>

<button onclick={() => (visible = !visible)}>Toggle</button>

{#if visible}
  <p transition:fade>Default fade</p>
  <p transition:fly={{ x: 20, duration: 250 }}>Fly with params</p>
{/if}
```

### in and out transitions

```svelte
<script>
  import { fade, fly } from 'svelte/transition';
  let show = $state(false);
</script>

{#if show}
  <section in:fly={{ y: 16 }} out:fade={{ duration: 140 }}>Card</section>
{/if}
```

### Custom CSS and JS transitions

```svelte
<script>
  function pop(node, { duration = 220 } = {}) {
    return {
      duration,
      css: (t) => `transform:scale(${0.9 + 0.1 * t});opacity:${t};`
    };
  }

  function draw(node, { duration = 300 } = {}) {
    return {
      duration,
      tick: (t) => {
        node.style.opacity = String(t);
        node.style.clipPath = `inset(${(1 - t) * 100}% 0 0 0)`;
      }
    };
  }
</script>

<div transition:pop>CSS transition</div>
<div transition:draw>JS transition</div>
```

### Transition events, global transitions, and key blocks

```svelte
<script>
  import { fade } from 'svelte/transition';
  let show = $state(false);
  let outer = $state(true);
  let step = $state(0);
  let status = $state('idle');
</script>

{#if show}
  <div
    transition:fade
    onintrostart={() => (status = 'intro started')}
    onintroend={() => (status = 'intro ended')}
    onoutrostart={() => (status = 'outro started')}
    onoutroend={() => (status = 'outro ended')}
  >
    Transition events
  </div>
{/if}

{#if outer}
  <p transition:fade|global>Transitions on parent block changes too</p>
{/if}

{#key step}
  <h3 transition:fade>Step {step}</h3>
{/key}
```

## Practical Checklist

- Keep state minimal and local first.
- Prefer derived values over duplicated state.
- Keep `$effect` for side effects, not data derivation.
- Use `bind:` for form sync and event handlers for intent-driven actions.
- Use attachments for mount/update/unmount behavior.
- Use `transition:` for symmetric enter/leave and `in:`/`out:` for asymmetric behavior.
