<script>
	import { API_URL } from '$lib/config';
	import { fly, fade, slide } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';
	import { base } from '$app/paths';
	const url = API_URL;

	// Control which form is active
	let isLoginActive = true;

	// Login form data and state
	let loginData = {
		username: '',
		password: ''
	};

	// Registration form data and state
	let registerData = {
		username: '',
		password: '',
		firstName: '',
		lastName: '',
		email: '',
		phone: '',
		address: '',
		city: '',
		zip: '',
		education: '',
		schoolName: '',
		collegeName: '',
		branch: '',
		jobTitle: '',
		skills: '',
		interests: '',
		bio: '',
		passOutYear: '',
		isAlumni: ''
	};

	let errors = {};
	let errorMessage = '';
	let successMessage = '';
	let isSubmitting = false;

	// Toggle between login and register forms with animation
	function toggleForm() {
		errorMessage = '';
		successMessage = '';
		errors = {};
		isLoginActive = !isLoginActive;
	}

	// Validate login form
	function validateLoginForm() {
		errors = {};
		if (!loginData.username.trim()) errors.username = 'Username is required';
		if (!loginData.password.trim()) errors.password = 'Password is required';
		return Object.keys(errors).length === 0;
	}

	// Validate registration form
	function validateRegisterForm() {
		errors = {};
		if (!registerData.username.trim()) errors.username = 'Username is required';
		if (!registerData.password.trim() || registerData.password.length < 6)
			errors.password = 'Password must be at least 6 characters';
		if (!registerData.firstName.trim()) errors.firstName = 'First name is required';
		if (!registerData.email.trim() || !registerData.email.includes('@'))
			errors.email = 'Valid email is required';
		if (!registerData.phone.trim() || registerData.phone.length < 10)
			errors.phone = 'Valid phone number is required';
		return Object.keys(errors).length === 0;
	}

	// Handle login submission
	async function handleLogin() {
		if (!validateLoginForm()) return;
		isSubmitting = true;
		errorMessage = '';
		successMessage = '';

		try {
			const response = await fetch(url + 'auth/login', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify(loginData)
			});

			if (!response.ok) throw new Error('Invalid username or password');

			let token = await response.text();
			localStorage.setItem('token', token);
			const userRes = await fetch(url + 'users/me', {
				headers: {
					Authorization: `Bearer ${token}`
				}
			});

			if (!userRes.ok) {
				throw new Error('Fetching user info failed');
			}

			const user = await userRes.json();
			localStorage.setItem('user', JSON.stringify(user));
			successMessage = 'Login successful! Redirecting...';
			setTimeout(() => (window.location.href = base + '/home'), 1500);
		} catch (err) {
			errorMessage = err.message || 'Something went wrong. Please try again.';
		} finally {
			isSubmitting = false;
		}
	}

	// Handle registration submission
	async function handleRegister() {
		if (!validateRegisterForm()) return;
		isSubmitting = true;
		errorMessage = '';
		successMessage = '';

		try {
			const response = await fetch(url + 'auth/register', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify(registerData)
			});
			const data = await response.text();
			if (!response.ok) {
				console.error(response);
				let eee = await response.text();
				console.log('WAS HERE');
				console.error(eee);
				throw new Error(data.message || 'Failed to register');
			}

			successMessage = 'Registration successful! You can now log in.';
			setTimeout(() => {
				isLoginActive = true;
			}, 1000);
		} catch (err) {
			errorMessage = err.message || 'Something went wrong. Please try again.';
		} finally {
			isSubmitting = false;
		}
	}
</script>

