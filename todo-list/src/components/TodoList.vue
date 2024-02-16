<template>
  <div class="todo-list">
    <h1>Welcome, {{ currentUser ? currentUser.userDetails : 'Guest' }}</h1>
    <div class="todo-item">
      <div class="todo-content">
        <input type="text" v-model="newTodo" @keyup.enter="addTodo" class="rounded-input">
      </div>
      <button @click="addTodo" class="add-button">+</button>
    </div>

    <div class="todo-item" v-for="(todo, index) in todos" :key="index">
      <div class="todo-content">
        <input type="checkbox" v-model="todo.completed">
        <label :class="{ 'completed': todo.completed }">{{ todo.label }}</label>
      </div>
      <button @click="removeTodo(index)" class="delete-button">X</button>
    </div>

  </div>
</template>

<script>
export default {
  data() {
    return {
      newTodo: '',
      todos: [],
      currentUser: null
    };
  },
  created() {},
  methods: {
    async addTodo() {
      if (this.newTodo.trim() !== '') {
        const todo = {
          label: this.newTodo,
          completed: false
        };
        this.todos.push(todo);
        this.newTodo = '';
      }
    },
    removeTodo(index) {
      this.todos.splice(index, 1);
    },
  }
};
</script>

<style scoped>
.todo-list {
  margin: 20px auto;
  max-width: 400px;
  padding: 20px;
  background-color: #f5f5f5;
  border-radius: 10px;
}

.delete-button {
  justify-content: center;
  align-items: center;
  background-color: red;
  color: white;
  text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;
  border: none;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  font-weight: bold;
  cursor: pointer;
}

.add-button {
  justify-content: center;
  align-items: center;
  background-color: green;
  color: white;
  font-size: 20px;
  text-shadow: -1px 0 black, 0 1px black, 1px 0 black, 0 -1px black;
  border: none;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  font-weight: bold;
  cursor: pointer;
}

.rounded-input,
.rounded-button,
.rounded-todo {
  border-radius: 5px;
  margin-bottom: 5px;
}

.rounded-input,
.rounded-button,
.edit-todo-input {
  border: none;
  padding: 5px 10px;
}

rounded-input {
  width: 100%;
}

.rounded-input {
  width: 90%;
}

.completed {
  text-decoration: line-through;
  transition: text-decoration 0.3s ease-in-out;
}

.edit-todo-input {
  margin-bottom: 5px;
  width: 90%;
}

ul {
  list-style-type: none;
}

.todo-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.todo-content {
  display: flex;
  width: 100%;
  flex: 1;
}

#label {
  justify-content: left;
}

</style>
