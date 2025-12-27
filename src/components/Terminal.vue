<template>
	<div class="mac-window-container" data-aos="fade-down-right">
		<div class="taskbar">
			<div class="red-btn"></div>
			<div class="yellow-btn"></div>
			<div class="green-btn"></div>
		</div>
		<div class="content">
			<!-- <div v-if="showTerminalAnimation" class="terminal-output">
				<pre
					class="typing-text"
				><span class="cursor">|</span>{{ displayText }}</pre>
			</div>
			<slot v-else /> -->
			<div class="terminal-output">
				<!-- <span style="margin-right: 5px">{{ base }}</span> -->
				<textarea class="input" />
			</div>
		</div>
	</div>
</template>

<script setup>
import { ref } from 'vue';

const base = ref('ata.parvin@root');
const path = ref('');
</script>

<style lang="scss" scoped>
.mac-window-container {
	border-radius: 16px;
	backdrop-filter: blur(8px);
	background-color: #232325e6;
	padding: 0 0.25rem 0.25rem;
	border: 1px solid rgba(#fff, 0.09);

	.taskbar {
		display: flex;
		align-items: center;
		$btn-size: 9px;
		justify-content: flex-end;
		padding: 0.75rem;
		gap: 8px;

		> * {
			width: $btn-size;
			height: $btn-size;
			border-radius: 50%;
		}

		.red-btn {
			background-color: #ec6a5e;
		}

		.yellow-btn {
			background-color: #f5bf4f;
		}

		.green-btn {
			background-color: #61c554;
		}
	}

	.content {
		text-align: left;
		border-radius: 12px;
		background: linear-gradient(129deg, #0000007a 22.38%, #0000 136.5%);
		// padding: 20px;
		overflow: hidden;
	}
}

.terminal-output {
	background-color: #1e1e1e;
	color: #f0f0f0;
	padding: 20px;
	font-family: 'Courier New', monospace;
	height: 100%;

	.input {
		background-color: red;
		border: none;
		width: 100%;
		height: 100%;
		min-height: 200px;

		&:focus {
			outline: none;
		}
	}
}

.typing-text {
	margin: 0;
	white-space: pre-wrap;
	line-height: 1.5;
}

.cursor {
	display: inline-block;
	width: 8px;
	animation: blink 1s step-end infinite;
}

@keyframes blink {
	from,
	to {
		opacity: 1;
	}
	50% {
		opacity: 0;
	}
}
</style>
