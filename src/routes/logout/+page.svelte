<script>
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';

	async function handleLogout() {
		localStorage.removeItem('token');
		localStorage.removeItem('user');
		goto('/login');
	}
	onMount(() => {
		handleLogout();
	});
</script>
