<script>
	export let show = false;
	export let onClose;
	export let onSubmit;

	let title = '';
	let content = '';

	function handleSubmit() {
		console.log('Submitting new post:', title, content);
		if (title.trim() && content.trim()) {
			onSubmit(title, content);
			title = '';
			content = '';
			onClose();
		}
	}

	function handleKeyClose(event) {
		if (event.key === 'Escape') {
			onClose();
		}
	}
</script>

{#if show}
	<div
		class="modal-overlay"
		role="button"
		tabindex="0"
		aria-label="Close modal"
		on:click={() => onClose()}
		on:keydown={handleKeyClose}
	>
		<div class="modal-content" on:click|stopPropagation>
			<h2>Create a New Post</h2>
			<input type="text" bind:value={title} placeholder="Post title" />
			<textarea bind:value={content} placeholder="What's on your mind?"></textarea>
			<div class="modal-actions">
				<button on:click={() => onClose()}>Cancel</button>
				<button class="primary" on:click={handleSubmit}>Post</button>
			</div>
		</div>
	</div>
{/if}

<style>
	.modal-overlay {
		position: fixed;
		inset: 0;
		background: rgba(0, 0, 0, 0.5);
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 100;
		outline: none;
	}

	.modal-content {
		background: #fff;
		border-radius: 1rem;
		padding: 2rem;
		width: 400px;
		max-width: 90%;
		box-shadow: 0 8px 30px rgba(0, 0, 0, 0.2);
		animation: fadeIn 0.3s ease;
	}

	.modal-content h2 {
		margin-bottom: 1.2rem;
		font-size: 1.5rem;
		color: #333;
	}

	input[type='text'],
	textarea {
		width: 100%;
		padding: 0.85rem;
		margin-bottom: 1.1rem;
		border: 1px solid #ddd;
		border-radius: 0.6rem;
		font-size: 1rem;
		transition:
			border 0.2s,
			box-shadow 0.2s;
	}

	input[type='text']:focus,
	textarea:focus {
		border-color: #007aff;
		box-shadow: 0 0 0 3px rgba(0, 122, 255, 0.2);
		outline: none;
	}

	textarea {
		resize: vertical;
		min-height: 100px;
	}

	.modal-actions {
		display: flex;
		justify-content: flex-end;
		gap: 1rem;
	}

	button {
		padding: 0.6rem 1.3rem;
		border: none;
		border-radius: 0.6rem;
		cursor: pointer;
		font-size: 1rem;
		background: #f0f0f0;
		transition: background 0.2s;
	}

	button:hover {
		background: #e0e0e0;
	}

	.primary {
		background: #007aff;
		color: white;
	}

	.primary:hover {
		background: #0063cc;
	}

	@keyframes fadeIn {
		from {
			opacity: 0;
			transform: translateY(-20px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
</style>
