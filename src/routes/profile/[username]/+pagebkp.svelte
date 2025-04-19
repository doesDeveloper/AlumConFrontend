<script>
	import { API_URL } from '$lib/config';
	import { onMount } from 'svelte';
	import { page } from '$app/stores';
	import Sidebar from '$lib/components/Sidebar.svelte';

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
<Sidebar/>
{#if error}
	<p class="error">{error}</p>
{:else if publicUser}
	<div class="profile-card">
		<div class="profile-header">
			<div  class="logo-icon profile-pic">
			A</div>
			<div>
				<h2>{publicUser.firstName} {publicUser.lastName}</h2>
				<p class="username">@{publicUser.username}</p>
				<p class="location">{publicUser.city}</p>
			</div>
		</div>

		<div class="stats">
			<div><strong>{posts.length}</strong> Posts</div>
			<div><strong>{publicUser.followers || '12.7K'}</strong> Followers</div>
			<div><strong>{publicUser.following || '221'}</strong> Following</div>
		</div>

		<div class="about">
			<h3>About Me</h3>
			<p>{publicUser.bio}</p>
		</div>

		<div class="contact-info">
			<h3>Contact</h3>
			<p>📞 {publicUser.phoneNumber }</p>
			<p>✉️ {publicUser.email }</p>
			<p>🔗 <a href={publicUser.website || '#'} target="_blank">{publicUser.website || 'example.com'}</a></p>
		</div>

		<h3 style="margin-top: 2rem;">Posts by @{publicUser.username}</h3>
		{#if loading}
			<p>Loading posts…</p>
		{:else if posts.length === 0}
			<p>No posts yet.</p>
		{:else}
			{#each posts as post}
				<div class="post">
					<h4>{post.title}</h4>
					<p>{post.content}</p>
				</div>
			{/each}
		{/if}
	</div>
{:else}
	<p class="loading">Loading profile…</p>
{/if}
</div>

<style>
	.profile-card {
		max-width: 600px;
		margin: 2rem auto;
		padding: 2rem;
		border-radius: 1rem;
		box-shadow: 0 8px 24px rgba(0,0,0,0.08);
		background: #fff;
		font-family: 'Poppins', sans-serif;
	}
	.profile-header {
		display: flex;
		align-items: center;
		gap: 1rem;
	}
	.profile-pic {
		width: 80px;
		height: 80px;
		border-radius: 50%;
		object-fit: cover;
	}
	.username {
		color: #777;
		font-size: 0.9rem;
	}
	.location {
		color: #555;
		font-size: 0.85rem;
	}
	.stats {
		display: flex;
		justify-content: space-between;
		margin: 1.5rem 0;
		font-size: 0.95rem;
	}
	.about, .contact-info {
		margin-top: 1.5rem;
	}
	.about h3, .contact-info h3 {
		margin-bottom: 0.5rem;
	}
	.contact-info p {
		margin: 0.3rem 0;
	}
	.post {
		margin: 1rem 0;
		padding: 1rem;
		border: 1px solid #eee;
		border-radius: 0.5rem;
		background: #fafafa;
	}
	.post h4 {
		margin: 0 0 0.5rem;
	}
	.error {
		color: red;
		text-align: center;
	}
	.loading {
		text-align: center;
		color: #666;
	}
</style>
