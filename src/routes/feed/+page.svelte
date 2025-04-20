<script>
	import { onMount } from 'svelte';
	import { API_URL } from '$lib/config';

	let token;
	let posts = [];
	let title = '';
	let content = '';
	let username = '';
	let error = '';
	let success = '';
	let loading = false;

	const votePost = async (postId, voteValue) => {
		try {
			// loading = true;

			const res = await fetch(`${API_URL}posts/${postId}/vote?voteValue=${voteValue}`, {
				method: 'POST',
				headers: {
					Authorization: `Bearer ${token}`
				}
			});

			if (!res.ok) throw new Error('Failed to vote on post');
			let data = await res.text();

			// Find the voted post in the posts array and update its voteCount locally
			posts = posts.map((post) => {
				if (post.id === postId) {
					return { ...post, voteCount: data };
				}
				return post;
			});

			console.log(`Voted post: ${postId}, value: ${voteValue}`);
		} catch (err) {
			console.error(err);
			error = err.message;
		} finally {
			// loading = false;
		}
	};

	const fetchPosts = async () => {
		try {
			loading = true;
			// const res = await fetch(`${API_URL}posts/feed?page=0`, {
			//   headers: {
			//     'Authorization': `Bearer ${token}`
			//   }
			// });
			const res = await fetch(`${API_URL}posts/feed?page=0`, {
				headers: {
					Authorization: `Bearer ${token}`
				}
			});

			if (!res.ok) throw new Error('Failed to fetch posts');
			const data = await res.json();
			posts = data.content;
			console.log(data);
			// posts = data.reverse(); // optional: reverse for newest first
		} catch (err) {
			error = err.message;
		} finally {
			loading = false;
		}
	};

	const createPost = async () => {
		if (!title.trim() || !content.trim()) {
			error = 'Title and content cannot be empty.';
			return;
		}

		try {
			loading = true;
			const res = await fetch(`${API_URL}posts/create`, {
				method: 'POST',
				headers: {
					Authorization: `Bearer ${token}`,
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({ title: title.trim(), content: content.trim() })
			});

			if (!res.ok) {
				const errorData = await res.json();
				console.error('Server error:', errorData);
				throw new Error(errorData.message || 'Failed to create post');
			}

			const data = await res.json();
			console.log('Created post:', data);
			title = '';
			content = '';
			success = 'Post created!';
			error = '';
			await fetchPosts();
		} catch (err) {
			console.error(err);
			error = err.message;
			success = '';
		} finally {
			loading = false;
		}
	};
	const fetchPost = async () => {
		try {
			loading = true;
			const res = await fetch(`${API_URL}posts/user/${username}?page=0`, {
				headers: {
					Authorization: `Bearer ${token}`
				}
			});

			if (!res.ok) throw new Error('Failed to fetch user posts');
			const data = await res.json();
			posts = data.content;
			console.log(data);
		} catch (err) {
			error = err.message;
		} finally {
			loading = false;
		}
	};
	onMount(async () => {
		token = localStorage.getItem('token');
		if (!token) {
			error = 'Please log in.';
			window.location.href = base + '/login';
			return;
		}

		await fetchPosts();
	});
</script>

<h1>Feed</h1>

{#if error}
	<p style="color: red;">{error}</p>
{/if}

{#if success}
	<p style="color: green;">{success}</p>
{/if}

<section style="margin-bottom: 2rem;">
	<h2>Create Post</h2>
	<form on:submit|preventDefault={createPost}>
		<input type="text" bind:value={title} placeholder="Title" required />
		<textarea bind:value={content} placeholder="Content" required></textarea>
		<button type="submit">Post</button>
	</form>
	<form on:submit|preventDefault={fetchPost}>
		<input type="text" bind:value={username} placeholder="Username" required />
		<button type="submit">Search by user</button>
	</form>
</section>

<h2>Posts</h2>
{#if loading}
	<p>Loading...</p>
{:else if posts.length === 0}
	<p>No posts yet.</p>
{:else}
	{#each posts as post}
		<div class="post">
			<h3>{post.title}</h3>
			<p>{post.content}</p>
			<small>by {post.username}</small>

			<div class="votes">
				<button on:click={() => votePost(post.id, 1)}>👍</button>
				{post.voteCount}
				<button on:click={() => votePost(post.id, -1)}>👎</button>
			</div>
		</div>
	{/each}
{/if}

<style>
	.votes {
		margin-top: 0.5rem;
	}

	.votes button {
		margin-right: 0.5rem;
		padding: 0.3rem 0.6rem;
		font-size: 0.9rem;
		border-radius: 0.3rem;
		border: none;
		cursor: pointer;
	}

	.votes button:first-child {
		background-color: #28a745;
		color: white;
	}

	.votes button:last-child {
		background-color: #dc3545;
		color: white;
	}

	form {
		display: flex;
		flex-direction: column;
		max-width: 500px;
	}

	input,
	textarea {
		margin: 0.5rem 0;
		padding: 0.5rem;
		font-size: 1rem;
	}

	button {
		padding: 0.5rem;
		font-size: 1rem;
		background: #0077ff;
		color: white;
		border: none;
		border-radius: 0.4rem;
		cursor: pointer;
	}

	.post {
		margin-bottom: 1.5rem;
		padding: 1rem;
		border: 1px solid #ddd;
		border-radius: 0.5rem;
	}
</style>
