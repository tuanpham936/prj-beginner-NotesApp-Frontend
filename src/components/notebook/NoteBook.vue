<template>
    <div>
        <header class="sticky-header">

        </header>

        <div class="layout-container">
            <aside class="sidebar left sticky-scrollable">
				<notesFolder v-for="folder in folders" :folder-name="folder.name"  @edit-folder-name="UpdateFolderName" @remove-folder="RemoveFolder"/>
            </aside>

            <main class="main-content">
                <writenote v-model:content="noteContent" @edit-note="UpdateNote"/>
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
	import { ref } from 'vue'; //Declare primitive variables
	import { reactive } from 'vue'; //Declare object, array variables

	//data
	const noteContent = ref('');
	const folders = reactive([
		{
			name: 'Folder A',
			files: null,
		},
	]);

	function UpdateNote(newNote) {
		noteContent.value = newNote;
	}

	function UpdateFolderName(oldName, newName) {
		const folder = folders.find(f => f.name === oldName);
		folder.name = newName;
	}

	function RemoveFolder(name) {
		const index = folders.findIndex(f => f.name === name);
		folders.splice(index, 1);
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
  padding: 20px;
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

</style>