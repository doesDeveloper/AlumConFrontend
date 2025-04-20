<script>
	import { base } from '$app/paths';
	import Sidebar from '$lib/components/Sidebar.svelte';
	import '$lib/styles/alumnicard.css';
	import { API_URL } from '$lib/config';
	import { onMount } from 'svelte';
	import { slide } from 'svelte/transition';

    import UserAvatar from '$lib/components/UserAvatar.svelte';

	// User data variables
	let users = null;
	let error = null;
	let token = '';
	let loading = false;

	// Search filters
	let firstName = '';
	let city = '';
	let jobTitle = '';
	let collegeName = '';
	let sortBy = 'username'; // default sort

	// Pagination variables
	let page = 0; // zero-based page index
	let size = 10; // items per page
	let totalPages = 1; // will be set by API response

	// UI state
	let expanded = false;

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
			loading = true;
			const params = new URLSearchParams();
			if (firstName) params.append('firstName', firstName);
			if (city) params.append('city', city);
			if (jobTitle) params.append('jobTitle', jobTitle);
			if (collegeName) params.append('collegeName', collegeName);
			params.append('sortBy', sortBy);
			params.append('page', page.toString());
			params.append('size', size.toString());

			const endpoint = `${API_URL}users/searchalumnis?${params.toString()}`;
			const response = await fetch(endpoint, {
				method: 'GET',
				headers: {
					Authorization: `Bearer ${token}`
				}
			});

			if (!response.ok) throw new Error('Failed to fetch user data');
			users = await response.json();

			// Set pagination data
			totalPages = users.totalPages ?? Math.ceil(users.totalElements / size);

			error = null;
		} catch (err) {
			error = err.message;
			users = null;
		} finally {
			loading = false;
		}
	}

	function handleSearch() {
		page = 0; // Reset to first page when searching
		fetchUsers();
	}

	let centerContainer;
	function changePage(newPage) {
		if (newPage < 0 || newPage >= totalPages) return;
		page = newPage;
		// Scroll to top when changing pages
		centerContainer.scrollTo(0, 0);
		fetchUsers();
	}

	function expand() {
		expanded = true;
	}

	function toggleExpand() {
		expanded = !expanded;
	}
</script>

<div class="layout">
	<Sidebar />
	<div class="main-content">
		<div class="center-container" bind:this={centerContainer}>
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
			<h2 class="feed-heading"> 🧑‍🎓 Alumni List</h2>
			{#if error}
				<p class="error-message">{error}</p>
			{:else if loading}
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
			{:else if users && users.content && users.content.length > 0}
				{#each users.content as user (user.username)}
					<div class="person-card">
						<!-- <div class="logo-icon">{user.username.charAt(0).toUpperCase()}</div> -->
						<UserAvatar username={user.username} size="40px" />
						<div class="info">
							<div class="name">
								<a href="{base}/profile/others?name={user.username}"
									>{user.firstName} {user.lastName}</a
								>
							</div>
							<div class="title">
								{user.jobTitle || 'No job title'} from {user.city || 'Unknown location'}
							</div>
							<div class="bio">{user.bio || 'No bio available'}</div>
						</div>
						<a href="{base}/profile/others?name={user.username}"
							><button class="connect-btn">Profile</button></a
						>
					</div>
				{/each}

				<!-- Pagination Controls -->
				{#if totalPages > 1}
					<div class="pagination">
						<button class="pagination-btn" disabled={page === 0} on:click={() => changePage(0)}>
							« First
						</button>
						<button
							class="pagination-btn"
							disabled={page === 0}
							on:click={() => changePage(page - 1)}
						>
							‹ Previous
						</button>

						<div class="page-info">
							<span class="current-page">{page + 1}</span> of {totalPages}
						</div>

						<button
							class="pagination-btn"
							disabled={page >= totalPages - 1}
							on:click={() => changePage(page + 1)}
						>
							Next ›
						</button>
						<button
							class="pagination-btn"
							disabled={page >= totalPages - 1}
							on:click={() => changePage(totalPages - 1)}
						>
							Last »
						</button>
					</div>
				{/if}
			{:else}
				<div class="no-results">
					<div class="no-results-icon">🔍</div>
					<h3>No alumni found</h3>
					<p>Try adjusting your search filters or check back later.</p>
				</div>
			{/if}
		</div>
	</div>
</div>

<style>
	.search-filters {
		max-width: inherit;
		transition: all 0.3s ease;
		width: 100%;
	}

	.search-container {
		display: flex;
		flex-direction: column;
		gap: 15px;
		position: relative;
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

	/* Pagination Styles */
	.pagination {
		display: flex;
		justify-content: center;
		align-items: center;
		margin: 30px 0;
		gap: 8px;
	}

	.pagination-btn {
		background-color: #f5f7fa;
		border: 1px solid #ddd;
		border-radius: 6px;
		padding: 8px 16px;
		font-size: 14px;
		cursor: pointer;
		transition: all 0.2s ease;
	}

	.pagination-btn:hover:not([disabled]) {
		background-color: #eef2ff;
		border-color: #0077ff;
		color: #0077ff;
	}

	.pagination-btn[disabled] {
		opacity: 0.5;
		cursor: not-allowed;
	}

	.page-info {
		padding: 8px 16px;
		background-color: #f5f7fa;
		border-radius: 6px;
		border: 1px solid #ddd;
		font-size: 14px;
	}

	.current-page {
		font-weight: bold;
		color: #0077ff;
	}

	/* No results styling */
	.no-results {
		text-align: center;
		padding: 40px 20px;
		background-color: #f9f9f9;
		border-radius: 8px;
		margin: 20px 0;
	}

	.no-results-icon {
		font-size: 48px;
		margin-bottom: 16px;
	}

	.no-results h3 {
		color: #333;
		margin-bottom: 8px;
	}

	.no-results p {
		color: #666;
	}

	/* Make sure the layout is responsive */
	@media (max-width: 768px) {
		.search-row {
			flex-direction: column;
		}

		.pagination {
			flex-wrap: wrap;
		}
	}
	.feed-heading {
		font-size: 1.8rem;
		font-weight: 600;
		color: #333;
		margin: 50px 0 16px 0; /* increased top margin here */
		padding-left: 10px;
		border-left: 4px solid #4caf50;
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}
</style>
