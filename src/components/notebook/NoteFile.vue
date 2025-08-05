<template>
    <div class="file-button-wrapper">
        <div class="file-button" @click.left.self="OpenFile">
            <i class="fa-solid fa-file"></i>
            <div class="file-name" type="text" >{{ fileName }}</div>
            <div class="side-button" @mouseover="ActiveOptionsMenu" @mouseleave="UnactiveOptionsMenu">
                <i class="fa-solid fa-bars"></i>
                <ul class="file-options-menu" v-show="enableOptionsMenu">
                    <li class="menu-item" @click.left.stop="ChangeFolder"><i class="fa-solid fa-folder-tree"></i> Change Folder</li>
                    <li class="menu-item" @click.left="RemoveFile"><i class="fa-solid fa-trash"></i> Remove</li>
                </ul>
            </div>
        </div>
    </div>
    
</template>

<script setup>
    //import
    import { ref } from 'vue';
    
    //var                                                                                           
    const enableOptionsMenu = ref(false);
    const props = defineProps({
        fileName: {
            Type: String,
            default: 'New File',
        },
        id: {
            Type: String,
            required: true,
        }

    });
    const emits = defineEmits(['fileChangeFolder', 'removeFile', 'openFile']);

    //func
    function ActiveOptionsMenu() {
        enableOptionsMenu.value = true;
    }

    function UnactiveOptionsMenu() {
        enableOptionsMenu.value = false;
    }

    function ChangeFolder() {
        emits('fileChangeFolder', props.id);
    }

    function RemoveFile() {
        emits('removeFile', props.id);
    }

    function OpenFile() {
        emits('openFile', props.id);
    }
</script>

<style scoped>
.file-button-wrapper {
    position: relative;
}

.file-button-wrapper:hover .file-button {
    background-color: #9bb2c4;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.15);
}

.file-button {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    background-color: #acc6d8;
    padding: 8px 12px;
    border-radius: 6px;
    gap: 10px;
    color: white;
    max-width: 100%;
    cursor: pointer;
    overflow-wrap: normal;
    transition: background-color 0.2s ease, box-shadow 0.2s ease;
}

.file-button:hover .side-button {
    display: flex;
}

.file-name {
    font-size: 100%;
    font-weight: 500;
    border: none;
    outline: none;
    background: transparent;
    color: white;
    pointer-events: none;
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

.file-options-menu {
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

.file-options-menu .menu-item {
    padding: 10px 16px;
    cursor: pointer;
    font-size: auto;
    transition: background-color 0.2s ease;
    white-space: nowrap;
}

.file-options-menu .menu-item:hover {
    background-color: #f1f2f6;
}
</style>