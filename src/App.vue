<template>
    <div id="app" class="book-list-container">
        <div v-if="loading" class="loading-text">{{ loadingText }}</div>
        <div v-else-if="error" class="error">{{ error }}</div>
        <div v-else>
            <div class="create-book-button-container">
                <button class="btn btn-success btn-sm create-book-button" @click="showCreateModal()">Create
                    Book</button>
            </div>
            <div class="book-list-header row">
                <div class="col-2 header-item image-header">Image</div>
                <div class="col-2 header-item title-header">Title</div>
                <div class="col-2 header-item title-header">Genre</div>
                <div class="col-2 header-item title-header">Author</div>
                <div class="col-2 header-item title-header">Year of Publication</div>
                <div class="col-2 header-item actions-header">Actions</div>
            </div>
            <ul class="book-list">
                <li v-for="book in books" :key="book.id" class="book-item row">
                    <img :src="book.image_url" alt="book image" class="col-2 book-image" @click="showDetails(book)" />
                    <div class="col-2 book-title">{{ book.title }}</div>
                    <div class="col-2 book-genre">{{ book.genre }}</div>
                    <div class="col-2 book-author">{{ book.author }}</div>
                    <div class="col-2 book-year-of-publication">{{ book.year_of_publication }}</div>
                    <div class="col-2 book-actions">
                        <button class="btn btn-danger btn-sm delete-button"
                            @click="showConfirmModal(book)">Delete</button>
                        <button class="btn btn-primary btn-sm edit-button" @click="showEditModal(book)">Edit</button>
                    </div>
                </li>
            </ul>
        </div>

        <DetailModal :visible="isModalVisible" :detail="selectedBook" @close="isModalVisible = false" />
        <FormModal :visible="isFormModalVisible" :book="selectedBook" @close="isFormModalVisible = false"
            @create="createBook" @edit="editBook" />
        <ErrorModal :visible="isErrorModalVisible" :error="errorMessage" @close="isErrorModalVisible = false" />
        <ConfirmModal :visible="isConfirmModalVisible" message="Are you sure you want to delete this book?"
            @confirm="deleteBook(selectedBookId)" @close="isConfirmModalVisible = false" />
    </div>
</template>

<script>
import axios from 'axios';
import { ref, onMounted, defineComponent } from 'vue';
import DetailModal from './components/Book/DetailModal.vue';
import FormModal from './components/Book/FormModal.vue';
import ErrorModal from './components/ErrorModal.vue'
import ConfirmModal from './components/ConfirmModal.vue';

