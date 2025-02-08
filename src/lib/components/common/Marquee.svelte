<script lang="ts">
	import { fade, fly } from 'svelte/transition';
	import { onMount, onDestroy } from 'svelte';

	let idx = 0;
	let interval: NodeJS.Timer;

	export let className = '';
	export let words = ['lorem', 'ipsum'];
	export let duration = 5000;

	onMount(() => {
		interval = setInterval(async () => {
			if (idx === words.length - 1) {
				idx = 0;
			} else {
				idx = idx + 1;
			}
		}, duration);
	});

	onDestroy(() => {
		if (interval) {
			clearInterval(interval);
		}
	});
</script>

<div class={className}>
	<div>
		{#key idx}
			<div class="marquee-item" in:fly|local={{ y: '20%', duration: 1500, easing: quintOut }}>
				{words.at(idx)}
			</div>
		{/key}
	</div>
</div>

<script context="module">
	import { quintOut } from 'svelte/easing';
</script>
