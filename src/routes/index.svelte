<script lang="ts">
	import { writable, Writable } from 'svelte/store';
	import { propertyStore } from 'svelte-writable-derived';

	import type { LayoutConfig, ResolvedLayoutConfig, VirtualLayout } from 'golden-layout';
	import '../lib/css/themes/goldenlayout-light-theme.css';

	import GoldenLayout from '../lib';
	import Test from './_Test.svelte';

	const names = ['foo', 'bar', 'baz'];
	const files = writable<{ [name: string]: string }>({
		foo: '',
		bar: '',
		baz: '',
	});

	function fileStore(name: string): Writable<string> {
		return propertyStore(files, name);
	}

	let display = true;
	let rows = 4;

	const components = { Test };

	let layout: LayoutConfig;
	let saved: ResolvedLayoutConfig = undefined;

	let goldenLayout: VirtualLayout;

	$: layout = {
		root: {
			type: 'column',
			content: Array.from({ length: rows }, (_, i) => {
				const name = names[i % names.length];

				return {
					type: 'component',
					title: name,
					componentType: 'Test',
					componentState: {
						name,
						// file: propertyStore(files, name),
					},
				};
			}),
		},
	};

	function handleSave() {
		saved = goldenLayout.saveLayout();
	}

	function handleRestore() {
		layout = saved as unknown as LayoutConfig;
	}
</script>

<main class:popout={goldenLayout?.isSubWindow}>
	<div>
		<h1>Config</h1>
		<p>
			<input type="checkbox" bind:checked={display} /> display
			<input type="number" min="0" max="10" bind:value={rows} /> columns
			<button on:click={handleSave}>Save Layout</button>
			<button on:click={handleRestore} disabled={saved === undefined}>Restore Layout</button>
		</p>
		{#each Object.entries($files) as [name, content]}
			<p>{name}: {content}</p>
		{/each}
		<h2>Saved Layout</h2>
		{#if saved !== undefined}
			<pre>{JSON.stringify(saved, undefined, 2)}</pre>
		{:else}
			<p>(none)</p>
		{/if}
	</div>
	<div>
		<h1>Demo</h1>
		<div class="layout-container">
			{#if display}
				<GoldenLayout config={layout} bind:goldenLayout let:componentType let:componentState>
					<svelte:component this={components[componentType]} {...componentState} />
				</GoldenLayout>
			{/if}
		</div>
	</div>
</main>

<style>
	main {
		display: flex;
		flex-direction: row;
	}

	main > div {
		width: 50%;
		padding: 2rem;
	}

	input[type='number'] {
		width: 3.5em;
	}

	main:not(.popout) .layout-container {
		height: 600px;

		border: 1px solid black;
	}

	/* popout styling */

	main.popout {
		visibility: hidden;
	}

	main.popout .layout-container {
		visibility: visible;

		position: fixed;
		inset: 0;
	}
</style>