export default defineComponent({
    name: 'App',
    components: {
        DetailModal,
        FormModal,
        ErrorModal,
        ConfirmModal,
    },

    setup() {
        const books = ref([]);
        const loading = ref(true);
        const loadingText = ref('Loading');
        const error = ref(null);
        const isModalVisible = ref(false);
        const selectedBook = ref(null);
        const isFormModalVisible = ref(false);
        const isSubmitting = ref(false);
        const isErrorModalVisible = ref(false);
        const errorMessage = ref('');
        const isConfirmModalVisible = ref(false);
        const selectedBookId = ref(null);

        const fetchData = async () => {
            try {
                updateLoadingText();
                const response = await axios.get('http://localhost:9000/api/books/');
                books.value = response.data;

                setTimeout(() => {
                    loading.value = false;
                }, 1000);
            } catch (error) {
                handleError(error)
            } finally {
                loading.value = true
            }
        };

        const handleError = (err) => {
            if (err.response) {
                error.value = `Error: ${err.response.status} - ${err.response.statusText}`;
            } else if (err.request) {
                error.value = 'Network Error: No response received from the server';
            } else {
                error.value = `Error: ${err.message}`;
            }
            loading.value = false;
        };

        const showDetails = (book) => {
            selectedBook.value = book;
            isModalVisible.value = true;
        }

        const deleteBook = async (id) => {
            try {
                await axios.delete(`http://localhost:9000/api/books/${id}`);
                fetchData();
                updateLoadingText();
            } catch (error) {
                console.error('Error deleting book:', error);
            }
        }

        const showCreateModal = () => {
            isFormModalVisible.value = true;
        }

        const createBook = async (book) => {
            try {
                if (book instanceof SubmitEvent) {
                    book.preventDefault();
                    return;
                }

                if (isSubmitting.value) return;
                isSubmitting.value = true;;
                await axios.post('http://localhost:9000/api/books/', JSON.stringify({
                    ...book,
                    price: parseFloat(book.price),
                    year_of_publication: parseInt(book.year_of_publication),
                }), {
                    headers: {
                        'Content-Type': 'application/json',
                    },
                });
                fetchData();
                updateLoadingText();
            } catch (error) {
                errorMessage.value = error.message;
                isErrorModalVisible.value = true;
            } finally {
                isSubmitting.value = false;
            }
        }

        const showEditModal = (book) => {
            isFormModalVisible.value = true;
            selectedBook.value = book;
        }

        const editBook = async (book) => {
            try {
                if (book instanceof SubmitEvent) {
                    book.preventDefault();
                    return;
                }

                if (isSubmitting.value) return;
                isSubmitting.value = true;
                await axios.put(`http://localhost:9000/api/books/${book.id}`, JSON.stringify({
                    ...book,
                    price: parseFloat(book.price),
                    year_of_publication: parseInt(book.year_of_publication),
                }), {
                    headers: {
                        'Content-Type': 'application/json',
                    },
                });
                fetchData();
                updateLoadingText();
            } catch (error) {
                errorMessage.value = error.message;
                isErrorModalVisible.value = true;
            } finally {
                isSubmitting.value = false;
            }
        }

        const updateLoadingText = () => {
            let count = 0;
            const intervalId = setInterval(() => {
                loadingText.value += '.';
                count++;
                if (count > 3) {
                    loadingText.value = 'Loading';
                    count = 0;
                }
            }, 500);

            setTimeout(() => {
                clearInterval(intervalId);
                loading.value = false;
            }, 5000);
        }

        const showConfirmModal = (book) => {
            isConfirmModalVisible.value = true;
            selectedBookId.value = book.id;
        }

        onMounted(fetchData);

        return {
            loading,
            loadingText,
            books,
            error,
            fetchData,
            deleteBook,
            showDetails,
            isModalVisible,
            selectedBook,
            isFormModalVisible,
            showCreateModal,
            createBook,
            showEditModal,
            editBook,
            isErrorModalVisible,
            errorMessage,
            showConfirmModal,
            isConfirmModalVisible,
            selectedBookId,
        };
    },
})
</script>

<style scoped>
.book-list-container {
    font-family: 'Arial', sans-serif;
    max-width: 800px;
    margin: 20px auto;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    padding: 20px;
    background-color: #fff;
}

.book-list-header {
    display: flex;
    margin-bottom: 20px;
    padding: 15px;
    background-color: #007bff;
    color: #ffffff;
    border-radius: 5px;
}

.header-item {
    flex: 1;
    text-align: center;
    font-weight: bold;
}

.image-header {
    flex: 0 0 100px;
}

.actions-header {
    flex: 0 0 220px;
}

.book-list {
    list-style-type: none;
    padding: 0;
}

.book-item {
    display: flex;
    align-items: center;
    margin-bottom: 20px;
    padding: 10px;
    border-bottom: 1px solid #eee;
    transition: transform 0.2s ease;
}

.book-item:hover {
    transform: translateY(-5px);
    background-color: #f9f9f9;
}

.book-image {
    width: 100px;
    height: 140px;
    object-fit: cover;
    border-radius: 5px;
    margin-right: 20px;
}

.book-title,
.book-genre,
.book-author,
.book-year-of-publication {
    flex: 1;
    text-align: left;
}

.book-actions {
    display: flex;
    justify-content: center;
    gap: 10px;
}

.btn {
    padding: 5px 15px;
    font-size: 14px;
    border-radius: 5px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.delete-button,
.edit-button {
    transition: transform 0.2s;
}

.delete-button:hover,
.edit-button:hover {
    transform: scale(1.05);
}

.delete-button {
    background-color: #dc3545;
}

.edit-button {
    background-color: #28a745;
}

.create-book-button-container {
    text-align: right;
    margin-bottom: 20px;
}
</style>