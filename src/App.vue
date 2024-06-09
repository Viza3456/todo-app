<template>
  <div id="app" class="card">
    <h1>To-Do List</h1>
    <form @submit.prevent="addOrUpdateTodo" class="todo-form">
      <input
        v-model="newTodo.name"
        placeholder="Enter task name"
        class="form-input"
      />
      <input
        v-model="newTodo.label"
        placeholder="Enter task label"
        class="form-input"
      />
      <label class="checkbox-label">
        <input type="checkbox" v-model="newTodo.done" class="checkbox-input" />
        Done
      </label>
      <button type="submit" class="btn-update">
        <div class="d-flex">
          <div>
            <span class="material-icons">{{
              editIndex !== null ? "update" : "add"
            }}</span>
          </div>
          <div class="pt-5">
            {{ formButtonLabel }}
          </div>
        </div>
      </button>
    </form>
    <div class="body-table">
      <table class="todo-table">
        <thead>
          <tr>
            <th>Name</th>
            <th>Label</th>
            <th>Status</th>
            <th class="actions">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(todo, index) in todos" :key="index">
            <td>{{ todo.name }}</td>
            <td>{{ todo.label }}</td>
            <td>{{ todoStatus(todo.done) }}</td>
            <td class="actions">
              <div class="d-flex">
                <button @click="deleteTodo(index)" class="btn-delete">
                  <div class="d-flex">
                    <div><span class="material-icons">delete</span></div>
                    <div class="pt-5">Delete</div>
                  </div>
                </button>
                <button @click="editTodo(index, todo)" class="btn-update">
                  <div class="d-flex">
                    <div>
                      <span class="material-icons">{{
                        editIndex !== index ? "edit" : "cancel"
                      }}</span>
                    </div>
                    <div class="pt-5">{{ editButtonLabel(index) }}</div>
                  </div>
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import Swal from "sweetalert2";

const NULL_INDEX = -1;

const newTodo = ref({ name: "", label: "", done: false });
const todos = ref([]);
const editIndex = ref(NULL_INDEX);

const addOrUpdateTodo = () => {
  const { name, label, done } = newTodo.value;
  if (name.trim() && label.trim()) {
    if (editIndex.value !== NULL_INDEX) {
      todos.value[editIndex.value] = { name, label, done };
      editIndex.value = NULL_INDEX;
    } else {
      todos.value.push({ name, label, done });
    }
    resetForm();
    updateLocalStorage();
  }
};

const deleteTodo = (index) => {
  Swal.fire({
    title: "Are you sure?",
    text: "You will not be able to recover this todo!",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "Yes, delete it!",
    cancelButtonText: "No, keep it",
  }).then((result) => {
    if (result.isConfirmed) {
      todos.value.splice(index, 1);
      updateLocalStorage();
      Swal.fire("Deleted!", "Your todo has been deleted.", "success");
    }
  });
};

const editTodo = (index, todo) => {
  if (editIndex.value !== index) {
    newTodo.value = { ...todo };
    editIndex.value = index;
  } else {
    resetForm();
  }
};

const updateLocalStorage = () => {
  localStorage.setItem("todos", JSON.stringify(todos.value));
};

const loadTodos = () => {
  const storedTodos = localStorage.getItem("todos");
  if (storedTodos) {
    todos.value = JSON.parse(storedTodos);
  }
};

const resetForm = () => {
  newTodo.value = { name: "", label: "", done: false };
  editIndex.value = NULL_INDEX;
};

const todoStatus = (done) => (done ? "Done" : "Not");

const editButtonLabel = (index) =>
  editIndex.value !== index ? "Update" : "Cancel";

const formButtonLabel = computed(() =>
  editIndex.value !== NULL_INDEX ? "Update" : "Add"
);

onMounted(() => {
  loadTodos();
});
</script>