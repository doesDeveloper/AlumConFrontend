<script>
	import { onMount } from 'svelte';
	import { API_URL } from '$lib/config';
	import Sidebar from '$lib/components/Sidebar.svelte';
	import NewPostModal from '$lib/components/NewPostModal.svelte';
	import PostCard from '$lib/components/PostCard.svelte';
	import { base } from '$app/paths';

	let showNewPostModal = false;
	let username = 'Abbasi';
	let token;
	let posts = [];
	let error = '';
	let success = '';
	let loading = false;

	// pagination state
	let page = 0; // zero-based page index
	let size = 10; // items per page
	let totalPages = 1; // will be set by API response

	// Fetch posts for current page
	async function fetchPosts() {
		loading = true;
		error = '';
		try {
			const res = await fetch(`${API_URL}posts/feed?page=${page}&size=${size}`, {
				headers: { Authorization: `Bearer ${token}` }
			});
			const data = await res.json();
			if (!res.ok) throw new Error(data.message || 'Failed to fetch posts');

			posts = data.content;
			totalPages = data.totalPages ?? Math.ceil(data.totalElements / size);
		} catch (err) {
			error = err.message;
		} finally {
			loading = false;
		}
	}

	let centerContainer;
	function changePage(newPage) {
		if (newPage < 0 || newPage >= totalPages) return;
		page = newPage;
		centerContainer.scrollTo(0, 0);
		fetchPosts();
	}

	// Handle post updates from the PostCard component
	function handlePostUpdate(event) {
		const { postId, voteCount, ownVote } = event.detail;

		// Update the post in the posts array
		posts = posts.map((post) => {
			if (post.id === postId) {
				return { ...post, voteCount, ownVote };
			}
			return post;
		});
	}

	// Create a new post
	const createPost = async (newTitle, newContent) => {
		if (!newTitle.trim() || !newContent.trim()) {
			error = 'Title and content cannot be empty.';
			return false;
		}

		try {
			loading = true;
			const res = await fetch(`${API_URL}posts/create`, {
				method: 'POST',
				headers: {
					Authorization: `Bearer ${token}`,
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({ title: newTitle.trim(), content: newContent.trim() })
			});

			const payload = await res.json();
			if (!res.ok) {
				// show field-level errors if any
				if (payload.fieldErrors) {
					error = payload.fieldErrors.map((fe) => `${fe.field}: ${fe.message}`).join('; ');
				} else {
					error = payload.message || 'Failed to create post';
				}
				return false;
			}

			success = 'Post created!';
			error = '';
			await fetchPosts();
			return true;
		} catch (err) {
			console.error(err);
			error = err.message;
			success = '';
			return false;
		} finally {
			loading = false;
		}
	};

	async function checkAuth() {
		token = localStorage.getItem('token');
		let user = JSON.parse(localStorage.getItem('user'));
		if (!token || !user) {
			error = 'Please log in.';
			window.location.href = base + '/login';
			return;
		}
	}

	onMount(async () => {
		await checkAuth();
		await fetchPosts();
	});
</script>

<div class="layout">
	<Sidebar />
	<div class="main-content">
		<div class="center-container" bind:this={centerContainer}>
			<!-- Top Search Bar -->
			<div class="top-bar">
				<input type="text" placeholder="Use the search box in alumni section." />
				<button class="add-post-btn" on:click={() => (showNewPostModal = true)}>Post ➕</button>
			</div>
			<h2 class="feed-heading">📢 Community Feed</h2>

			{#if loading}
				{#each [...Array(10)] as _, i}
					<div class="post-card loading">
						<div class="post-header">
							<div class="logo-icon user-avatar skeleton-circle"></div>
							<div class="user-info">
								<div class="skeleton-text short"></div>
								<div class="skeleton-text"></div>
							</div>
							<div class="post-menu skeleton-rect"></div>
						</div>

						<div class="post-content">
							<div class="skeleton-text"></div>
							<div class="skeleton-text"></div>
							<div class="skeleton-text short"></div>
						</div>

						<div class="post-actions">
							<div class="skeleton-text tiny"></div>
							<div class="skeleton-text tiny"></div>
							<div class="skeleton-text tiny"></div>
						</div>
					</div>
				{/each}
			{:else if posts.length === 0}
				<p>No posts yet.</p>
			{:else}
				<!-- Using the PostCard component for each post -->
				{#each posts as post (post.id)}
					<PostCard
						{post}
						{token}
						on:postUpdated={handlePostUpdate}
						on:error={(e) => (error = e.detail)}
					/>
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
			{/if}
		</div>
	</div>
</div>

<NewPostModal
	show={showNewPostModal}
	onClose={() => (showNewPostModal = false)}
	onSubmit={async (t, c) => createPost(t, c)}
/>

<style>
	/* Your existing styles from the original component */
	.add-post-btn {
		background: #007aff;
		color: white;
		padding: 0.6rem 1rem;
		border: none;
		border-radius: 0.6rem;
		cursor: pointer;
		font-weight: bold;
		font-size: 1rem;
		transition: 0.3s;
	}
	.add-post-btn:hover {
		background: #005ec2;
	}

	/* Skeleton loading styles */
	.loading .skeleton-circle {
		width: 40px;
		height: 40px;
		border-radius: 50%;
		background-color: #f0f0f0;
	}

	.loading .skeleton-text {
		height: 16px;
		background-color: #f0f0f0;
		border-radius: 4px;
		margin: 8px 0;
		width: 70%;
	}

	.loading .skeleton-text.short {
		width: 40%;
	}

	.loading .skeleton-text.tiny {
		width: 20%;
		height: 12px;
	}

	.loading .skeleton-rect {
		width: 24px;
		height: 24px;
		background-color: #f0f0f0;
		border-radius: 4px;
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

	/* Responsiveness */
	@media (max-width: 768px) {
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
