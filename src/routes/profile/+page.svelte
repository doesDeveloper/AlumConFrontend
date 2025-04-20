<script>
	import { API_URL } from '$lib/config';
	import { onMount } from 'svelte';
	import { page } from '$app/stores';
	import Sidebar from '$lib/components/Sidebar.svelte';
	import '$lib/styles/publicprof.css';
	import { base } from '$app/paths';

	// Example usage
	const url = API_URL;
	let user = null;
	let error = null;
	let success = null;
	let isEditing = false;
	let loading = false;

	let posts = [];
	let postsData = null;
	let token;
	const fetchUserPosts = async () => {
		try {
			loading = true;
			const res = await fetch(`${API_URL}posts/feed/${user.username}?page=0`, {
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
	async function updateProfile() {
		success = null;
		error = null;
		try {
			const response = await fetch(url + 'users/update', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json',
					Authorization: `Bearer ${token}`
				},
				body: JSON.stringify(user)
			});

			if (!response.ok) throw new Error('Failed to update profile');
			user = await response.json();
			success = 'Profile updated successfully!';
			isEditing = false;
		} catch (err) {
			error = err.message;
		}
	}
	onMount(async () => {
		token = localStorage.getItem('token');
		if (!token) {
			error = 'You are not logged in.';
			window.location.href = base + '/login';
			return;
		}

		try {
			const response = await fetch(url + 'users/me', {
				method: 'GET',
				headers: {
					Authorization: `Bearer ${token}`
				}
			});

			if (!response.ok) throw new Error('Failed to fetch user data');
			user = await response.json();
		} catch (err) {
			error = err.message;
		}
		await fetchUserPosts();
	});
</script>

<div class="layout">
	<Sidebar />

	<div class="main-content">
	<div class="center-container">
		<div class="top-bar">
			<input type="text" placeholder="Search for friends, groups, pages" />
			<button class="add-post-btn">Add New Post ➕</button>
		</div>
		{#if error}
			<p class="error">{error}</p>
		{:else if user}
			<div class="profile-card">
				<div class="profile-header">
					<div class="logo-icon profile-pic logo-text">A</div>
					<div>
						<h2>{user.firstName} {user.lastName}</h2>
						<p class="username">@{user.username}</p>
						<p class="location">From {user.city}</p>
					</div>
				</div>
				<div class="stats">
					<div><strong>{postsData?.totalElements || '12'}</strong><br /> Posts</div>
					<div><strong>{user.followers || '12.7K'}</strong><br /> Followers</div>
					<div><strong>{user.following || '221'}</strong> <br />Following</div>
				</div>

				<div class="about">
					<h3>About Me</h3>
					<p>{user.bio}</p>
				</div>
			</div>
			<div class="profile-card education-card">
				<h3 class="card-heading">Contact</h3>
				<div class="education-details">
					<div class="degree-info">📞 {user.phone || '+123 456 789 000'}</div>
					<div class="degree-info">✉️ {user.email || 'hello@example.com'}</div>
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
					<div class="institution-name">{user.collegeName}</div>
					<div class="degree-info">Degree: {user.education}</div>
				</div>
			</div>
			<div class="profile-card education-card">
				<h3 class="card-heading">Current Job</h3>
				<div class="education-details">
					<div class="institution-name">WHAT JOB?</div>
					<div class="degree-info">Job Title: {user.jobTitle}</div>
					<div class="degree-info">Skills: {user.skills}</div>
				</div>
			</div>
			<div class="profile-card education-card">
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
						<div class="post-card">
							<div class="post-header">
								<div class="logo-icon user-avatar">{post.username.charAt(0).toUpperCase()}</div>
								<div class="user-info">
									<div class="user-name">
										<a
											href="{base}/profile/others?name={post.username}st.username}st.username}st.username}st.username}"
											>{post.username}</a
										>
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
</div>

<style>
</style>
