<template>
    <div>
        <header class="sticky-header">

        </header>

        <div class="layout-container">
            <aside class="sidebar left sticky-scrollable">
				<notesFolder v-for="folder in folders" :folder-name="folder.name" :files="folder.files"  @edit-folder-name="UpdateFolderName" @remove-folder="RemoveFolder" @file-change-folder="FileChangeFolder" @remove-file="RemoveFile" @open-file="OpenFile"/>
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
    //Note data
  const noteID = ref('');
	const noteContent = ref('');
    //Note Organizer
	const folders = reactive([
		{
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

	function UpdateFolderName(oldName, newName) {
		const folder = folders.find(f => f.name === oldName);
		folder.name = newName;
	}

	function RemoveFolder(name) {
		const index = folders.findIndex(f => f.name === name);
		folders.splice(index, 1);
	}

  function FileChangeFolder(fileID, folderName) {
		const folder = folders.find(f => f.name === folderName);
    const file = folder.files.find(f => f.id == fileID);
    
  }

  function RemoveFile(fileID, folderName) {
		const folder = folders.find(f => f.name === folderName);
    const index = folder.files.findIndex(f => f.id == fileID);
    folder.files.splice(index, 1);
  }

  function OpenFile(fileID) {
    noteID.value = fileID;
    noteContent.value = filesContent.find(f => f.id === fileID).content;
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