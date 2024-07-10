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
                <b-form-input id="book-image-url" v-model="imageUrl"
                    placeholder="Enter image URL"></b-form-input>
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

        const formData = ref({
            id: '',
            title: '',
            price: 0,
            image_url: '',
            genre: '',
            author: '',
            year_of_publication: 0,
        });

        // Define validation schema
        const schema = yup.object({
            title: yup.string().required('Title is required'),
            price: yup.number().positive('Price must be positive').required('Price is required'),
            image_url: yup.string(),
            genre: yup.string().required('Genre is required'),
            author: yup.string().required('Author is required'),
            year_of_publication: yup.number().positive('Year must be positive').integer('Year must be an integer').required('Year of publication is required'),
        });

        // Use useForm hook to create a form context
        const { handleSubmit, resetForm } = useForm({
            validationSchema: schema,
        });

        // Use useField hook to bind fields to the schema
        const { value: title, errorMessage: titleErr } = useField('title');
        const { value: price, errorMessage: priceErr } = useField('price');
        const { value: imageUrl } = useField('image_url');
        const { value: genre, errorMessage: genreErr } = useField('genre');
        const { value: author, errorMessage: authorErr } = useField('author');
        const { value: yearOfPublication, errorMessage: yearOfPublicationErr } = useField('year_of_publication');

        watch(
            () => book.value,
            (coming) => {
                if (coming) {
                    console.log("coming:",coming);
                    console.log("image_url:",coming.image_url);
                    formData.value = { ...coming };
                    // Set the field values
                    title.value = coming.title;
                    price.value = coming.price;
                    imageUrl.value = coming.image_url;
                    genre.value = coming.genre;
                    author.value = coming.author;
                    yearOfPublication.value = coming.year_of_publication;
                    console.log(imageUrl.value);
                } else {
                    formData.value = {
                        id: '',
                        title: '',
                        price: 0,
                        image_url: '',
                        genre: '',
                        author: '',
                        year_of_publication: 0,
                    };
                }
            },
            { immediate: true }
        );

        const resetFormData = () => {
            formData.value = {
                id: '',
                title: '',
                price: 0,
                image_url: '',
                genre: '',
                author: '',
                year_of_publication: 0,
            };
        };

        const close = () => {
            emit('close');
            resetFormData();
            resetForm();
        };

        const submitForm = handleSubmit((values) => {
            // Update formData with values from validated fields
            formData.value = {
                id: formData.value.id, // Keep the existing id if any
                title: title.value,
                price: price.value,
                image_url: imageUrl.value,
                genre: genre.value,
                author: author.value,
                year_of_publication: yearOfPublication.value,
            };

            // Your existing submit logic
            alert('Form submitted successfully');
            console.log("test:",formData.value);
            if (formData.value.id) {
                emit('edit', formData.value);
            } else {
                emit('create', formData.value);
            }
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
