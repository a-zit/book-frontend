<template>
    <b-modal id="confirm-modal" v-model="visible" @hide="close" title="Confirm" centered hide-footer>
        <div class="">
            <p>{{ message }}</p>
            <div class="button-action">
                <b-button variant="btn btn-danger btn-sm delete-button" @click="confirm">Confirm</b-button>
                <b-button variant="btn btn-secondary btn-sm cancel-button" @click="close">Cancel</b-button>
            </div>
        </div>
    </b-modal>
</template>

<script>
import { defineComponent, toRefs } from 'vue';

export default defineComponent({
    name: 'ConfirmModal',
    props: {
        visible: {
            type: Boolean,
            required: true,
        },
        message: {
            type: Object,
            required: true,
        },
    },

    setup(props, { emit }) {
        const { visible, message } = toRefs(props);

        const close = () => {
            emit('close');
        };

        const confirm = () => {
            emit('confirm');
            close();
        };

        return {
            visible,
            message,
            confirm,
            close,
        };
    },
});
</script>
<style scoped>
.b-modal .modal-header h5 {
    font-size: 1.25rem;
    color: #333;
}

@media (max-width: 768px) {
    .modal-body-content {
        padding: 15px;
    }
}

.btn {
    padding: 5px 15px;
    /* Adjusted padding */
    font-size: 14px;
    /* Adjusted font size */
    border-radius: 5px;
    /* Rounded corners for buttons */
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    /* Added shadow for depth */
}

.delete-button,
.cancel-button {
    transition: transform 0.2s;
    /* Smooth transition for hover effect */
}

.delete-button:hover,
.cancel-button:hover {
    transform: scale(1.05);
    /* Slightly enlarge buttons on hover */
}

.delete-button {
    background-color: #dc3545;
    /* Kept red background for delete button */
}

.button-action {
    display: flex;
    justify-content: center;
    gap: 10px;
}
</style>