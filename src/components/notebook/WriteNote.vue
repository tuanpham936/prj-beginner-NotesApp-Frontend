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

	<div ref="storage" style="display: none;">
		<div ref="noteTable" class="note-table" contenteditable="false">
			<table contenteditable="true">
				<tbody>
					<tr class="note-table-row">
						<td class="note-table-col"><br></td>
					</tr>
				</tbody>
			</table>

			<button contenteditable="false" class="table-options">
				<i class="fa-solid fa-chevron-down"></i>
				<ul class="table-options-menu">
					<li class="table-menu-item">
						<i class="fa-solid fa-table icon" style="width: 100%;text-align: center;"> Table</i>
						<i class="fa-solid fa-left-right button" data-table-mode="expand" @click="AdjustTableMode"></i>
						<i class="fa-solid fa-trash button" @click="RemoveTable"></i>
					</li>
					<li class="table-menu-item">
						<i class="icon" style="width: 100%;;text-align: center;">Col <span>at</span> <input style="width: 35px;" type="number" name="" class="table-col-numb" min="1" value="1"></i>
						<i class="fa-solid fa-plus button" @click="AdjustTableCol(true, $event.target)"></i>
						<i class="fa-solid fa-minus button" @click="AdjustTableCol(false, $event.target)"></i>
					</li>
					<li class="table-menu-item">
						<i class="icon" style="width: 100%;text-align: center;">Row <span>at</span> <input style="width: 35px;" type="number" name="" class="table-row-numb" min="1" value="1"></i>
						<i class="fa-solid fa-plus button" @click="AdjustTableRow(true, $event.target)"></i>
						<i class="fa-solid fa-minus button" @click="AdjustTableRow(false, $event.target)"></i>
					</li>
				</ul>
			</button>
		</div>

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

	//Selection
	const selectedRange = ref(null);

	//Table
	const noteTable = ref(null);

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
		selection.removeAllRanges();
		selection.addRange(selectedRange.value);

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
				let node = range.startContainer;

				if (range.startContainer.nodeType === Node.TEXT_NODE) {
					node = range.startContainer;
				} else {
					node = range.startContainer.firstChild;
				}
			
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
				let node = range.startContainer;

				if (range.startContainer.nodeType === Node.TEXT_NODE) {
					node = range.startContainer;
				} else {
					node = range.startContainer.firstChild;
				}
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
		else if (cmd === 'createTable') {
			if (selection.rangeCount > 0) {
				const range = selection.getRangeAt(0).cloneRange();
				// if (range.startContainer === range.endContainer) {
				// 	if (range.startOffset === range.endOffset) {
				// 		Notify('Cannot attach comment to nothing', 'alert');
				// 		return;
				// 	}
				// }
				// else {
				// 	Notify('Cannot attach comment cross paragraphs', 'alert');
				// 	return;
				// }

				let node = range.commonAncestorContainer;
				let parentNode = node.parentElement;
				if (node.id === 'note-editor') {
					parentNode = node;
					node = null;
				}
				while (parentNode.id !== 'note-editor') {
					node = parentNode;
					parentNode = node.parentElement;
				}
				const newLine = document.createElement('div');
				const newBr = document.createElement('br');
				newLine.appendChild(newBr);
				if (node === null) {
					parentNode.appendChild(newLine.cloneNode(true));
					parentNode.appendChild(noteTable.value.cloneNode(true));
					parentNode.appendChild(newLine.cloneNode(true));
				}
				else {
					node.insertAdjacentElement('afterend', newLine.cloneNode(true));
					node.insertAdjacentElement('afterend', noteTable.value.cloneNode(true));
					node.insertAdjacentElement('afterend', newLine.cloneNode(true));
				}
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
		const value = e.target.value;
		UnactiveLinkInput();
		const selection = window.getSelection();
		selection.removeAllRanges();
		selection.addRange(linkRangeStorage.value);

		if (selection.rangeCount > 0) {
			const range = selection.getRangeAt(0);
			let node = range.startContainer;

			if (range.startContainer.nodeType === Node.TEXT_NODE) {
				node = range.startContainer;
			} else {
				node = range.startContainer.firstChild;
			}
			
			const startOffset = range.startOffset
			const endOffset = range.endOffset

			const text = node.textContent;
			const before = text.slice(0, startOffset);
			const after = text.slice(endOffset);

			const beforeNode = document.createTextNode(before);
			const afterNode = document.createTextNode(after);

			const insertNode = document.createElement('a');
			insertNode.href = value;
			insertNode.textContent = selection.toString();

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
		const value = e.target.value;
		UnactiveCmtInput();
		const selection = window.getSelection();
		selection.removeAllRanges()
		selection.addRange(cmtRangeStorage.value);

		if (selection.rangeCount > 0) {
			const range = selection.getRangeAt(0);
			let node = range.startContainer;

			if (range.startContainer.nodeType === Node.TEXT_NODE) {
				node = range.startContainer;
			} else {
				node = range.startContainer.firstChild;
			}
			
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

	function getHTMLSelectedRange() {
		const selection = window.getSelection();
		if (!selection.rangeCount) return null;

		// const range = selection.getRangeAt(0);
		// // const node = range.startContainer.parentElement;
		return selection.getRangeAt(0).cloneRange();
	}

	document.addEventListener('mouseup', () => {
		selectedRange.value = getHTMLSelectedRange();
	});

	//Table 
	function AdjustTableCol(incs, element) {
		let menuItemParent = element.parentElement;
		const index = menuItemParent.firstChild.lastChild.value;

		let table = menuItemParent.parentElement.parentElement.parentElement.firstChild.firstChild;

		const newCol = document.createElement('td');
		const br = document.createElement('br');
		newCol.appendChild(br);
		newCol.classList.add('note-table-col');

		for (let i = 0; i < table.childNodes.length; i++) {
			const row = table.childNodes[i];
			
			if (row.childNodes.length < index - 1) {
				if (incs) row.appendChild(newCol.cloneNode(true));
				else row.lastChild.remove();
			}
			else {
				if (incs) row.insertBefore(newCol.cloneNode(true), row.childNodes[index - 1]);
				else row.childNodes[index - 1].remove();
			}
		}
	}	

	function AdjustTableRow(incs, element) {
		let menuItemParent = element.parentElement;
		const index = menuItemParent.firstChild.lastChild.value;

		let table = menuItemParent.parentElement.parentElement.parentElement.firstChild.firstChild;

		const newCol = document.createElement('td');
		const br = document.createElement('br');
		newCol.appendChild(br);
		newCol.classList.add('note-table-col');

		const count = table.firstChild.childNodes.length;

		const newRow = document.createElement('tr');
		newRow.classList.add('note-table-row');

		for (let i = 0; i < count; i++) {
			newRow.appendChild(newCol.cloneNode(true));
		}

		if (table.childNodes.length < index - 1) {
			if (incs) table.appendChild(newRow);
			else table.lastChild.remove();
		}
		else {
			if (incs) table.insertBefore(newRow, table.childNodes[index - 1]);
			else table.childNodes[index - 1].remove();
		}
	}	

	function AdjustTableMode(e) {
		const tableMode = e.target.dataset.tableMode;

		let table = e.target.parentElement.parentElement.parentElement.parentElement.firstChild;
		
		if (tableMode === 'expand') {
			e.target.dataset.tableMode = 'fit';
			table.style.width = 'fit-content';
		}
		else {
			e.target.dataset.tableMode = 'expand';
			table.style.width = '100%';
		}
	}

	function RemoveTable(e) {
		let table = e.target.parentElement.parentElement.parentElement.parentElement;

		table.remove();
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

.note-table {
	display: flex;
	justify-content: center;
}

.note-table table {
	box-sizing: border-box;
	padding: 0;
	margin: 0;
	border: none;
	border-spacing: 0;
	width: 100%;
}

[contenteditable="true"] :deep(.note-table-row) {
	outline:1px #000 solid;
}

[contenteditable="true"] :deep(.note-table-col) {
	outline:1px #000 solid;
	padding: 0 2px;
}

.table-options {
	box-sizing: border-box;
    position: absolute;
    right: -1%;
	border-radius: 100%;
	border: none;
	padding: 1px;
	width: max-content;
	cursor: pointer;
	outline: #333 2px solid;
	background: none;
}

.table-options:hover .table-options-menu, .table-options-menu:hover {
	visibility:visible;
}

.table-options-menu {
	visibility: hidden;
    color: #333;
    position: absolute;
	top: 60%;
	left: 0;
    background-color: white;
    border: 1px solid #ccc;
    border-radius: 6px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    margin-top: 6px;
    list-style: none;
    padding: 6px 0;
    width: auto;
    z-index: 100;
	text-align: left;
}

.table-options-menu .table-menu-item {
    padding: 10px 16px;
    cursor:auto;
    font-size: auto;
    transition: background-color 0.2s ease;
    white-space: nowrap;
    position: relative;
	display: flex;
	justify-content: space-evenly;
}

/* .table-options-menu .table-menu-item:hover {
    background-color: #f1f2f6;
} */

.table-menu-item .icon {
	border-right: 2px solid #333;
}

.table-menu-item i {
	padding: 5px 10px;
	font-weight: 900;
	text-transform: uppercase;
	font-style: normal;
	font-family: "Font Awesome 6 Free";
	margin: auto 0;
}

.table-menu-item .button {
    transition: background-color 0.2s ease;
	cursor: pointer;
}

.table-menu-item .button:hover {
    background-color: #dddee0;
}
</style>