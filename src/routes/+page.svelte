<script>
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
	import CelebrationPage from '$lib/components/celebration/CelebrationPage.svelte';
	import { user } from '$lib/stores';

	// 如果用户已登录，直接跳转到 app 路由
	onMount(async () => {
		try {
			if ($user) {
				await goto('/app');
			}
		} catch (error) {
			console.error('Navigation error:', error);
		}
	});
</script>

<CelebrationPage />
