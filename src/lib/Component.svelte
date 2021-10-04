<script lang="ts">
	import type { SvelteComponent } from 'svelte';

	import type { ComponentContainer, LogicalZIndex } from 'golden-layout';
	import { VirtualLayout, LayoutConfig, ResolvedComponentItemConfig } from 'golden-layout';

	import './css/goldenlayout-base.css';

	type Bounds = {
		left: number;
		top: number;
		width: number;
		height: number;
	};
	type ComponentConfig = {
		key: ComponentContainer;
		type: typeof SvelteComponent;
		props: any;
		bounds: Bounds;
		visible: boolean;
		zIndex: string;
	};

	export let components: Record<string, typeof SvelteComponent> = {};
	export let config: LayoutConfig;

	let goldenLayout: VirtualLayout | undefined;
	let goldenLayoutBoundingClientRect: DOMRect | undefined;
	let componentInstances: ComponentConfig[] = [];

	let width: number, height: number;

	function onResize() {
		goldenLayout?.setSize(goldenLayout.container.offsetWidth, goldenLayout.container.offsetHeight);
	}

	$: {
		// evaluate width & height to establish a reactive dependency
		width;
		height;

		onResize();
	}

	function handleBindComponentEvent(
		container: ComponentContainer,
		itemConfig: ResolvedComponentItemConfig,
	) {
		// Use ResolvedComponentItemConfig.resolveComponentTypeNamecan to resolve component types to a unique name
		const componentTypeName = ResolvedComponentItemConfig.resolveComponentTypeName(itemConfig);
		if (componentTypeName === undefined) {
			throw new Error('handleBindComponentEvent: Undefined componentTypeName');
		}

		const componentType = components[componentTypeName];
		if (componentType === undefined) {
			throw new Error(
				`handleBindComponentEvent: Unknown componentTypeName: '${componentTypeName}'`,
			);
		}

		const component = {
			key: container,
			type: componentType,
			props: itemConfig.componentState,
			bounds: {
				left: 0,
				top: 0,
				width: 0,
				height: 0,
			},
			visible: true,
			zIndex: '',
		};

		componentInstances.push(component);
		componentInstances = componentInstances;

		container.virtualRectingRequiredEvent = handleContainerVirtualRectingRequiredEvent;
		container.virtualVisibilityChangeRequiredEvent = handleContainerVisibilityChangeRequiredEvent;

		return {
			component: undefined,
			virtual: true,
		};
	}

	function handleUnbindComponentEvent(container: ComponentContainer) {
		const index = componentInstances.findIndex((value) => value.key === container);

		if (index === -1) {
			throw new Error('handleUnbindComponentEvent: Component not found');
		}

		const component = componentInstances[index];

		componentInstances.splice(index, 1);
		componentInstances = componentInstances;
	}

	function handleContainerVirtualRectingRequiredEvent(
		container: ComponentContainer,
		width: number,
		height: number,
	) {
		const component = componentInstances.find((value) => value.key === container);

		if (component === undefined) {
			throw new Error('handleContainerVirtualRectingRequiredEvent: Component not found');
		}

		const containerBoundingClientRect = container.element.getBoundingClientRect();
		const left = containerBoundingClientRect.left - goldenLayoutBoundingClientRect.left;
		const top = containerBoundingClientRect.top - goldenLayoutBoundingClientRect.top;

		component.bounds = { left, top, width, height };
		componentInstances = componentInstances;
	}

	function handleContainerVisibilityChangeRequiredEvent(
		container: ComponentContainer,
		visible: boolean,
	) {
		const component = componentInstances.find((value) => value.key === container);

		if (component === undefined) {
			throw new Error('handleContainerVisibilityChangeRequiredEvent: Component not found');
		}

		component.visible = visible;
		componentInstances = componentInstances;
	}

	function handleContainerVirtualZIndexChangeRequiredEvent(
		container: ComponentContainer,
		logicalZIndex: LogicalZIndex,
		defaultZIndex: string,
	) {
		const component = componentInstances.find((value) => value.key === container);

		if (component === undefined) {
			throw new Error('handleContainerVirtualZIndexChangeRequiredEvent: Component not found');
		}

		component.zIndex = defaultZIndex;
		componentInstances = componentInstances;
	}

	function initRoot(root: HTMLDivElement) {
		const components = new Map<ComponentContainer, SvelteComponent>();

		goldenLayout = new VirtualLayout(root, handleBindComponentEvent, handleUnbindComponentEvent);
		goldenLayout.beforeVirtualRectingEvent = (count: number) => {
			goldenLayoutBoundingClientRect = root.getBoundingClientRect();
		};

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
			goldenLayout?.loadLayout(config);
		}
	}

	function componentStyle(component: ComponentConfig): string {
		let style = '';
		style += ['left', 'top', 'width', 'height']
			.map((key) => `${key}: ${component.bounds[key]}px;`)
			.join(' ');
		if (!component.visible) {
			style += ` display: none;`;
		}
		if (component.zIndex !== '') {
			style += ` z-index: ${component.zIndex};`;
		}
		return style;
	}
</script>

<svelte:window on:resize={onResize} />

<div class="root" bind:offsetWidth={width} bind:offsetHeight={height} use:initRoot>
	{#each componentInstances as component (component.key)}
		<div class="component-root" style={componentStyle(component)}>
			<svelte:component this={component.type} {...component.props} />
		</div>
	{/each}
</div>

<style>
	.root {
		width: 100%;
		height: 100%;
	}

	.component-root {
		position: absolute;
	}
</style>
