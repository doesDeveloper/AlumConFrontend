<script>
	import { API_URL } from '$lib/config';
	import Field from '$lib/components/Field.svelte';

	const url = API_URL;

	let formData = {
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
		skills: '',
		interests: '',
		branch: '',
		jobTitle: '',
		bio: ''
	};

	let errors = {};
	let errorMessage = '';
	let isSubmitting = false;

	function validateForm() {
		errors = {};

		if (!formData.username.trim()) errors.username = 'Username is required';
		if (!formData.password.trim() || formData.password.length < 6)
			errors.password = 'Password must be at least 6 characters';
		if (!formData.firstName.trim()) errors.firstName = 'First name is required';
		if (!formData.email.trim() || !formData.email.includes('@'))
			errors.email = 'Valid email is required';
		if (!formData.phone.trim() || formData.phone.length < 10)
			errors.phone = 'Valid phone number is required';

		return Object.keys(errors).length === 0;
	}

	async function handleRegister() {
		if (!validateForm()) return;

		isSubmitting = true;
		errorMessage = '';

		try {
			const response = await fetch(url + 'auth/register', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify(formData)
			});

			const data = await response.json();

			if (!response.ok) {
				console.error(response);
				throw new Error(data.message || 'Failed to register');
			}

			alert('Registration successful! Please login.');
			window.location.href = '/login';
		} catch (error) {
			errorMessage = error.message || 'Something went wrong. Please try again.';
		} finally {
			isSubmitting = false;
		}
	}
</script>

<h1>Register</h1>

{#if errorMessage}
	<p class="error">{errorMessage}</p>
{/if}

<form on:submit|preventDefault={handleRegister}>
	<fieldset>
		<legend>Account Info</legend>
		<Field label="Username*" id="username" bind:value={formData.username} error={errors.username} />
		<Field
			label="Password*"
			id="password"
			type="password"
			bind:value={formData.password}
			error={errors.password}
		/>
	</fieldset>

	<fieldset>
		<legend>Personal Details</legend>
		<Field
			label="First Name*"
			id="firstName"
			bind:value={formData.firstName}
			error={errors.firstName}
		/>
		<Field label="Last Name" id="lastName" bind:value={formData.lastName} />
		<Field
			label="Email*"
			id="email"
			type="email"
			bind:value={formData.email}
			error={errors.email}
		/>
		<Field label="Phone*" id="phone" type="tel" bind:value={formData.phone} error={errors.phone} />
		<Field label="Address" id="address" bind:value={formData.address} />
		<Field label="City" id="city" bind:value={formData.city} />
		<Field label="Zip" id="zip" bind:value={formData.zip} />
	</fieldset>

	<fieldset>
		<legend>Education & Career</legend>
		<Field label="Education" id="education" bind:value={formData.education} />
		<Field label="School Name" id="schoolName" bind:value={formData.schoolName} />
		<Field label="College Name" id="collegeName" bind:value={formData.collegeName} />
		<Field label="Branch" id="branch" bind:value={formData.branch} />
		<Field label="Job Title" id="jobTitle" bind:value={formData.jobTitle} />
		<Field label="Skills" id="skills" bind:value={formData.skills} />
		<Field label="Interests" id="interests" bind:value={formData.interests} />
	</fieldset>

	<fieldset>
		<legend>Bio & Submit</legend>
		<div class="field">
			<label for="bio">Bio</label>
			<textarea id="bio" bind:value={formData.bio}></textarea>
		</div>
		<button type="submit" disabled={isSubmitting}>
			{isSubmitting ? 'Registering...' : 'Register'}
		</button>
	</fieldset>
</form>

<style>
	form {
		display: flex;
		flex-direction: column;
		max-width: 500px;
		margin-top: 1rem;
	}

	.field {
		margin-bottom: 1rem;
		display: flex;
		flex-direction: column;
	}

	label {
		font-weight: bold;
	}

	input,
	textarea {
		padding: 8px;
		border: 1px solid #ccc;
		border-radius: 4px;
		margin-top: 4px;
	}

	input.error {
		border-color: red;
	}

	.field-error {
		color: red;
		font-size: 0.875rem;
		margin-top: 2px;
	}

	.error {
		color: red;
		margin-bottom: 1rem;
		font-weight: bold;
	}

	button {
		margin-top: 10px;
		padding: 10px;
		background-color: #0077ff;
		color: white;
		border: none;
		border-radius: 4px;
		cursor: pointer;
	}

	button:disabled {
		opacity: 0.6;
		cursor: not-allowed;
	}
</style>
