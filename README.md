# IndexedDB Hook for Managing a Todo App

This code snippet demonstrates how to set up a custom React hook, `useIndexedDB`, to manage a `TodoApp` using IndexedDB. The hook leverages the `idb` library for easier interactions with IndexedDB and provides CRUD operations tailored to a `Todo` database schema. 

---

## Key Features

1. **Database Initialization:**
   - Defines the database schema using TypeScript's `DBSchema` interface.
   - Automatically sets up the database with a `todos` object store and a `by-date` index for querying by creation date.

2. **CRUD Operations:**
   - `addTodo`: Adds a new todo item.
   - `getTodos`: Retrieves all todo items.
   - `updateTodo`: Updates an existing todo item.
   - `deleteTodo`: Deletes a todo item by its ID.
   - `getTodosByDateRange`: Fetches todos within a specific date range using the `by-date` index.

3. **Error Handling:**
   - Tracks database initialization errors using React state.

4. **Automatic Cleanup:**
   - Ensures the database connection is closed when the component using the hook is unmounted.

---

## Code Structure

### Database Schema

The `TodoDB` interface defines the schema for the IndexedDB database, including the structure of the `todos` object store:

```typescript
interface TodoDB extends DBSchema {
    todos: {
        value: Todo;
        key: number;
        indexes: { 'by-date': Date };
    };
}
```

### Todo Type

Defines the structure of a Todo item:

```typescript
export interface Todo {
    id?: number;
    title: string;
    completed: boolean;
    createdAt: Date;
}
```

### Database Initialization

Initializes the `TodoApp` database and configures the `todos` object store with an auto-incrementing key and an index on `createdAt`:

```typescript
useEffect(() => {
    async function initDB() {
        try {
            const database = await openDB<TodoDB>(DB_NAME, DB_VERSION, {
                upgrade(db) {
                    if (!db.objectStoreNames.contains('todos')) {
                        const store = db.createObjectStore('todos', {
                            keyPath: 'id',
                            autoIncrement: true,
                        });
                        store.createIndex('by-date', 'createdAt');
                    }
                },
            });
            setDb(database);
        } catch (err) {
            setError(err instanceof Error ? err : new Error('Failed to initialize database'));
        }
    }

    initDB();

    return () => {
        db?.close();
    };
}, []);
```

### CRUD Operations

#### Add a Todo
```typescript
const addTodo = useCallback(async (todo: Omit<Todo, 'id'>) => {
    if (!db) throw new Error('Database not initialized');
    return await db.add('todos', { ...todo, createdAt: new Date() });
}, [db]);
```

#### Get All Todos
```typescript
const getTodos = useCallback(async () => {
    if (!db) throw new Error('Database not initialized');
    return await db.getAll('todos');
}, [db]);
```

#### Update a Todo
```typescript
const updateTodo = useCallback(async (todo: Todo) => {
    if (!db) throw new Error('Database not initialized');
    return await db.put('todos', todo);
}, [db]);
```

#### Delete a Todo
```typescript
const deleteTodo = useCallback(async (id: number) => {
    if (!db) throw new Error('Database not initialized');
    return await db.delete('todos', id);
}, [db]);
```

#### Get Todos by Date Range
```typescript
const getTodosByDateRange = useCallback(async (startDate: Date, endDate: Date) => {
    if (!db) throw new Error('Database not initialized');
    const range = IDBKeyRange.bound(startDate, endDate);
    return await db.getAllFromIndex('todos', 'by-date', range);
}, [db]);
```

---

## Usage

Here's how you can use the `useIndexedDB` hook in a React component:

```typescript
import React, { useEffect } from 'react';
import { useIndexedDB, Todo } from './useIndexedDB';

const TodoApp = () => {
    const { addTodo, getTodos, error } = useIndexedDB();

    useEffect(() => {
        async function fetchTodos() {
            const todos = await getTodos();
            console.log('Todos:', todos);
        }
        fetchTodos();
    }, [getTodos]);

    const handleAddTodo = async () => {
        const newTodo: Omit<Todo, 'id'> = { title: 'Learn IndexedDB', completed: false, createdAt: new Date() };
        await addTodo(newTodo);
        console.log('Todo added!');
    };

    return (
        <div>
            <h1>Todo App</h1>
            {error && <p>Error: {error.message}</p>}
            <button onClick={handleAddTodo}>Add Todo</button>
        </div>
    );
};

export default TodoApp;
```

---

## Dependencies

- **idb**: A modern wrapper for IndexedDB.
  ```bash
  npm install idb
  ```

---

## Advantages

- **Type Safety**: Strongly typed schema and operations using TypeScript.
- **Performance**: Efficiently manages large data sets using IndexedDB.
- **Reusability**: Encapsulates database logic into a reusable custom hook.

---

This hook provides a robust solution for integrating IndexedDB into React applications, making it easier to manage offline data and enhance application performance.
