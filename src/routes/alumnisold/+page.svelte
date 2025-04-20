<script>
	import { API_URL } from '$lib/config';
	import { onMount } from 'svelte';
	import { base } from '$app/paths';

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
			window.location.href = base + '/login';
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

	function handleSearch() {
		fetchUsers();
	}
</script>

<h1>Alumni List</h1>

<div class="search-bar">
	<input type="text" bind:value={firstName} placeholder="First Name" />
	<input type="text" bind:value={city} placeholder="City" />
	<input type="text" bind:value={jobTitle} placeholder="Job Title" />
	<input type="text" bind:value={collegeName} placeholder="College Name" />

	<select bind:value={sortBy}>
		<option value="username">Sort by Username</option>
		<option value="city">Sort by City</option>
		<option value="jobTitle">Sort by Job Title</option>
		<option value="collegeName">Sort by College</option>
	</select>

	<button on:click={handleSearch}>Search</button>
</div>

{#if error}
	<p>{error}</p>
{:else if users}
	{#each users.content as user (user.username)}
		<div class="user-card">
			<h2><a href="{base}profile/others?name={user.username}">@{user.username}</a></h2>
			<p><strong>Name:</strong> {user.firstName} {user.lastName}</p>
			<p><strong>Job Title:</strong> {user.jobTitle}</p>
			<p><strong>College:</strong> {user.collegeName}</p>
			<p><strong>City:</strong> {user.city}</p>
			<p><strong>Bio:</strong> {user.bio}</p>
		</div>
	{/each}
{:else}
	<p>Loading...</p>
{/if}

<style>
	.search-bar {
		display: flex;
		gap: 0.5rem;
		margin-bottom: 1.5rem;
	}

	.search-bar input {
		flex: 1;
		padding: 0.6rem 1rem;
		font-size: 1rem;
		border: 1px solid #ccc;
		border-radius: 0.5rem;
		outline: none;
		transition: border-color 0.2s ease;
	}

	.search-bar input:focus {
		border-color: #0077ff;
	}

	.search-bar button {
		padding: 0.6rem 1.2rem;
		font-size: 1rem;
		background-color: #0077ff;
		color: white;
		border: none;
		border-radius: 0.5rem;
		cursor: pointer;
		transition: background-color 0.2s ease;
	}

	.search-bar button:hover {
		background-color: #005fd1;
	}
	.user-card {
		border: 1px solid #ccc;
		border-radius: 0.5rem;
		padding: 1rem;
		margin-bottom: 1rem;
		background-color: #f9f9f9;
	}
	.user-card h2 {
		margin: 0 0 0.5rem;
	}
	.user-card p {
		margin: 0.2rem 0;
	}
	.user-card strong {
		font-weight: bold;
	}
	.user-card:hover {
		background-color: #e9e9e9;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
	}
	.user-card:hover h2 {
		color: #0077ff;
	}
	.user-card:hover p {
		color: #555;
	}
	.user-card:hover strong {
		color: #333;
	}
	.user-card:hover {
		transform: scale(1.02);
		transition: transform 0.2s ease;
	}
</style>
