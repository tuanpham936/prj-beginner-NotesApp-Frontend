<template>
    <div class="folder-item">
        <input class="folder-title" v-model="editName" v-on:dblclick.left="toggleReadonly" :readonly="isReadonly">
            
            <button class="folder-menu-button"><i class="fa-solid fa-bars"></i></button>
        </input>
        <div class="submenu" v-show="files.length">
            <br>
            <notefile v-for="file in files" :keys="file.name" :name="file.name"/>
        </div>
    </div>
</template>

<script setup>
    import { ref } from 'vue';
    import notefile from './NoteFile.vue';

    const props = defineProps({
        name: {
            type: String,
            default: 'New Folder',
            required: false,
        },
        files: {
            type: Array,
            required: false,
        }
    })

    const isReadonly = ref(true);

    function toggleReadonly() {
        isReadonly.value = !isReadonly.value;
    }

    const emits = defineEmits(['editfoldername'])

    function editName() {
        
    }
</script>

<style scoped>
.folder-item {
    margin-bottom: 10px;
}

.folder-title {
    position: relative;
    cursor: pointer;
    font-weight: bold;
    padding: 8px;
    background-color: #dfe6e9;
    border-radius: 4px;
    user-select: none;
    display: flex;
    justify-content: space-between;
    align-items: center;
    border: none;
}

.submenu {
    margin-left: 15px;
}

.folder-menu-button {
  display: none;
  background: none;
  border: none;
  font-size: 18px;
  cursor: pointer;
  padding: 0;
  margin: 0;
  color: #636e72;
  height: fit-content;
}

.folder-title:hover {
    background-color: #b2bec3;
}

.folder-title:hover .folder-menu-button {
    display: inline;
}
</style>