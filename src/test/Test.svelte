<script lang="ts">
	import { onMount } from 'svelte';

	export let name: string;

	let greeting = 'Hello';

	onMount(() => {
		console.log("mount", name);

		return () => {
			console.log("unmount", name);
		};
	});
</script>

<p>Greeting: <input type="text" size="5" bind:value={greeting} /></p>
<p>{greeting} {name}!</p>
