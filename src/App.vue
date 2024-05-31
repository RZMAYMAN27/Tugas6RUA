<template>
  <div class="container">
    <h1 class="title">TEMPAT WISATA ANTAR NEGARA</h1>
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.country" placeholder="Country" class="input" /><br>
      <input type="text" v-model="form.location" placeholder="Location" class="input" /><br>
      <textarea v-model="form.description" placeholder="Description" class="textarea"></textarea><br>
      <button type="submit" class="button">Save</button>
    </form>
    <div class="cards">
      <div v-for="destination in destinations" :key="destination.id" class="card">
        <h3>{{ destination.location }}</h3>
        <h4>{{ destination.country }}</h4>
        <p>{{ destination.description }}</p>
        <div class="buttons">
          <button @click="edit(destination)" class="button button-edit">
            Edit
          </button>
          <button @click="confirmDelete(destination.id)" class="button button-delete">
            Delete
          </button>
        </div>
      </div>
    </div>
    <button @click="load" class="button load-button">Load</button>

    <!-- Delete Confirmation Modal -->
    <div v-if="showModal" class="modal">
      <div class="modal-content">
        <p>Are you sure you want to delete this destination?</p>
        <button @click="deleteDestination" class="button button-danger">Yes, Delete</button>
        <button @click="closeModal" class="button">Cancel</button>
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
      country: '',
      location: '',
      description: ''
    });

    const destinations = ref([]);
    const showModal = ref(false);
    const destinationToDelete = ref(null);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/destinations');
        destinations.value = response.data;
      } catch (error) {
        console.error('Error loading destinations:', error);
      }
    }

    async function save() {
      try {
        const url = form.id ? `http://localhost:3000/destinations/${form.id}` : 'http://localhost:3000/destinations';
        const method = form.id ? 'put' : 'post';
        const response = await axios[method](url, form);

        if (method === 'put') {
          const index = destinations.value.findIndex(destination => destination.id === response.data.id);
          if (index !== -1) {
            destinations.value[index] = response.data;
          }
        } else {
          destinations.value.push(response.data);
        }

        form.id = null;
        form.country = '';
        form.location = '';
        form.description = '';
      } catch (error) {
        console.error('Error saving destination:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/destinations/${id}`);
        destinations.value = destinations.value.filter(destination => destination.id !== id);
      } catch (error) {
        console.error('Error deleting destination:', error);
      }
    }

    function edit(destination) {
      form.id = destination.id;
      form.country = destination.country;
      form.location = destination.location;
      form.description = destination.description;
    }

    function confirmDelete(id) {
      destinationToDelete.value = id;
      showModal.value = true;
    }

    function deleteDestination() {
      remove(destinationToDelete.value);
      closeModal();
    }

    function closeModal() {
      showModal.value = false;
      destinationToDelete.value = null;
    }

    onMounted(load);

    return { form, destinations, save, edit, confirmDelete, deleteDestination, closeModal, showModal };
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap');

body {
  background-color: #f0f2f5;
  color: #333;
  font-family: 'Roboto', sans-serif;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.title {
  font-size: 2em;
  text-align: center;
  margin-bottom: 20px;
  font-weight: 700;
  color: #007bff;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.input, .textarea {
  padding: 10px;
  font-size: 1em;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-family: 'Roboto', sans-serif;
}

.button {
  padding: 10px 20px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 1em;
  font-family: 'Roboto', sans-serif;
  transition: background-color 0.3s ease;
}

.button:hover {
  background-color: #0056b3;
}

.button-danger {
  background-color: #dc3545;
}

.button-danger:hover {
  background-color: #c82333;
}

.cards {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-top: 20px;
}

.card {
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  padding: 20px;
  width: calc(33.333% - 20px);
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.card h3 {
  margin: 0;
  font-size: 1.25em;
  color: #333;
}

.card h4 {
  margin: 0;
  font-size: 1em;
  color: #555;
}

.card p {
  flex-grow: 1;
  font-size: 0.9em;
  color: #666;
}

.buttons {
  display: flex;
  gap: 10px;
}

.button-edit {
  background-color: #17a2b8;
}

.button-edit:hover {
  background-color: #138496;
}

.button-delete {
  background-color: #dc3545;
}

.button-delete:hover {
  background-color: #c82333;
}

.load-button {
  background-color: #007bff;
  margin-top: 10px;
}

.load-button:hover {
  background-color: #0056b3;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal-content {
  background-color: #fff;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
}

.modal-content p {
  margin-bottom: 20px;
  color: #000;
}
</style>
