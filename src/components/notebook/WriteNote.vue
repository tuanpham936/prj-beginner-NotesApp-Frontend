<template>
    <div class="notebook-wrapper" >
        <!-- <textarea class="title-textarea" placeholder="Tiêu đề..." :value="title" v-on:keydown.tab.prevent="tabIndent" @input="editNote"></textarea>
		<br>
		<hr> -->
        <div ref="textarea" contenteditable="true" id="note-editor" class="lined-textarea" placeholder="Nội dung..." v-on:keydown.tab.prevent="tabIndent" @click="" @input="editNote">
		</div>
		<!-- <div class="lined-textarea bottom-textarea">
			<p>This is bottom of sheet - Can't write</p>
			<br>
			<br>
			<br>
			<br>
			<br>
			<br>
			<br>
			<br>
			<br>
			<br>
			<br>
		</div> -->
	</div>

	<div ref="storage" style="visibility: hidden;">
		<span ref="linkInput" class="input-tooltip-wrapper">
			<span v-show="enableLinkInput" class="input-tooltip" contenteditable="false">
				<span style="user-select: none;" for="URL"><b>URL</b></span>
				<input ref="linkInputBox" type="url" name="URL" id="link-input" @blur="AttachLink">
			</span>
		</span>
		
		<span ref="cmtInput" class="input-tooltip-wrapper">
			<span v-show="enableCmtInput" class="input-tooltip" contenteditable="false">
				<span style="user-select: none;" for="comment"><b>Comment</b></span>
				<input ref="cmtInputBox" type="text" name="comment" id="cmt-input" @blur="AttachCmt">
			</span>
		</span>
	</div>


</template>

