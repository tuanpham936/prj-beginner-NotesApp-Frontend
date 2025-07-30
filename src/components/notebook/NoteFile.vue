<template>
    <div class="file-item">
        <div class="file-title">
            {{ name }}
            <button class="file-menu-button"><i class="fa-solid fa-bars"></i></button>
        </div>
    </div>
</template>

<script setup>
    const props = defineProps({
        name: {
            Type: String,
            default: 'New File',
            required: false
        }
    })
</script>

<style scoped>
.file-item {
    margin-bottom: 10px;    
}

.file-title {
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
}

.file-title:hover {
    background-color: #b2bec3;
}

.file-menu-button {
    display: none;
    background: none;
    border: none;
    font-size: 18px;
    cursor: pointer;
    padding: 0;
    margin-left: 8px;
    color: #636e72;
}

.file-title:hover .file-menu-button {
    display: inline;
}

</style>