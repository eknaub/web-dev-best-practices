# Advanced Svelte Best Practices

## Navigation Index

- [Reusing Content](#reusing-content)
- [Advanced Reactivity](#advanced-reactivity)
- [State vs Raw State](#state-vs-raw-state)
- [Derived vs Derived by](#derived-vs-derived-by)
- [Reactive Classes](#reactive-classes)
- [Reactive Built-ins](#reactive-built-ins)
- [Stores Update Patterns](#stores-update-patterns)
- [Motion](#motion)
- [Advanced Bindings](#advanced-bindings)
- [Advanced Transitions](#advanced-transitions)
- [Context API](#context-api)
- [Special Elements](#special-elements)
- [Script Module](#script-module)

## Reusing Content

Use snippets for reusable UI fragments and `{@render ...}` for placement. Components can receive snippets explicitly as props, and can also use implicit snippet props, including `children`.

```svelte
<!-- CardList.svelte -->
<script>
	// explicit snippet prop
	let { items = [], row, children } = $props();
</script>

<section class="card-list">
	<header>{@render children?.()}</header>

	{#each items as item (item.id)}
		<article>{@render row(item)}</article>
	{/each}
</section>
```

```svelte
<!-- App.svelte -->
<script>
	import CardList from './CardList.svelte';

	let users = $state([
		{ id: 1, name: 'Ada', role: 'admin' },
		{ id: 2, name: 'Linus', role: 'editor' }
	]);
</script>

<CardList items={users}>
	<!-- implicit snippet prop named children -->
	<h3>Team Members</h3>

	<!-- implicit snippet prop named row (matches CardList prop name) -->
	{#snippet row(user)}
		<strong>{user.name}</strong>
		<small> ({user.role})</small>
	{/snippet}
</CardList>
```

Why this pattern works:

- `CardList` owns structure and behavior.
- Parent owns the rendered content.
- One component API supports many visual variants.

## Advanced Reactivity

Advanced reactivity is mostly about control: what should be deeply tracked, what should stay plain, and where reactive logic should live.

## State vs Raw State

Use $state for deep reactivity (object and array mutations are tracked).
Use $state.raw when you want reassignment-only updates and less proxy overhead.

```svelte
<script>
	let user = $state({ name: 'Ada', score: 1 });
	let bigList = $state.raw([{ id: 1 }, { id: 2 }]);

	function updateState() {
		user.score += 1; // tracked
	}

	function updateRaw() {
		// bigList.push({ id: 3 }) would not be the update pattern you want here
		bigList = [...bigList, { id: 3 }]; // reassignment pattern
	}
</script>
```

Quick rule:

- $state: frequent nested mutation.
- $state.raw: large mostly-replaced data blobs.

## Derived vs Derived by

$derived(expression) is for short, direct formulas.
$derived.by(() => { ... }) is for multi-step logic.

```svelte
<script>
	let price = $state(120);
	let tax = $state(0.2);
	let discounts = $state([5, 10]);

	let gross = $derived(price * (1 + tax));

	let net = $derived.by(() => {
		let totalDiscount = 0;
		for (const d of discounts) totalDiscount += d;
		return gross - totalDiscount;
	});
</script>

<p>Gross: {gross}</p>
<p>Net: {net}</p>
```

Equivalent mental model:

- $derived(a + b) ~= $derived.by(() => a + b)

## Reactive Classes

Class instances are not proxied. Put $state/$derived on class fields.

```svelte
<script>
	class CartItem {
		// public reactive fields
		name = $state('Keyboard');
		qty = $state(1);
		price = $state(89);

		// private reactive field
		#currency = $state('EUR');

		// derived class field
		total = $derived(this.qty * this.price);

		// accessor (public API)
		get label() {
			return `${this.name} (${this.#currency})`;
		}

		set currency(next) {
			this.#currency = next;
		}

		// arrow keeps this bound when passed as callback
		increment = () => {
			this.qty += 1;
		};

		reset() {
			this.qty = 1;
		}
	}

	let item = new CartItem();
</script>

<p>{item.label}</p>
<p>Total: {item.total}</p>
<button onclick={item.increment}>+</button>
<button onclick={() => item.reset()}>Reset</button>
```

What to remember:

- Prefer small class APIs (methods/getters) over exposing internal details.
- If passing normal methods to events, wrap them or use arrow methods.

## Reactive Built-ins

Svelte provides reactive versions of built-ins in svelte/reactivity.

```svelte
<script>
	import { SvelteDate, SvelteMap } from 'svelte/reactivity';

	const now = new SvelteDate();
	const counts = new SvelteMap();

	$effect(() => {
		const id = setInterval(() => now.setTime(Date.now()), 1000);
		return () => clearInterval(id);
	});

	function vote(key) {
		counts.set(key, (counts.get(key) ?? 0) + 1);
	}
</script>

<p>{now.toLocaleTimeString()}</p>
<button onclick={() => vote('svelte')}>Svelte: {counts.get('svelte') ?? 0}</button>
```

Also available:

- SvelteSet
- SvelteURL
- SvelteURLSearchParams
- MediaQuery

## Stores Update Patterns

With writable stores, you can update in three styles.

```svelte
<script>
	import { writable } from 'svelte/store';

	const count = writable(0);

	function withSet() {
		count.set(10);
	}

	function withUpdate() {
		count.update((n) => n + 1);
	}

	function withDollar() {
		$count += 1;
	}
</script>

<p>Count: {$count}</p>
<button onclick={withSet}>set(10)</button>
<button onclick={withUpdate}>update(+1)</button>
<button onclick={withDollar}>$count += 1</button>
```

When to use:

- set: replace with known value.
- update: next value depends on previous value.
- $store shorthand: concise in component code.

## Motion

Use motion for continuously changing values and transitions for enter/leave. They work well together.

```svelte
<script>
	import { Spring, prefersReducedMotion } from 'svelte/motion';
	import { fly } from 'svelte/transition';

	let open = $state(false);
	const x = new Spring(0, { stiffness: 0.15, damping: 0.8 });

	function toggle() {
		open = !open;
		x.target = open ? 140 : 0;
	}
</script>

<button onclick={toggle}>Toggle panel</button>

{#if open}
	<aside
		transition:fly={{ y: prefersReducedMotion.current ? 0 : 18, duration: 220 }}
		style:transform={`translateX(${x.current}px)`}
	>
		Motion drives position, transition handles mount/unmount
	</aside>
{/if}
```

Quick rule:

- Motion (`Spring`/`Tween`): animate value changes over time.
- Transition (`transition:`/`in:`/`out:`): animate element lifecycle.

## Advanced Bindings

### Contenteditable bindings

Use contenteditable bindings when editing rich or plain text in place.

```svelte
<script>
	let html = $state('<b>Hello</b>');
	let text = $state('');
</script>

<div contenteditable bind:innerHTML={html}></div>
<div contenteditable bind:textContent={text}></div>
```

### Each block bindings

Bind to object properties inside each blocks, not to temporary loop primitives.

```svelte
<script>
	let todos = $state([
		{ id: 1, text: 'Read docs' },
		{ id: 2, text: 'Build demo' }
	]);
</script>

{#each todos as todo (todo.id)}
	<input bind:value={todo.text} />
{/each}
```

### Media elements

Media bindings are great for custom players.

```svelte
<script>
	let currentTime = $state(0);
	let duration = $state(0);
	let paused = $state(true);
	let volume = $state(0.8);
</script>

<video
	src="/media/demo.mp4"
	controls
	bind:currentTime
	bind:duration
	bind:paused
	bind:volume
></video>

<p>{Math.round(currentTime)} / {Math.round(duration)}s</p>
```

### Dimensions

Use dimension bindings for measurement-driven UI.

```svelte
<script>
	let clientWidth = $state(0);
	let clientHeight = $state(0);
</script>

<div bind:clientWidth bind:clientHeight class="panel">Resize me</div>
<p>{clientWidth} x {clientHeight}</p>
```

### This

Use `bind:this` for direct element references.

```svelte
<script>
	let inputEl;
</script>

<input bind:this={inputEl} />
<button onclick={() => inputEl?.focus()}>Focus</button>
```

### Component bindings

To make parent-to-child two-way binding work, the child prop should be bindable.

```svelte
<!-- Counter.svelte -->
<script>
	let { value = $bindable(0) } = $props();
</script>

<button onclick={() => (value += 1)}>{value}</button>
```

```svelte
<!-- App.svelte -->
<script>
	import Counter from './Counter.svelte';
	let count = $state(0);
</script>

<Counter bind:value={count} />
```

### Binding to component instances

Use `bind:this` on components when you need a tiny imperative API.

```svelte
<!-- VideoPlayer.svelte -->
<script>
	let videoEl;

	export function play() {
		videoEl?.play();
	}
</script>

<video bind:this={videoEl} src="/media/demo.mp4" controls></video>
```

```svelte
<!-- App.svelte -->
<script>
	import VideoPlayer from './VideoPlayer.svelte';
	let player;
</script>

<VideoPlayer bind:this={player} />
<button onclick={() => player?.play()}>Play</button>
```

## Advanced Transitions

### Deferred transitions with crossfade (send/receive)

`crossfade` creates a matched `send`/`receive` pair. When an element leaves one place and appears in another, they coordinate to animate as if the element physically moved between the two locations.

### animate:flip

`animate:flip` runs whenever a keyed each block reorders. It calculates each element's old and new position and smoothly interpolates between them.

### Combined example: moving items between two lists

This is the canonical use-case for both — moving a todo between lists triggers `crossfade` on the item that jumps, while `animate:flip` smoothly slides remaining items to fill the gap.

```svelte
<script>
  import { crossfade } from 'svelte/transition';
  import { flip } from 'svelte/animate';

  const [send, receive] = crossfade({ duration: 300 });

  let todo = $state([
    { id: 1, text: 'Write tests' },
    { id: 2, text: 'Fix bug' }
  ]);
  let done = $state([
    { id: 3, text: 'Deploy' }
  ]);

  function move(item, from, to) {
    from.splice(from.indexOf(item), 1);
    to.push(item);
  }
</script>

<div class="cols">
  <ul>
    {#each todo as item (item.id)}
      <li
        animate:flip={{ duration: 250 }}
        in:receive={{ key: item.id }}
        out:send={{ key: item.id }}
      >
        {item.text}
        <button onclick={() => move(item, todo, done)}>Done</button>
      </li>
    {/each}
  </ul>

  <ul>
    {#each done as item (item.id)}
      <li
        animate:flip={{ duration: 250 }}
        in:receive={{ key: item.id }}
        out:send={{ key: item.id }}
      >
        {item.text}
        <button onclick={() => move(item, done, todo)}>Undo</button>
      </li>
    {/each}
  </ul>
</div>
```

What each part does:

- `send` — outro transition, hands the element off to its counterpart.
- `receive` — intro transition, catches the element and animates it into place.
- `key` — the shared ID that matches the two sides together.
- `animate:flip` — repositions surrounding elements that shift after the move.

## Context API

Use `setContext` and `getContext` to share values across a component tree without prop-drilling. Context is synchronous and scoped to the component subtree where it is set.

```svelte
<!-- ThemeProvider.svelte -->
<script>
  import { setContext } from 'svelte';

  let { theme = 'light', children } = $props();

  // Reactive object — consumers stay in sync when theme changes
  const ctx = { get theme() { return theme; } };
  setContext('theme', ctx);
</script>

{@render children()}
```

```svelte
<!-- Button.svelte -->
<script>
  import { getContext } from 'svelte';

  const { theme } = getContext('theme');
</script>

<button class={theme}>Click</button>
```

```svelte
<!-- App.svelte -->
<script>
  import ThemeProvider from './ThemeProvider.svelte';
  import Button from './Button.svelte';

  let theme = $state('dark');
</script>

<ThemeProvider {theme}>
  <Button />
</ThemeProvider>
```

Key points:

- Context is not global — it is scoped to the subtree below the component that calls `setContext`.
- `getContext` must be called at component init time, not inside effects or event handlers.
- Pass a reactive object (with getters) so consumers react to changes, not a plain snapshot.

## Special Elements

### svelte:head

Inject content into `<head>` from any component — great for page titles and meta tags.

```svelte
<svelte:head>
  <title>My Page</title>
  <meta name="description" content="Best page ever" />
</svelte:head>
```

### svelte:window, svelte:document, svelte:body

Attach event listeners or bindings to global browser targets without `addEventListener` in effects.

```svelte
<script>
  let scrollY = $state(0);
  let online = $state(true);
</script>

<svelte:window bind:scrollY />
<svelte:document onvisibilitychange={() => console.log(document.visibilityState)} />
<svelte:body class="dark-mode" />
```

### svelte:element

Render an element whose tag is not known until runtime (e.g. driven by a CMS).

```svelte
<script>
  let tag = $state('h2');
</script>

<svelte:element this={tag}>Dynamic heading</svelte:element>
```

### svelte:component

Render a component whose type is resolved at runtime.

```svelte
<script>
  import IconA from './IconA.svelte';
  import IconB from './IconB.svelte';

  let active = $state(true);
  let Icon = $derived(active ? IconA : IconB);
</script>

<svelte:component this={Icon} size={24} />
```

### svelte:boundary

Isolate rendering errors and show a fallback. Works with async pending states too.

```svelte
<svelte:boundary>
  <RiskyWidget />

  {#snippet pending()}
    <p>Loading...</p>
  {/snippet}

  {#snippet failed(error, reset)}
    <p>{error.message}</p>
    <button onclick={reset}>Retry</button>
  {/snippet}
</svelte:boundary>
```

## Script Module

Code in `<script module>` runs once when the module is first imported, not per component instance. Use it for shared state, one-time setup, or named exports that live outside the component lifecycle.

```svelte
<!-- AudioPlayer.svelte -->
<script module>
  // Shared across all instances — only one player active at a time
  let currentPlayer = $state(null);

  export function stopAll() {
    currentPlayer?.pause();
    currentPlayer = null;
  }
</script>

<script>
  let audioEl;

  function play() {
    if (currentPlayer && currentPlayer !== audioEl) {
      currentPlayer.pause();
    }
    currentPlayer = audioEl;
    audioEl.play();
  }
</script>

<audio bind:this={audioEl} src="/media/track.mp3"></audio>
<button onclick={play}>Play</button>
```

```svelte
<!-- App.svelte -->
<script>
  import AudioPlayer, { stopAll } from './AudioPlayer.svelte';
</script>

<AudioPlayer />
<AudioPlayer />
<button onclick={stopAll}>Stop all</button>
```

Key points:

- Runs once per module import, not once per instance.
- Can export values and functions that callers import like any JS module.
- Cannot access instance-level state (`$props`, `$state` in the regular `<script>`) directly.
