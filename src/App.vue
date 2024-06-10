<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <h2>Buat Artikel Baru</h2>
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button save-button">Save</button>
    </form>
    <div class="article-section">
      <h2>List Article</h2>
      <div class="article-list" ref="articleList">
        <div v-for="article in articles" :key="article.id" class="article-item">
          <div v-if="editId === article.id">
            <h3>Edit Article</h3>
            <input type="text" v-model="editForm.title" class="input" /><br />
            <textarea v-model="editForm.content" class="textarea"></textarea><br />
            <button @click="updateArticle(article.id)" class="button save-button">Update</button>
            <button @click="cancelEdit" class="button cancel-button">Cancel</button>
          </div>
          <div v-else>
            <strong>{{ article.title }}</strong><br />
            <p>{{ article.content }}</p>
            <button @click="edit(article)" class="button edit-button">Edit</button>
            <button @click="deleteArticle(article.id)" class="button delete-button">Delete</button>
          </div>
        </div>
      </div>
      <button @click="scrollToTop" class="button scroll-top-button">Scroll to Top</button>
    </div>
    <button @click="load" class="button load-button">Load Articles</button>
  </div>
</template>

<script>
import { ref, onMounted, reactive, nextTick } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: null,
      title: '',
      content: ''
    });
    const editForm = reactive({
      id: null,
      title: '',
      content: ''
    });
    const articles = ref([]);
    const editId = ref(null);
    const articleList = ref(null);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
        await nextTick();
        if (articles.value.length > 5) {
          articleList.value.scrollTo({
            top: articleList.value.scrollHeight,
            behavior: 'smooth'
          });
        }
      } catch (error) {
        showError('Gagal memuat artikel');
      }
    }

    async function save() {
      if (!validateForm()) return;

      try {
        const url = form.id
          ? `http://localhost:3000/articles/${form.id}`
          : 'http://localhost:3000/articles';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, { title: form.title, content: form.content });

        if (form.id) {
          const index = articles.value.findIndex(article => article.id === form.id);
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
        showError('Gagal menyimpan artikel');
      }
    }

    async function updateArticle(id) {
      try {
        const response = await axios.put(`http://localhost:3000/articles/${id}`, {
          title: editForm.title,
          content: editForm.content
        });

        const index = articles.value.findIndex(article => article.id === id);
        if (index !== -1) {
          articles.value[index] = response.data;
        }

        cancelEdit();
      } catch (error) {
        showError('Gagal memperbarui artikel');
      }
    }

    async function deleteArticle(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        showError('Gagal menghapus artikel');
      }
    }

    function edit(article) {
      editId.value = article.id;
      editForm.id = article.id;
      editForm.title = article.title;
      editForm.content = article.content;
    }

    function cancelEdit() {
      editId.value = null;
      editForm.id = null;
      editForm.title = '';
      editForm.content = '';
    }

    function scrollToTop() {
      articleList.value.scrollTo({ top: 0, behavior: 'smooth' });
    }

    function validateForm() {
      if (!form.title.trim() || !form.content.trim()) {
        alert('Title dan Content harus diisi');
        return false;
      }
      return true;
    }

    function showError(message) {
      alert('Terjadi kesalahan: ' + message);
    }

    onMounted(load);

    return { form, editForm, articles, save, edit, updateArticle, deleteArticle, load, cancelEdit, editId, articleList, scrollToTop };
  }
}
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: 'Arial', sans-serif;
  background-color: #f9f9f949;
  border-radius: 10px;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.1);
}

.form {
  margin-bottom: 20px;
  background: #ffffff56;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.input, .textarea {
  width: 100%;
  margin-bottom: 10px;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ddd;
}

.button {
  margin-right: 10px;
  padding: 10px 15px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.save-button {
  background-color: #000000;
  color: white;
}

.cancel-button {
  background-color: #000000;
  color: white;
}

.edit-button {
  background-color: #000000;
  color: white;
}

.delete-button {
  background-color: #000000;
  color: white;
}

.scroll-top-button {
  background-color: #555;
  color: white;
}

.load-button {
  background-color: #555;
  color: white;
}

.article-section {
  background: #ffffff56;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  max-height: 600px; /* Ganti dengan tinggi maksimum yang Anda inginkan */
}

.article-list {
  margin-top: 20px;
}

.article-item {
  margin-bottom: 20px;
  padding: 15px;
  background: #ffffff56;
  border-radius: 5px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}

.article-item strong {
  font-size: 1.2em;
}

.article-item p {
  margin: 10px 0;
}

.scroll-top-button {
  margin-top: 20px;
}
</style>