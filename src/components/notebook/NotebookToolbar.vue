<template>
    <div class="toolbar">
        <div>
            <button class="tooltip" @click="exec('folderhier')"><i class="fa-solid fa-folder-tree"></i><span class="tooltiptext">Folder Hierarchy<br>(Ctrl + E)</span></button>
            <div><button class="tooltip" @click="exec('newNote')"><i class="fa-solid fa-file"></i><span class="tooltiptext">New Note<br>(Ctrl + N)</span></button></div>
        </div>
        <div>
            <button class="tooltip" @click="exec('undo')"><i class="fa-solid fa-rotate-left"></i><span class="tooltiptext">Undo<br>(Ctrl + Z)</span></button>
            <button class="tooltip" @click="exec('redo')"><i class="fa-solid fa-rotate-right"></i><span class="tooltiptext">Redo<br>(Ctrl + Y)</span></button>
            <div class="btn-badge"><button class="tooltip" @click="exec('save')"><i class="fa-solid fa-floppy-disk"></i><span class="badge-dot" v-show="!saveStatus"></span><span class="tooltiptext">Save<br>(Ctrl + S)</span></button></div>
        </div>
        <div>
            <button class="tooltip" @click="exec('cut')"><i class="fa-solid fa-scissors"></i><span class="tooltiptext">Cut<br>(Ctrl + X)</span></button>
            <button class="tooltip" @click="exec('copy')"><i class="fa-solid fa-copy"></i><span class="tooltiptext">Copy<br>(Ctrl + C)</span></button>
            <!-- <button class="tooltip" @click="exec('paste')"><i class="fa-solid fa-paste"></i><span class="tooltiptext">Paste<br>(Ctrl + V)</span></button> -->
        </div>
        <div>
            <select id="fontSelector" @change="exec(this.value)">
                <option disabled selected>Font Family</option>
                <option value="Arial">Arial</option>
                <option value="Times New Roman">Times New Roman</option>
                <option value="Courier New">Courier New</option>
                <option value="Georgia">Georgia</option>
                <option value="Verdana">Verdana</option>
            </select>
        </div>
        <div>
            <select id="fontSizeSelector" @change="exec('fontSize', $event.target.value)">
                <option disabled selected>Font Size</option>
                <option value="1">Super Small</option>
                <option value="2">Small</option>
                <option value="3">Medium</option>
                <option value="5">Large</option>
                <option value="6">Super Large</option>
                <option value="7">Super Super Large</option>
            </select>
        </div>
        <div>
            <button class="tooltip" @click="exec('bold')"><i class="fa-solid fa-bold"></i><span class="tooltiptext">Bold<br>(Ctrl + B)</span></button>
            <button class="tooltip" @click="exec('italic')"><i class="fa-solid fa-italic"></i><span class="tooltiptext">Italic<br>(Ctrl + I)</span></button>
            <button class="tooltip" @click="exec('underline')"><i class="fa-solid fa-underline"></i><span class="tooltiptext">Underline<br>(Ctrl + U)</span></button>
        </div>
        <div>
            <button class="color tooltip" @click="changeTextColor">
                <i class="fa-solid fa-font"></i>
                <span class="tooltiptext">Text Color<br>(Ctrl + G)</span>
            </button>
                <input ref="textColorInput" type="color" @input="changeTextColor" title="Chọn màu" value="#f00000"/>
            <button class="color tooltip" @click="highlight">
                <i class="fa-solid fa-highlighter"></i>
                <span class="tooltiptext">Text Highlight Color<br>(Ctrl + H)</span>
            </button>
                <input ref="highlightInput" type="color" @input="highlight" title="Chọn màu" value="#ffff00"/>
        </div>
        <div>
            <button class="tooltip" @click="exec('unorderList')"><i class="fa-solid fa-list-ul"></i><span class="tooltiptext">Unorder List</span></button>
            <button class="tooltip" @click="exec('orderList')"><i class="fa-solid fa-list-ol"></i><span class="tooltiptext">Order List</span></button>
            <!-- <button class="tooltip" @click="exec('unorderList')"><i class="fa-solid fa-list-check"></i><span class="tooltiptext">Check List</span></button> -->
        </div>
        <div>
            <button class="tooltip" @click="exec('justifyLeft')"><i class="fa-solid fa-align-left"></i><span class="tooltiptext">Align Left</span></button>
            <button class="tooltip" @click="exec('justifyRight')"><i class="fa-solid fa-align-right"></i><span class="tooltiptext">Align Right</span></button>
        </div>
        <div>
            <button class="tooltip" @click="exec('attachLink')"><i class="fa-solid fa-link"></i><span class="tooltiptext">Attach link<br>(Ctrl + Q)</span></button>
            <button class="tooltip" @click="exec('attachComment')"><i class="fa-solid fa-comment"></i><span class="tooltiptext">Attach comment<br>(Ctrl + W)</span></button>
            <button class="tooltip" @click="exec('createTable')"><i class="fa-solid fa-table"></i><span class="tooltiptext">Create Table</span></button>
        </div>
    </div>
