<script setup>
import { ref } from "vue";
import todos from "../data.js";
let nextId = todos.length + 1;
import fn from "../functions.js";

const newTaskInput = ref("");
const currentTab = ref("all");
const currentEditing = ref("");

const getStorageData = () => {
  todos.length = 0;
  const storedTodos = fn.getTodosStorage();
  storedTodos.forEach((todo) => {
    todos.push(todo);
  });
  nextId = todos.length + 1;
};

getStorageData();

const addNewTodo = () => {
  todos.unshift({ id: nextId, task: newTaskInput.value, done: false });
  newTaskInput.value = "";
  nextId++;
  fn.updateTodosStorage(todos);
};

const getTodos = () => {
  if (currentTab.value == "all") {
    return todos;
  } else if (currentTab.value == "active") {
    return todos.filter((todo) => {
      if (!todo.done) {
        return todo;
      }
    });
  } else if (currentTab.value == "completed") {
    return todos.filter((todo) => {
      if (todo.done) {
        return todo;
      }
    });
  }
};

const getRemainingText = () => {
  let remaining = 0;
  let total = todos.length;
  todos.forEach((todo) => {
    if (!todo.done) {
      remaining++;
    }
  });

  return `${remaining} of ${total} left`;
};

const clearTodos = () => {
  if (currentTab.value == "all") {
    todos.length = 0;
  } else if (currentTab.value == "active") {
    let tempTodos = JSON.parse(JSON.stringify(todos));
    todos.length = 0;
    tempTodos.forEach((todo) => {

      console.log(todo);
      if (todo.done) {
        todos.push(todo);
      }
    });
  } else if (currentTab.value == "completed") {
    let tempTodos = JSON.parse(JSON.stringify(todos));
    todos.length = 0;
    tempTodos.forEach((todo) => {
      if (!todo.done) {
        todos.push(todo);
      }
    });
  }

  fn.updateTodosStorage(todos);
};

const toggleTaskStatus = () => {
  fn.updateTodosStorage(todos);
}

const deleteTask = (id) => {
  let tempTodos = JSON.parse(JSON.stringify(todos));
  todos.length = 0;
  tempTodos.map((todo) => {
    if (todo.id != id) {
      todos.push(todo);
    }
  });

  fn.updateTodosStorage(todos);
}

const updateTask = () => {
  currentEditing.value = null;
  fn.updateTodosStorage(todos);
}
</script>

<template>
  <div class="bg-gray-100 dark:bg-slate-900 h-screen">
    <section
      class="h-screen flex items-center flex-col justify-center py-12 sm:px-6 lg:px-8"
    >
      <div
        class="max-w-md w-full p-3 bg-white dark:bg-slate-800 shadow overflow-hidden sm:rounded-lg space-y-8"
      >
        <div class="flex justify-between">
          <h1 class="font-medium dark:text-white">Todo App</h1>
        </div>

        <form @submit.prevent="addNewTodo()" class="flex">
          <input
            v-model="newTaskInput"
            type="text"
            placeholder="New Task"
            class="h-10 px-3 py-2 bg-white dark:bg-slate-700 dark:text-slate-200 dark:border-slate-500 border shadow-sm border-slate-300 placeholder-slate-400 disabled:bg-slate-50 disabled:text-slate-500 disabled:border-slate-200 focus:outline-none focus:border-violet-500 focus:ring-violet-500 block w-full rounded-tl-lg rounded-bl-lg sm:text-sm focus:ring-1 invalid:border-pink-500 invalid:text-pink-600 focus:invalid:border-pink-500 focus:invalid:ring-pink-500 disabled:shadow-none"
          /><button
            type="submit"
            aria-label="Add todo"
            class="h-10 bg-violet-500 hover:bg-violet-600 focus:outline-none focus:ring focus:ring-violet-300 active:bg-violet-700 px-5 py-2 text-sm leading-5 rounded-tr-lg rounded-br-lg font-semibold text-white"
          >
            Add
          </button>
        </form>

        <div class="h-80 overflow-x-hidden overflow-y-auto todo-list">
          <template v-for="todo in getTodos()" :key="index">
            <div v-if="todo.id != currentEditing"
              class="shadow rounded-lg p-3 mt-4 bg-gray-50 dark:bg-slate-700 flex justify-between"
            >
              <div @dblclick="currentEditing = todo.id" class="flex items-center">
                <input type="checkbox" @change="toggleTaskStatus()" v-model="todo.done" />
                <label
                  class="ml-2 font-medium text-slate-800 dark:text-gray-50 text-sm"
                  :class="{ 'line-through': todo.done }"
                  >{{ todo.task }}</label
                >
              </div>
              <div class="">
                <button
                  @click="deleteTask(todo.id)"
                  class="bg-violet-500 hover:bg-violet-600 focus:outline-none focus:ring focus:ring-violet-300 active:bg-violet-700 px-2 py-1 text-sm leading-5 rounded-full font-semibold text-white"
                  aria-label="Delete a todo"
                >
                  X
                </button>
              </div>
            </div>

            <div
              v-else
              class="shadow rounded-lg p-3 mt-4 bg-gray-50 dark:bg-slate-700"
            >
              <form @submit.prevent="updateTask()" class="flex">
                <input
                  @keyup.enter="updateTask()"
                  v-model="todo.task"
                  type="text"
                  placeholder="Edit Task"
                  class="h-10 px-3 py-2 bg-white dark:bg-slate-700 dark:text-slate-200 dark:border-slate-500 border shadow-sm border-slate-300 placeholder-slate-400 disabled:bg-slate-50 disabled:text-slate-500 disabled:border-slate-200 focus:outline-none focus:border-violet-500 focus:ring-violet-500 block w-full rounded-tl-lg rounded-bl-lg sm:text-sm focus:ring-1 invalid:border-pink-500 invalid:text-pink-600 focus:invalid:border-pink-500 focus:invalid:ring-pink-500 disabled:shadow-none"
                /><button
                  type="submit"
                  aria-label="Add todo"
                  class="h-10 bg-violet-500 hover:bg-violet-600 focus:outline-none focus:ring focus:ring-violet-300 active:bg-violet-700 px-5 py-2 text-sm leading-5 rounded-tr-lg rounded-br-lg font-semibold text-white"
                >
                  Update
                </button>
              </form>
            </div>

          </template>
        </div>

        <div class="flex justify-between">
          <div class="flex justify-center gap-5">
            <p class="text-gray-800">{{ getRemainingText() }}</p>
            <button
              @click="currentTab = 'all'"
              :class="currentTab == 'all' ? 'text-green-500' : 'text-gray-500'"
            >
              All
            </button>
            <button
              @click="currentTab = 'active'"
              :class="
                currentTab == 'active' ? 'text-green-500' : 'text-gray-500'
              "
            >
              Active
            </button>
            <button
              @click="currentTab = 'completed'"
              :class="
                currentTab == 'completed' ? 'text-green-500' : 'text-gray-500'
              "
            >
              Completed
            </button>
          </div>
          <button @click="clearTodos()" class="text-red-600">Clear</button>
        </div>
      </div>
    </section>
  </div>
</template>

<style></style>
