<script lang="ts">
	import type { LayoutConfig } from 'golden-layout';
	import '../lib/css/themes/goldenlayout-light-theme.css';

	import GoldenLayout from '../lib';
	import Test from '../test/Test.svelte';

	let display = true;
	let columns = 2;

	const components = { Test };
	const names = ['foo', 'bar', 'baz'];

	let layout: LayoutConfig;

	$: layout = {
		root: {
			type: 'row',
			content: Array.from({ length: columns }, (_, i) => {
				const componentState: any = {};
				componentState.name = names[i % names.length];
				if (i == 0) componentState.extraClass = 'bold';

				return {
					type: 'component',
					componentType: 'Test',
					componentState,
				};
			}),
		},
	};
</script>

<main>
	<p>
		<input type="checkbox" bind:checked={display} /> display
		<input type="number" min="0" max="10" bind:value={columns} /> columns
	</p>
	<div class="layout-container">
		{#if display}
			<GoldenLayout {components} config={layout} />
		{/if}
	</div>
</main>

<style>
	main {
		width: 800px;
		margin: 150px auto;
	}

	input[type='number'] {
		width: 3.5em;
	}

	.layout-container {
		width: 800px;
		height: 600px;

		border: 1px solid black;
	}

	:global(.bold) {
		font-weight: bold;
	}
</style>
