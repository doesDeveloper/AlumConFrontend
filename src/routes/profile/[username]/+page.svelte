<script>
	import { API_URL } from '$lib/config';
	import { onMount } from 'svelte';
	import { page } from '$app/stores';
	import Sidebar from '$lib/components/Sidebar.svelte';
	import '$lib/styles/publicprof.css';

	let publicUser = null;
	let error = null;
	let username;
	let token;
	let posts = [];
	let loading = false;

	$: username = $page.params.username;

	onMount(async () => {
		token = localStorage.getItem('token');
		if (!token) {
			error = 'You are not logged in.';
			window.location.href = '/login';
			return;
		}

		await fetchUserProfile();
		await fetchUserPosts();
	});

	const fetchUserProfile = async () => {
		try {
			const response = await fetch(`${API_URL}users/profile/${username}`, {
				headers: { Authorization: `Bearer ${token}` }
			});
			if (!response.ok) throw new Error('Failed to fetch profile');
			publicUser = await response.json();
			console.log(publicUser);
		} catch (err) {
			error = err.message;
		}
	};

	const fetchUserPosts = async () => {
		try {
			loading = true;
			const res = await fetch(`${API_URL}posts/feed/${username}?page=0`, {
				headers: { Authorization: `Bearer ${token}` }
			});
			if (!res.ok) throw new Error('Failed to fetch user posts');
			const data = await res.json();
			posts = data.content;
		} catch (err) {
			error = err.message;
		} finally {
			loading = false;
		}
	};
</script>

<div class="layout">
	<Sidebar />

	<div class="center-container">
		<div class="top-bar">
			<input type="text" placeholder="Search for friends, groups, pages" />
			<button class="add-post-btn">Add New Post ➕</button>
		</div>
		{#if error}
			<p class="error">{error}</p>
		{:else if publicUser}
			<div class="profile-card">
				<div class="profile-header">
					<div class="logo-icon profile-pic logo-text">A</div>
					<div>
						<h2>{publicUser.firstName} {publicUser.lastName}</h2>
						<p class="username">@{publicUser.username}</p>
						<p class="location">From {publicUser.city}</p>
					</div>
				</div>
				<div class="stats">
					<div><strong>{posts.length}</strong><br /> Posts</div>
					<div><strong>{publicUser.followers || '12.7K'}</strong><br /> Followers</div>
					<div><strong>{publicUser.following || '221'}</strong> <br />Following</div>
				</div>

				<div class="about">
					<h3>About Me</h3>
					<p>{publicUser.bio}</p>
				</div>
			</div>
			<div class="profile-card">
				<!-- <div class="contact-info">
				<h3>Contact</h3>
				<p>📞 {publicUser.phone || '+123 456 789 000'}</p>
				<p>✉️ {publicUser.email || 'hello@example.com'}</p>
				<p>
					🔗 <a href={publicUser.website || '#'} target="_blank"
						>{publicUser.website || 'example.com'}</a
					>
				</p>
			</div> -->

				<h3 style="font-size:larger;">Posts</h3>
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
									<div class="user-name">
										<a href="/profile/{post.username}">{post.username}</a>
									</div>
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
		{:else}
			<p class="loading">Loading profile…</p>
		{/if}
	</div>
</div>

<style>
</style>
