<script lang="ts">
	import Chat from '$lib/components/chat/Chat.svelte';
	import Help from '$lib/components/layout/Help.svelte';
	import CelebrationPage from '$lib/components/celebration/CelebrationPage.svelte';
	import { onMount } from 'svelte';

	let showCelebration = false;

	onMount(() => {
		// 检查是否是第一次访问
		const hasVisited = localStorage.getItem('has_visited_app');
		if (!hasVisited) {
			showCelebration = true;
			localStorage.setItem('has_visited_app', 'true');
		}
	});
</script>

{#if showCelebration}
	<CelebrationPage />
{/if}

<Help />
<Chat />
