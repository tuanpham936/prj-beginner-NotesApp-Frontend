<template>
    <div class="folder-button">
        <div class="wide-input-button" @click.left.self="ToggleSubfiles">
            <i class="fa-solid fa-folder"></i>
            <input class="readonly-name" ref="inputName" type="text" :value="folderName" :readonly="!enableNameEdit" @blur="UnactiveEditName" />

            <div class="side-button" @mouseover="ActiveOptionsMenu" @mouseleave="UnactiveOptionsMenu">
                <i class="fa-solid fa-bars"></i>
                <ul class="folder-options-menu" v-show="enableOptionsMenu">
                    <li class="menu-item" @click.left="ActiveEditName"><i class="fa-solid fa-pen"></i> Rename</li>
                    <li class="menu-item" @click.left="RemoveFolder"><i class="fa-solid fa-trash"></i> Remove</li>
                </ul>
            </div>
        </div>
    </div>

    <ul ref="subfiles" class="subfiles slide-in-anim" v-show="enableSubfiles">
        <li v-show="files.length == 0" class="empty-msg">Folder is empty</li>
        <noteFile v-for="file in files" :file-name="file.name" :id="file.id" @file-change-folder="FileChangeFolder" @remove-file="RemoveFile" @open-file="OpenFile" />
    </ul>
    <br>
</template>

<script setup>
    //import
    import { ref } from 'vue';
    import noteFile from './NoteFile.vue';

    //expose
    defineExpose({ActiveEditName});

    //var
    const enableSubfiles = ref(false);
    const enableOptionsMenu = ref(false);
    const props = defineProps({
        id: {
            Type: String,
            required: true,
        },
        folderName: {
            Type: String,
            default: 'New Folder',
        },
        files: {
            Type: Array,
        }
    });
    const enableNameEdit = ref(false);
    const inputName = ref(null);
    const emits = defineEmits(['editFolderName', 'removeFolder', 'fileChangeFolder', 'removeFile', 'openFile']);

    const subfiles = ref(null);

    //func
    async function ToggleSubfiles() {
        if (enableSubfiles.value) {
            subfiles.value.classList.add('slide-out-anim');
            await wait(500);
        }
        else {
            subfiles.value.classList.remove('slide-out-anim');
        }
        enableSubfiles.value = !enableSubfiles.value;
    }

    function ActiveOptionsMenu() {
        enableOptionsMenu.value = true;
    }

    function UnactiveOptionsMenu() {
        enableOptionsMenu.value = false;
    }

    function ActiveEditName() {
        UnactiveOptionsMenu();
        enableSubfiles.value = false;
        enableNameEdit.value = true;
        inputName.value?.focus();
    }

    function UnactiveEditName(e) {
        emits('editFolderName', props.id, e.target.value);
    }

    function RemoveFolder() {
        UnactiveOptionsMenu();
        ToggleSubfiles();
        emits('removeFolder', props.id);
    }

    function FileChangeFolder(fileID) {
        emits('fileChangeFolder', fileID, props.id);
    }

    function RemoveFile(fileID) {
        emits('removeFile', fileID, props.id);
    }

    function OpenFile(fileID) {
        emits('openFile', fileID);
    }

    //Animation
    function wait(ms) {
        return new Promise(resolve => setTimeout(resolve, ms));
    }
</script>

<style scoped>
.folder-button {
    position: relative;
    width: 100%;
    top: -100%;
    animation: slide 0.5s forwards;   
}

@keyframes slide {
    100% { top: 0; }
}

.folder-button:hover .wide-input-button {
    background-color: #2980b9;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
}

.wide-input-button {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    background-color: #3498db;
    padding: 8px 12px;
    border-radius: 6px;
    gap: 10px;
    width: 100%;
    color: white;
    max-width: 100%;
    cursor: pointer;
    overflow-wrap: normal;
    transition: background-color 0.2s ease, box-shadow 0.2s ease;
}

.wide-input-button:hover .side-button {
    display: flex;
}

.readonly-name {
    font-size: 100%;
    font-weight: 500;
    border: none;
    outline: none;
    background: transparent;
    color: white;
    pointer-events: none;
    width: auto;
    width: 100%;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}

.side-button {
    display: none;
    background: none;
    border: none;
    font-size: inherit;
    color: white;
    cursor: pointer;
    padding: 0;
    transition: background-color 0.2s ease;
}

.side-button:hover {
    background-color: rgba(255, 255, 255, 0.15);
}

.subfiles {
    list-style: none;         
    margin: 8px 0 0 20px;      
    padding: 0;
    display: flex;
    flex-direction: column;
    gap: 6px;       
    position: relative;
    /* left: -100%; */
}

.folder-options-menu {
    color: #333;
    position: absolute;
    top: 55%;             
    right: 0;           
    background-color: white;
    border: 1px solid #ccc;
    border-radius: 6px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    margin-top: 6px;
    list-style: none;
    padding: 6px 0;
    width: auto;
    z-index: 1000;
}

.folder-options-menu .menu-item {
    padding: 10px 16px;
    cursor: pointer;
    font-size: auto;
    transition: background-color 0.2s ease;
    white-space: nowrap;
}

.folder-options-menu .menu-item:hover {
    background-color: #f1f2f6;
}

.empty-msg {
    text-align: center;
    background-color: #2980b9;
    padding: 8px 12px;
    border-radius: 6px;
    gap: 10px;
    width: 100%;
    color: white;
    max-width: 100%;
}
</style>