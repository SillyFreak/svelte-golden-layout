<script lang="ts">
	import type { LayoutConfig } from 'golden-layout';
	import '../lib/css/themes/goldenlayout-light-theme.css';

	import GoldenLayout from '../lib';
	import Test from '../test/Test.svelte';

	const components = { Test };

	const layout: LayoutConfig = {
		root: {
			type: 'row',
			content: [
				{
					type: 'component',
					componentType: 'Test',
				},
				{
					type: 'component',
					componentType: 'Test',
				},
			],
		},
	};
</script>

<div class="layout-container">
	<GoldenLayout {components} config={layout} />
</div>

<style>
	.layout-container {
		width: 800px;
		height: 600px;

		margin: 150px auto;
		border: 1px solid black;
	}
</style>
