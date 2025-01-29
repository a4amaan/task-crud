<template>
  <q-page class="q-pa-md">
    <q-form @submit="submitTask" class="q-gutter-md">
      <q-input v-model="task.title" label="Title" required square dense filled />
      <q-input v-model="task.description" label="Description" type="textarea" required square dense filled />
      <q-toggle v-model="task.completed" label="Completed" />
      <q-btn type="submit" color="primary" :label="isEditing ? 'Update Task' : 'Add Task'" />
    </q-form>

    <q-table :rows="tasks" :columns="columns" row-key="id" class="q-mt-md">
      <template v-slot:body-cell-actions="props">
        <q-td :props="props">
          <q-btn flat dense color="green" icon="edit" @click="loadTask(props.row)" />
          <q-btn flat dense color="red" icon="delete" @click="deleteTask(props.row.id)" />
        </q-td>
      </template>
    </q-table>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

const task = ref({ id: null, title: "", description: "", completed: false });
const tasks = ref([]);
const isEditing = ref(false);

const columns = [
  { name: "title", label: "Title", field: "title", align: "left" },
  { name: "description", label: "Description", field: "description", align: "left" },
  { name: "completed", label: "Completed", field: "completed", align: "center", format: val => (val ? "Yes" : "No") },
  { name: "actions", label: "Actions", align: "center" }
];

const fetchTasks = async () => {
  const response = await axios.get("http://localhost:5000/tasks");
  tasks.value = response.data;
};

const submitTask = async () => {
  if (isEditing.value) {
    await axios.put(`http://localhost:5000/tasks/${task.value.id}`, task.value);
  } else {
    await axios.post("http://localhost:5000/tasks", task.value);
  }
  resetForm();
  fetchTasks();
};

const loadTask = (taskData) => {
  task.value = { ...taskData };
  isEditing.value = true;
};

const deleteTask = async (id) => {
  await axios.delete(`http://localhost:5000/tasks/${id}`);
  fetchTasks();
};

const resetForm = () => {
  task.value = { id: null, title: "", description: "", completed: false };
  isEditing.value = false;
};

onMounted(fetchTasks);
</script>
