<template>
  <div class="container">
    <aside class="sidebar">
      <h2>Halil Fahriza Story Book</h2>
      <button @click="load">Load Articles</button>
      <p class="welcome-text">Welcome to Story Book Halil Fahriza</p>
    </aside>
    <main class="content">
      <header class="header">
        <h1>Articles Book List Today</h1>
      </header>
      <form @submit.prevent="save" class="article-form">
        <input type="text" v-model="form.title" placeholder="Title" />
        <textarea v-model="form.content" placeholder="Content"></textarea>
        <button type="submit">Save</button>
      </form>
      <div class="articles-list">
        <div v-for="article in articles" :key="article.id" class="article-item">
          <div class="article-header">
            <h3>{{ article.title }}</h3>
            <div class="article-actions">
              <button @click="edit(article)">Edit</button>
              <button @click="confirmDelete(article.id)">Delete</button>
            </div>
          </div>
          <p>{{ article.content }}</p>
        </div>
      </div>
    </main>

    <!-- Delete Confirmation Modal -->
    <div v-if="showDeleteModal" class="modal">
      <div class="modal-content">
        <span class="close" @click="showDeleteModal = false">&times;</span>
        <p>Are you sure you want to delete this article?</p>
        <button @click="remove(deleteArticleId)">Yes</button>
        <button @click="showDeleteModal = false">No</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, reactive, onMounted } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });

    const articles = ref([]);
    const showDeleteModal = ref(false);
    const deleteArticleId = ref(null);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        const url = form.id ? `http://localhost:3000/articles/${form.id}` : 'http://localhost:3000/articles';
        const method = form.id ? "put" : "post";
        const response = await axios[method](url, form);
        console.log('Article saved:', response.data);

        if (method === 'put') {
          const index = articles.value.findIndex(article => article.id === response.data.id);
          if (index !== -1) {
            articles.value[index] = response.data;
          }
        } else {
          articles.value.push(response.data);
        }

        form.id = null;
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        console.log('Article deleted:', id);
        articles.value = articles.value.filter(article => article.id !== id);
        showDeleteModal.value = false;
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function confirmDelete(id) {
      deleteArticleId.value = id;
      showDeleteModal.value = true;
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load, showDeleteModal, confirmDelete, deleteArticleId };
  }
}
</script>

<style scoped>
/* General Styling */
body {
  font-family: 'Arial, sans-serif';
  background-color: #f5f5f5;
  color: #333;
  margin: 0;
  padding: 0;
  display: flex;
  height: 100vh;
}

/* Container Layout */
.container {
  display: flex;
  width: 100%;
}

/* Sidebar Styling */
.sidebar {
  width: 250px;
  background-color: #3a3a3a;
  color: white;
  padding: 20px;
  box-shadow: 2px 0 5px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.sidebar h2 {
  margin: 0;
  font-size: 24px;
}

.sidebar button {
  background-color: #ff5722;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 20px;
  font-size: 16px;
}

.sidebar button:hover {
  background-color: #e64a19;
}

.welcome-text {
  margin-top: auto;
  text-align: center;
  font-size: 16px;
  padding: 10px;
  color: #cccccc;
}

/* Content Styling */
.content {
  flex: 1;
  display: flex;
  flex-direction: column;
  padding: 20px;
  overflow-y: auto;
}

.header {
  background-color: #2196f3;
  color: white;
  padding: 15px;
  text-align: center;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.header h1 {
  margin: 0;
  font-size: 24px;
}

.article-form {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  margin-top: 20px;
}

.article-form input[type="text"],
.article-form textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
}

.article-form button[type="submit"] {
  background-color: #2196f3;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s ease;
}

.article-form button[type="submit"]:hover {
  background-color: #1976d2;
}

/* Articles List Styling */
.articles-list {
  margin-top: 20px;
}

.article-item {
  background-color: #ffffff;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
}

.article-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.article-header h3 {
  margin: 0;
}

.article-actions button {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  margin-right: 5px;
  transition: background-color 0.3s ease;
}

.article-actions button:hover {
  background-color: #388e3c;
}

.article-actions button:last-child {
  background-color: #f44336;
}

.article-actions button:last-child:hover {
  background-color: #d32f2f;
}

/* Modal Styling */
.modal {
  display: flex;
  align-items: center;
  justify-content: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
}

.modal-content {
  background: #ffffff;
  padding: 20px;
  border-radius: 8px;
  text-align: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.modal-content p {
  margin-bottom: 20px;
}

.modal-content button {
  margin: 0 10px;
}

.close {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 20px;
  cursor: pointer;
}
</style>
