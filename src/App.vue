<script>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";

export default {
  setup() {
    const form = reactive({
      id: null,
      title: "",
      content: "",
    });

    const articles = ref([]);
    const loading = ref(false);
    const error = ref("");

    // Validasi form
    const isFormValid = () => {
      return form.title.trim() !== "" && form.content.trim() !== "";
    };

    // Reset form
    const resetForm = () => {
      form.id = null;
      form.title = "";
      form.content = "";
    };

    async function load() {
      loading.value = true;
      error.value = "";
      try {
        const response = await axios.get(
          "https://6d270502-63ef-411b-8d4a-20d1c93aa350-00-m1dgx7b3mnl.sisko.replit.dev/articles"
        );
        articles.value = response.data;
      } catch (err) {
        error.value = "Error loading articles: " + err.message;
        console.error("Error loading articles:", err);
      } finally {
        loading.value = false;
      }
    }

    async function save() {
      if (!isFormValid()) {
        error.value = "Title and content are required";
        return;
      }

      loading.value = true;
      error.value = "";

      try {
        const url = form.id
          ? `https://6d270502-63ef-411b-8d4a-20d1c93aa350-00-m1dgx7b3mnl.sisko.replit.dev/articles/${form.id}`
          : "https://6d270502-63ef-411b-8d4a-20d1c93aa350-00-m1dgx7b3mnl.sisko.replit.dev/articles";
        const method = form.id ? "put" : "post";
        const response = await axios[method](url, {
          title: form.title,
          content: form.content,
        });

        if (form.id) {
          // Update existing article
          articles.value = articles.value.map((article) =>
            article.id === response.data.id ? response.data : article
          );
        } else {
          // Add new article
          articles.value.push(response.data);
        }

        resetForm();
      } catch (err) {
        error.value = "Error saving article: " + err.message;
        console.error("Error saving article:", err);
      } finally {
        loading.value = false;
      }
    }

    async function deleteArticle(id) {
      if (!confirm("Apakah anda yakin mau menghapus artikel tersebut?")) {
        return;
      }

      loading.value = true;
      error.value = "";

      try {
        await axios.delete(
          `https://6d270502-63ef-411b-8d4a-20d1c93aa350-00-m1dgx7b3mnl.sisko.replit.dev/articles/${id}`
        );
        articles.value = articles.value.filter((article) => article.id !== id);
      } catch (err) {
        error.value = "Error deleting article: " + err.message;
        console.error("Error deleting article:", err);
      } finally {
        loading.value = false;
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    function cancelEdit() {
      resetForm();
    }

    onMounted(load);

    return {
      form,
      articles,
      loading,
      error,
      save,
      edit,
      deleteArticle,
      cancelEdit,
      isFormValid,
    };
  },
};
</script>

<template>
  <div>
    <!-- Error message -->
    <div v-if="error" class="error" style="color: red; margin-bottom: 10px">
      {{ error }}
    </div>

    <!-- Loading indicator -->
    <div v-if="loading" style="margin-bottom: 10px">Loading...</div>

    <!-- Form -->
    <form @submit.prevent="save">
      <div>
        <label>Title:</label>
        <input
          type="text"
          v-model="form.title"
          placeholder="Enter article title"
          required
        />
      </div>
      <div>
        <label>Content:</label>
        <textarea
          v-model="form.content"
          placeholder="Enter article content"
          rows="4"
          required
        ></textarea>
      </div>
      <div>
        <button type="submit" :disabled="loading || !isFormValid()">
          {{ form.id ? "Update" : "Create" }} Article
        </button>
        <button
          type="button"
          @click="cancelEdit"
          v-if="form.id"
          :disabled="loading"
        >
          Cancel
        </button>
      </div>
    </form>

    <hr style="margin: 20px 0" />

    <!-- Articles list -->
    <div>
      <h3>Articles</h3>
      <button @click="load" :disabled="loading">Refresh Articles</button>

      <ul v-if="articles.length > 0">
        <li
          v-for="article in articles"
          :key="article.id"
          style="margin-bottom: 15px; border: 1px solid #ccc; padding: 10px"
        >
          <h4>{{ article.title }}</h4>
          <p>{{ article.content }}</p>
          <div>
            <button @click="edit(article)" :disabled="loading">Edit</button>
            <button
              @click="deleteArticle(article.id)"
              :disabled="loading"
              style="margin-left: 5px; background-color: #dc3545; color: white"
            >
              Delete
            </button>
          </div>
        </li>
      </ul>

      <p v-else-if="!loading">No articles found.</p>
    </div>
  </div>
</template>

<style scoped>
form div {
  margin-bottom: 10px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input,
textarea {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  background-color: #007bff;
  color: white;
  cursor: pointer;
}

button:disabled {
  background-color: #6c757d;
  cursor: not-allowed;
}

button:hover:not(:disabled) {
  background-color: #0056b3;
}

.error {
  padding: 10px;
  background-color: #f8d7da;
  border: 1px solid #f5c6cb;
  border-radius: 4px;
}
</style>
