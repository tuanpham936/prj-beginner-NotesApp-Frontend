<template>
    <div class="toolbar">
        <div>
            <button class="tooltip"><i class="fa-solid fa-rotate-left"></i><span class="tooltiptext">Undo(Ctrl + Z)</span></button>
            <button class="tooltip"><i class="fa-solid fa-rotate-right"></i><span class="tooltiptext">Redo(Ctrl + Y)</span></button>
            <button class="tooltip"><i class="fa-solid fa-floppy-disk"></i><span class="tooltiptext">Save(Ctrl + S)</span></button>
        </div>
        <div>
            <button class="tooltip"><i class="fa-solid fa-scissors"></i><span class="tooltiptext">Cut(Ctrl + X)</span></button>
            <button class="tooltip"><i class="fa-solid fa-copy"></i><span class="tooltiptext">Copy(Ctrl + C)</span></button>
            <button class="tooltip"><i class="fa-solid fa-paste"></i><span class="tooltiptext">Paste(Ctrl + V)</span></button>
        </div>
        <div>
            <select id="fontSelector" onchange="changeFont(this.value)">
                <option disabled selected>Font Family</option>
                <option value="Arial">Arial</option>
                <option value="Times New Roman">Times New Roman</option>
                <option value="Courier New">Courier New</option>
                <option value="Georgia">Georgia</option>
                <option value="Verdana">Verdana</option>
            </select>
        </div>
        <div>
            <select id="fontSizeSelector" onchange="changeFontSize(this.value)">
                <option disabled selected>Font Size</option>
                <option value="1">Rất nhỏ</option>
                <option value="2">Nhỏ</option>
                <option value="3">Trung bình</option>
                <option value="4">Hơi lớn</option>
                <option value="5">Lớn</option>
                <option value="6">Rất lớn</option>
                <option value="7">Cực lớn</option>
            </select>
        </div>
        <div>
            <button class="tooltip"><i class="fa-solid fa-bold"></i><span class="tooltiptext">Bold(Ctrl + B)</span></button>
            <button class="tooltip"><i class="fa-solid fa-italic"></i><span class="tooltiptext">Italic(Ctrl + I)</span></button>
            <button class="tooltip"><i class="fa-solid fa-underline"></i><span class="tooltiptext">Underline(Ctrl + U)</span></button>
        </div>
        <div>
            <div class="color tooltip">
                <i class="fa-solid fa-font"></i>
                <input type="color" onchange="changeColor(this.value)" title="Chọn màu" />
                <span class="tooltiptext">Text Color</span>
            </div>
            <div class="color tooltip">
                <i class="fa-solid fa-highlighter"></i>
                <input type="color" onchange="changeColor(this.value)" title="Chọn màu" />
                <span class="tooltiptext">Text Highlight Color</span>
            </div>
        </div>
        <div>
            <button class="tooltip"><i class="fa-solid fa-list-ul"></i><span class="tooltiptext">Unorder List</span></button>
            <button class="tooltip"><i class="fa-solid fa-list-ol"></i><span class="tooltiptext">Order List</span></button>
            <button class="tooltip"><i class="fa-solid fa-list-check"></i><span class="tooltiptext">Check List</span></button>
        </div>
        <div>
            <button class="tooltip"><i class="fa-solid fa-align-left"></i><span class="tooltiptext">Align Left</span></button>
            <button class="tooltip"><i class="fa-solid fa-align-right"></i><span class="tooltiptext">Align Right</span></button>
        </div>
        <div>
            <button class="tooltip"><i class="fa-solid fa-link"></i><span class="tooltiptext">Attach link(Ctrl + A)</span></button>
            <button class="tooltip"><i class="fa-solid fa-comment"></i><span class="tooltiptext">Attach comment(Ctrl + Q)</span></button>
        </div>
    </div>
</template>

<script setup>

</script>

<style scoped>
.toolbar {
    display: flex;
    width: 100%;
    justify-content:stretch;
}

.toolbar > div {
    width: 100%;
    padding: 10px 0;
    border-right: #3babf7 solid 2px;
    border-left: #3babf7 solid 2px;
    display:flex;
    justify-content: space-evenly;
}

.toolbar select, .toolbar button, .color {
    border: none;
    background: white;
    padding: 6px 10px;
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
}

.toolbar button:hover {
    background-color: #b2bec3;
}
</style>