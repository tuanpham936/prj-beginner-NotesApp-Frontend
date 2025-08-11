<template>
	<div class="notebook">
		<header class="sticky-header note-toolbar">
			<toolbar @exec-command="execCommand" :save-status="saveStatus"/>
		</header>	

		<div class="layout-container">
			<main class="main-content">
				<div class="note-editor-wrapper">
					<writenote style="z-index: 5;" ref="noteEditor" @edit-note="UpdateNote" @notify="Notify"/>
				</div>
			</main>
		</div>

		<div ref="sidebar" class="sidebar-overlay slide-out-anim" v-show="enableFolderHier">
			<aside class="sidebar left sticky-scrollable slide-in-anim" ref="noteOrganizer">
				<div class="pill-wrapper" id="pillContainer">
					<button class="pill-button" id="pillButton" @click.left="AddNewFolder"><i class="fa-solid fa-plus"></i> Add Folder</button>
					<span class="folder-hier-exit" @click.left="UnactiveFolderHierarchy"><i class="fa-solid fa-circle-xmark"></i></span>
				</div>
				<br>
				<notesFolder v-for="folder in folders" ref="folderButtons" :id="folder.id" :folder-name="folder.name" :files="folder.files"  @edit-folder-name="UpdateFolderName" @remove-folder="RemoveFolder" @file-change-folder="FileChangeFolder" @remove-file="RemoveFile" @open-file="OpenFile"/>
			</aside>
		</div>

		<chooseFolderModal v-show="enableChooseFolderModal" :folders="folders" @closeModal="UnactiveModal" @chooseFolder="ChooseChangeFolder"/>
		
	</div>
	<message v-show="enableMessage" :msg="messageText" :type="messageType" @msg-confirm="UnactiveMessage"/>
</template>

<script setup>
	//import
	import writenote from './WriteNote.vue';
	import notesFolder from './NotesFolder.vue';
	import chooseFolderModal from './ChooseFolderModal.vue';
	import toolbar from './NotebookToolbar.vue';
	import message from '../ultils/Message.vue';
	import { ref, watch } from 'vue'; //Declare primitive variables
	import { reactive } from 'vue'; //Declare object, array variables

	//data
    //Note data
  	const noteID = ref('');
	const noteContent = ref('');
	const enableChooseFolderModal = ref(false);
	const noteEditor = ref(null);
	const saveStatus = ref(true);
    //Note Organizer
	const enableFolderHier = ref(false);
	const folders = ref([
		{
			id: 'F-1',
			name: 'Folder A',
			files: [
				{
				name: 'File 1',
				id: 'ABC-123',
				},
				{
				name: 'File 2',
				id: 'DEF-456',
				}
			],
		},
		{
			id: 'F-2',
			name: 'Folder B',
			files: [
				{
				name: 'File 3',
				id: 'ABCD-1234',
				},
				{
				name: 'File 4',
				id: 'DEFG-4567',
				}
			],
		},
	]);
	const folderButtons = ref([]);
	//File Change Folder Data
	const fileChange = reactive({
		name: '',
		id: '',
	});
	const folderId = ref('');
	//Message
	const enableMessage = ref(false);
	const messageText = ref('');
	const messageType = ref('');

	//Sidebar
	const sidebar = ref(null);

	const filesContent = ref([
		{
		name: 'File 1',
		id: 'ABC-123',
		content: 'This is content<div>of</div><div>File 1</div>',
		},
		{
		name: 'File 2',
		id: 'DEF-456',
		content: 'This is content of File 2',
		},
		{
		name: 'File 3',
		id: 'ABCD-1234',
		content: 'This is content of File 3',
		},
		{
		name: 'File 4',
		id: 'DEFG-4567',
		content: 'This is content of File 4',
		},
	]);

	function NewNote() {

	}

	function UpdateNote(newNote) {
		if (saveStatus.value) saveStatus.value = false;
		noteContent.value = newNote;
	}

	function UpdateFolderName(id, newName) {
		const folder = folders.value.find(f => f.id === id);
		folder.name = newName;
		folders.value = sortByNameAsc(folders.value);
	}

	function RemoveFolder(id) {
		const index = folders.value.findIndex(f => f.id === id);
		folders.value.splice(index, 1);
		folders.value = sortByNameAsc(folders.value);
	}

	function FileChangeFolder(fileID, id) {
		const folder = folders.value.find(f => f.id === id);
		const file = folder.files.find(f => f.id == fileID);
		fileChange.id = file.id;
		fileChange.name = file.name;
		folderId.value = id;
		ActiveModal();
	}

	function SaveFile() {
		if (noteID.value === null) {
			
		}
		else {
			const file = filesContent.value.find(f => f.id === noteID.value);
			file.content = noteContent.value;
		}
	}

	function CheckSaveStatus() {
		if (!saveStatus.value) {
			Notify('Note is not saved yet!', 'alert');
		}
		
		return saveStatus.value;
	}

	function RemoveFile(fileID, id) {
		const folder = folders.value.find(f => f.id === id);
		const index = folder.files.findIndex(f => f.id == fileID);
		if (folder.files[index].id === noteID.value) noteID.value = '';
		folder.files.splice(index, 1);
		folder.files = sortByNameAsc(folder.files);
	}

	function OpenFile(fileID) {
		if (!CheckSaveStatus()) return;

		if (fileID === null) {	
			noteID.value = null;
			noteContent.value = '';
			noteEditor.value?.openFile(null);
		}
		else {
			noteID.value = fileID;
			noteContent.value = filesContent.value.find(f => f.id === fileID).content;
			noteEditor.value?.openFile(noteContent.value);
		}
	}

	function ActiveModal() {
		enableChooseFolderModal.value = true;
	}

	function UnactiveModal() {
		enableChooseFolderModal.value = false;
	}

	function ChooseChangeFolder(id) {
		if (id !== folderId.value) {
			RemoveFile(fileChange.id, folderId.value);
			const folder = folders.value.find(f => f.id === id);
			const newFile = reactive({
				id: fileChange.id,
				name: fileChange.name,
			});
			folder.files.push(newFile);
			folder.files = sortByNameAsc(folder.files);
		}
		UnactiveModal();
	}
	
	function sortByNameAsc(list) {
		return list.slice().sort((a, b) => a.name.localeCompare(b.name));
	}

	function AddNewFolder() {
		const newFolder = {
			id: 'F-' + (folders.value.length + 1),
			name: 'New Folder',
			files: [],
		};
		folders.value.unshift(newFolder);
		folderButtons.value[0]?.ActiveEditName();
	}

	function execCommand(cmd, value) {
		switch (cmd) {
			case 'save':
				SaveFile();
				return;
			case 'folderhier':
				ActiveFolderHierarchy();
				return;
			case 'newNote':

				return;
			default:
				break;
		}

		noteEditor.value?.executeCommand(cmd, value);
	}

	//Message Method
	function Notify(msg, type) {
		messageText.value = msg;
		messageType.value = type;
		ActiveMessage();
	}

	function ActiveMessage() {
		enableMessage.value = true;
	}
	
	function UnactiveMessage() {
		enableMessage.value = false;
		messageText.value = '';
		messageType.value = '';
	}

	//Folder Hierarchy
	function ActiveFolderHierarchy() {
		sidebar.value.classList.remove('slide-out-anim');
		enableFolderHier.value = true;
	}

	async function UnactiveFolderHierarchy() {
		sidebar.value.classList.add('slide-out-anim');
		await wait(500);
		enableFolderHier.value = false;
	}
	
    function wait(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  height: 100vh;
  display: flex;
  flex-direction: column;
  font-family: sans-serif;
  overflow-y: hidden;
  bottom: 0;
}

