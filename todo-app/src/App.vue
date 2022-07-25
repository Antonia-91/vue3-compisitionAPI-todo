<template>
  <main class="app">
    <section class="create-todo">
      <h3>CREATE A TODO</h3>
      <form id="new-todo-form" @submit.prevent="addTodo">
        <h4>What's on your todo-list?</h4>
        <input
          type="text"
          name="content"
          id="content"
          placeholder="e.g. make a video"
          v-model="state.input_content"
        />

        <h4>Pick a category</h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              id="category1"
              value="business"
              v-model="state.input_category"
            />
            <span class="bubble business"></span>
            <div>Business</div>
          </label>

          <label>
            <input
              type="radio"
              name="category"
              id="category2"
              value="personal"
              v-model="state.input_category"
            />
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>
        </div>
        <input type="submit" value="Add todo" />
      </form>
    </section>

    <section class="todo-list">
      <h3>TODO LIST</h3>
      <div class="list" id="todo-list">
        <div
          v-for="todo in state.todos"
          :class="`todo-item ${todo.done && 'done'}`"
        >
          <label>
            <input type="checkbox" v-model="todo.done" @click="check(todo)" />
            <span
              :class="`bubble ${
                todo.category == 'business' ? 'business' : 'personal'
              }`"
            ></span>
          </label>

          <div class="todo-content">
            <input type="text" v-model="todo.content" />
          </div>

          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<script>
import axios from "axios";
import { defineComponent, reactive, watch } from "vue";

export default defineComponent({
  setup() {
    const state = reactive({
      todos: [],
      input_content: "",
      input_category: null,
    });

    ///add todo
    function addTodo() {
      if (state.input_content.trim() === "" || state.input_category === null) {
        return;
      }
      axios
        .post("http://localhost:3000/todos", {
          content: state.input_content,
          category: state.input_category,
          done: false,
          editable: false,
          createdAt: new Date().getTime(),
        })
        .then(() => {
          (state.input_content = ""), (state.input_category = null);
          getTodos();
        });
    }

    ///update todo
    function check(todo) {
      console.log(todo.done);
      axios
        .patch("http://localhost:3000/todos/" + todo.id, {
          done: !todo.done,
        })
        .then(() => getTodos());
    }

    /// remove todo
    function removeTodo(todo) {
      console.log(todo.id);
      axios({
        method: "DELETE",
        url: "http://localhost:3000/todos/" + todo.id,
      }).then(() => getTodos());
    }

    ///fetch all todos
    function getTodos() {
      axios.get("http://localhost:3000/todos").then((response) => {
        state.todos = response.data;

        console.log(state.todos);
      });
    }

    getTodos();

    /// return
    return {
      state,
      addTodo,
      check,
      removeTodo,
    };
  },
});
</script>
