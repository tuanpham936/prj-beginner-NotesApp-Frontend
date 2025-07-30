<template>
    <div>
        <header class="sticky-header">
            
        </header>

        <div class="layout-container">
            <aside class="sidebar left sticky-scrollable">
                <notefolder v-for="folder in folders" :name="folder.name" :files="folder.files"/>
            </aside>

            <main class="main-content">
                <writenote/>
            </main>

            <aside class="sidebar right sticky-scrollable">

            </aside>
        </div>
    </div>

</template>

<script setup>
    import { ref } from 'vue';
    const folders = ref([
        {
        name: 'Menu 1',
        files: [
            {
                name: 'File 1'
            },
            {
                name: 'File 2'
            }
        ]},
        {
        name: 'Menu 2',
        files: [
            {
                name: 'File 3'
            },
            {
                name: 'File 4'
            }
        ]},
    ]);

    import writenote from './WriteNote.vue';
    import notefolder from './NoteFolder.vue';
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