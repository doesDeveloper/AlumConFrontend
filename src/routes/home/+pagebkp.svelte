<script>
	import { onMount } from 'svelte';
	import { API_URL } from '$lib/config';
	import Sidebar from '$lib/components/Sidebar.svelte';
	import NewPostModal from '$lib/components/NewPostModal.svelte';
  
	let showNewPostModal = false;
	let posts = [];
	let loading = false;
	let error = '';
	let success = '';
	let token = '';
  
	// pagination state
	let page = 0;       // zero-based page index
	let size = 10;      // items per page
	let totalPages = 1; // will be set by API response
  
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
			error = payload.fieldErrors.map(fe => `${fe.field}: ${fe.message}`).join('; ');
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
  
	// Fetch posts for current page
	async function fetchPosts() {
	  loading = true;
	  error = '';
	  try {
		const res = await fetch(
		  `${API_URL}posts/feed?page=${page}&size=${size}`,
		  { headers: { Authorization: `Bearer ${token}` } }
		);
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
  
	function changePage(newPage) {
	  if (newPage < 0 || newPage >= totalPages) return;
	  page = newPage;
	  fetchPosts();
	}
  
	onMount(() => {
	  token = localStorage.getItem('token') || '';
	  if (!token) {
		window.location.href = '/login';
		return;
	  }
	  fetchPosts();
	});
  </script>
  
  <div class="layout">
	<Sidebar />
	<div class="center-container">
	  <div class="top-bar">
		<input type="text" placeholder="Search for friends, groups, pages" />
		<button class="add-post-btn" on:click={() => (showNewPostModal = true)}>
		  Post
		</button>
	  </div>
  
	  {#if loading}
		<p>Loading posts...</p>
	  {:else if error}
		<p class="error">{error}</p>
	  {:else if posts.length === 0}
		<p>No posts yet.</p>
	  {:else}
		{#each posts as post}
		  <div class="post-card">
			<div class="post-header">
			  <div class="logo-icon user-avatar">{post.username.charAt(0).toUpperCase()}</div>
			  <div class="user-info">
				<div class="user-name">{post.username}</div>
				<div class="user-role">Product Designer, slothUI</div>
			  </div>
			</div>
			<div class="post-content"><p>{post.content}</p></div>
			<div class="post-actions">
			  <div>❤️ {post.voteCount} Likes</div>
			  <div>💬 Comments</div>
			  <div>🔁 Share</div>
			</div>
		  </div>
		{/each}
  
		<!-- Pagination Controls -->
		<div class="pagination">
		  <button on:click={() => changePage(page - 1)} disabled={page === 0}>
			Previous
		  </button>
  
		  {#each Array(totalPages) as _, i}
			<button
			  class:selected={i === page}
			  on:click={() => changePage(i)}
			>
			  {i + 1}
			</button>
		  {/each}
  
		  <button on:click={() => changePage(page + 1)} disabled={page >= totalPages - 1}>
			Next
		  </button>
		</div>
	  {/if}
	</div>
  </div>
  
  <NewPostModal
	show={showNewPostModal}
	onClose={() => (showNewPostModal = false)}
	onSubmit={async (t, c) => createPost(t, c)}
  />
  
  <style>
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
	.add-post-btn:hover { background: #005ec2; }
  
	.pagination {
	  display: flex;
	  gap: 0.5rem;
	  margin: 1rem 0;
	  flex-wrap: wrap;
	}
	.pagination button.selected {
	  font-weight: bold;
	  text-decoration: underline;
	}
  
	.error { color: #c00; margin: 1rem 0; }
  </style>
  