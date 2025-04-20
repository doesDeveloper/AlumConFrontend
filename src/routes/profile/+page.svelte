<script>
	import { API_URL } from '$lib/config';
	import { onMount } from 'svelte';
	import { page } from '$app/stores';
	import Sidebar from '$lib/components/Sidebar.svelte';
	import '$lib/styles/publicprof.css';
	import { base } from '$app/paths';
	import PostCard from '$lib/components/PostCard.svelte';
	import { goto } from '$app/navigation';
    import UserAvatar from '$lib/components/UserAvatar.svelte';

	// Example usage
	const url = API_URL;
	let user = null;
	let error = null;
	let success = null;
	let isEditing = false;
	let loading = false;
	// Create a copy of user data for editing
	let editableUser = null;

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
	
	let centerContainer;
	function startEditing() {
		// Create a deep copy of user to edit
		editableUser = JSON.parse(JSON.stringify(user));
		isEditing = true;
		// Scroll to top when starting edit mode
		centerContainer.scrollTo({ top: 0, behavior: 'smooth' });
	}

	function cancelEditing() {
		isEditing = false;
		success = null;
		error = null;
	}

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
				body: JSON.stringify(editableUser)
			});

			if (!response.ok) throw new Error('Failed to update profile');
			user = await response.json();
			success = 'Profile updated successfully!';
			isEditing = false;
			// Scroll to top to see success message
			centerContainer.scrollTo({ top: 0, behavior: 'smooth' });
		} catch (err) {
			error = err.message;
		}
	}
	function handleLogout() {
		goto(base + '/logout');
	}
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
		<div class="center-container" bind:this={centerContainer}>
			{#if error}
				<p class="error">{error}</p>
			{:else if user}
				{#if success}
					<div class="success-message">
						<span class="success-icon">✓</span>
						{success}
					</div>
				{/if}

				{#if isEditing}
					<div class="edit-mode-banner">
						<span class="edit-icon">✏️</span> You are in edit mode. Make your changes and save when you're
						done.
					</div>
				{/if}

				<div class="profile-card">
					<div class="profile-header">
						<UserAvatar username={user.username} size="60px" />
						<div>
							{#if !isEditing}
								<h2>{user.firstName} {user.lastName}</h2>
								<p class="username">@{user.username}</p>
								<p class="location">From {user.city}</p>
								<button class="edit-profile-btn" on:click={startEditing}>
									<span class="edit-icon"></span> Edit Profile
								</button>
							{:else}
								<div class="edit-form">
									<h3 class="edit-section-title">Personal Information</h3>
									<div class="form-group">
										<label for="firstName">First Name</label>
										<input type="text" id="firstName" bind:value={editableUser.firstName} />
									</div>
									<div class="form-group">
										<label for="lastName">Last Name</label>
										<input type="text" id="lastName" bind:value={editableUser.lastName} />
									</div>
									<div class="form-group">
										<label for="username"
											>Username <span class="non-editable">(Not Editable)</span></label
										>
										<input type="text" id="username" value={user.username} disabled />
									</div>
									<div class="form-group">
										<label for="city">City</label>
										<input type="text" id="city" bind:value={editableUser.city} />
									</div>
								</div>
							{/if}
						</div>
						<button class="logout-btn" on:click={handleLogout}>
							<i class="fa fa-sign-out-alt"></i> Logout
						</button>
					</div>
					<div class="stats">
						<div><strong>{postsData?.totalElements || '0'}</strong><br /> Posts</div>
						<div><strong>{user.followers || '12.7K'}</strong><br /> Followers</div>
						<div><strong>{user.following || '221'}</strong> <br />Following</div>
					</div>

					<div class="about">
						<h3>About Me</h3>
						{#if !isEditing}
							<p>{user.bio}</p>
						{:else}
							<div class="form-group">
								<label for="bio">Bio</label>
								<textarea
									id="bio"
									bind:value={editableUser.bio}
									rows="4"
									placeholder="Tell us about yourself..."
								></textarea>
							</div>
						{/if}
					</div>
				</div>

				<div class="profile-card education-card">
					<h3 class="card-heading">Contact</h3>
					<div class="education-details">
						<div class="degree-info">📞 {user.phone || '+123 456 789 000'}</div>
						<div class="degree-info">✉️ {user.email || 'hello@example.com'}</div>
						{#if isEditing}
							<div class="form-note">
								<span class="note-icon">ℹ️</span> Contact information cannot be edited from this page.
								Please contact support to update these details.
							</div>
						{/if}
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
					{#if !isEditing}
						<div class="education-details">
							<div class="institution-name">{user.collegeName}</div>
							<div class="degree-info">Degree: {user.education}</div>
						</div>
					{:else}
						<div class="edit-form">
							<h3 class="edit-section-title">Education Details</h3>
							<div class="form-group">
								<label for="collegeName">College/University</label>
								<input
									type="text"
									id="collegeName"
									bind:value={editableUser.collegeName}
									placeholder="Enter your university name"
								/>
							</div>
							<div class="form-group">
								<label for="education">Degree</label>
								<input
									type="text"
									id="education"
									bind:value={editableUser.education}
									placeholder="e.g. Bachelor of Science in Computer Science"
								/>
							</div>
						</div>
					{/if}
				</div>

				<div class="profile-card education-card">
					<h3 class="card-heading">Current Job</h3>
					{#if !isEditing}
						<div class="education-details">
							<div class="institution-name">{user.company || 'Not specified'}</div>
							<div class="degree-info">Job Title: {user.jobTitle || 'Not specified'}</div>
							<div class="degree-info">Skills: {user.skills || 'Not specified'}</div>
						</div>
					{:else}
						<div class="edit-form">
							<h3 class="edit-section-title">Professional Information</h3>
							<div class="form-group">
								<label for="company">Company</label>
								<input
									type="text"
									id="company"
									bind:value={editableUser.company}
									placeholder="Where do you work?"
								/>
							</div>
							<div class="form-group">
								<label for="jobTitle">Job Title</label>
								<input
									type="text"
									id="jobTitle"
									bind:value={editableUser.jobTitle}
									placeholder="Your position or role"
								/>
							</div>
							<div class="form-group">
								<label for="skills">Skills</label>
								<input
									type="text"
									id="skills"
									bind:value={editableUser.skills}
									placeholder="e.g. JavaScript, Project Management, Marketing"
								/>
							</div>
						</div>
					{/if}
				</div>

				{#if isEditing}
					<div class="save-actions-container">
						<div class="save-actions">
							<button class="save-btn" on:click={updateProfile}>
								<span class="save-icon"></span> Save All Changes
							</button>
							<button class="cancel-btn" on:click={cancelEditing}>
								<span class="cancel-icon">✖</span> Cancel
							</button>
						</div>
					</div>
				{/if}

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
						{#each posts as post (post.id)}
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
	.edit-profile-btn {
		margin-top: 12px;
		padding: 8px 16px;
		background-color: #4a76a8;
		color: white;
		border: none;
		border-radius: 6px;
		cursor: pointer;
		font-size: 14px;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: all 0.2s ease;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
	}

	.edit-profile-btn:hover {
		background-color: #3a5b85;
		transform: translateY(-2px);
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
	}

	.edit-icon,
	.save-icon,
	.cancel-icon,
	.note-icon {
		margin-right: 6px;
		font-size: 16px;
	}

	.edit-form {
		margin: 15px 0;
		width: 100%;
	}

	.edit-section-title {
		font-size: 16px;
		color: #4a76a8;
		margin-bottom: 12px;
		border-bottom: 2px solid #e6e6e6;
		padding-bottom: 8px;
	}

	.form-group {
		margin-bottom: 16px;
	}

	.form-group label {
		display: block;
		margin-bottom: 6px;
		font-weight: 500;
		font-size: 14px;
		color: #333;
	}

	.non-editable {
		color: #888;
		font-size: 12px;
		font-style: italic;
	}

	.form-group input,
	.form-group textarea {
		width: 100%;
		padding: 12px;
		border: 1px solid #ddd;
		border-radius: 6px;
		font-size: 14px;
		transition: border 0.2s ease;
	}

	.form-group input:focus,
	.form-group textarea:focus {
		border-color: #4a76a8;
		outline: none;
		box-shadow: 0 0 0 2px rgba(74, 118, 168, 0.2);
	}

	.form-group input:disabled {
		background-color: #f5f5f5;
		color: #888;
		cursor: not-allowed;
	}

	.save-actions-container {
		position: sticky;
		bottom: 20px;
		width: 100%;
		display: flex;
		justify-content: center;
		z-index: 100;
		margin-top: 20px;
		margin-bottom: 20px;
	}

	.save-actions {
		display: flex;
		gap: 12px;
		background-color: white;
		padding: 15px 20px;
		border-radius: 12px;
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
		border: 1px solid #e0e0e0;
	}

	.save-btn {
		padding: 12px 24px;
		background-color: #4caf50;
		color: white;
		border: none;
		border-radius: 6px;
		cursor: pointer;
		font-weight: 600;
		font-size: 15px;
		display: flex;
		align-items: center;
		transition: all 0.2s ease;
	}

	.save-btn:hover {
		background-color: #3e8e41;
		transform: translateY(-2px);
		box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
	}

	.cancel-btn {
		padding: 12px 24px;
		background-color: #f5f5f5;
		color: #333;
		border: 1px solid #ddd;
		border-radius: 6px;
		cursor: pointer;
		font-weight: 500;
		font-size: 15px;
		display: flex;
		align-items: center;
		transition: all 0.2s ease;
	}

	.cancel-btn:hover {
		background-color: #e8e8e8;
	}

	.form-note {
		font-size: 14px;
		color: #666;
		background-color: #f9f9f9;
		padding: 10px;
		border-radius: 6px;
		border-left: 3px solid #4a76a8;
		margin-top: 12px;
		display: flex;
		align-items: center;
	}

	.success-message {
		background-color: #dff0d8;
		color: #3c763d;
		padding: 12px 16px;
		border-radius: 6px;
		margin-bottom: 20px;
		border-left: 4px solid #4caf50;
		display: flex;
		align-items: center;
	}

	.success-icon {
		margin-right: 8px;
		background-color: #4caf50;
		color: white;
		border-radius: 50%;
		width: 24px;
		height: 24px;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 14px;
	}

	.error {
		background-color: #f2dede;
		color: #a94442;
		padding: 12px 16px;
		border-radius: 6px;
		margin-bottom: 20px;
		border-left: 4px solid #f44336;
	}

	.edit-mode-banner {
		background-color: #fff3cd;
		color: #856404;
		padding: 12px 16px;
		border-radius: 6px;
		margin-bottom: 20px;
		border-left: 4px solid #ffc107;
		display: flex;
		align-items: center;
	}
	.logout-btn {
		padding: 0.5rem 1rem;
		background-color: #6b5bff;
		color: #fff;
		border-radius: 0.5rem;
		cursor: pointer;
		margin-left: auto;
		transition: all 0.2s ease;
	}
	.logout-btn:hover {
		background-color: #5840ff;
		transform: translateY(-2px);
		box-shadow: 0 4px 12px rgba(107, 91, 255, 0.4);
	}
</style>
