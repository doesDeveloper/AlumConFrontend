<script>
	import '$lib/styles/sidebar.css';
	import '$lib/styles/postcard.css';
	import { base } from '$app/paths';

	import { onMount } from 'svelte';
	let username = '';
	let fullName = '';

	onMount(() => {
		const user = JSON.parse(localStorage.getItem('user'));
		if (user) {
			username = user.username;
			fullName = user.firstName + ' ' + user.lastName;
		} else {
			window.location.href = base + '/login';
		}
	});
	function displaySidebar() {
		// add visible class to sidebar using svelte
		const sidebar = document.querySelector('.sidebar');
		sidebar.classList.add('visible');
	}
	function closeSidebar() {
		const sidebar = document.querySelector('.sidebar');
		sidebar.classList.remove('visible');
	}
</script>

<div class="sidebar">
	<div class="logo">
		<div class="logo-icon">A</div>
		<span>AlumCon</span>
	</div>
	<div class="search-box">
		<input type="text" placeholder="Search..." />
	</div>
	<ul class="menu">
		<li>
			<a href="{base}/home" on:click={closeSidebar}
				><span class="icon">🏠</span> Feed <span class="badge">10</span></a
			>
		</li>
		<li>
			<a href="{base}/alumnis" on:click={closeSidebar}
				><span class="icon">👥</span> Alumnis <span class="badge">2</span></a
			>
		</li>
		<li>
			<a href="{base}/" on:click={closeSidebar}><span class="icon">❓</span> Help & Support</a>
		</li>
	</ul>
	<a href="{base}/profile">
		<div class="user-profile">
			<div class="logo-icon">{fullName.charAt(0).toUpperCase()}</div>
			<div class="user-info">
				<div class="name">{fullName}</div>
				<div class="role">@{username}</div>
			</div>
			<div class="logout-icon"><i class="fas fa-angle-right"></i></div>
		</div>
	</a>
</div>
<div class="floating-button" on:click={displaySidebar}>
	<i class="fas fa-bars" style="color: white; font-size: medium;"></i>
</div>
