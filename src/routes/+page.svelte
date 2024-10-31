<script lang="ts">
	import { fade, fly, slide } from 'svelte/transition';

	type Todo = {
		text: string;
		done: boolean;
	};
	type Filters = 'all' | 'active' | 'completed';
	let todos = $state<Todo[]>([
		{ text: 'Learn Svelte', done: false },
		{ text: 'Learn Sapper', done: false },
		{ text: 'Learn SvelteKit', done: false }
	]);

	let filter = $state<Filters>('all');
	let filteredTodos = $derived(filterTodos());

	$effect(() => {
		const savedTodos = localStorage.getItem('todos');
		savedTodos && (todos = JSON.parse(savedTodos));
	});

	$effect(() => {
		localStorage.setItem('todos', JSON.stringify(todos));
	});

	function addTodo(event: KeyboardEvent) {
		if (event.key !== 'Enter') return;

		const todoEl = event.target as HTMLInputElement;
		const text = todoEl.value;
		const done = false;

		todos = [...todos, { text, done }];

		todoEl.value = '';
	}

	function editTodo(event: Event) {
		const inputEl = event.target as HTMLInputElement;
		const index = +inputEl.dataset.index!;
		todos[index].text = inputEl.value;
	}

	function toggleTodo(event: Event) {
		const inputEl = event.target as HTMLInputElement;
		const index = +inputEl.dataset.index!;
		todos[index].done = !todos[index].done;
	}

	function setFilter(newFilter: Filters) {
		filter = newFilter;
	}

	function filterTodos() {
		switch (filter) {
			case 'all':
				return todos;
			case 'active':
				return todos.filter((todo) => !todo.done);
			case 'completed':
				return todos.filter((todo) => todo.done);
		}
	}

	function remaining() {
		return todos.filter((todo) => !todo.done).length;
	}
</script>

<div>
	<input type="text" placeholder="Add Todo" onkeydown={addTodo} />
</div>
<div>
	<div class="todos">
		{#each filteredTodos as todo, i}
			<div class="todo">
				<input oninput={editTodo} data-index={i} type="text" value={todo.text} />
				<input type="checkbox" bind:checked={todo.done} />
			</div>
		{/each}
	</div>
</div>
<div class="filters">
	{#each ['all', 'active', 'completed'] as filterName}
		<button
			class={filter === filterName ? 'active-btn' : ''}
			onclick={() => setFilter(filterName as Filters)}>{filterName}</button
		>
	{/each}
</div>

<p>{remaining()} remaining</p>

<style>
	.todos {
		display: grid;
		gap: 1rem;
		height: 100%;

		margin-block-start: 1rem;
	}

	.todo {
		position: relative;
	}
	input {
		background-color: hsl(220 10% 12%);
		color: hsl(220 10% 98%);
		border: none;
		border-radius: 0.5rem;
		font-weight: 400;
		font-size: 1.2rem;
	}
	input[type='text'] {
		width: 100%;
		padding: 1rem;
	}
	input[type='checkbox'] {
		position: absolute;
		right: 0;
		top: 50%;
		translate: 0% -50%;

		width: 1.1rem;
		height: 1.1rem;
	}
	input[type='checkbox']:checked {
		background-color: hsl(220 10% 98%);
	}
	input[type='checkbox']:hover {
		cursor: pointer;
	}

	.filters {
		display: flex;
		gap: 1rem;
		margin-block-start: 1rem;
	}

	.filters button {
		background-color: hsl(220 10% 12%);
		color: hsl(220 10% 98%);
		border: none;
		border-radius: 0.5rem;
		font-weight: 400;
		font-size: 1.2rem;
		padding: 10px 20px;
	}
	.active-btn {
		background-color: blue !important;
	}
</style>
