<script lang="ts">
	import type { SvelteComponent } from 'svelte';

	import type { ComponentContainer, JsonValue, LogicalZIndex } from 'golden-layout';
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
		id: string;
		componentType: string;
		componentState?: JsonValue;
		bounds: Bounds;
		visible: boolean;
		zIndex: string;
	};

	export let config: LayoutConfig;
	export let goldenLayout: VirtualLayout = undefined;

	let goldenLayoutBoundingClientRect: DOMRect | undefined;
	let components: ComponentConfig[] = [];

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
		const { id, componentState } = itemConfig;
		// Use ResolvedComponentItemConfig.resolveComponentTypeNamecan to resolve component types to a unique name
		const componentType = ResolvedComponentItemConfig.resolveComponentTypeName(itemConfig);

		const component = {
			key: container,
			id,
			componentType,
			componentState,
			bounds: {
				left: 0,
				top: 0,
				width: 0,
				height: 0,
			},
			visible: true,
			zIndex: '',
		};

		components.push(component);
		components = components;

		container.virtualRectingRequiredEvent = handleContainerVirtualRectingRequiredEvent;
		container.virtualVisibilityChangeRequiredEvent = handleContainerVisibilityChangeRequiredEvent;
		container.virtualZIndexChangeRequiredEvent = handleContainerVirtualZIndexChangeRequiredEvent;

		return {
			component: undefined,
			virtual: true,
		};
	}

	function handleUnbindComponentEvent(container: ComponentContainer) {
		const index = components.findIndex((value) => value.key === container);

		if (index === -1) {
			throw new Error('handleUnbindComponentEvent: Component not found');
		}

		const component = components[index];

		components.splice(index, 1);
		components = components;
	}

	function handleContainerVirtualRectingRequiredEvent(
		container: ComponentContainer,
		width: number,
		height: number,
	) {
		const component = components.find((value) => value.key === container);

		if (component === undefined) {
			throw new Error('handleContainerVirtualRectingRequiredEvent: Component not found');
		}

		const containerBoundingClientRect = container.element.getBoundingClientRect();
		const left = containerBoundingClientRect.left - goldenLayoutBoundingClientRect.left;
		const top = containerBoundingClientRect.top - goldenLayoutBoundingClientRect.top;

		component.bounds = { left, top, width, height };
		components = components;
	}

	function handleContainerVisibilityChangeRequiredEvent(
		container: ComponentContainer,
		visible: boolean,
	) {
		const component = components.find((value) => value.key === container);

		if (component === undefined) {
			throw new Error('handleContainerVisibilityChangeRequiredEvent: Component not found');
		}

		component.visible = visible;
		components = components;
	}

	function handleContainerVirtualZIndexChangeRequiredEvent(
		container: ComponentContainer,
		logicalZIndex: LogicalZIndex,
		defaultZIndex: string,
	) {
		const component = components.find((value) => value.key === container);

		if (component === undefined) {
			throw new Error('handleContainerVirtualZIndexChangeRequiredEvent: Component not found');
		}

		component.zIndex = defaultZIndex;
		components = components;
	}

	function initRoot(root: HTMLDivElement) {
		const components = new Map<ComponentContainer, SvelteComponent>();

		goldenLayoutBoundingClientRect = root.getBoundingClientRect();

		goldenLayout = new VirtualLayout(root, handleBindComponentEvent, handleUnbindComponentEvent);
		goldenLayout.beforeVirtualRectingEvent = (count: number) => {
			goldenLayoutBoundingClientRect = root.getBoundingClientRect();
		};

		if (goldenLayout.isSubWindow) {
			goldenLayout.checkAddDefaultPopinButton();
		}

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
			// in case of a popout (subwindow), we don't want to load a layout
			if (!goldenLayout?.isSubWindow) {
				goldenLayout?.loadLayout(config);
			}
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

<div class="wrapper" bind:offsetWidth={width} bind:offsetHeight={height}>
	<div class="root" use:initRoot />
	{#each components as component (component.key)}
		<div class="component-root" style={componentStyle(component)}>
			<slot
				id={component.id}
				componentType={component.componentType}
				componentState={component.componentState}
			/>
		</div>
	{/each}
</div>

<style>
	.wrapper {
		position: relative;
		width: 100%;
		height: 100%;
		overflow: hidden;
	}

	.root {
		position: absolute;
		left: 0;
		top: 0;
		right: 0;
		bottom: 0;
	}

	.component-root {
		position: absolute;
	}
</style>
