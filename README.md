# svelte-golden-layout

[GoldenLayout 2.x](https://github.com/golden-layout/golden-layout) Wrapper for Svelte. The `GoldenLayout` component accepts an object mapping names to `SvelteComponent` constructors, which are instantiated for every tab. The layout contains the component names, and any props to pass to the components.

This was extracted from a personal project, so its initial scope was rather limited. Expect rough edges, but bug reports and contributions are welcome!

**Features**

- pass component types (`components`) and layout (`config`) via props
- pass props to components in the layout
- pass a CSS class to the container element of a component
- automatic resizing of the layout within its containing HTML element

**Known limitations**

- changing the layout recreates the whole thing, i.e. new components are created for each tab
	- this includes props, as props are given within the layout JSON
- components are not destroyed when tabs are closed or the layout is recreated (this might be an upstream GoldenLayout bug)

This means that right now, the `config` prop should better not be changed after the fact, and that there is a lifecycle problem and memory leak of individual tabs. It's a starting point!

## Installation & development

An **npm package** does not exist yet. If reception is good, I will publish this and develop it further.

```sh
# clone this repo and package the library
git clone https://github.com/SillyFreak/svelte-golden-layout
cd svelte-golden-layout
npm i
npm run package

# add it to your own project
cd ../your-own-project
npm i ../svelte-golden-layout/package
```

## simple example

```svelte
<script lang="ts">
	import type { LayoutConfig } from 'golden-layout';
	import 'svelte-golden-layout/css/themes/goldenlayout-light-theme.css';

	import GoldenLayout from 'svelte-golden-layout';
	// ... import a Test component

	const components = { Test };

	const layout: LayoutConfig = {
		root: {
			type: 'row',
			content: [
				{
					type: 'component',
					componentType: 'Test',
					componentState: {
						extraClass = 'bold',
						someProp = 1,
						anotherProp = 1,
					}
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

	:global(.bold) {
		font-weight: bold;
	}
</style>
```

You can also run this repo as an app; the example code is in [routes/index.svelte](src/routes/index.svelte) and [test/Test.svelte](src/test/Test.svelte), and demonstates both features and shortcomings of the library.