<script setup>
	//import
	import { ref } from 'vue';
	import { nextTick } from 'vue';

	//expose
	defineExpose({openFile, executeCommand});

	//var
	const textarea = ref(null);
	const emits = defineEmits(['editNote', 'notify']);
	const historyLimit = 24;
	const history = ref([]);

	const storage = ref(null);

	//Attach link
    const enableLinkInput = ref(false);
	const linkInput = ref(null);
	const linkRangeStorage = ref(null);
	const linkInputBox = ref(null);

	//Attach Comment 
	const enableCmtInput = ref(false);
	const cmtInput = ref(null);
	const cmtRangeStorage = ref(null);
	const cmtInputBox = ref(null);

	//func
	function tabIndent() {
		document.execCommand('insertText', false, '\t')
	}

	function openFile(content) {
		textarea.value.innerHTML = content;
	}

	function editNote() {
		emits('editNote', textarea.value.innerHTML);
	}

	function executeCommand(cmd, value) {
		textarea.value?.focus();
		const selection = window.getSelection();

		if (cmd === 'backColor' || cmd === 'foreColor') 
		{
			if (selection.rangeCount > 0) {
				const range = selection.getRangeAt(0)
				const node = range.startContainer.parentElement;

				if  (cmd === 'foreColor') {
					if (rgbToHex(getComputedStyle(node).color) === value) {
						document.execCommand(cmd, false, '#000000');
					}
					else {
						document.execCommand(cmd, false, value);
					}
				}

				if (cmd === 'backColor') {
					if (rgbToHex(getComputedStyle(node).backgroundColor) === value) {
						document.execCommand(cmd, false, 'white');
					}
					else {
						document.execCommand(cmd, false, value);
					}
				}
			}
		}
		else if (cmd === 'unorderList' || cmd === 'orderList') {
			const listTag = cmd === 'unorderList' ? 'ul' : 'ol';

			if (selection.rangeCount > 0) {
				const range = selection.getRangeAt(0);
				let parentNode = range.commonAncestorContainer.nodeType === Node.ELEMENT_NODE ? range.commonAncestorContainer : range.commonAncestorContainer.parentElement;
				let nodes = Array.from(parentNode.childNodes);
				nodes = nodes.filter(node => range.intersectsNode(node));

				if (parentNode.tagName === 'LI') {
					parentNode = parentNode.parentElement;
					nodes = Array.from(parentNode.childNodes);
				}

				if (parentNode.tagName === 'UL' || parentNode.tagName === 'OL') {
					if (listTag.toUpperCase() === parentNode.tagName) {
						nodes.forEach(node => {
							const newNode = document.createElement('div');
							if (node.childNodes.length === 0) {
								newNode.appendChild(node);
							}
							else {
								node.childNodes.forEach(childNode => {		
									
									newNode.appendChild(childNode);
								});
							}
							parentNode.parentElement.insertBefore(newNode, parentNode);
						});	
						parentNode.remove();
					}
					else {
						const newList = document.createElement(listTag);
						nodes.forEach(node => {
							const newNode = document.createElement('li');
							if (node.childNodes.length === 0) {
								newNode.appendChild(node);
							}
							else {
								node.childNodes.forEach(childNode => {		
									newNode.appendChild(childNode);
								});
								node.remove();
							}
							newList.appendChild(newNode);
						});
						parentNode.parentElement.insertBefore(newList, parentNode);
						parentNode.remove();
					}
				} 
				else {
					if (nodes.length === 0) {
						const newList = document.createElement(listTag);
						const newListitem = document.createElement('li');
						newList.appendChild(newListitem);
						parentNode.appendChild(newList);
					}
					else {
						const newList = document.createElement(listTag);
						parentNode.insertBefore(newList, nodes[0]);
						nodes.forEach(node => {
							const newNode = document.createElement('li');
							if (node.childNodes.length === 0) {
								newNode.appendChild(node);
							}
							else {
								node.childNodes.forEach(childNode => {		
									newNode.appendChild(childNode);
								});
								node.remove();
							}
							newList.append(newNode);
						});	
					}
				}
			}
		}
		else if (cmd === 'attachLink') {
			if (selection.rangeCount > 0) {
				const range = selection.getRangeAt(0).cloneRange();
				if (range.startContainer === range.endContainer) {
					if (range.startOffset === range.endOffset) {
						Notify('Cannot attach link to nothing', 'alert');
						return;
					}
				}
				else {
					Notify('Cannot attach link cross paragraphs', 'alert');
					return;
				}
				const node = range.startContainer;
				const startOffset = range.startOffset
				const endOffset = range.endOffset

				const text = node.textContent;
				const before = text.slice(0, startOffset);
				const after = text.slice(startOffset);

				const beforeNode = document.createTextNode(before);
				const afterNode = document.createTextNode(after);

				const insertNode = (linkInput.value);

				const parent = node.parentNode

				parent.replaceChild(afterNode, node)
				parent.insertBefore(insertNode, afterNode)
				parent.insertBefore(beforeNode, insertNode)

				const newRange = document.createRange()
				newRange.setStart(afterNode, 0)
				newRange.setEnd(afterNode, endOffset - beforeNode.length)

				selection.removeAllRanges()
				selection.addRange(newRange)

				linkRangeStorage.value = newRange;

				ActiveLinkInput();
			}
		}
		else if (cmd === 'attachComment') {
			if (selection.rangeCount > 0) {
				const range = selection.getRangeAt(0).cloneRange();
				if (range.startContainer === range.endContainer) {
					if (range.startOffset === range.endOffset) {
						Notify('Cannot attach comment to nothing', 'alert');
						return;
					}
				}
				else {
					Notify('Cannot attach comment cross paragraphs', 'alert');
					return;
				}
				const node = range.startContainer;
				const startOffset = range.startOffset
				const endOffset = range.endOffset

				const text = node.textContent;
				const before = text.slice(0, startOffset);
				const after = text.slice(startOffset);

				const beforeNode = document.createTextNode(before);
				const afterNode = document.createTextNode(after);

				const insertNode = (cmtInput.value);

				const parent = node.parentNode

				parent.replaceChild(afterNode, node)
				parent.insertBefore(insertNode, afterNode)
				parent.insertBefore(beforeNode, insertNode)

				const newRange = document.createRange()
				newRange.setStart(afterNode, 0)
				newRange.setEnd(afterNode, endOffset - beforeNode.length)

				selection.removeAllRanges()
				selection.addRange(newRange)

				cmtRangeStorage.value = newRange;

				ActiveCmtInput();
			}
		}
		else {
			document.execCommand(cmd, false, value);
		}
	}

	function rgbToHex(rgb) {
		const result = rgb.match(/\d+/g)
		if (!result) return '#000000'

		const [r, g, b] = result.map(x => parseInt(x).toString(16).padStart(2, '0'))
		return `#${r}${g}${b}`
	}

	//Attach Link
    async function ActiveLinkInput() {
        enableLinkInput.value = true;
		await nextTick();
		document.getSelection()?.removeAllRanges();
		document.activeElement.blur();
		linkInputBox.value?.focus();
    }

    function UnactiveLinkInput() {
        enableLinkInput.value = false;
		storage.value.appendChild(linkInput.value);
    }

	function AttachLink(e) {
		const selection = window.getSelection();
		selection.removeAllRanges()
		selection.addRange(linkRangeStorage.value)
		const value = e.target.value;
		UnactiveLinkInput();
		executeCommand('createLink', value);
	}

	function Notify(msg, type) {
		emits('notify', msg, type);
	}

	//Attach Comment
    async function ActiveCmtInput() {
        enableCmtInput.value = true;
		await nextTick();
		document.getSelection()?.removeAllRanges();
		document.activeElement.blur();
		cmtInputBox.value?.focus();
    }

    function UnactiveCmtInput() {
        enableCmtInput.value = false;
		storage.value.appendChild(cmtInput.value);
    }

	function AttachCmt(e) {
		const selection = window.getSelection();
		selection.removeAllRanges()
		selection.addRange(cmtRangeStorage.value);
		const value = e.target.value;
		UnactiveCmtInput();

		if (selection.rangeCount > 0) {
			const range = selection.getRangeAt(0);
			const node = range.startContainer;
			
			const startOffset = range.startOffset
			const endOffset = range.endOffset

			const text = node.textContent;
			const before = text.slice(0, startOffset);
			const after = text.slice(endOffset);

			const beforeNode = document.createTextNode(before);
			const afterNode = document.createTextNode(after);

			const insertNode = document.createElement('span');
			insertNode.classList.add('tooltip');
			insertNode.textContent = selection.toString();

			const commentNode = document.createElement('span');
			commentNode.classList.add('tooltiptext');
			commentNode.textContent = value;

			insertNode.appendChild(commentNode);

			const parent = node.parentNode;

			parent.replaceChild(afterNode, node);
			parent.insertBefore(insertNode, afterNode);
			parent.insertBefore(beforeNode, insertNode);

			const newRange = document.createRange();
			newRange.setStartAfter(insertNode);

			selection.removeAllRanges();
			selection.addRange(newRange);
		}
	}

</script>

<style scoped>
.notebook-wrapper {
	width: 100%;
	height: fit-content;
	position: relative;
	display: flex;
	flex-direction: column;
	/* overflow: hidden; */
	font-family: 'Courier New', monospace;
	box-sizing: border-box;
	margin: 20px 0;
	z-index: 0;
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
	z-index: 0;
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
	height: 100%;
	/* border-top: red solid 2px; */
	z-index: 0;
}

.input-tooltip-wrapper {
	position: relative;
}

.input-tooltip {
	background-color:whitesmoke;
	width: fit-content;
	box-sizing: border-box;
	text-align: center;
	border-radius: 6px;
	padding: 5px 10px;
	position: absolute;
	z-index: 10;
	top: auto;
	left: calc(50%);
  	transform: translateX(-50%);
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
	overflow:-moz-hidden-unscrollable;
	
	margin-top: 24px;

	font-size: 16px;
}

.input-tooltip::after {
	content: "";
	position: absolute;
	bottom: 100%;
	left: 50%;
	margin-left: -5px;
	border-width: 5px;
	border-style: solid;
	z-index: 9;
	border-color: transparent transparent whitesmoke transparent;
}
</style>