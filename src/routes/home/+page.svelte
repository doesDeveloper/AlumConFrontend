<script>
	import { onMount } from 'svelte';
	import { API_URL } from '$lib/config';
	import Sidebar from '$lib/components/Sidebar.svelte';
	let username = 'Abbasi';
	let token;
	let posts = [];
	let title = '';
	let content = '';
	// let username = '';
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
	// const fetchPost = async () => {
	// 	try {
	// 		loading = true;
	// 		const res = await fetch(`${API_URL}posts/user/${username}?page=0`, {
	// 			headers: {
	// 				Authorization: `Bearer ${token}`
	// 			}
	// 		});

	// 		if (!res.ok) throw new Error('Failed to fetch user posts');
	// 		const data = await res.json();
	// 		posts = data.content;
	// 		console.log(data);
	// 	} catch (err) {
	// 		error = err.message;
	// 	} finally {
	// 		loading = false;
	// 	}
	// };
	onMount(async () => {
		token = localStorage.getItem('token');
		if (!token) {
			error = 'Please log in.';
			window.location.href = '/login';
			return;
		}

		await fetchPosts();
	});
</script>

<div class="layout">
	<Sidebar />

	<div class="center-container">
		<!-- Top Search Bar -->
		<div class="top-bar">
			<input type="text" placeholder="Search for friends, groups, pages" />
			<button class="add-post-btn">Add New Post ➕</button>
		</div>

		{#if loading}
			{#each [...Array(10)] as i}
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
			<!-- Post Card -->
			{#each posts as post}
				<div class="post-card">
					<div class="post-header">
						<div class="logo-icon user-avatar">{post.username.charAt(0).toUpperCase()}</div>
						<div class="user-info">
							<div class="user-name"><a href="/profile/{post.username}">{post.username}</a></div>
							<!-- <div class="user-role">Product Designer, slothUI</div> -->
						</div>
						<div class="post-menu">⋮</div>
					</div>
                    
					<div class="post-content">
                        <div class="post-title">{post.title}</div>
						<p>
							{post.content}
						</p>
						<!-- <span class="hashtags">#amazing #great #lifetime #uiux #machinelearning</span> -->
					</div>

					<!-- <div class="post-image">
					<img src="assets/images/postimg.png" alt="Post Image" />
				</div> -->

					<div class="post-actions">
						<div>❤️ {post.voteCount} Likes</div>
						<div>💬 Comments</div>
						<div>🔁 Share</div>
					</div>

					<!-- <div class="post-comment">
                    <img src="user-avatar.jpg" alt="User" class="comment-avatar">
                    <input type="text" placeholder="Write your comment...">
                    <div class="comment-icons">
                        😊 📎 📨
                    </div>
                </div> -->
				</div>
			{/each}
		{/if}
	</div>
</div>
