<template>
	<div class="container" @click="focusTextArea">
		<div class="history">
			<div v-for="(entry, index) in history" :key="index" class="history-line">
				<div class="command-row">
					<span class="prompt">{{ entry.prompt }}</span>
					<span class="command-text">{{ entry.command }}</span>
				</div>
				<div v-if="entry.output" class="output-text">{{ entry.output }}</div>
			</div>
		</div>

		<div class="input-line">
			<span class="prompt">{{ base }}:{{ path }}#</span>
			<div class="input-wrapper">
				<span class="text-mirror">{{ value }}</span>

				<textarea
					ref="inputField"
					v-model="value"
					@keydown.enter.prevent="handleCommand"
					rows="1"
					spellcheck="false"
					autocomplete="off"
				/>
				<span class="cursor" :style="{ left: cursorOffset + 'px' }"></span>
			</div>
		</div>
	</div>
</template>

<script setup>
import { ref, nextTick, onMounted, watch } from 'vue';

const base = ref('root@ata.parvin');
const path = ref('~');
const value = ref('');
const history = ref([]);
const inputField = ref(null);
const cursorOffset = ref(0);

// Directory Data
const directories = [
	{
		name: 'about',
		content:
			'Name: Ata Parvin\nRole: Full Stack Developer\nStatus: Looking for root access...',
	},
	{
		name: 'experience',
		content:
			'1. Frontend Dev - Vue.js\n2. Backend Architect - Node.js\n3. Linux Enthusiast',
	},
	{
		name: 'contact',
		content: 'Email: hello@ata.parvin\nGitHub: github.com/atapavin',
	},
];

const handleCommand = () => {
	const currentCommand = value.value;
	const trimmed = currentCommand.trim();
	const args = trimmed.split(' ');
	const cmd = args[0].toLowerCase();
	const target = args[1];

	let output = '';

	// Capture prompt before changing path
	const currentPrompt = `${base.value}:${path.value}#`;

	// COMMAND LOGIC
	if (cmd === 'ls') {
		if (path.value === '~') {
			output = directories.map((d) => d.name).join('    ');
		} else {
			const currentDirName = path.value.replace('~/', '');
			const dir = directories.find((d) => d.name === currentDirName);
			if (dir) {
				output = dir.content;
			} else {
				output = '';
			}
		}
	} else if (cmd === 'cd') {
		if (!target || target === '~' || target === '/') {
			path.value = '~';
		} else if (target === '..') {
			path.value = '~';
		} else {
			const exists = directories.find((d) => d.name === target);
			if (exists) {
				path.value = `~/${target}`;
			} else {
				output = `bash: cd: ${target}: No such directory`;
			}
		}
	} else if (cmd === 'cat') {
		const currentDirName = path.value.replace('~/', '');
		const dir = directories.find((d) => d.name === currentDirName);
		if (dir) {
			output = dir.content;
		} else {
			output = 'cat: No content in current directory.';
		}
	} else if (cmd === 'help') {
		output = 'Commands: ls, cd [dir], cat, clear, help';
	} else if (cmd === 'clear') {
		history.value = [];
		value.value = '';
		return;
	} else if (trimmed !== '') {
		output = `bash: ${cmd}: command not found`;
	}

	// Record History
	history.value.push({
		prompt: currentPrompt,
		command: currentCommand,
		output: output,
	});

	value.value = '';
	scrollToBottom();
};

// Layout & Cursor Logic
const updateCursor = () => {
	nextTick(() => {
		const mirror = document.querySelector('.text-mirror');
		if (mirror) {
			cursorOffset.value = mirror.offsetWidth;
		}
	});
};

watch(value, () => updateCursor());

onMounted(() => {
	focusTextArea();
});

const focusTextArea = () => inputField.value?.focus();

const scrollToBottom = () =>
	nextTick(() =>
		window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' })
	);
</script>

<style lang="scss" scoped>
.container {
	min-height: 100vh;
	background-color: black;
	color: white;
	padding: 20px;
	font-family: 'Courier New', monospace;
	font-size: 18px;
	font-weight: 500;
	display: flex;
	flex-direction: column;
	cursor: text;

	.history-line {
		.command-row {
			display: flex;
			align-items: center;
		}

		.command-text {
			margin-left: 2px;
			white-space: pre;
		}

		.output-text {
			color: #aaa;
			white-space: pre-wrap;
			line-height: 1.4;
		}
	}

	.input-line {
		display: flex;
		align-items: center;
	}

	.prompt {
		color: #ff5555;
		font-weight: bold;
		margin-right: 10px;
		white-space: nowrap;
	}

	.input-wrapper {
		position: relative;
		display: flex;
		flex: 1;
		align-items: center;

		.text-mirror {
			visibility: hidden;
			position: absolute;
			white-space: pre;
			font-family: inherit;
			font-size: inherit;
			font-weight: inherit;
		}

		textarea {
			background: transparent;
			color: white;
			border: none;
			outline: none;
			resize: none;
			flex: 1;
			font-family: inherit;
			font-size: inherit;
			font-weight: inherit;
			padding: 0;
			margin: 0;
			caret-color: transparent;
			z-index: 10;
			overflow: hidden;
			white-space: pre;
		}

		.cursor {
			position: absolute;
			display: inline-block;
			width: 10px;
			height: 1.2rem;
			background-color: white;
			animation: blink 1s step-end infinite;
			pointer-events: none;
			z-index: 5;
		}
	}

	@keyframes blink {
		0%,
		100% {
			opacity: 1;
		}
		50% {
			opacity: 0;
		}
	}
}
</style>
