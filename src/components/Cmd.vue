<template>
	<div class="container" @click="focusTextArea">
		<div class="history">
			<div v-for="(entry, index) in history" :key="index" class="history-line">
				<span class="prompt">{{ entry.prompt }}</span>
				<span class="command-text">{{ entry.command }}</span>
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
				/>
				<span class="cursor" :style="{ left: cursorOffset + 'px' }"></span>
			</div>
		</div>
	</div>
</template>

<script setup>
import { ref, nextTick, onMounted, computed, watch } from 'vue';

const base = ref('root@ata.parvin');
const path = ref('~');
const value = ref('');
const history = ref([]);
const inputField = ref(null);
const cursorOffset = ref(0);

const handleCommand = () => {
	const currentCommand = value.value; // Keep original for history
	let output = '';

	const cmd = currentCommand.trim().toLowerCase();

	if (cmd === 'help') {
		output = 'Available: bio, projects, clear';
	} else if (cmd === 'clear') {
		history.value = [];
		value.value = '';
		return;
	} else if (cmd !== '') {
		output = `bash: ${cmd}: command not found`;
	}

	history.value.push({
		prompt: `${base.value}:${path.value}#`,
		command: currentCommand,
		output: output,
	});

	value.value = '';
	scrollToBottom();
};

// Calculate the width of the text to move the cursor
const updateCursor = () => {
	nextTick(() => {
		const mirror = document.querySelector('.text-mirror');
		if (mirror) {
			cursorOffset.value = mirror.offsetWidth;
		}
	});
};

// Watch the input value to move the cursor in real-time
watch(value, () => {
	updateCursor();
});

onMounted(() => {
	focusTextArea();
});

const focusTextArea = () => inputField.value.focus();
const scrollToBottom = () =>
	nextTick(() => window.scrollTo(0, document.body.scrollHeight));
</script>

<style lang="scss" scoped>
.container {
	min-height: 100vh;
	background-color: black;
	color: white;
	padding: 20px;
	font-family: 'Courier New', monospace;
	display: flex;
	flex-direction: column;
	white-space: pre; // Crucial to maintain spaces width
	font-family: 'Courier New', monospace;
	font-size: 18px;
	font-weight: 500;

	.history-line {
		.command-text {
			margin-left: 2px;
		}
		.output-text {
			color: #aaa;
			white-space: pre-wrap;
			margin-top: 2px;
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

		// The Mirror must have the exact same font properties as the textarea
		.text-mirror {
			visibility: hidden;
			position: absolute;
			white-space: pre; // Crucial to maintain spaces width
			font-family: 'Courier New', monospace;
			font-size: 18px;
			font-weight: 500;
		}

		textarea {
			background: transparent;
			color: white;
			border: none;
			outline: none;
			resize: none;
			flex: 1;
			font-family: 'Courier New', monospace;
			font-size: 18px;
			font-weight: 500;
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
