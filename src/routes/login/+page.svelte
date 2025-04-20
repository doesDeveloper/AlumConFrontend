<script>
	import { API_URL } from '$lib/config';

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
			window.location.href = '/profile';
		} catch (error) {
			console.error('Error during login:', error);
			alert('Invalid username or password. Please try again.');
		}
		loading = false;
	}
</script>

<h1>Login</h1>

{#if loading}
	<p>Loading...</p>
{/if}
<form on:submit|preventDefault={handleLogin}>
	<label>
		Username:
		<input bind:value={username} required />
	</label>

	<label>
		Password:
		<input type="password" bind:value={password} required />
	</label>

	<button type="submit">Login</button>
</form>

<a href="/">Back to Home</a>
