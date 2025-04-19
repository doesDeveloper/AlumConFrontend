<script>
	import Sidebar from '$lib/components/Sidebar.svelte';
	import '$lib/styles/alumnicard.css';

	import { API_URL } from '$lib/config';
	import { onMount } from 'svelte';
	let users = null;
	let error = null;
	let token = '';

	// Search filters
	let firstName = '';
	let city = '';
	let jobTitle = '';
	let collegeName = '';
	let sortBy = 'username'; // default sort

	onMount(async () => {
		token = localStorage.getItem('token');
		if (!token) {
			error = 'You are not logged in.';
			window.location.href = '/login';
			return;
		}

		fetchUsers(); // load initial feed
	});

	async function fetchUsers() {
		try {
			const params = new URLSearchParams();
			if (firstName) params.append('firstName', firstName);
			if (city) params.append('city', city);
			if (jobTitle) params.append('jobTitle', jobTitle);
			if (collegeName) params.append('collegeName', collegeName);
			params.append('sortBy', sortBy);
			params.append('page', '0');

			const endpoint = `${API_URL}users/search?${params.toString()}`;

			const response = await fetch(endpoint, {
				method: 'GET',
				headers: {
					Authorization: `Bearer ${token}`
				}
			});

			if (!response.ok) throw new Error('Failed to fetch user data');

			users = await response.json();
			error = null;
		} catch (err) {
			error = err.message;
			users = null;
		}
	}
	let i = 0;
	function handleSearch() {
		fetchUsers();
	}
</script>

<div class="layout">
	<Sidebar />

	<div class="center-container">
		<div class="top-bar">
			<input type="text" placeholder="Search for friends, groups, pages" />
			<button class="add-post-btn">Add New Post ➕</button>
		</div>

		{#if error}
			<p>{error}</p>
		{:else if users}
			{#each users.content as user (user.username)}
				<div class="person-card">
					<div class="logo-icon">{user.username.charAt(0).toUpperCase()}</div>
					<div class="info">
						<div class="name">{user.firstName} {user.lastName}</div>
						<div class="title">{user.jobTitle}</div>
						<div class="bio">{user.bio}</div>
					</div>
					<button class="connect-btn">Connect</button>
				</div>
			{/each}
		{:else}
			{#each [...Array(10)] as i}
				<div class="person-card loading skeleton">
					<div class="logo-icon loading animate-pulse"></div>
					<div class="info">
						<div class="name loading placeholder animate-pulse"></div>
						<div class="title loading placeholder animate-pulse"></div>
						<div class="bio loading placeholder animate-pulse"></div>
					</div>
					<div class="connect-btn loading placeholder animate-pulse"></div>
				</div>
			{/each}
		{/if}
	</div>
</div>
