<script>
  import { API_URL } from '$lib/config';

// Example usage
    const url = API_URL;
  import { onMount } from 'svelte';

  let user = null;
  let error = null;
  let success = null;
  let isEditing = false;

  let token;

  onMount(async () => {
    token = localStorage.getItem('token');
    if (!token) {
      error = 'You are not logged in.';
      window.location.href = '/login';
      return;
    }

    try {
      const response = await fetch(url + 'users/me', {
        method: 'GET',
        headers: {
          'Authorization': `Bearer ${token}`,
        },
      });

      if (!response.ok) throw new Error('Failed to fetch user data');
      user = await response.json();
    } catch (err) {
      error = err.message;
    }
  });

  async function updateProfile() {
    success = null;
    error = null;
    try {
      const response = await fetch(url + 'users/update', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${token}`,
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
</script>

{#if error}
  <p style="color: red;">{error}</p>
{:else if user}
  <h1>{user.firstName} {user.lastName}</h1>
  <p><strong>Username:</strong> {user.username}</p>
  <p><strong>Email:</strong> {user.email}</p>

  {#if isEditing}
  <form on:submit|preventDefault={updateProfile}>
    <label>
      Username:
      <input type="text" value={user.username} readonly />
      <small>This field cannot be edited</small>
    </label><br/>

    <label>
      Email:
      <input type="email" value={user.email} readonly />
      <small>This field cannot be edited</small>
    </label><br/>

    <label>
      Phone:
      <input type="text" value={user.phone} readonly />
      <small>This field cannot be edited</small>
    </label><br/>

    <label>
      Address:
      <input type="text" value={user.address} readonly />
      <small>This field cannot be edited</small>
    </label><br/>

    <label>Bio: <textarea bind:value={user.bio}></textarea></label><br/>
    <label>City: <input type="text" bind:value={user.city} /></label><br/>
    <label>Zip: <input type="text" bind:value={user.zip} /></label><br/>
    <label>Education: <input type="text" bind:value={user.education} /></label><br/>
    <label>School Name: <input type="text" bind:value={user.schoolName} /></label><br/>
    <label>College Name: <input type="text" bind:value={user.collegeName} /></label><br/>
    <label>Skills: <input type="text" bind:value={user.skills} /></label><br/>
    <label>Interests: <input type="text" bind:value={user.interests} /></label><br/>
    <label>Branch: <input type="text" bind:value={user.branch} /></label><br/>
    <label>Job Title: <input type="text" bind:value={user.jobTitle} /></label><br/>

    <button type="submit">Save</button>
    <button type="button" on:click={() => isEditing = false}>Cancel</button>
  </form>

  
  {:else}
    <p><strong>Phone:</strong> {user.phone}</p>
    <p><strong>Bio:</strong> {user.bio}</p>
    <p><strong>Address:</strong> {user.address}</p>
    <p><strong>City:</strong> {user.city}</p>
    <p><strong>Zip:</strong> {user.zip}</p>
    <p><strong>Education:</strong> {user.education}</p>
    <p><strong>School Name:</strong> {user.schoolName}</p>
    <p><strong>College Name:</strong> {user.collegeName}</p>
    <p><strong>Skills:</strong> {user.skills}</p>
    <p><strong>Interests:</strong> {user.interests}</p>
    <p><strong>Branch:</strong> {user.branch}</p>
    <p><strong>Job Title:</strong> {user.jobTitle}</p>

    <button on:click={() => isEditing = true}>Edit Profile</button>
  {/if}

  {#if success}
    <p style="color: green;">{success}</p>
  {/if}
{:else}
  <p>Loading...</p>
{/if}

<style>
  label {
    display: block;
    margin: 0.5rem 0;
  }

  input, textarea {
    width: 100%;
    max-width: 500px;
    padding: 0.5rem;
    margin-top: 0.2rem;
  }

  button {
    margin-right: 1rem;
    padding: 0.6rem 1.2rem;
    font-size: 1rem;
    background-color: #0077ff;
    color: white;
    border: none;
    border-radius: 0.4rem;
    cursor: pointer;
  }

  button:hover {
    background-color: #005fd1;
  }
</style>
