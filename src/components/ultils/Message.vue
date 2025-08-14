<template>
    <div class="static-overlay">
        <div ref="msgBox" class="static-message-box message">
            <i ref="icon" class="fa-solid fa-envelope notif-icon message"></i>
            <p class="message-text">{{ msg }}</p>
            <button ref="msgBtn" class="confirm-button message" @click="Confirm">Confirm</button>
        </div>
    </div>
</template>

<script setup>
    import { ref } from 'vue';

    //expose
    defineExpose({setMessage});
    //var
    const msg = ref('');
    const mtype = ref('');
    const icon = ref('');
    const msgBox = ref(null);
    const msgBtn = ref(null);
    const emits = defineEmits(['msgConfirm']);

    function Confirm() {
        emits('msgConfirm');
    }
    function setMessage(m, t) {
        msg.value = m;
        mtype.value = t;
        setType();
    }

    function setType() {
        msgBox.value.classList.remove('message');
        msgBox.value.classList.remove('error');
        msgBox.value.classList.remove('warning');
        msgBtn.value.classList.remove('message');
        msgBtn.value.classList.remove('error');
        msgBtn.value.classList.remove('warning');
        icon.value.classList.remove('message');
        icon.value.classList.remove('error');
        icon.value.classList.remove('warning');
        icon.value.classList.remove('fa-envelope');
        icon.value.classList.remove('fa-circle-exclamation');
        icon.value.classList.remove('fa-triangle-exclamation');
        switch (mtype.value) {
            case 'message':
                msgBox.value.classList.add('message');
                msgBtn.value.classList.add('message');
                icon.value.classList.add('message');
                icon.value.classList.add('fa-envelope');
                break;
            case 'warning':
                msgBox.value.classList.add('warning');
                msgBtn.value.classList.add('warning');
                icon.value.classList.add('warning');
                icon.value.classList.add('fa-triangle-exclamation');
                break;
            case 'error':
                msgBox.value.classList.add('error');
                msgBtn.value.classList.add('error');
                icon.value.classList.add('error');
                icon.value.classList.add('fa-circle-exclamation');
                break;
        }
    }
</script>

<style scoped>
.notif-icon {
    font-size:xx-large;
    color: #3498db;
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
    background-color: #3498db;
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

.static-message-box.message {
    border-top: 4px solid #3498db; /* message = xanh dương */
}

.confirm-button.message {
    background-color: #3498db;
}
.confirm-button.message:hover {
    background-color: #1f6391;
}

.static-message-box.error {
    border-top: 4px solid #db3434; /* message = xanh dương */
}

.confirm-button.error {
    background-color: #db3434;
}
.confirm-button.error:hover {
    background-color: #c12e2e;
}

.static-message-box.warning {
    border-top: 4px solid #ddd921; /* message = xanh dương */
}

.confirm-button.warning {
    background-color: #ddd921;
}
.confirm-button.warning:hover {
    background-color: #c6c31e;
}

.notif-icon.message {
    color: #3498db;
}

.notif-icon.error {
    color: #db3434;
}

.notif-icon.warning {
    color: #ddd921;
}
</style>