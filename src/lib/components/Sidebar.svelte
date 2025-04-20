<script>
	import '$lib/styles/sidebar.css';
	import '$lib/styles/postcard.css';
	import { base } from '$app/paths';
	import UserAvatar from '$lib/components/UserAvatar.svelte';

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
	<div class="close-btn" style="" on:click={closeSidebar}>
		<i class="fas fa-times"></i>
	</div>
	<div class="logo">
		<div class="logo-icon graduation-logo"></div>
		<div class="tassel"></div>
		<span>AlumCon</span>
	</div>
	<div class="search-box">
		<input type="text" placeholder="Visual Search Bar..." />
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
			<a href="{base}/chats" on:click={closeSidebar}><span class="icon">🗨️</span> Chats </a>
		</li>
		<li>
			<a href="{base}/" on:click={closeSidebar}><span class="icon">❓</span> Help & Support</a>
		</li>
	</ul>
	<a href="{base}/profile">
		<div class="user-profile">
			<UserAvatar {username} size="60px" />
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

<style>
	.logo-icon.graduation-logo {
		width: 50px;
		height: 50px;
		border-radius: 50%;
		background: linear-gradient(to bottom, #f4e9e2 60%, #000 60%);
		position: relative;
		overflow: hidden;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	/* Head */
	.logo-icon.graduation-logo::before {
		content: '';
		position: absolute;
		width: 20px;
		height: 20px;
		background: #fca;
		border-radius: 50%;
		top: 15px;
		left: 15px;
		z-index: 2;
	}

	/* Cap (mortarboard) */
	.logo-icon.graduation-logo::after {
		content: '';
		position: absolute;
		width: 26px;
		height: 26px;
		background: #222;
		transform: rotate(45deg);
		top: 0;
		left: 12px;
		z-index: 3;
		box-shadow: 1px 1px 0 #333;
	}

	/* Tassel */
	.logo-icon.graduation-logo .tassel {
		position: absolute;
		width: 2px;
		height: 15px;
		background: #e36b5b;
		top: 12px;
		left: 24px;
		transform: rotate(10deg);
		z-index: 4;
	}

	.logo-icon.graduation-logo .tassel::after {
		content: '';
		position: absolute;
		width: 6px;
		height: 6px;
		background: #e36b5b;
		border-radius: 50%;
		bottom: -4px;
		left: -2px;
	}
</style>
