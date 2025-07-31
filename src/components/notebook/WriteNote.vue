<template>
    <div class="notebook-wrapper">
        <!-- <textarea class="title-textarea" placeholder="Tiêu đề..." :value="title" v-on:keydown.tab.prevent="tabIndent" @input="editNote"></textarea>
		<br>
		<hr> -->
        <textarea class="lined-textarea" placeholder="Nội dung..." :value="content" v-on:keydown.tab.prevent="tabIndent" @input="editNote"></textarea>
    </div>
</template>

<script setup>
	const props = defineProps({
		content: {
			Type: String
		}
	})

	function tabIndent(e) {
		const textarea = e.target
		const start = textarea.selectionStart
		const end = textarea.selectionEnd
		const value = textarea.value
		const indent = "    "
		textarea.value = textarea.value.substring(0, start) + indent + textarea.value.substring(end)
		textarea.selectionStart = textarea.selectionEnd = start + indent.length
	}

	const emits = defineEmits(['editNote']);

	function editNote(e) {
		emits('editNote', e.target.value);
	}
</script>

<style scoped>
.notebook-wrapper {
	width: 100%;
	height: fit-content;
	position: relative;
	display: flex;
	flex-direction: column;
	overflow: hidden;
	font-family: 'Courier New', monospace;
}

.title-textarea {
    flex: 1;
    font-size: 42px;
    line-height: 50px;
    padding: 0 20px;
    border: none;
    outline: none;
    resize: none;
    box-sizing: border-box;
    background-image: repeating-linear-gradient(
        to bottom,
        transparent,
        transparent 23px,
        #ccc 24px
    );
    background-size: 100% 24px;
	text-align: center;
    field-sizing: content;
}

.lined-textarea {
	flex: 1;
	font-size: 16px;
	line-height: 24px;
	padding: 0 20px;
	border: none;
	outline: none;
	resize: none;
	box-sizing: border-box;

	background-image: repeating-linear-gradient(
		to bottom,
		transparent,
		transparent 23px,
		#ccc 24px
	);

	background-size: 100% 24px;
    field-sizing: content;
	height: auto;
}
</style>