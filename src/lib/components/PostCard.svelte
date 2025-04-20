<script>
    import { API_URL } from '$lib/config';
    import { base } from '$app/paths';
    import UserAvatar from '$lib/components/UserAvatar.svelte';
    // Component props
    export let post;
    export let token;

    // Function to format date from timestamp
    function formatDate(timestamp) {
        const timestampNum = Number(timestamp);
        const date = new Date(timestampNum);
        const options = { year: 'numeric', month: 'long', day: 'numeric' };
        return date.toLocaleDateString('en-US', options);
    }

    // Vote function with toggle behavior
    const votePost = async (postId, voteValue) => {
        try {
            // If already voted with same value, toggle it off (set to 0)
            const newVoteValue = post.ownVote === voteValue ? 0 : voteValue;
            
            const res = await fetch(`${API_URL}posts/indiv/${postId}/vote?voteValue=${newVoteValue}`, {
                method: 'POST',
                headers: {
                    Authorization: `Bearer ${token}`
                }
            });

            if (!res.ok) throw new Error('Failed to vote on post');
            const data = await res.text();
            
            // Update post data locally and dispatch event
            post = { 
                ...post, 
                voteCount: data,
                ownVote: newVoteValue
            };
            
            // Dispatch an event to notify parent component
            dispatch('postUpdated', { postId, voteCount: data, ownVote: newVoteValue });
            
            console.log(`Voted post: ${postId}, value: ${newVoteValue}`);
        } catch (err) {
            console.error(err);
            dispatch('error', err.message);
        }
    };

    // Event dispatcher
    import { createEventDispatcher } from 'svelte';
    const dispatch = createEventDispatcher();
</script>

<div class="post-card">
    <div class="post-header">
        <!-- <div class="logo-icon user-avatar">{post.username.charAt(0).toUpperCase()}</div> -->
        <UserAvatar username={post.username} size="40px" />
        <div class="user-info">
            <div class="user-name">
                <a href="{base}/profile/others?name={post.username}">{post.username}</a>
            </div>
            <div class="user-role">{formatDate(post.timeStamp)}</div>
        </div>
        <div class="post-menu">⋮</div>
    </div>

    <div class="post-content">
        <div class="post-title">{post.title}</div>
        <p>{post.content}</p>
    </div>

    <div class="post-actions">
        <span class="post-upvote">
            <button 
                on:click={() => votePost(post.id, 1)} 
                aria-label="Upvote"
                class="vote-btn upvote-btn {post.ownVote === 1 ? 'active' : ''}"
            >
                <i class="fas fa-arrow-up"></i>
            </button>
            <span class="vote-count">{post.voteCount}</span>
            <button 
                on:click={() => votePost(post.id, -1)} 
                aria-label="Downvote"
                class="vote-btn downvote-btn {post.ownVote === -1 ? 'active' : ''}"
            >
                <i class="fas fa-arrow-down"></i>
            </button>
        </span>
        <div>💬 Comments</div>
        <div>🔁 Share</div>
    </div>
</div>

<style>
    .post-card {
        background-color: white;
        border-radius: 12px;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
        margin-bottom: 20px;
        padding: 20px;
    }

    .post-header {
        display: flex;
        align-items: center;
        margin-bottom: 15px;
    }

    .user-avatar {
        width: 40px;
        height: 40px;
        border-radius: 50%;
        background-color: #3a86ff;
        color: white;
        display: flex;
        align-items: center;
        justify-content: center;
        font-weight: bold;
        margin-right: 10px;
    }

    .user-info {
        flex-grow: 1;
    }

    .user-name {
        font-weight: 600;
        font-size: 16px;
    }

    .user-name a {
        color: #333;
        text-decoration: none;
    }

    .user-name a:hover {
        text-decoration: underline;
    }

    .user-role {
        color: #666;
        font-size: 14px;
    }

    .post-menu {
        cursor: pointer;
        font-size: 18px;
        font-weight: bold;
        color: #666;
    }

    .post-content {
        margin-bottom: 15px;
    }

    .post-title {
        font-size: 18px;
        font-weight: 600;
        margin-bottom: 8px;
    }

    .post-actions {
        display: flex;
        gap: 20px;
        color: #555;
        font-size: 14px;
    }

    .post-upvote {
        display: flex;
        align-items: center;
        gap: 5px;
    }

    /* Modern voting buttons */
    .vote-btn {
        background: none;
        border: 1px solid #e0e0e0;
        cursor: pointer;
        padding: 5px 10px;
        border-radius: 6px;
        transition: all 0.2s ease;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .vote-btn:hover {
        background-color: #f5f5f5;
    }

    /* Active state styling - more visible */
    .upvote-btn.active {
        background-color: rgba(46, 125, 50, 0.15);
        border-color: #2e7d32;
        color: #2e7d32;
        box-shadow: 0 0 0 1px rgba(46, 125, 50, 0.2);
    }

    .downvote-btn.active {
        background-color: rgba(198, 40, 40, 0.15);
        border-color: #c62828;
        color: #c62828;
        box-shadow: 0 0 0 1px rgba(198, 40, 40, 0.2);
    }

    .vote-count {
        font-weight: 600;
        min-width: 24px;
        text-align: center;
    }

    /* Make sure the post actions items look clickable */
    .post-actions div {
        cursor: pointer;
        padding: 5px 10px;
        border-radius: 6px;
        transition: background-color 0.2s;
    }

    .post-actions div:hover {
        background-color: #f5f5f5;
    }
</style>