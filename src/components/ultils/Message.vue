<template>
    <div class="static-overlay">
        <div class="static-message-box">
            <i class="fa-solid fa-envelope notif-icon"></i>
            <p class="message-text">{{ msg }}</p>
            <button class="confirm-button" @click="Confirm">Confirm</button>
        </div>
    </div>
</template>

<script setup>
    //var
    const props = defineProps({
        msg: {
            Type: String,
            required: true,
        },
        type: {
            Type: String,
        }
    });
    const emits = defineEmits(['msgConfirm']);

    function Confirm() {
        emits('msgConfirm');
    }
</script>

<style scoped>
.notif-icon {
    font-size:xx-large;
    color: #2980b9;
}

.static-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    backdrop-filter: blur(4px);
    background-color: rgba(0, 0, 0, 0.3);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 9999;
}

/* Hộp thông báo */
.static-message-box {
    background-color: white;
    padding: 24px;
    border-radius: 8px;
    max-width: 320px;
    width: 90%;
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);
    text-align: center;
    border-top: 4px solid #3498db; /* message = xanh dương */
}

/* Nội dung chữ */
.message-text {
    font-size: 16px;
    margin-bottom: 20px;
    color: #2d3436;
}

/* Nút xác nhận */
.confirm-button {
    padding: 8px 16px;
    background-color: #2980b9;
    border: none;
    border-radius: 4px;
    color: white;
    font-size: 14px;
    cursor: pointer;
    transition: background-color 0.2s ease;
}

.confirm-button:hover {
    background-color: #1f6391;
}


</style>