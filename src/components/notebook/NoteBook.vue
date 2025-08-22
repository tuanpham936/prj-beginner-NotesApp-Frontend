<template>
	<div class="notebook">
		<header class="sticky-header note-toolbar">
			<toolbar @exec-command="execCommand" :save-status="saveStatus"/>
		</header>	

		<div class="layout-container">
			<main class="main-content">
				<div class="note-editor-wrapper">
					<writenote style="z-index: 5;" ref="noteEditor" @edit-note="UpdateNote" @notify="Notify" @edit-title="UpdateTitle"/>
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

		<chooseFolderModal v-show="enableChooseFolderModal" :folders="folders" @closeModal="UnactiveModal" @chooseFolder="ChooseFolder"/>
		
	</div>
	<message ref="msgBox" v-show="enableMessage" @msg-confirm="UnactiveMessage"/>
</template>

<script setup>
	//import
	import writenote from './WriteNote.vue';
	import notesFolder from './NotesFolder.vue';
	import chooseFolderModal from './ChooseFolderModal.vue';
	import toolbar from './NotebookToolbar.vue';
	import message from '../ultils/Message.vue';
	import { onMounted, ref, watch } from 'vue'; //Declare primitive variables
	import { reactive } from 'vue'; //Declare object, array variables
	import { getFolders, postFolder, updateFolder, deleteFolder, getFilesByFolderId, postFile, updateFile, deleteFile, postNote, getNote, updateNote, deleteNote } from '../ultils/axios.vue'

	//data
    //Note data
  	const noteID = ref('');
	const noteTitle = ref('');
	const noteContent = ref('');
	const enableChooseFolderModal = ref(false);
	const noteEditor = ref(null);
	const saveStatus = ref(true);
    //Note Organizer Data
	const enableFolderHier = ref(false);
	const folders = ref([
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
	const msgBox = ref(null);

	//Sidebar
	const sidebar = ref(null);

	//Choose Folder
	const cfmstate = ref('');

	//Start
	onMounted(async () => {
		// folders.value.splice(0);
		folders.value = await getFolders(); 
		if (!folders.value) {
			Notify('Error: Cannot get folders', 'error');
			folders.value = [];
			return;
		}
		folders.value.forEach(async (folder) => {
			folder.files = [];
			const data = await getFilesByFolderId(folder.id);
			if (!data) {
				Notify('Error: Cannot get note in folders', 'error');
				folder.files = [];
				return;
			}
			folder.files = data;
		})
	});

	async function UpdateTitle(newTitle) {
		noteTitle.value = newTitle;

		if (noteID.value === '') return;
		
		const folder = folders.value.find(fd => fd.files.find(fl => fl.id === noteID.value));

		const flag = await updateFile(noteID.value, newTitle, folder.id);
		if (!flag) {
			Notify('Error: Cannot update note title', 'error');
			return;
		}

		const file = folder.files.find(fl => fl.id === noteID.value);
		file.name = newTitle;
	}

	function UpdateNote(newNote) {
		if (saveStatus.value) saveStatus.value = false;
		noteContent.value = newNote;
	}

	async function UpdateFolderName(id, newName) {
		const flag = await updateFolder(id, newName);
		if (!flag) {
			Notify('Error: Cannot update folder name', 'error');
			return;
		}
		const folder = folders.value.find(f => f.id === id);
		folder.name = newName;
		folders.value = sortByNameAsc(folders.value);
	}

	async function RemoveFolder(id) {
		const folder = folders.value.find(f => f.id === id);
		folder.files.forEach(file => {
			RemoveFile(file.id);
		});
		const flag = await deleteFolder(id);
		if (!flag) {
			Notify('Error: Cannot remove folder', 'error');
			return;
		}
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
		ActiveModal('changefolder');
	}

	function SaveFile() {
		if (saveStatus.value) return;

		if (noteID.value.trim() === '') {
			ActiveModal('newfile');
		}
		else {
			const flag = updateNote(noteID.value, noteContent.value);
			if (!flag) {
				Notify('Error: Cannot save note', 'error');
				return;
			}
			saveStatus.value = true;
		}
	}

	function CheckSaveStatus() {
		if (!saveStatus.value) {
			Notify('Note is not saved yet!', 'error');
		}
		
		return saveStatus.value;
	}

	function RemoveFileLocal(fileID, fdId) {
		const folder = folders.value.find(f => f.id === fdId);
		// const file = folder.files.find(f => f.id === fileID);
		const index = folder.files.findIndex(f => f.id == fileID);
		if (folder.files[index].id === noteID.value) noteID.value = '';
		folder.files.splice(index, 1);
		folder.files = sortByNameAsc(folder.files);
	}

	function RemoveFile(fileID, fdId) {
		const flag1 = deleteFile(fileID, fdId);
		if (!flag1) {
			Notify('Error: Cannot remove note', 'error');
			return;
		}
		RemoveFileLocal(fileID, fdId);
	}

	async function OpenFile(fileID, fileTitle) {
		if (!CheckSaveStatus()) return;

		if (fileID === null) {	
			noteID.value = '';
			noteTitle.value = '';
			noteContent.value = '';
			noteEditor.value?.openFile(null, null);
		}
		else {
			const openNoteContent = await getNote(fileID);
			if (!openNoteContent) {
				Notify('Error: Cannot open note', 'error');
				return;
			}
			noteID.value = fileID;
			noteTitle.value = fileTitle;
			noteContent.value = openNoteContent;
			noteEditor.value?.openFile(noteTitle.value, noteContent.value);
		}

		saveStatus.value = true;
	}

	function ActiveModal(state) {
		cfmstate.value = state;
		enableChooseFolderModal.value = true;
	}

	function UnactiveModal() {
		enableChooseFolderModal.value = false;
	}

	async function ChooseFolder(newFolderId) {
		if (newFolderId === null) return;
		if (cfmstate.value === 'newfile') {
			const newFile = await postFile(noteTitle.value, newFolderId);
			if (!newFile) {
				Notify('Error: Cannot add new note', 'error');
				return;
			}

			// const newNote = postNote(newFile.id, noteContent.value);
			// if (!newNote) {
			// 	deleteFile(newFile.id, newFolderId);
			// 	Notify('Error: Cannot add new note', 'error');
			// 	return;
			// }

			const folder = folders.value.find(f => f.id === newFolderId);
			folder.files.push(newFile);
			folder.files = sortByNameAsc(folder.files);

			noteID.value = newFile.id;
			saveStatus.value = true;
		}
		else if (cfmstate.value === 'changefolder') {
			if (newFolderId !== folderId.value) {
				const flag = await updateFile(fileChange.id, fileChange.name, newFolderId);
				if (!flag) {
					Notify('Error: Cannot change note\'folder', 'error');
					return;
				}

				RemoveFileLocal(fileChange.id, folderId.value);
				const folder = folders.value.find(f => f.id === newFolderId);
				const newFile = reactive({
					id: fileChange.id,
					name: fileChange.name,
				});
				folder.files.push(newFile);
				folder.files = sortByNameAsc(folder.files);
			}
		}
		UnactiveModal();
	}
	
	function sortByNameAsc(list) {
		return list.slice().sort((a, b) => a.name.localeCompare(b.name));
	}

	async function AddNewFolder() {
		const newFolder = await postFolder();
		if (!newFolder) {
			Notify('Error: Cannot add new folder', 'error');
			return;
		}
		newFolder.files = [];
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
				OpenFile(null);
				return;
			default:
				break;
		}

		noteEditor.value?.executeCommand(cmd, value);
	}

	//Message Method
	function Notify(msg, type) {
		msgBox.value?.setMessage(msg, type);
		ActiveMessage();
	}

	function ActiveMessage() {
		enableMessage.value = true;
	}
	
	function UnactiveMessage() {
		enableMessage.value = false;
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