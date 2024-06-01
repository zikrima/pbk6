<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Judul" class="input"/><br />
      <textarea v-model="form.content" placeholder="Konten" class="textarea"></textarea><br />
      <button type="submit" class="button">Simpan</button>
    </form>
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

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: '',
    });

    const articles = ref([]);

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
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles'; 
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (form.id) {
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
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
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, remove };
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

.input, .textarea {
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
</style>
