<template>
    <section class="todoapp">
        <header>
            <h1>todos</h1>
            <input class="new-todo" autofocus placeholder="What needs to be done?" @keyup.enter="addTodo">
        </header>
        <section class="main" v-show="todos.length">
            <input id="toggle-all" class="toggle-all" type="checkbox" :checked="remaining === 0" @change="toggleAll">
            <label for="toggle-all">Mark all as complete</label>
            <ul class="todo-list">
                <li class="todo" v-for="todo in filteredTodos" :key="todo.id" :class="{completed: todo.completed, editing: todo === editedTodo}">
                    <div class="view">
                        <input class="toggle" type="checkbox" v-model="todo.completed">
                        <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
                        <button class="destroy" @click="removeTodo(todo)"></button>
                    </div>
                    <!-- <input class="edit" type="text" v-if="todo === editedTodo" v-model="todo.title" @vnode-mounted="({ el }) => el.focus()" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.escape="cancelEdit(todo)"> -->
                    <input class="edit" type="text" v-focus="todo === editedTodo" v-model="todo.title" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)" @keyup.escape="cancelEdit(todo)">
                </li>
            </ul>
        </section>
        <footer class="footer" v-show="todos.length">
            <span class="todo-count">
                <strong>{{ remaining }}</strong>
                <span>{{ remaining === 1 ? ' item' : ' items' }} left</span>
            </span>
            <ul class="filters">
                <li><a href="#/all" :class="{ selected: visibility === 'all' }">All</a></li>
                <li><a href="#/active" :class="{ selected: visibility === 'active' }">Active</a></li>
                <li><a href="#/completed" :class="{ selected: visibility === 'completed' }">Completed</a></li>
            </ul>
            <button class="clear-completed" v-show="todos.length > remaining" @click="removeCompleted">Clear completed</button>
        </footer>
    </section>
    <footer class="info">
        <p>Double-click to edit a todo</p>
        <!-- <p>Written by <a href="">Rui Xiao</a></p>
        <p>Part of <a href="">TodoMVC</a></p> -->
    </footer>
</template>

<script setup>
import { ref, computed, watchEffect } from 'vue'
const STORAGE_KEY = 'vue-todo'

// 筛选按钮handle
const filters = {
    all: (todos) => todos,
    active: (todos) => todos.filter((todo) => !todo.completed),
    completed: (todos) => todos.filter((todo) => todo.completed)
}

// 状态｜数据
const todos = ref(JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'))
const visibility = ref('all')
const editedTodo = ref()

// 计算状态｜数据
const filteredTodos = computed(() => filters[visibility.value](todos.value))
const remaining = computed(() => filters.active(todos.value).length)

// todo的相关操作
function addTodo(e) {
    const value = e.target.value.trim()
    if (value) {
        todos.value.push({
            id: Date.now(),
            title: value,
            completed: false
        })
    }
    e.target.value = ''
}
function removeTodo(todo) {
    todos.value.splice(todos.value.indexOf(todo), 1)
}
let memoryEditValue = ''
function editTodo(todo) {
    editedTodo.value = todo
    memoryEditValue = todo.title
}
function doneEdit(todo) {
    if (editedTodo.value) {
        todo.title = todo.title.trim()
        if (!todo.title) {
            removeTodo(todo)
        }
        editedTodo.value = null
    }
}
function cancelEdit(todo) {
    editedTodo.value = null
    todo.title = memoryEditValue
}

// todo全部完成
function toggleAll(e) {
    todos.value.forEach((todo) => {
        todo.completed = e.target.checked
    })
}

// 筛选，通过路由来实现
function onHashChange() {
    const route = window.location.hash.replace(/#\/?/, '')
    if (filters[route]) {
        visibility.value = route
    } else {
        window.location.hash = ''
        visibility.value = 'all'
    }
}
window.addEventListener('hashchange', onHashChange)
onHashChange() // 首次加载页面先处理一次

// 删除完成
function removeCompleted() {
    todos.value = filters.active(todos.value)
}

// 数据持久化
watchEffect(() => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos.value))
})

// 输入框聚焦指令
const vFocus = function (el, binding) {
    if (binding.value) {
        el.focus()
    }
}
</script>