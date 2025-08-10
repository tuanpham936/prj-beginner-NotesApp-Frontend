<template>
    <div class="modal-overlay">
        <div class="modal-popup">
            <div class="modal-header">
                <span class="modal-title">Choose Folder</span>
                <button class="modal-close-button" @click="ChooseFolder('')"><i class="fa-solid fa-x"></i></button>
            </div>

            <ul class="modal-item-list">
                <li class="modal-item" v-for="folder in folders" @click="ChooseFolder(folder.id)">
                    <i class="fa-solid fa-folder"></i>
                    <span>{{ folder.name }}</span>
                </li>
            </ul>
        </div>
    </div>
</template>

<script setup>
    const props = defineProps({
        folders: {
            Type: Array
        }
    });

    const emits = defineEmits(['chooseFolder', 'closeModal']);

    function ChooseFolder(id) {
        if (id === '') {
            emits('closeModal');
        }
        else {
            emits('chooseFolder', id);
        }
    }
</script>

<style scoped>
/* Lớp phủ */
.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;

    background-color: rgba(0, 0, 0, 0.4);
    display: flex;
    align-items: center;
    justify-content: center;

    z-index: 30;
}

/* Bảng nổi */
.modal-popup {
    background-color: white;
    padding: 20px;
    border-radius: 8px;
    width: 300px;
    box-shadow: 0 8px 16px rgba(0,0,0,0.2);
    position: relative;
}

/* Header bảng */
.modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;
}

.modal-title {
    font-weight: bold;
    font-size: 16px;
    color: #2d3436;
}

.modal-close-button {
    background: none;
    border: none;
    font-size: 20px;
    color: #636e72;
    cursor: pointer;
    padding: 0 4px;
}

.modal-close-button:hover {
    color: #d63031;
}

/* Danh sách các mục */
.modal-item-list {
    list-style: none;
    padding: 5px;
    margin: 0;
    display: flex;
    flex-direction: column;
    gap: 10px;
    border: #2d3436 solid 2px;
    border-radius: 15px;
    overflow-y: auto;
    height: calc(60vh - 80px);
}

.modal-item {
    padding: 10px 14px;
    border-radius: 4px;
    cursor: pointer;
    background-color: #f1f2f6;
    transition: background-color 0.2s ease;
}

.modal-item:hover {
    background-color: #dfe6e9;
}

</style>