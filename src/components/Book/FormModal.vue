<template>
    <b-modal id="book-modal" v-model="visible" @hide="close" :title="modalTitle" centered hide-footer>
        <b-form @submit.prevent="submitForm">
            <b-form-group label="Title" label-for="book-title">
                <b-form-input id="book-title" v-model="title" placeholder="Enter title"></b-form-input>
                <div v-if="titleErr" class="text-danger">{{ titleErr }}</div>
            </b-form-group>

            <b-form-group label="Price" label-for="book-price">
                <b-form-input id="book-price" v-model="price" placeholder="Enter price" type="number"></b-form-input>
                <div v-if="priceErr" class="text-danger">{{ priceErr }}</div>
            </b-form-group>

            <b-form-group label="Image URL" label-for="book-image-url">
                <b-form-input id="book-image-url" v-model="imageUrl" placeholder="Enter image URL"></b-form-input>
            </b-form-group>

            <b-form-group label="Genre" label-for="book-genre">
                <b-form-input id="book-genre" v-model="genre" placeholder="Enter genre"></b-form-input>
                <div v-if="genreErr" class="text-danger">{{ genreErr }}</div>
            </b-form-group>

            <b-form-group label="Author" label-for="book-author">
                <b-form-input id="book-author" v-model="author" placeholder="Enter author"></b-form-input>
                <div v-if="authorErr" class="text-danger">{{ authorErr }}</div>
            </b-form-group>

            <b-form-group label="Year Of Publication" label-for="book-year-of-publication">
                <b-form-input id="book-year-of-publication" v-model="yearOfPublication"
                    placeholder="Enter year of publication" type="number"></b-form-input>
                <div v-if="yearOfPublicationErr" class="text-danger">{{ yearOfPublicationErr }}</div>
            </b-form-group>

            <b-button type="submit" variant="primary">{{ buttonText }}</b-button>
        </b-form>
    </b-modal>
</template>

<script>
import { computed, defineComponent, ref, watch, toRefs } from 'vue';
import { useForm, useField } from 'vee-validate';
import * as yup from 'yup';

export default defineComponent({
    name: 'BookModal',
    props: {
        visible: {
            type: Boolean,
            required: true,
        },
        book: {
            type: Object,
        },
    },
    setup(props, { emit }) {
        const { visible, book } = toRefs(props);
        const formData = ref(createEmptyFormData());

        const schema = defineValidationSchema();
        const { handleSubmit, resetForm } = useForm({
            validationSchema: schema,
        });

        const { value: title, errorMessage: titleErr } = useField('title');
        const { value: price, errorMessage: priceErr } = useField('price');
        const { value: imageUrl } = useField('image_url');
        const { value: genre, errorMessage: genreErr } = useField('genre');
        const { value: author, errorMessage: authorErr } = useField('author');
        const { value: yearOfPublication, errorMessage: yearOfPublicationErr } = useField('year_of_publication');

        watch(book, updateFormDataOnBookChange, { immediate: true });

        function createEmptyFormData() {
            return {
                id: '',
                title: '',
                price: 0,
                image_url: '',
                genre: '',
                author: '',
                year_of_publication: 0,
            };
        }

        function defineValidationSchema() {
            return yup.object({
                title: yup.string().required('Title is required'),
                price: yup.number().positive('Price must be positive').required('Price is required'),
                image_url: yup.string(),
                genre: yup.string().required('Genre is required'),
                author: yup.string().required('Author is required'),
                year_of_publication: yup.number().positive('Year must be positive').integer('Year must be an integer').required('Year of publication is required'),
            });
        }

        function updateFormFields(coming) {
            title.value = coming ? coming.title : '';
            price.value = coming ? coming.price : 0;
            imageUrl.value = coming ? coming.image_url : '';
            genre.value = coming ? coming.genre : '';
            author.value = coming ? coming.author : '';
            yearOfPublication.value = coming ? coming.year_of_publication : 0;
        }

        function updateFormDataOnBookChange(coming) {
            formData.value = coming ? { ...coming } : createEmptyFormData();
            if (coming) {
                updateFormFields(coming);
            }
        }

        const close = () => {
            emit('close');
            formData.value = createEmptyFormData();
            resetForm();
        };

        const submitForm = handleSubmit((values) => {
            formData.value = { ...formData.value, ...values };
            emit(formData.value.id ? 'edit' : 'create', formData.value);
            close();
        });

        const modalTitle = computed(() => (formData.value.id ? 'Edit Book' : 'Add Book'));
        const buttonText = computed(() => (formData.value.id ? 'Update' : 'Create'));

        return {
            visible,
            close,
            submitForm,
            modalTitle,
            buttonText,
            formData,
            title,
            imageUrl,
            price,
            genre,
            author,
            yearOfPublication,
            titleErr,
            priceErr,
            genreErr,
            authorErr,
            yearOfPublicationErr,
        };
    },
});
</script>
