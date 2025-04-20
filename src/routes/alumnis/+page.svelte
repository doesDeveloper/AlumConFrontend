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
	function handleSearch() {
		fetchUsers();
	}

	// Add the existing imports and variables
	import { slide } from 'svelte/transition';

	// Add this new variable to track expanded state
	let expanded = false;

	// Add these functions
	function expand() {
		expanded = true;
	}

	function toggleExpand() {
		expanded = !expanded;
	}
</script>

<div class="layout">
	<Sidebar />
	<div class="center-container">
		<div class="top-bar">
			<div class="search-filters {expanded ? 'expanded' : ''}">
				<div class="search-container">
					<!-- Always visible search field -->
					<div class="search-field main-field">
						<div class="icon">👤</div>
						<input
							type="text"
							bind:value={firstName}
							on:focus={expand}
							placeholder="Search by name..."
						/>
						<button class="expand-toggle" on:click|stopPropagation={toggleExpand}>
							{expanded ? '▲' : '▼'}
						</button>
					</div>

					<!-- Expandable section -->
					{#if expanded}
						<div class="expanded-fields" transition:slide={{ duration: 300 }}>
							<div class="search-row">
								<div class="search-field">
									<div class="icon">🏙️</div>
									<input type="text" bind:value={city} placeholder="City" />
								</div>
								<div class="search-field">
									<div class="icon">💼</div>
									<input type="text" bind:value={jobTitle} placeholder="Job Title" />
								</div>
							</div>
							<div class="search-row">
								<div class="search-field">
									<div class="icon">🎓</div>
									<input type="text" bind:value={collegeName} placeholder="College Name" />
								</div>
								<div class="sort-container">
									<div class="icon">🔄</div>
									<select bind:value={sortBy}>
										<option value="username">Sort by Username</option>
										<option value="city">Sort by City</option>
										<option value="jobTitle">Sort by Job Title</option>
										<option value="collegeName">Sort by College</option>
									</select>
								</div>
							</div>
							<div class="search-actions">
								<button class="search-btn" on:click={handleSearch}>
									<span class="search-icon">🔍</span>
									<span>Search</span>
								</button>
							</div>
						</div>
					{/if}
				</div>
			</div>
		</div>

		<!-- Add search bar from old code -->

		{#if error}
			<p class="error-message">{error}</p>
		{:else if users}
			{#each users.content as user (user.username)}
				<div class="person-card">
					<div class="logo-icon">{user.username.charAt(0).toUpperCase()}</div>
					<div class="info">
						<div class="name">
							<a href="/profile/others?name={user.username}">{user.firstName} {user.lastName}</a>
						</div>
						<div class="title">{user.jobTitle} from {user.city}</div>
						<div class="bio">{user.bio}</div>
					</div>
					<a href="/profile/others?name={user.username}"
						><button class="connect-btn">Profile</button></a
					>
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

<style>
	.search-filters {
		/* background-color: white; */
		/* border-radius: 12px; */
		/* box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08); */
		/* padding: 16px; */
		/* margin-bottom: 20px; */
		max-width: inherit;
		transition: all 0.3s ease;
		width: 100%;
	}

	.search-container {
		display: flex;
		flex-direction: column;
		gap: 15px;
		position: relative;
		/* max-width: 500px; */
	}

	.search-field {
		display: flex;
		align-items: center;
		background-color: #f5f7fa;
		border-radius: 99999px;
		padding: 0px 6px;
		transition: all 0.2s ease;
	}

	.main-field {
		cursor: text;
	}

	.search-field:focus-within {
		background-color: #eef2ff;
		box-shadow: 0 0 0 2px rgba(0, 119, 255, 0.2);
	}

	.icon {
		font-size: 1.2rem;
		margin-right: 12px;
		margin-left: 12px;
		color: #555;
	}

	.search-field input {
		flex: 1;
		border: none;
		background: transparent;
		/* padding: 10px 8px; */
		font-size: 1rem;
		outline: none;
	}

	.expanded-fields {
		display: flex;
		flex-direction: column;
		gap: 12px;
	}

	.search-row {
		display: flex;
		gap: 10px;
	}

	.search-row > * {
		flex: 1;
	}

	.sort-container {
		display: flex;
		align-items: center;
		background-color: #f5f7fa;
		border-radius: 9999px;
		padding: 8px 16px;
	}

	.sort-container select {
		flex: 1;
		border: none;
		background: transparent;
		padding: 10px 8px;
		font-size: 1rem;
		outline: none;
		cursor: pointer;
	}

	.search-actions {
		display: flex;
		justify-content: flex-end;
		margin-top: 8px;
	}

	.search-btn {
		display: flex;
		align-items: center;
		gap: 8px;
		background-color: #0077ff;
		color: white;
		border: none;
		border-radius: 8px;
		padding: 12px 24px;
		font-weight: 500;
		font-size: 0.95rem;
		cursor: pointer;
		transition: all 0.2s ease;
	}

	.search-btn:hover {
		background-color: #0055cc;
		transform: translateY(-1px);
	}

	.search-icon {
		font-size: 1.1rem;
	}

	.error-message {
		color: #e53935;
		padding: 10px;
		background-color: #ffebee;
		border-radius: 6px;
		margin-bottom: 16px;
	}

	/* Make sure the layout is responsive */
	@media (max-width: 768px) {
		.search-row {
			flex-direction: column;
		}
	}
</style>
