<script lang="ts">
	import type { SvelteComponent } from 'svelte';

	import type { ComponentContainer, ResolvedComponentItemConfig } from 'golden-layout';
	import { GoldenLayout, LayoutConfig } from 'golden-layout';

	import './css/goldenlayout-base.css';

	export let components: Record<string, typeof SvelteComponent> = {};
	export let config: LayoutConfig;

	$: componentsTypeMap = new Map(Object.entries(components));

	let goldenLayout: GoldenLayout | undefined;

	let width: number, height: number;

	function onResize() {
		goldenLayout?.setSize(goldenLayout.container.offsetWidth, goldenLayout.container.offsetHeight);
	}

	function initRoot(root: HTMLDivElement) {
		const components = new Map<ComponentContainer, SvelteComponent>();

		function bindComponentEvent(
			container: ComponentContainer,
			itemConfig: ResolvedComponentItemConfig,
		): ComponentContainer.BindableComponent {
			const { componentType, componentState } = itemConfig;
			if (typeof componentType !== 'string') throw new Error('Invalid component type.');

			const component = componentsTypeMap.get(componentType);
			if (!component) throw new Error(`Component not found: '${componentType}'`);

			// eslint-disable-next-line @typescript-eslint/no-explicit-any
			const { extraClass, ...props } = componentState as any;

			if (extraClass) container.element.classList.add(extraClass);

			const node = new component({
				target: container.element,
				props,
			});

			components.set(container, node);

			return {
				component: node,
				virtual: false,
			};
		}

		function unbindComponentEvent(container: ComponentContainer) {
			const node = components.get(container);
			components.delete(container);
			node.$destroy();
		}

		goldenLayout = new GoldenLayout(root, bindComponentEvent, unbindComponentEvent);

		return {
			destroy() {
				goldenLayout.destroy();
				goldenLayout = undefined;
			},
		};
	}

	$: {
		if (config) {
			// TODO goldenLayout is now reloaded whenever the config changes,
			// not only when the component is first loaded.
			// does that hurt in any particular scenario?
			goldenLayout?.loadLayout(config);
		}
	}

	$: {
		// evaluate width & height to establish a reactive dependency
		width;
		height;

		onResize();
	}
</script>

<svelte:window on:resize={onResize} />

<div class="root" bind:offsetWidth={width} bind:offsetHeight={height} use:initRoot />

<style>
	.root {
		width: 100%;
		height: 100%;
	}
</style>
