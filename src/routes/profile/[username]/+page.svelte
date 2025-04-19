<script>
	import { API_URL } from '$lib/config';
	import { onMount } from 'svelte';
	import { page } from '$app/stores';

	let publicUser = null;
	let error = null;
	let username;
	let token;
	let posts = [];
	let loading = false;

	// Extract the username from the route params
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
			console.log(data)
		} catch (err) {
			error = err.message;
		} finally {
			loading = false;
		}
	};
</script>

{#if error}
	<p style="color: red;">{error}</p>
{:else if publicUser}
	<h1>{publicUser.firstName} {publicUser.lastName}</h1>

	<p><strong>Bio:</strong> {publicUser.bio}</p>
	<p><strong>City:</strong> {publicUser.city}</p>
	<p><strong>Education:</strong> {publicUser.education}</p>
	<p><strong>School Name:</strong> {publicUser.schoolName}</p>
	<p><strong>College Name:</strong> {publicUser.collegeName}</p>
	<p><strong>Skills:</strong> {publicUser.skills}</p>
	<p><strong>Interests:</strong> {publicUser.interests}</p>
	<p><strong>Branch:</strong> {publicUser.branch}</p>
	<p><strong>Job Title:</strong> {publicUser.jobTitle}</p>

	<h2 style="margin-top: 2rem;">Posts by {publicUser.username}</h2>
	
	{#if loading}
		<p>Loading posts…</p>
	{:else if posts.length === 0}
		<p>No posts yet.</p>
	{:else}
		{#each posts as post}
			<div class="post">
				<h3>{post.title}</h3>
				<p>{post.content}</p>
			</div>
		{/each}
	{/if}

{:else}
	<p>Loading profile…</p>
{/if}

<style>
	p {
		margin: 0.5rem 0;
	}
	.post {
		margin: 1rem 0;
		padding: 1rem;
		border: 1px solid #ccc;
		border-radius: 0.5rem;
	}
</style>
