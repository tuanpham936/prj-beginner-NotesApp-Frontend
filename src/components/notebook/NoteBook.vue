<template>
    <div>
        <header class="sticky-header">
			<toolbar/>
        </header>

        <div class="layout-container">
            <aside class="sidebar left sticky-scrollable" ref="noteOrganizer">
				<div class="pill-wrapper" id="pillContainer" @click.left="AddNewFolder">
					<button class="pill-button" id="pillButton"><i class="fa-solid fa-plus"></i> Add Folder</button>
				</div>
				<br>
				<notesFolder v-for="folder in folders" ref="folderButtons" :id="folder.id" :folder-name="folder.name" :files="folder.files"  @edit-folder-name="UpdateFolderName" @remove-folder="RemoveFolder" @file-change-folder="FileChangeFolder" @remove-file="RemoveFile" @open-file="OpenFile"/>
            </aside>

            <main class="main-content">
              	<writenote v-model:content="noteContent" @edit-note="UpdateNote"/>
				<chooseFolderModal v-show="enableChooseFolderModal" :folders="folders" @closeModal="UnactiveModal" @chooseFolder="ChooseChangeFolder"/>
            </main>

            <aside class="sidebar right sticky-scrollable">

            </aside>
        </div>
    </div>
</template>

<script setup>
	//import
	import writenote from './WriteNote.vue';
	import notesFolder from './NotesFolder.vue';
	import chooseFolderModal from './ChooseFolderModal.vue';
	import toolbar from './NotebookToolbar.vue'
	import { ref, watch } from 'vue'; //Declare primitive variables
	import { reactive } from 'vue'; //Declare object, array variables

	//data
    //Note data
  	const noteID = ref('');
	const noteContent = ref('');
	const enableChooseFolderModal = ref(false);
    //Note Organizer
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
  
	const filesContent = reactive([
		{
		name: 'File 1',
		id: 'ABC-123',
		content: 'This is content of File 1',
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

	function UpdateNote(newNote) {
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

	function RemoveFile(fileID, id) {
		const folder = folders.value.find(f => f.id === id);
		const index = folder.files.findIndex(f => f.id == fileID);
		if (folder.files[index].id === noteID.value) noteID.value = '';
		folder.files.splice(index, 1);
		folder.files = sortByNameAsc(folder.files);
	}

	function OpenFile(fileID) {
		//request infor from server
		noteID.value = fileID;
		noteContent.value = filesContent.find(f => f.id === fileID).content;
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
  overflow: hidden; /* Ngăn scroll toàn trang */
}

.sticky-header {
  position: sticky;
  top: 0;
  background: #3498db;
  color: white;
  text-align: center;
  font-weight: bold;
  z-index: 10;
}

/* Wrapper layout */
.layout-container {
  flex: 1;
  display: flex;
  height: 100%;
}

/* Sticky, scrollable sidebar */
.sidebar {
  width: 15%;
  background: #ecf0f1;
  padding: 20px;
  overflow-y: auto;
}

.sticky-scrollable {
  position: sticky;
  top: 80px; /* Giống chiều cao header */
  max-height: calc(100vh - 80px); /* Giới hạn chiều cao dưới */
}

/* Main content scroll riêng */
.main-content {
  flex: 1;
  background: #fff;
  padding: 20px;
  overflow-y: auto;
  height: calc(100vh - 80px);
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


</style>