</template>

<script setup>
    //import
    import { ref } from 'vue';
    import { onMounted } from 'vue';
    //var 
    const emits = defineEmits(['execCommand']);
    const props = defineProps({
        saveStatus: {
            Type: Boolean,
            default: true,
            required: true,
        }
    })
    //func
    onMounted(() => {
        window.addEventListener('keydown', handleKeydown)
    })

    function handleKeydown(e) {
        if (e.ctrlKey && e.key === 's') {
            e.preventDefault()
            exec('save')
        }
        if (e.ctrlKey && e.key === 'g') {
            e.preventDefault();
            changeTextColor();
        }
        if (e.ctrlKey && e.key === 'h') {
            e.preventDefault();
            highlight();
        }
        if (e.ctrlKey && e.key === 'q') {
            e.preventDefault();
            exec('attachLink');
        }
        if (e.ctrlKey && e.key === 'e') {
            e.preventDefault();
            exec('folderhier');
        }
        if (e.ctrlKey && e.key === 'z') {
            e.preventDefault();
            exec('undo');
        }
        if (e.ctrlKey && e.key === 'y') {
            e.preventDefault();
            exec('redo');
        }
        if (e.ctrlKey && e.key === 'n') {
            e.preventDefault();
            exec('newNote');
        }
    }

    function exec(cmd, value) {
        emits('execCommand', cmd, value);
    }

    const textColorInput = ref(null);
    function changeTextColor() {
        emits('execCommand', 'foreColor', textColorInput.value.value);
    }

    const highlightInput = ref(null);
    function highlight() {
        emits('execCommand', 'backColor', highlightInput.value.value);
    }
</script>

<style scoped>
.toolbar {
    display: flex;
    width: 100%;
    flex-wrap: wrap;
    justify-content: center;
}

.toolbar > div {
    width: fit-content;
    padding: 10px;
    border: #318fce solid 2px;
    display:flex;

}

.toolbar select, .toolbar button, .color {
    border: none;
    background: white;
    padding: 6px 10px;
    margin: 0 5px;
    font-size: 14px;
    cursor: pointer;
    border-radius: 4px;
    height: 100%;
    color: #000;
    transition: background-color 0.2s ease;
}

.toolbar input[type="color"] {
    border: none;
    background: none;
    cursor: pointer;
    height: 100%;
    width: 20px;
    margin: 0;
}

.toolbar button:hover, .color:hover {
    background-color: #b2bec3;
}

.btn-badge {
    position: relative; /* Để chứa dấu đỏ */
    display: inline-block;
}

.badge-dot {
    position: absolute;
    top: -5%;
    right: -5%;
    width: 30%;
    height: 30%;
    background: red;
    border-radius: 50%;
}
</style>