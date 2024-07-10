<template>
    <b-modal id="error-notification-modal" v-model="visible" @hide="close" title="An Error Occurred" centered
        hide-footer :no-close-on-backdrop="true" :no-close-on-esc="true">
        <template #modal-header="{ close }">
            <h2 class="text-danger">Error</h2>
            <b-button size="sm" variant="outline-danger" @click="close">Close</b-button>
        </template>
        <div v-if="error" class="modal-body-content">
            <p class="text-danger">{{ error }}</p>
        </div>
    </b-modal>
</template>

<script>
import { toRefs, defineComponent } from 'vue';
import { BModal, BButton } from 'bootstrap-vue-3';

export default defineComponent({
    components: {
        BModal,
        BButton
    },
    props: {
        visible: {
            type: Boolean,
            required: true
        },
        error: {
            type: String,
            required: true
        }
    },
    setup(props, { emit }) {
        const { visible, error } = toRefs(props);

        const close = () => {
            emit('close');
        };

        return { visible, error, close };
    }
});
</script>

<style scoped>
.modal-body-content {
    padding: 20px;
    background-color: #f8d7da;
    border-color: #f5c6cb;
    color: #721c24;
    border-radius: 0.25rem;
}
</style>