<div class="auth-page">
	<div class="auth-container">
		<div class="branding">
			<div class="logo">
				<span>AlumCon</span>
			</div>
			<p class="tagline">Connect with your alumni network</p>
		</div>

		<div class="card-container">
			<div class="card">
				<div class="card-header">
					<div class="tabs">
						<button
							class="tab {isLoginActive ? 'active' : ''}"
							on:click={() => (isLoginActive ? null : toggleForm())}
						>
							Sign In
						</button>
						<button
							class="tab {!isLoginActive ? 'active' : ''}"
							on:click={() => (!isLoginActive ? null : toggleForm())}
						>
							Register
						</button>
					</div>
				</div>

				{#if errorMessage}
					<div class="error-message" in:fade={{ duration: 300 }}>{errorMessage}</div>
				{/if}
				{#if successMessage}
					<div class="success-message" in:fade={{ duration: 300 }}>{successMessage}</div>
				{/if}

				<div class="card-content">
					{#if isLoginActive}
						<div
							in:fly={{ x: -30, duration: 300, delay: 300, easing: quintOut }}
							out:fly={{ x: 30, duration: 300 }}
						>
							<h2 class="title">Welcome Back</h2>
							<p class="subtitle">Sign in to your AlumCon account</p>

							<form on:submit|preventDefault={handleLogin}>
								<div class="field">
									<label for="login-username">Username</label>
									<input
										id="login-username"
										type="text"
										bind:value={loginData.username}
										required
										autocomplete="username"
									/>
									{#if errors.username}<p class="field-error">{errors.username}</p>{/if}
								</div>

								<div class="field">
									<label for="login-password">Password</label>
									<input
										id="login-password"
										type="password"
										bind:value={loginData.password}
										required
										autocomplete="current-password"
									/>
									{#if errors.password}<p class="field-error">{errors.password}</p>{/if}
								</div>

								<button class="btn" type="submit" disabled={isSubmitting}>
									{isSubmitting ? 'Signing in...' : 'Sign In'}
								</button>
							</form>
						</div>
					{:else}
						<div
							in:fly={{ x: 30, duration: 300, delay: 300, easing: quintOut }}
							out:fly={{ x: -30, duration: 300 }}
						>
							<h2 class="title">Create Account</h2>
							<p class="subtitle">Join the AlumCon community</p>

							<form on:submit|preventDefault={handleRegister} class="register-form">
								<div class="form-section">
									<h3 class="section-title">Account Information</h3>
									<div class="field">
										<label for="username">Username*</label>
										<input id="username" type="text" bind:value={registerData.username} required />
										{#if errors.username}<p class="field-error">{errors.username}</p>{/if}
									</div>

									<div class="field">
										<label for="password">Password*</label>
										<input
											id="password"
											type="password"
											bind:value={registerData.password}
											required
										/>
										{#if errors.password}<p class="field-error">{errors.password}</p>{/if}
									</div>
								</div>

								<div class="form-section">
									<h3 class="section-title">Personal Information</h3>
									<div class="field">
										<label for="firstName">First Name*</label>
										<input
											id="firstName"
											type="text"
											bind:value={registerData.firstName}
											required
										/>
										{#if errors.firstName}<p class="field-error">{errors.firstName}</p>{/if}
									</div>

									<div class="field">
										<label for="lastName">Last Name*</label>
										<input id="lastName" type="text" bind:value={registerData.lastName} required />
									</div>

									<div class="field">
										<label for="email">Email*</label>
										<input id="email" type="email" bind:value={registerData.email} required />
										{#if errors.email}<p class="field-error">{errors.email}</p>{/if}
									</div>

									<div class="field">
										<label for="phone">Phone*</label>
										<input id="phone" type="tel" bind:value={registerData.phone} required />
										{#if errors.phone}<p class="field-error">{errors.phone}</p>{/if}
									</div>
								</div>

								<div class="form-section">
									<h3 class="section-title">Address</h3>
									<div class="field">
										<label for="address">Address*</label>
										<input id="address" type="text" bind:value={registerData.address} required />
									</div>

									<div class="field">
										<label for="city">City*</label>
										<input id="city" type="text" bind:value={registerData.city} required />
									</div>

									<div class="field">
										<label for="zip">Zip*</label>
										<input id="zip" type="text" bind:value={registerData.zip} required />
									</div>
								</div>

								<div class="form-section">
									<h3 class="section-title">Education & Career</h3>
									<div class="field">
										<label for="education">Education*</label>
										<input
											id="education"
											type="text"
											bind:value={registerData.education}
											required
										/>
									</div>
									<div class="field">
										<label for="schoolName">School Name*</label>
										<input
											id="schoolName"
											type="text"
											bind:value={registerData.schoolName}
											required
										/>
									</div>
									<!-- <div class="field">
										<label for="collegeName">College Name*</label>
										<input
											id="collegeName"
											type="text"
											bind:value={registerData.collegeName}
											required
										/>
									</div> -->
									<div class="field">
										<label for="university">College Name*</label>
										<select id="university" bind:value={registerData.collegeName} required>
											<option value="" disabled selected>Select a university</option>
											<option value="Jamia Millia Islamia">Jamia Millia Islamia</option>
											<option value="Aligarh Muslim University">Aligarh Muslim University</option>
											<option value="Delhi University">Delhi University</option>
										</select>
									</div>
									<div class="field">
										<label for="passOutYear">Pass Out Year*</label>
										<input id="passOutYear" type="number" bind:value={registerData.passOutYear} required />
									</div>

									<div class="field">
										<label for="branch">Branch*</label>
										<input id="branch" type="text" bind:value={registerData.branch} required />
									</div>
									<div class="field">
										<label for="jobTitle">Job Title*</label>
										<input id="jobTitle" type="text" bind:value={registerData.jobTitle} required />
									</div>
								</div>

								<div class="form-section">
									<h3 class="section-title">Skills & Interests</h3>
									<div class="field">
										<label for="skills">Skills*</label>
										<input id="skills" type="text" bind:value={registerData.skills} required />
									</div>
									<div class="field">
										<label for="interests">Interests*</label>
										<input
											id="interests"
											type="text"
											bind:value={registerData.interests}
											required
										/>
									</div>
									<div class="field">
										<label for="isAlumni">Are you an alumni?</label>
										<select id="isAlumni" bind:value={registerData.isAlumni} required>
											<option value="" disabled selected>Select an option</option>
											<option value="true">Yes</option>
											<option value="false">No</option>
										</select>
									</div>
								</div>

								<div class="form-section">
									<h3 class="section-title">Bio</h3>
									<div class="field">
										<label for="bio">Bio</label>
										<textarea
											id="bio"
											bind:value={registerData.bio}
											placeholder="Tell us about yourself..."
										></textarea>
									</div>
								</div>

								<button class="btn" type="submit" disabled={isSubmitting}>
									{isSubmitting ? 'Registering…' : 'Register'}
								</button>
							</form>
						</div>
					{/if}
				</div>
			</div>
		</div>
	</div>
</div>

<style>
	/* Page wrapper */
	.auth-page {
		display: flex;
		justify-content: center;
		padding: 2rem 1rem;
		background: linear-gradient(135deg, #eef2f7, #ffffff);
		min-height: 100vh;
	}

	.auth-container {
		display: flex;
		flex-direction: column;
		width: 100%;
		max-width: 600px; /* Reduced max-width for single column layout */
	}

	/* Branding */
	.branding {
		text-align: center;
		margin-bottom: 2rem;
	}

	.logo {
		font-size: 2.5rem;
		font-weight: 700;
		color: #0077ff;
		margin-bottom: 0.5rem;
		letter-spacing: -0.5px;
	}

	.logo span {
		background: linear-gradient(135deg, #0077ff, #00c6ff);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	.tagline {
		font-size: 1.1rem;
		color: #555;
		margin: 0;
	}

	/* Card container with perspective */
	.card-container {
		perspective: 1000px;
	}

	/* Card */
	.card {
		background: #fff;
		border-radius: 12px;
		box-shadow: 0 12px 24px rgba(0, 0, 0, 0.08);
		padding: 0;
		width: 100%;
		transition:
			transform 0.3s ease,
			box-shadow 0.3s ease;
		transform-style: preserve-3d;
	}
	.card:hover {
		transform: translateY(-4px);
		box-shadow: 0 20px 32px rgba(0, 0, 0, 0.12);
	}

	/* Card Header with Tabs */
	.card-header {
		padding: 0;
		border-bottom: 1px solid #eaeaea;
	}

	.tabs {
		display: flex;
		width: 100%;
	}

	.tab {
		flex: 1;
		padding: 1.25rem 0;
		background: transparent;
		border: none;
		cursor: pointer;
		font-size: 1rem;
		font-weight: 500;
		color: #777;
		transition: all 0.3s ease;
		position: relative;
	}

	.tab.active {
		color: #0077ff;
		font-weight: 600;
	}

	.tab.active::after {
		content: '';
		position: absolute;
		bottom: 0;
		left: 0;
		width: 100%;
		height: 3px;
		background: linear-gradient(90deg, #0077ff, #00c6ff);
	}

	.tab:hover:not(.active) {
		color: #444;
		background-color: rgba(0, 119, 255, 0.05);
	}

	/* Card Content */
	.card-content {
		padding: 2.5rem;
		min-height: 300px;
	}

	.title {
		margin: 0;
		font-size: 1.75rem;
		color: #333;
		font-weight: 600;
		text-align: center;
	}
	.subtitle {
		margin: 0.5rem 0 1.5rem;
		text-align: center;
		color: #666;
		font-size: 0.95rem;
	}

	/* Form Sections */
	.form-section {
		margin-bottom: 2rem;
		padding-bottom: 1.5rem;
		border-bottom: 1px solid #eaeaea;
	}

	.form-section:last-child {
		border-bottom: none;
	}

	.section-title {
		font-size: 1.1rem;
		color: #333;
		margin: 0 0 1rem 0;
		font-weight: 600;
	}

	.error-message {
		background: #ffe0e0;
		color: #d62828;
		padding: 0.75rem;
		border-radius: 6px;
		margin: 1rem 2.5rem 0;
		text-align: center;
	}

	.success-message {
		background: #e6f9f0;
		color: #2f9e44;
		padding: 0.75rem;
		border-radius: 6px;
		margin: 1rem 2.5rem 0;
		text-align: center;
	}

	form {
		display: flex;
		flex-direction: column;
	}

	/* Single column layout for fields */
	.field {
		display: flex;
		flex-direction: column;
		margin-bottom: 1.25rem;
		width: 100%;
	}

	.field:last-child {
		margin-bottom: 0;
	}

	.field label {
		margin-bottom: 0.4rem;
		font-size: 0.9rem;
		color: #555;
	}
	.field input,
	.field textarea,
	.field select {
		padding: 0.75rem 1rem;
		border: 1px solid #d1d5db;
		border-radius: 6px;
		font-size: 1rem;
		transition:
			border-color 0.2s ease,
			box-shadow 0.2s ease;
		width: 100%;
	}
	.field input:focus,
	.field textarea:focus,
	.field select:focus {
		outline: none;
		border-color: #0077ff;
		box-shadow: 0 0 0 3px rgba(0, 119, 255, 0.2);
	}
	.field-error {
		margin: 0.25rem 0 0;
		font-size: 0.85rem;
		color: #d62828;
	}

	.field textarea {
		min-height: 120px;
		resize: vertical;
	}

	/* Remember and forgot */
	.remember-forgot {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 1.5rem;
		font-size: 0.9rem;
	}

	.remember {
		display: flex;
		align-items: center;
	}

	.remember input[type='checkbox'] {
		margin-right: 0.5rem;
	}

	.forgot {
		color: #0077ff;
		text-decoration: none;
		transition: color 0.2s ease;
	}

	.forgot:hover {
		color: #0056cc;
		text-decoration: underline;
	}

	/* Button */
	.btn {
		margin-top: 1rem;
		padding: 0.85rem;
		background: linear-gradient(135deg, #0077ff, #00c6ff);
		color: #fff;
		font-weight: 600;
		border: none;
		border-radius: 6px;
		cursor: pointer;
		transition:
			filter 0.2s ease,
			transform 0.1s ease;
	}
	.btn:hover {
		filter: brightness(1.1);
	}
	.btn:active {
		transform: scale(0.98);
	}
	.btn:disabled {
		opacity: 0.7;
		cursor: not-allowed;
	}

	/* Register form specific styles */
	.register-form {
		max-height: 600px;
		overflow-y: auto;
		padding-right: 10px; /* Allow space for scrollbar */
	}

	.register-form::-webkit-scrollbar {
		width: 8px;
	}

	.register-form::-webkit-scrollbar-track {
		background: #f1f1f1;
		border-radius: 4px;
	}

	.register-form::-webkit-scrollbar-thumb {
		background: #c1c1c1;
		border-radius: 4px;
	}

	.register-form::-webkit-scrollbar-thumb:hover {
		background: #a8a8a8;
	}

	/* Responsive */
	@media (max-width: 768px) {
		.card-content {
			padding: 2rem 1.5rem;
		}
		.title {
			font-size: 1.5rem;
		}
		.error-message,
		.success-message {
			margin: 1rem 1.5rem 0;
		}
	}

	@media (max-width: 480px) {
		.logo {
			font-size: 2rem;
		}
		.card-content {
			padding: 1.5rem 1.25rem;
		}
		.section-title {
			font-size: 1rem;
		}
	}
</style>
