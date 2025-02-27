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
        <input type="checkbox" v-model="todo.complete" @change="toggleCompleted(index)">
        <label :class="{ 'completed': todo.complete }">{{ todo.label }}</label>
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
  async created() {
    await this.getCurrentUser();

    const todos = await this.list()
    this.todos = todos;
  },
  methods: {
    async getCurrentUser() {
      try {
        const response = await fetch('/.auth/me');
        if (response.ok) {
          const data = await response.json();
          this.currentUser = data.clientPrincipal;
        } else {
          console.error('Failed to fetch user information');
        }
      } catch (error) {
        console.error('Error fetching user information:', error);
      }
    },
    async addTodo() {
      if (this.newTodo.trim() !== '') {
        const todo = {
          id: Math.random().toString(36).substr(2, 9),
          userId: Math.random().toString(36).substr(2, 9),
          label: this.newTodo,
          complete: false
        };
        // this.todos.push(todo);
        // this.newTodo = '';

        const createdTodo = await this.create(todo)
        this.todos.push(createdTodo);
        this.newTodo = '';
      }
    },
    async removeTodo(index) {
      // this.todos.splice(index, 1);

      const todo = this.todos[index];
      this.todos.splice(index, 1);
      await this.delete(todo.id);
    },
    async toggleCompleted(index){
      const todo = this.todos[index];
      console.log(todo);
      await this.update(todo.id, todo);
    },
    async list() {
      const query = `
        query GetTodoItems {
          todoItems {
            items {
              id
              userId
              label
              complete
            }
          }
        }`;
          
      const endpoint = "/data-api/graphql";
      const response = await fetch(endpoint, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ query: query })
      });
      
      const result = await response.json();
      return result.data.todoItems.items;
    },
    async update(id, newData) {
      const gql = `
        mutation UpdateTodoItem($id: ID!, $_partitionKeyValue: String!, $item: UpdateTodoItemsInput!) {
          updateTodoItems(id: $id, _partitionKeyValue: $_partitionKeyValue, item: $item) {
            id
            userId
            label
            complete
          }
        }`;

      const query = {
        query: gql,
        variables: {
          id: id,
          _partitionKeyValue: id,
          item: newData
        }
      };

      const endpoint = "/data-api/graphql";
      const response = await fetch(endpoint, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(query)
      });

      const result = await response.json();
      return result.data.updateTodoItems;
    },
    async create(data) {
      const gql = `
        mutation CreateTodoItem($item: CreateTodoItemsInput!) {
          createTodoItems(item: $item) {
            id
            userId
            label
            complete
          }
        }`;

      const query = {
        query: gql,
        variables: {
          item: data
        }
      };

      const endpoint = "/data-api/graphql";
      const response = await fetch(endpoint, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(query)
      });

      const result = await response.json();
      return result.data.createTodoItems;
    },
    async delete(id) {
      const gql = `
        mutation DeleteTodoItem($id: ID!, $_partitionKeyValue: String!) {
          deleteTodoItems(id: $id, _partitionKeyValue: $_partitionKeyValue) {
            id
          }
        }`;

      const query = {
        query: gql,
        variables: {
          id: id,
          _partitionKeyValue: id
        }
      };

      const endpoint = "/data-api/graphql";
      const response = await fetch(endpoint, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(query)
      });

      const result = await response.json();
      return result.data.deleteTodoItems;
    }
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
