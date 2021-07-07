<template>
    <div class="hello">
        <section class="todoapp">
            <header class="header">
                <h1>todos</h1>
                <input class="new-todo"
                       autofocus
                       autocomplete="off"
                       placeholder="What needs to be done?"
                       v-model="newTodo"
                       @keyup.enter="addTodo"/>
            </header>
            <section class="main" v-show="todoList.length" v-cloak>
                <input
                        id="toggle-all"
                        class="toggle-all"
                        type="checkbox"
                        v-model="allDone"
                />
                <label for="toggle-all"></label>
                <ul class="todo-list">
                    <li v-for="todo in filteredTodos"
                        class="todo"
                        :key="todo.id"
                            :class="{ completed: todo.completed, editing: todo == editedTodo }">
                        <div class="view">
                            <input class="toggle" type="checkbox" v-model="todo.completed"/>
                            <label @dblclick="editTodo(todo)">{{ todo.title }}</label>
                            <button class="destroy" @click="removeTodo(todo)"></button>
                        </div>
                        <input class="edit"
                               type="text"
                               v-model="todo.title"
                               v-todo-focus="todo == editedTodo"
                               @blur="doneEdit(todo)"
                               @keyup.enter="doneEdit(todo)"
                               @keyup.esc="cancelEdit(todo)"/>
                    </li>
                </ul>
            </section>
            <footer class="footer" v-show="todoList.length" v-cloak>
        <span class="todo-count">
          <strong>{{ remaining }}</strong> {{ remaining | pluralize }} left
        </span>
                <ul class="filters">
                    <li>
                        <a href="#/all" :class="{ selected: visibility == 'all' }">All</a>
                    </li>
                    <li>
                        <a href="#/active" :class="{ selected: visibility == 'active' }"
                        >Active</a
                        >
                    </li>
                    <li>
                        <a href="#/completed"
                           :class="{ selected: visibility == 'completed' }"
                        >Completed</a
                        >
                    </li>
                </ul>
                <button class="clear-completed"
                        @click="removeCompleted"
                        v-show="todoList.length > remaining">
                    Clear completed
                </button>
            </footer>
        </section>
        <footer class="info">
            <p>Double-click to edit a todo</p>
            <p>Written by <a href="http://evanyou.me">Evan You</a></p>
            <p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
        </footer>
    </div>
</template>

<script>
    import '../static/hello.css';

    // localStorage persistence
    let STORAGE_KEY = "todoList-vuejs-2.0";
    let todoStorage = {
        fetch: function () {
            let todoList = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
            todoList.forEach(function (todo, index) {
                todo.id = index;
            });
            todoStorage.uid = todoList.length;
            console.log(todoList)
            return todoList;
        },
        save: function (todoList) {
            localStorage.setItem(STORAGE_KEY, JSON.stringify(todoList));
        }
    };

    // visibility filters
    let filters = {
        all: function (todoList) {
            return todoList;
        },
        active: function (todoList) {
            return todoList.filter(function (todo) {
                return !todo.completed;
            });
        },
        completed: function (todoList) {
            return todoList.filter(function (todo) {
                return todo.completed;
            });
        }
    };
    let dataSource = {
        todoList: todoStorage.fetch(),
        newTodo: "",
        editedTodo: null,
        visibility: "all"
    };
    export default {
        name: 'HelloWorld',
        // app initial state
        data(){
            return dataSource
        } ,
        // watch todoList change for localStorage persistence
        watch: {
            todoList: {
                handler: function (todoList) {
                    todoStorage.save(todoList);
                },
                deep: true
            }
        },

        // computed properties
        // http://vuejs.org/guide/computed.html
        computed: {
            filteredTodos: function () {
                return filters[this.visibility](this.todoList);
            },
            remaining: function () {
                return filters.active(this.todoList).length;
            },
            allDone: {
                get: function () {
                    return this.remaining === 0;
                },
                set: function (value) {
                    this.todoList.forEach(function (todo) {
                        todo.completed = value;
                    });
                }
            }
        },

        filters: {
            pluralize: function (n) {
                return n === 1 ? "item" : "items";
            }
        },

        // methods that implement data logic.
        // note there's no DOM manipulation here at all.
        methods: {
            addTodo: function () {
                let value = this.newTodo && this.newTodo.trim();
                if (!value) {
                    return;
                }
                this.todoList.push({
                    id: todoStorage.uid++,
                    title: value,
                    completed: false
                });
                this.newTodo = "";
            },

            removeTodo: function (todo) {
                this.todoList.splice(this.todoList.indexOf(todo), 1);
            },

            editTodo: function (todo) {
                this.beforeEditCache = todo.title;
                this.editedTodo = todo;
            },

            doneEdit: function (todo) {
                if (!this.editedTodo) {
                    return;
                }
                this.editedTodo = null;
                todo.title = todo.title.trim();
                if (!todo.title) {
                    this.removeTodo(todo);
                }
            },

            cancelEdit: function (todo) {
                this.editedTodo = null;
                todo.title = this.beforeEditCache;
            },

            removeCompleted: function () {
                this.todoList = filters.active(this.todoList);
            }
        },

        // a custom directive to wait for the DOM to be updated
        // before focusing on the input field.
        // http://vuejs.org/guide/custom-directive.html
        directives: {
            "todo-focus": function (el, binding) {
                if (binding.value) {
                    el.focus();
                }
            }
        }
    }

    // handle routing
    function onHashChange() {
        let visibility = window.location.hash.replace(/#\/?/, "");
        if (filters[visibility]) {
            dataSource.visibility = visibility;
        } else {
            window.location.hash = "";
            dataSource.visibility = "all";
        }
    }

    window.addEventListener("hashchange", onHashChange);
    onHashChange();

</script>

<style>
    [v-cloak] {
        display: none;
    }
</style>
