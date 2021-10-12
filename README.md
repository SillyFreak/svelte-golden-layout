# svelte-golden-layout

[GoldenLayout 2.x](https://github.com/golden-layout/golden-layout) Wrapper for Svelte. The `GoldenLayout` component accepts child content (i.e. a slot) that renders each tab.

This was extracted from a personal project, so its initial scope was rather limited. Expect rough edges, but bug reports and contributions are welcome!

**Features**

- pass the layout (`config`) via props
- render tab content via the default [slot](https://svelte.dev/docs#slot_let); `let:id`, `let:componentType`, and `let:componentState` contain the specific tab's content
- automatic resizing of the layout within its containing HTML element

**Known limitations**

- changing the layout recreates the whole thing, i.e. new components are created for each tab
- dragging the tabset dividers results in text selection
- base and theme CSS is bundled, but image assets have to be added as static resources (see [static/img](static/img), in this repo for the example app)

This means that right now, the `config` prop should better not be changed after the fact. It's a starting point!

## Installation & development

An **npm package** does not exist yet. If reception is good, I will publish this and develop it further.

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

In this example, `svelte:component` is used to select a specific component for each tab, and the `componentState` object is passed as props to that component:

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

You can also run this repo as an app; the example code is in [index.svelte](src/routes/index.svelte) and [\_Test.svelte](src/routes/_Test.svelte), and demonstates both features and shortcomings of the library.
