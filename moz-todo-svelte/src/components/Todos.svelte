<h1> ✅ Svelte to-do list</h1>


<!-- Todos.svelte -->
<div class="todoapp stack-large">
  <!-- NewTodo -->
  <NewTodo autofocus on:addTodo={(e) => addTodo(e.detail)} />
  <!-- Filter -->
  <FilterButton bind:filter={filter} />
  <!-- TodosStatus -->
  <TodosStatus bind:this={todosStatus} {todos} />
    <!-- To-dos -->
  <ul class="todo-list stack-large" aria-labelledby="list-heading">
    {#each filterTodos(filter, todos) as todo (todo.id)}
    <li class="todo">
      <Todo {todo} on:update={(e) => updateTodo(e.detail)} on:remove={(e) => removeTodo(e.detail)} />
    </li>
    {:else}
    <li>Nothing to do here! 😴</li>
    {/each}
  </ul>
</div>
  <hr />

  <!-- MoreActions -->
  <MoreActions {todos}
    on:checkAll={(e) => checkAllTodos(e.detail)}
    on:removeCompleted={removeCompletedTodos}
  />

<script>
  import FilterButton from "./FilterButton.svelte";
  import Todo from "./Todo.svelte";
  import MoreActions from "./MoreActions.svelte";
  import NewTodo from "./NewTodo.svelte";
  import TodosStatus from "./TodosStatus.svelte";
  import { alert } from "../stores.js";

  let todosStatus; // reference to TodosStatus instance
  
  export let todos = [];
  $: totalTodos = todos.length;
  $: completedTodos = todos.filter((todo) => todo.completed).length;

  let newTodoName = "";

  function addTodo(name) {
    todos = [...todos, { id: newTodoId, name, completed: false }];
    $alert = `Todo '${name}' has been added`;
  }

  let newTodoId;
  $: newTodoId = todos.length ? Math.max(...todos.map((t) => t.id)) + 1 : 1;

  function removeTodo(todo) {
    todos = todos.filter((t) => t.id !== todo.id);
    todosStatus.focus(); // give focus to status heading
    $alert = `Todo '${todo.name}' has been deleted`;
  }

  function updateTodo(todo) {
    const i = todos.findIndex((t) => t.id === todo.id);
    if (todos[i].name !== todo.name)
      $alert = `todo '${todos[i].name}' has been renamed to '${todo.name}'`;
    if (todos[i].completed !== todo.completed)
      $alert = `todo '${todos[i].name}' marked as ${
        todo.completed ? "completed" : "active"
      }`;
    todos[i] = { ...todos[i], ...todo };
  }

  let filter = "all";
  const filterTodos = (filter, todos) =>
    filter === "active"
      ? todos.filter((t) => !t.completed)
      : filter === "completed"
        ? todos.filter((t) => t.completed)
        : todos;

  $: {
    if (filter === "all") {
      $alert = "Browsing all to-dos";
    } else if (filter === "active") {
      $alert = "Browsing active to-dos";
    } else if (filter === "completed") {
      $alert = "Browsing completed to-dos";
    }
  }

  const checkAllTodos = (completed) => {
    todos = todos.map((t) => ({ ...t, completed }));
    $alert = `${completed ? "Checked" : "Unchecked"} ${todos.length} to-dos`;
  };
  const removeCompletedTodos = () => {
    $alert = `Removed ${todos.filter((t) => t.completed).length} to-dos`;
    todos = todos.filter((t) => !t.completed);
  };
</script>