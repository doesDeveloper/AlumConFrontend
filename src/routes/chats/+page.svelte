<script>
	import '$lib/styles/chats.css';
	import Sidebar from '$lib/components/Sidebar.svelte';
	import { onMount } from 'svelte';
	import { API_URL } from '$lib/config';
	import { base } from '$app/paths';

	let roomName;
	let username;
	let messageInput = '';
	let roomId;
	let stompClient = null;
	let chatBox;
	let currentPage = 0;
	const pageSize = 10;
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
	function joinRoom(room) {
		roomId = room.roomId;
		roomName = room.roomName;
		hideContacts();

		if (closeChat()) {
			localStorage.setItem('roomId', roomId);
			localStorage.setItem('roomName', roomName);
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
	let reciever = '';
	onMount(() => {
		rooms = JSON.parse(localStorage.getItem('rooms') || '[]');
        
		reciever = localStorage.getItem('reciever');
		if (!reciever) localStorage.setItem('reciever', 'abbas');
		reciever = localStorage.getItem('reciever');
		username = JSON.parse(localStorage.getItem('user')).username;

		// Get Room name.
		roomName = localStorage.getItem('roomName');

		// messageInput = 'Heyyyy';
		roomId = localStorage.getItem('roomId')
			? localStorage.getItem('roomId')
			: generateRoomId('abbas', username);

		connect();
		if (!rooms.find((room) => room.roomId == roomId)) {
			rooms = [...rooms, { roomId: roomId, roomName: roomName }];
			localStorage.setItem('rooms', JSON.stringify(rooms));
		}
	});
</script>

<div class="layout">
	<Sidebar />
	<div class="chat-main-content">
		<div class="chat chat-box">
			<div class="contact bar">
				<i class="contactstoggle fas fa-bars fa-2x"></i>
				<div class="name" id="roomName">{roomName}</div>
				<div class="description" id="roomId">Username: @{reciever}</div>
				<div class="ppic stark"></div>
			</div>
			<div class="messages" id="chat-box" bind:this={chatBox}>
				<div class="time">Today at 11:41</div>
				<div class="message messagesend">
					<div class="sender">Peter Parker</div>
					<div class="content">Hey, What's going on?</div>
				</div>
				<div class="message messagesend">
					<div class="sender">Peter Parker</div>
					<div class="content">Hey, What's going on?</div>
				</div>
				<div class="message messagesend">
					<div class="sender">Peter Parker</div>
					<div class="content">Hey, What's going on?</div>
				</div>
				<div class="message messagesend">
					<div class="sender">Peter Parker</div>
					<div class="content">Hey, What's going on?</div>
				</div>
				<div class="message messagesend">
					<div class="sender">Peter Parker</div>
					<div class="content">Hey, What's going on?</div>
				</div>
				<div class="message messagesend">
					<div class="sender">Peter Parker</div>
					<div class="content">Hey, What's going on?</div>
				</div>
				<div class="message messagerec">
					<div class="sender">Tony Stark</div>
					<div class="content">Nothing much why even bother messaging?</div>
				</div>
				<div class="message messagesend">
					<div class="sender">Peter Parker</div>
					<div class="content">
						Just wanted to say that the person reading this chat has nothing else to do!
					</div>
				</div>
				<div class="message messagesend">
					<div class="sender">Peter Parker</div>
					<div class="content">Uh, what is this guy's problem</div>
				</div>
				<div class="message messagerec">
					<div class="sender">Steve Rogers</div>
					<div class="content">
						I guess he is just not in the right place he should join his room.
					</div>
				</div>
			</div>
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
		<h2>Rooms</h2>
		<div class="contact" id="addRoom">
			<div class="pic stark"></div>
			<div class="name">Add Room</div>
			<div class="message">Room Id: xxxx</div>
		</div>
	</div>
</div>
