<script>
	import { API_URL } from '$lib/config';
	import { onMount } from 'svelte';
	import { page } from '$app/stores';
	import Sidebar from '$lib/components/Sidebar.svelte';
	import '$lib/styles/publicprof.css';
	import { goto } from '$app/navigation';
	import { get } from 'svelte/store';
	import PostCard from '$lib/components/PostCard.svelte';
	import { base } from '$app/paths';
    import UserAvatar from '$lib/components/UserAvatar.svelte';

	let publicUser = null;
	let error = null;
	let username;
	let token;
	let posts = [];
	let postsData = null;
	let loading = false;

	onMount(async () => {
		const currentPage = get(page);
		const nameParam = currentPage.url.searchParams.get('name');
		username = nameParam;
		if (!username) {
			// goto(base +'/profile/');
			window.location.href = base + '/profile/';
		}
		token = localStorage.getItem('token');
		if (!token) {
			error = 'You are not logged in.';
			window.location.href = base + '/login';
			return;
		}

		await fetchUserProfile();
		await fetchUserPosts();
	});
	function handlePostUpdate(event) {
		const { postId, voteCount, ownVote } = event.detail;

		// Update the post in our posts array
		posts = posts.map((post) => {
			if (post.id === postId) {
				return { ...post, voteCount, ownVote };
			}
			return post;
		});
	}
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
			postsData = data;
		} catch (err) {
			error = err.message;
		} finally {
			loading = false;
		}
	};
	const startChat = (reciever) => {
		localStorage.setItem('reciever', reciever);
		window.location.href = base + '/chats';
	};
</script>

<div class="layout">
	<Sidebar />

	<div class="main-content">
		<div class="center-container">
			{#if error}
				<p class="error">{error}</p>
			{:else if publicUser}
				<div class="profile-card">
					<div class="profile-header">
						<UserAvatar username={publicUser.username} size="60px" />
						<div>
							<h2>{publicUser.firstName} {publicUser.lastName}</h2>
							<p class="username">@{publicUser.username}</p>
							<p class="location">From {publicUser.city}</p>
						</div>

						<!-- Added the chat button here -->

						<button class="chat-button" on:click={() => startChat(publicUser.username)}>
							<i class="fas fa-comments"></i>

							<span>Chat</span>
						</button>
					</div>
					<div class="stats">
						<div><strong>{postsData?.totalElements || '0'}</strong><br /> Posts</div>
						<div><strong>{publicUser.followers || '12.7K'}</strong><br /> Followers</div>
						<div><strong>{publicUser.following || '221'}</strong> <br />Following</div>
					</div>

					<div class="about">
						<h3>About Me</h3>
						<p>{publicUser.bio}</p>
					</div>
				</div>
				<div class="profile-card education-card">
					<div class="achievements">
						<h3 class="card-heading">Achievements 🏆</h3>
						<div class="achievements-grid">
							<div class="achievement-card-wrapper">
								<div class="animated-border"></div>
								<div class="achievement-card">
									<div>
										<h4>Top Contributor</h4>
										<p>Recognized for sharing 100+ quality posts.</p>
									</div>
								</div>
							</div>

							<div class="achievement-card-wrapper">
								<div class="animated-border"></div>
								<div class="achievement-card">
									<div>
										<h4>Verified Member</h4>
										<p>Officially verified profile badge earned.</p>
									</div>
								</div>
							</div>

							<div class="achievement-card-wrapper">
								<div class="animated-border"></div>
								<div class="achievement-card">
									<div>
										<h4>Early Adopter</h4>
										<p>Joined within the first 100 users!</p>
									</div>
								</div>
							</div>
						</div>
					</div>
				</div>

				<div class="profile-card education-card">
					<h3 class="card-heading">Education</h3>
					<div class="education-details">
						<div class="institution-name">{publicUser.collegeName}</div>
						<div class="degree-info">Degree: {publicUser.education}</div>
					</div>
				</div>
				<div class="profile-card education-card">
					<h3 class="card-heading">Current Job</h3>
					<div class="education-details">
						<div class="institution-name">WHAT JOB?</div>
						<div class="degree-info">Job Title: {publicUser.jobTitle}</div>
						<div class="degree-info">Skills: {publicUser.skills}</div>
					</div>
				</div>
				<div class="profile-card education-card">
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

					<h3 class="card-heading">Posts</h3>
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
							<PostCard
								{post}
								{token}
								on:postUpdated={handlePostUpdate}
								on:error={(e) => (error = e.detail)}
							/>
						{/each}
					{/if}
				</div>
			{:else}
				<p class="loading">Loading profile…</p>
			{/if}
		</div>
	</div>
</div>

<style>
	.chat-button {
		/* position: absolute; */
		/* top: 1rem; */
		/* right: 1rem; */
		padding: 0.5rem 1rem;
		background-color: #6b5bff;
		color: #fff;
		border-radius: 0.5rem;
		cursor: pointer;
		margin-left: auto;
		transition: all 0.2s ease;
	}
	.chat-button:hover {
		background-color: #5840ff;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(107, 91, 255, 0.4);
	}
</style>
