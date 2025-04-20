<script>
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';
	import {base} from '$app/paths';
	async function handleLogout() {
		localStorage.removeItem('token');
		localStorage.removeItem('user');
		// goto(base +'/login');
		window.location.href = base + '/login';
	}
	onMount(() => {
		handleLogout();
	});
</script>
