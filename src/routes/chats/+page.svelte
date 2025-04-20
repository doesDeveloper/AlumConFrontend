<script>
	import '$lib/styles/chats.css';
	import Sidebar from '$lib/components/Sidebar.svelte';
	import { onMount } from 'svelte';
	import { API_URL } from '$lib/config';
	import { base } from '$app/paths';
	import { get } from 'svelte/store';

	let username;
	let token;
	let roomName;
	let roomId;
	let reciever = '';
	let messageInput = '';
	let stompClient = null;
	let chatBox;
	let currentPage = 0;
	const pageSize = 30;
	let isLoadingChat = false;
	let contactsVisible = false;

	let rooms = [];
	let picsClass = ['thor', 'stark', 'rogers', 'banner', 'danvers'];

	function connect() {
		var socket = new SockJS(API_URL + '../ws');
		stompClient = Stomp.over(socket);
		stompClient.connect({}, function (frame) {
			stompClient.subscribe('/topic/chat/' + roomId, function (message) {
				showMessage(JSON.parse(message.body));
			});
		});
		hookChatUpdate();
		loadHistory();
	}
	function loadHistory() {
		if (isLoadingChat) return;
		isLoadingChat = true;

		const previousScrollHeight = chatBox.scrollHeight;
		const pageToFetch = currentPage;

		fetch(`${API_URL}../chat/history/${roomId}?page=${pageToFetch}&size=${pageSize}`)
			.then((res) => res.json())
			.then((messages) => {
				if (messages.length === 0) {
					return; // no need to decrement page; just stop loading
				}

				messages.forEach((msg) => {
					showHistory(msg); // should prepend to chatBox
				});

				const newScrollHeight = chatBox.scrollHeight;
				chatBox.scrollTop = newScrollHeight - previousScrollHeight;

				currentPage++; // increment only if successful and not empty
			})
			.catch((err) => {
				console.error('Failed to load history', err);
			})
			.finally(() => {
				isLoadingChat = false;
			});
	}
	function sendMessage() {
		createRoomIfNotExist(reciever, username);
		if (messageInput) {
			var message = {
				sender: username,
				timestamp: Date.now(),
				content: messageInput
			};
			stompClient.send('/app/chat/' + roomId, {}, JSON.stringify(message));
			messageInput = '';
		}
	}
	function showMessage(message) {
		var messageElement = createMessageElement(message);
		chatBox.appendChild(messageElement);
		chatBox.scrollTop = chatBox.scrollHeight;
	}
	function showHistory(message) {
		var messageElement = createMessageElement(message);
		chatBox.prepend(messageElement);
	}
	function createMessageElement(message) {
		var messageElement = document.createElement('div');
		messageElement.classList.add('message');
		messageElement.classList.add(message.sender == username ? 'messagesend' : 'messagerec');

		var contentElement = document.createElement('div');
		contentElement.classList.add('content');
		contentElement.textContent = message.content;
		messageElement.appendChild(contentElement);

		return messageElement;
	}
	function hookChatUpdate() {
		if (!chatBox) return;

		chatBox.innerHTML = '';
		chatBox.addEventListener('scroll', () => {
			if (chatBox.scrollTop === 0 && !isLoadingChat) {
				loadHistory();
			}
		});
		chatBox.scrollTop = chatBox.scrollHeight;
	}
	function submitConnect(roomIdInput, usernameInput, roomNameInput) {
		roomId = roomIdInput;
		username = usernameInput;
		roomName = roomNameInput;

		if (!rooms.find((room) => room.roomId == roomId)) {
			rooms = [...rooms, { roomId: roomId, roomName: roomName }];
			localStorage.setItem('rooms', JSON.stringify(rooms));
		}

		localStorage.setItem('roomId', roomId);
		localStorage.setItem('roomName', roomName);

		connect();
	}
	function hideContacts() {
		contactsVisible = false;
		document.getElementById('contacts').classList.remove('visible');
	}
	function showContacts() {
		contactsVisible = true;
		document.getElementById('contacts').classList.add('visible');
	}
	function joinRoom(room) {
		roomId = room.roomId;
		roomName = room.roomName;
		reciever = room.reciever;
		console.log(roomId, roomName, reciever);
		hideContacts();

		if (closeChat()) {
			localStorage.setItem('roomId', roomId);
			localStorage.setItem('roomName', roomName);
			localStorage.setItem('reciever', reciever);
			connect();
		}
	}

	function closeChat() {
		if (stompClient != null) {
			stompClient.disconnect();
			if (chatBox) chatBox.innerHTML = '';
			currentPage = 0;
			return true;
		} else {
			return false;
		}
	}
	function getRandomAvatar() {
		return picsClass[Math.floor(Math.random() * picsClass.length)];
	}
	function handleKeyDown(event) {
		if (event.key === 'Enter') {
			event.preventDefault();
			sendMessage();
		}
	}
	function generateRoomId(reciever, username) {
		const sortedNames = [username, reciever].sort().join('=');
		return sortedNames;
	}
	async function createRoomIfNotExist(reciever, username) {
		if (!rooms.find((room) => room.roomId == roomId && room.roomName == roomName)) {
			const recievers = await fetch(`${API_URL}users/me/chats`, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/text',
					Authorization: `Bearer ${token}`
				},
				body: reciever
			}).then(async (res) => {
				const data = await res.json().then((data) => {
					console.log(data);
					console.log(
						JSON.stringify(data.sort()) !==
							JSON.stringify(rooms.map((room) => room.reciever).sort())
					);
				});
			});
			const existingRoom = rooms.find((room) => room.roomId == roomId);
			if (existingRoom) {
				existingRoom.roomName = roomName;
			} else {
				rooms = [...rooms, { roomId, roomName, reciever }];
			}
			localStorage.setItem('rooms', JSON.stringify(rooms));
		}
	}
	async function getRoomName(reciever, token) {
		return fetch(`${API_URL}users/profile/${reciever}`, {
			method: 'GET',
			headers: {
				Authorization: `Bearer ${token}`
			}
		})
			.then((res) => res.json())
			.then((data) => {
				return data.firstName + ' ' + data.lastName;
			});
	}
	onMount(async () => {
		token = localStorage.getItem('token');
		if (!token) {
			window.location.href = base + '/login';
			return;
		}
		username = JSON.parse(localStorage.getItem('user')).username;
		rooms = JSON.parse(localStorage.getItem('rooms') || '[]');
		let remoteRecievers = await fetch(`${API_URL}users/me/chats`, {
			method: 'GET',
			headers: {
				Authorization: `Bearer ${token}`
			}
		}).then((res) => res.json());

		if (
			JSON.stringify(remoteRecievers.sort()) !==
			JSON.stringify(rooms.map((room) => room.reciever).sort())
		) {
			rooms = [];

			await Promise.all(
				remoteRecievers.map(async (reciever) => {
					if (!rooms.find((room) => room.reciever == reciever)) {
						rooms.push({
							roomId: generateRoomId(reciever, username),
							roomName: await getRoomName(reciever, token),
							reciever
						});
					}
				})
			).then(() => {
				rooms = rooms;
			});
		}

		localStorage.setItem('rooms', JSON.stringify(rooms));

		reciever = localStorage.getItem('reciever');
		if (!reciever) localStorage.setItem('reciever', 'abbas');
		reciever = localStorage.getItem('reciever');

		// Get Room name.
		roomName = await getRoomName(reciever, token);
		roomId = generateRoomId(reciever, username);

		connect();
	});
</script>

<div class="layout">
	<Sidebar />
	<div class="chat-main-content">
		<div class="chat chat-box">
			<div class="contact bar">
				<!-- // left arrow -->
				{#if !contactsVisible}
					<i class="contactstoggle fas fa-arrow-left fa-2x" on:click={showContacts}></i>
				{/if}
				<div class="name" id="roomName">{roomName}</div>
				<div class="description" id="roomId">Username: @{reciever}</div>
				<div class="ppic stark"></div>
			</div>
			<div class="messages" id="chat-box" bind:this={chatBox}></div>
			<div class="input">
				<input
					placeholder="Type your message here!"
					id="message"
					bind:value={messageInput}
					on:keydown={handleKeyDown}
					type="text"
				/>
				<i class="fas fa-paper-plane" on:click={sendMessage}></i>
			</div>
		</div>
	</div>
	<div class="contacts" id="contacts">
		<h2 class="text-2xl font-bold">Chats</h2>
		{#each rooms as room}
			<div class="contact" on:click={joinRoom(room)}>
				<div class="pic stark"></div>
				<div class="name">{room.roomName}</div>
				<div class="message">@{room.reciever}</div>
			</div>
		{/each}
	</div>
</div>
