<script>
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import CelebrationPage from '$lib/components/celebration/CelebrationPage.svelte';
	import { user } from '$lib/stores';

	// 如果用户已登录，直接跳转到主页
	onMount(async () => {
		if ($user) {
			goto('/chat');
		}
	});
</script>

<CelebrationPage />
