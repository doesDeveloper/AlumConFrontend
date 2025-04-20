<script>
	import { API_URL } from '$lib/config';

	import { base } from '$app/paths';

	// Example usage
	const url = API_URL;
	let username = '';
	let password = '';
	let loading = false;
	async function handleLogin() {
		try {
			loading = true;
			const response = await fetch(url + 'auth/login', {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({ username, password })
			});

			if (!response.ok) {
				throw new Error('Login failed');
			}

			const token = await response.text();
			localStorage.setItem('token', token);

			// Fetch user details
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
			// Navigate
			window.location.href = base + '/profile';
		} catch (error) {
			console.error('Error during login:', error);
			alert('Invalid username or password. Please try again.');
		}
		loading = false;
	}
</script>

{#if loading}
	<p>Loading...</p>
{/if}

<div class="login-page">
	<div class="card">
		<h2 class="title">Welcome Back</h2>

		<form on:submit|preventDefault={handleLogin}>
			<div class="field">
				<label for="username">Username</label>
				<input id="username" type="text" bind:value={username} required autocomplete="username" />
			</div>

			<div class="field">
				<label for="password">Password</label>
				<input
					id="password"
					type="password"
					bind:value={password}
					required
					autocomplete="current-password"
				/>
			</div>

			<button class="btn" type="submit">Login</button>
		</form>

		<p class="alt-action">
			Don’t have an account?
			<a href="{base}/register">Sign Up</a>
		</p>
	</div>
</div>

<style>
	/* Page background */
	.login-page {
		display: flex;
		align-items: center;
		justify-content: center;
		min-height: 100vh;
		background: linear-gradient(135deg, #eef2f7, #ffffff);
		padding: 1rem;
	}

	/* Card */
	.card {
		background: #fff;
		border-radius: 12px;
		box-shadow: 0 12px 24px rgba(0, 0, 0, 0.08);
		padding: 2.5rem;
		max-width: 360px;
		width: 100%;
		text-align: center;
		transition: transform 0.2s ease;
	}
	.card:hover {
		transform: translateY(-4px);
	}

	/* Title */
	.title {
		margin-bottom: 1.5rem;
		font-size: 1.75rem;
		color: #333;
		font-weight: 600;
	}

	/* Form fields */
	.field {
		display: flex;
		flex-direction: column;
		margin-bottom: 1.25rem;
		text-align: left;
	}
	.field label {
		margin-bottom: 0.5rem;
		font-size: 0.9rem;
		color: #555;
	}
	.field input {
		padding: 0.75rem 1rem;
		border: 1px solid #d1d5db;
		border-radius: 6px;
		font-size: 1rem;
		transition:
			border-color 0.2s ease,
			box-shadow 0.2s ease;
	}
	.field input:focus {
		outline: none;
		border-color: #0077ff;
		box-shadow: 0 0 0 3px rgba(0, 119, 255, 0.2);
	}

	/* Button */
	.btn {
		width: 100%;
		padding: 0.85rem;
		margin-top: 0.5rem;
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

	/* Alternative action link */
	.alt-action {
		margin-top: 1.5rem;
		font-size: 0.9rem;
		color: #666;
	}
	.alt-action a {
		color: #0077ff;
		text-decoration: none;
		font-weight: 500;
		transition: color 0.2s ease;
	}
	.alt-action a:hover {
		color: #0056cc;
		text-decoration: underline;
	}

	/* Responsive tweak */
	@media (max-width: 400px) {
		.card {
			padding: 2rem 1.5rem;
		}
		.title {
			font-size: 1.5rem;
		}
	}
</style>
