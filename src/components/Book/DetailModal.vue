<template>
    <b-modal id="detail-modal" v-model="visible" @hide="close" title="Detail Information" centered hide-footer>
        <div v-if="detail" class="modal-body-content">
            <b-row>
                <b-col md="4" class="text-center">
                    <b-img :src="detail.image_url" alt="book image" fluid thumbnail class="book-image mb-3"></b-img>
                </b-col>
                <b-col md="8">
                    <b-list-group flush>
                        <b-list-group-item>
                            <h5 class="mb-1">{{ detail.title }}</h5>
                        </b-list-group-item>
                        <b-list-group-item>Price: {{ detail.price }}$</b-list-group-item>
                        <b-list-group-item>Author: {{ detail.author }}</b-list-group-item>
                        <b-list-group-item>Genre: {{ detail.genre }}</b-list-group-item>
                        <b-list-group-item>Year Of Publication: {{ detail.year_of_publication }}</b-list-group-item>
                    </b-list-group>
                </b-col>
            </b-row>
        </div>
        <template #modal-footer>
            <b-button variant="secondary" @click="close">Close</b-button>
        </template>
    </b-modal>
</template>

<script>
import { defineComponent, toRefs } from 'vue';
import { BModal, BButton, BImg, BRow, BCol, BListGroup, BListGroupItem } from 'bootstrap-vue-3';

export default defineComponent({
    name: 'DetailModal',
    components: {
        BModal,
        BButton,
        BImg,
        BRow,
        BCol,
        BListGroup,
        BListGroupItem,
    },
    props: {
        visible: {
            type: Boolean,
            required: true,
        },
        detail: {
            type: Object,
            required: true,
        },
    },

    setup(props, { emit }) {
        const { visible, detail } = toRefs(props);

        const close = () => {
            emit('close');
        };

        return {
            visible,
            detail,
            close,
        };
    },
});
</script>

<style scoped>
.modal-body-content {
    background-color: #f8f9fa;
    border-radius: 8px;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.b-modal .modal-header h5 {
    font-size: 1.25rem;
    color: #333;
}

@media (max-width: 768px) {
    .modal-body-content {
        padding: 15px;
    }
}

.book-image:hover {
    cursor: pointer;
    transform: scale(1.05);
    transition: transform .2s ease-in-out;
}

.book-image:focus {
    outline: 2px solid #0056b3;
}
</style>