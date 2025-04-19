<script>
    import { API_URL } from '$lib/config';

// Example usage
    const url = API_URL;
    let username = '';
    let password = '';
  
    async function handleLogin() {
      try {
        const response = await fetch(url+"auth/login", {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify({ username, password })
        });
  
        if (!response.ok) {
          throw new Error('Login failed');
        }
  
        // const data = await response.json();
        // console.log('Login successful:', data);
  
        // Store the token in localStorage or sessionStorage
        localStorage.setItem('token', await response.text());
  
        // Redirect to the dashboard or home page after successful login
        window.location.href = '/profile'; // Or wherever you want to navigate
      } catch (error) {
        console.error('Error during login:', error);
        alert('Invalid username or password. Please try again.');
      }
    }
  </script>
  
  
  <h1>Login</h1>

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
  