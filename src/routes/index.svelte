<script lang="ts">
	import type { LayoutConfig } from 'golden-layout';
	import '../lib/css/themes/goldenlayout-light-theme.css';

	import GoldenLayout from '../lib';
	import Test from './_Test.svelte';

	let display = true;
	let columns = 2;

	const components = { Test };
	const names = ['foo', 'bar', 'baz'];

	let layout: LayoutConfig;

	$: layout = {
		root: {
			type: 'row',
			content: Array.from({ length: columns }, (_, i) => {
				return {
					type: 'component',
					componentType: 'Test',
					componentState: {
						name: names[i % names.length],
					},
				};
			}),
		},
	};
</script>

<main>
	<div>
		<h1>Config</h1>
		<p>
			<input type="checkbox" bind:checked={display} /> display
			<input type="number" min="0" max="10" bind:value={columns} /> columns
		</p>
	</div>
	<div>
		<h1>Demo</h1>
		<div class="layout-container">
			{#if display}
				<GoldenLayout config={layout} let:componentType let:componentState>
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

	.layout-container {
		height: 600px;

		border: 1px solid black;
	}
</style>
