<script>
	import { goto } from '$app/navigation';
	import { base } from '$app/paths';
	import { onMount } from 'svelte';
	onMount(() => {
		window.location.href = base + '/home';
	});
</script>

<h1>Redirecting.....</h1>
<!-- <nav>
	<a href="{base}/login">Login</a>
	<a href="{base}/register">Register</a>
	<a href="{base}/feed">feed me</a>
	<a href="{base}/alumnis">Alumni</a>
	<a href="{base}/profile">Profile</a>
</nav> -->