.notebook {
	display: flex;
	flex-direction: column;
	height: 100vh;
}

.sticky-header {
  background: #3498db;
  color: white;
  text-align: center;
  font-weight: bold;
  z-index: 9;
  height: fit-content;
  width: 100%;
  padding: 0 15%;
}

.sidebar-overlay {
	position: fixed;
	top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    backdrop-filter: blur(4px);
    background-color: rgba(0, 0, 0, 0.3);
	z-index: 10;
}

/* Sticky, scrollable sidebar */
.sidebar {
	background: #ecf0f1;
	padding: 20px;
	height: 100%;
	width: 15%;
	position: fixed;
	z-index: 10;
}

.sticky-scrollable {
	z-index: 10;
  	overflow-y: auto;
	scrollbar-width:none;
}

/* Wrapper layout */
.layout-container {
  	flex: 1;
	display:flex;
	height: 100%;
	z-index: 0;
	overflow: hidden;
	background-color: #3498db;
	box-sizing: border-box;
	padding-bottom: 20px;
  	overflow-y: auto;
}

/* Main content scroll riêng */
.main-content {
	flex: 1;
	background: white;
	padding: 20px;
	width: auto;
	min-height: 100%;
	height: max-content;
	z-index: 5;
	margin: 0 15%;
}

.pill-button {
    padding: 8px 20px;
    background-color: #3498db;
    color: white;
    font-size: 100%;
    font-weight: 500;
    border: none;
    border-radius: 999px; /* 👈 Bo tròn tuyệt đối */
    cursor: pointer;
    transition: background-color 0.2s ease, box-shadow 0.2s ease;
	width: 100%;
}

.pill-button:hover {
    background-color: #2980b9;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
}

.note-editor-wrapper {
	overflow-x: visible;
	position: relative;
}

.pill-wrapper {
	display: flex;
}

.folder-hier-exit {
	width: auto;
	height: 100%;
	text-align: center;
	margin: auto;
	padding: 5px;
	cursor: pointer;
	line-height:100%;
	font-size: 100%;
    color: #3498db;
}

.folder-hier-exit:hover {
   color: red;
}

</style>