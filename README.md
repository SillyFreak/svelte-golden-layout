# svelte-golden-layout

[GoldenLayout 2.x](https://github.com/golden-layout/golden-layout) Wrapper for Svelte. The `GoldenLayout` component accepts child content (i.e. a slot) that renders each tab.

This was extracted from a personal project, so its initial scope was rather limited. Expect rough edges, but bug reports and contributions are welcome!

[**Live Demo**](https://sillyfreak.github.io/svelte-golden-layout/)

**Features**

- pass the layout (`config`) via props
- render tab content via the default [slot](https://svelte.dev/docs#slot_let); `let:id`, `let:componentType`, and `let:componentState` contain the specific tab's content
- automatic resizing of the layout within its containing HTML element

**Known limitations**

- changing the layout recreates the whole thing, i.e. new components are created for each tab
- popout of tabs does not work yet
- dragging the tabset dividers results in text selection

## Installation & development

This library can be installed from NPM:

```sh
# from NPM
npm i svelte-golden-layout
```

For development, clone the repo and add the dependency as shown:

```sh
# clone this repo and package the library
git clone https://github.com/SillyFreak/svelte-golden-layout
cd svelte-golden-layout
npm i
npm run package

# add it & golden-layout to your own project
cd ../your-own-project
npm i golden-layout ../svelte-golden-layout/package
```

## simple example

You can run this repo as an app ([live demo](https://sillyfreak.github.io/svelte-golden-layout/)); the example code is in [index.svelte](src/routes/index.svelte) and [Test.svelte](src/routes/Test.svelte), and demonstates both features and shortcomings of the library.

In the less complete example below, `svelte:component` is used to select a specific component for each tab, and the `componentState` object is passed as props to that component:

```svelte
<script lang="ts">
	import 'svelte-golden-layout/css/themes/goldenlayout-light-theme.css';

	import GoldenLayout from 'svelte-golden-layout';
	// ... import a Test component

	const components = { Test };

	const layout = {
		root: {
			type: 'row',
			content: [
				{
					type: 'component',
					componentType: 'Test',
					componentState: {
						someProp: 1,
						anotherProp: 1,
					},
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
	<GoldenLayout config={layout} let:componentType let:componentState>
		<svelte:component this={components[componentType]} {...componentState} />
	</GoldenLayout>
</div>

<style>
	.layout-container {
		width: 800px;
		height: 600px;

		margin: 150px auto;
		border: 1px solid black;
	}
</style>
```
