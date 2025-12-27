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
					@keydown="handleKeydown"
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
const tabIndex = ref(-1);
const inputField = ref(null);
const cursorOffset = ref(0);

// Load history from localStorage on mount
onMounted(() => {
	const saved = localStorage.getItem('cmdHistory');
	if (saved) {
		history.value = JSON.parse(saved);
	}
});

// Save history to localStorage whenever it changes
watch(
	history,
	(newHistory) => {
		localStorage.setItem('cmdHistory', JSON.stringify(newHistory));
	},
	{ deep: true }
);

// Directory Data
const directories = {
	about: {
		content:
			'Name: Ata Parvin\nRole: Full Stack Developer\nStatus: Looking for root access...',
		subdirs: {},
		files: {},
	},
	experience: {
		content: '',
		subdirs: {
			'Critical Mass': {
				content:
					'Current company: Critical Mass\nRole: Full Stack Developer\nJoined: 2023',
				subdirs: {
					project1: {
						content:
							'E-commerce Platform\nTech: Vue.js, Node.js\nDescription: Built a scalable e-commerce site.',
						subdirs: {},
						files: {},
					},
					project2: {
						content:
							'Dashboard App\nTech: React, Express\nDescription: Real-time analytics dashboard.',
						subdirs: {},
						files: {},
					},
				},
				files: {
					'responsibilities.txt':
						'- Developed full-stack web applications\n- Integrated APIs and databases\n- Collaborated with design and product teams\n- Optimized performance and user experience',
				},
			},
			'Previous Company': {
				content:
					'Previous company: XYZ Corp\nRole: Junior Developer\nDuration: 2020-2023',
				subdirs: {},
				files: {
					'achievements.txt':
						'- Built initial prototypes\n- Learned modern web technologies',
				},
			},
		},
		files: {},
	},
	contact: {
		content:
			'Email: hello@ata.parvin\nGitHub: github.com/atapavin\nLinkedIn: linkedin.com/in/atapavin',
		subdirs: {},
		files: {},
	},
};

// Helper function to get current directory object
const getCurrentDir = () => {
	let current = { subdirs: directories, files: {} };
	if (path.value === '~') return current;
	const parts = path.value.replace('~/', '').split('/');
	for (const part of parts) {
		if (current.subdirs && current.subdirs[part]) {
			current = current.subdirs[part];
		} else {
			return null;
		}
	}
	return current;
};

const handleCommand = () => {
	const currentCommand = value.value;
	const trimmed = currentCommand.trim();
	const args = trimmed.split(' ');
	const cmd = args[0].toLowerCase();
	let target = '';

	// Parse target based on command
	if (cmd === 'cd' && trimmed.length > 3) {
		target = trimmed.slice(3).trim();
	} else if (cmd === 'cat' && trimmed.length > 4) {
		target = trimmed.slice(4).trim();
	} else {
		target = args[1] || '';
	}

	let output = '';

	// Capture prompt before changing path
	const currentPrompt = `${base.value}:${path.value}#`;

	// COMMAND LOGIC
	if (cmd === 'ls') {
		const currentDir = getCurrentDir();
		if (currentDir) {
			const subdirNames = Object.keys(currentDir.subdirs);
			const fileNames = Object.keys(currentDir.files);
			output = [...subdirNames, ...fileNames].join('    ');
		} else {
			output = 'ls: No such directory';
		}
	} else if (cmd === 'cd') {
		if (!target || target === '~' || target === '/') {
			path.value = '~';
		} else if (target === '..') {
			if (path.value === '~') {
				// stay
			} else {
				const parts = path.value.split('/');
				parts.pop();
				path.value = parts.length > 1 ? parts.join('/') : '~';
			}
		} else {
			const currentDir = getCurrentDir();
			if (currentDir && currentDir.subdirs[target]) {
				path.value =
					path.value === '~' ? `~/${target}` : `${path.value}/${target}`;
			} else {
				output = `bash: cd: ${target}: No such directory`;
			}
		}
	} else if (cmd === 'cat') {
		const currentDir = getCurrentDir();
		if (!target) {
			if (currentDir && currentDir.content) {
				output = currentDir.content;
			} else {
				output = 'cat: No content in current directory.';
			}
		} else {
			if (currentDir && currentDir.files[target]) {
				output = currentDir.files[target];
			} else {
				output = `cat: ${target}: No such file`;
			}
		}
	} else if (cmd === 'pwd') {
		output = path.value;
	} else if (cmd === 'echo') {
		output = args.slice(1).join(' ');
	} else if (cmd === 'help') {
		output =
			'Commands: ls, cd [dir], cat [file], pwd, echo [text], clear, help';
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

// Tab completion
const handleKeydown = (e) => {
	if (e.key === 'Tab') {
		e.preventDefault();
		handleTab();
	} else if (e.key !== 'Enter') {
		tabIndex.value = -1;
	}
};

const handleTab = () => {
	const trimmed = value.value.trim();
	if (trimmed === 'cd ') {
		const currentDir = getCurrentDir();
		if (currentDir) {
			const options = Object.keys(currentDir.subdirs);
			if (options.length > 0) {
				value.value = 'cd ' + options[0];
				tabIndex.value = 0;
			}
		}
	} else {
		const spaceIndex = trimmed.indexOf(' ');
		if (spaceIndex > 0) {
			const cmd = trimmed.slice(0, spaceIndex);
			const prefix = trimmed.slice(spaceIndex + 1).trim();
			const currentDir = getCurrentDir();
			if (cmd === 'cd' && currentDir) {
				const options = Object.keys(currentDir.subdirs);
				const matches = options.filter((opt) => opt.startsWith(prefix));
				if (matches.length > 0) {
					if (tabIndex.value === -1) {
						tabIndex.value = 0;
					} else {
						tabIndex.value = (tabIndex.value + 1) % matches.length;
					}
					value.value = 'cd ' + matches[tabIndex.value];
				}
			} else if (cmd === 'cat' && currentDir) {
				const options = Object.keys(currentDir.files);
				const matches = options.filter((opt) => opt.startsWith(prefix));
				if (matches.length > 0) {
					if (tabIndex.value === -1) {
						tabIndex.value = 0;
					} else {
						tabIndex.value = (tabIndex.value + 1) % matches.length;
					}
					value.value = 'cat ' + matches[tabIndex.value];
				}
			}
		} else {
			// Complete command names
			const commands = ['ls', 'cd', 'cat', 'pwd', 'echo', 'clear', 'help'];
			const matches = commands.filter((c) => c.startsWith(trimmed));
			if (matches.length > 0) {
				if (tabIndex.value === -1) {
					tabIndex.value = 0;
				} else {
					tabIndex.value = (tabIndex.value + 1) % matches.length;
				}
				value.value = matches[tabIndex.value];
			}
		}
	}
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
