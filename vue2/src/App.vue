<template>
	<div id="app">
		<section
			v-cloak
			class="todoapp"
		>
			<header class="header">
				<h1>todos</h1>
				<input
					v-model="newTodo"
					class="new-todo"
					autofocus
					autocomplete="off"
					placeholder="What needs to be done?"
					@keyup.enter="addTodo"
				>
			</header>
			<section
				v-show="todos.length"
				class="main"
			>
				<input
					id="toggle-all"
					v-model="allDone"
					class="toggle-all"
					type="checkbox"
				>
				<label for="toggle-all">Mark all as complete</label>
				<ul class="todo-list">
					<li
						v-for="(todo, index) in filteredTodos"
						:key="`todo-${index}`"
						class="todo"
						:class="{completed: todo.completed, editing: todo == editedTodo}"
					>
						<div class="view">
							<input
								v-model="todo.completed"
								class="toggle"
								type="checkbox"
							>
							<label @dblclick="editTodo(todo)">{{ todo.title }}</label>
							<button
								class="destroy"
								@click="removeTodo(todo)"
							/>
						</div>
						<input
							v-model="todo.title"
							v-todo-focus="todo == editedTodo"
							class="edit"
							type="text"
							@blur="doneEdit(todo)"
							@keyup.enter="doneEdit(todo)"
							@keyup.esc="cancelEdit(todo)"
						>
					</li>
				</ul>
			</section>
			<footer
				v-show="todos.length"
				class="footer"
			>
				<span class="todo-count">
					<strong v-text="remaining" /> {{ remaining | pluralize('item', remaining) }} left
				</span>
				<ul class="filters">
					<li>
						<a
							href="#/all"
							:class="{selected: visibility == 'all'}"
						>All</a>
					</li>
					<li>
						<a
							href="#/active"
							:class="{selected: visibility == 'active'}"
						>Active</a>
					</li>
					<li>
						<a
							href="#/completed"
							:class="{selected: visibility == 'completed'}"
						>Completed</a>
					</li>
				</ul>
				<button
					v-show="todos.length > remaining"
					class="clear-completed"
					@click="removeCompleted"
				>
					Clear completed
				</button>
			</footer>
		</section>
	</div>
</template>

<script>
// visibility filters
const filters = {
	all: function (todos) {
		return todos;
	},
	active: function (todos) {
		return todos.filter(function (todo) {
			return !todo.completed;
		});
	},
	completed: function (todos) {
		return todos.filter(function (todo) {
			return todo.completed;
		});
	}
};

export default {
	name: "App",
	filters: {
		pluralize(n) {
			return n === 1 ? "item" : "items";
		}
	},
	directives: {
		"todo-focus": function (el, binding) {
			if (binding.value) {
				el.focus();
			}
		}
	},
	data() {
		return {
			todos: [],
			newTodo: "",
			editedTodo: null,
			visibility: "all"
		};
	},
	computed: {
		filteredTodos() {
			return filters[this.visibility](this.todos);
		},
		remaining() {
			return filters.active(this.todos).length;
		},
		allDone: {
			get() {
				return this.remaining === 0;
			},
			set(value) {
				this.todos.forEach(function (todo) {
					todo.completed = value;
				});
			}
		}
	},
	mounted() {
		window.addEventListener("hashchange", this.onHashChange);
		this.onHashChange();
	},
	methods: {
		addTodo() {
			const value = this.newTodo && this.newTodo.trim();
			if (!value) {
				return;
			}
			this.todos.push({
				title: value,
				completed: false
			});
			this.newTodo = "";
		},

		removeTodo: function (todo) {
			this.todos.splice(this.todos.indexOf(todo), 1);
		},

		editTodo(todo) {
			this.beforeEditCache = todo.title;
			this.editedTodo = todo;
		},

		doneEdit(todo) {
			if (!this.editedTodo) {
				return;
			}
			this.editedTodo = null;
			todo.title = todo.title.trim();
			if (!todo.title) {
				this.removeTodo(todo);
			}
		},

		cancelEdit(todo) {
			this.editedTodo = null;
			todo.title = this.beforeEditCache;
		},

		removeCompleted() {
			this.todos = filters.active(this.todos);
		},

		// handle routing
		onHashChange() {
			const visibility = window.location.hash.replace(/#\/?/, "");
			if (filters[visibility]) {
				this.visibility = visibility;
			} else {
				window.location.hash = "";
				this.visibility = "all";
			}
		},
	}
};
</script>