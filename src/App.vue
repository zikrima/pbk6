<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Judul" class="input" /><br />
      <textarea v-model="form.content" placeholder="Konten" class="textarea"></textarea><br />
      <button type="submit" class="button">Simpan</button>
    </form>
    <div v-if="error" class="error-message">{{ error }}</div>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div class="article-content">
          <strong>{{ article.title }}</strong><br />
          <p>{{ article.content }}</p>
        </div>
        <div class="article-actions">
          <button @click="edit(article)" class="button">Edit</button>
          <button @click="remove(article.id)" class="button delete-button">Hapus</button>
        </div>
      </li>
    </ul>
    <button @click="load" class="button load-button">Muat</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from "axios";
import { v4 as uuidv4 } from 'uuid'; // Import uuidv4 sebagai uuid

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);
    const error = ref(null);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
        error.value = null;
      } catch (err) {
        console.error('Error loading articles:', err);
        error.value = 'Error loading articles. Please ensure the server is running.';
      }
    }

    async function save() {
      try {
        let url = 'http://localhost:3000/articles';

        if (form.id) {
          // Jika form.id terdefinisi, maka lakukan update (PUT)
          url += `/${form.id}`;

          const response = await axios.put(url, form);

          // Update artikel di dalam list dengan respons dari server
          articles.value = articles.value.map((article) =>
            article.id === form.id ? response.data : article
          );
        } else {
          // Jika form.id belum terdefinisi, maka lakukan create (POST)
          form.id = uuidv4();
          const response = await axios.post(url, form);

          // Tambahkan artikel baru ke dalam list
          articles.value.push(response.data);
        }

        // Kosongkan form setelah operasi berhasil
        form.id = null;
        form.title = '';
        form.content = '';
        error.value = null;
      } catch (err) {
        console.error('Error saving article:', err);
        error.value = 'Error saving article. Please ensure the server is running.';
      }
    }

    async function remove(id) {
      try {
        const response = await axios.delete(`http://localhost:3000/articles/${id}`);
        if (response.status === 200) {
          // Hapus artikel dari list lokal
          articles.value = articles.value.filter(article => article.id !== id);
          error.value = null;
        } else {
          console.error('Failed to delete article:', response);
          error.value = 'Failed to delete article. Please ensure the server is running.';
        }
      } catch (err) {
        console.error('Error deleting article:', err);
        error.value = 'Error deleting article. Please ensure the server is running.';
      }
    }

    function edit(article) {
      // Isi form dengan data artikel yang akan diedit
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, error, save, edit, remove, load };
  },
};
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.form {
  margin-bottom: 20px;
}

.input,
.textarea {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.textarea {
  min-height: 100px;
  resize: vertical;
}

.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  background-color: #28a745;
  color: white;
  cursor: pointer;
}

.button:hover {
  background-color: #218838;
}

.delete-button {
  background-color: #dc3545;
}

.delete-button:hover {
  background-color: #c82333;
}

.load-button {
  display: block;
  width: 100%;
  margin-top: 20px;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  border-bottom: 1px solid #ccc;
  padding: 10px 0;
}

.article-content {
  flex-grow: 1;
}

.article-actions {
  display: flex;
  gap: 10px;
}

.error-message {
  color: red;
  margin-bottom: 20px;
}
</style>
