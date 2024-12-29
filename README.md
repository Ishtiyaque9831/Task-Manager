// Setting up the project
// 1. Install Create React App or use Vite for React setup
// npx create-react-app task-manager
// cd task-manager
// npm install @reduxjs/toolkit react-redux axios react-router-dom bootstrap

// Folder Structure:
// src/
//   components/
//     TaskList.js
//     AddTaskForm.js
//     EditTaskForm.js
//   redux/
//     store.js
//     taskSlice.js
//   App.js
//   index.js

// Redux Setup (src/redux/taskSlice.js)
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';

const API_URL = 'https://your-api-endpoint/tasks';

export const fetchTasks = createAsyncThunk('tasks/fetchTasks', async () => {
  const response = await axios.get(API_URL);
  return response.data;
});

export const addTask = createAsyncThunk('tasks/addTask', async (task) => {
  const response = await axios.post(API_URL, task);
  return response.data;
});

export const updateTask = createAsyncThunk('tasks/updateTask', async ({ id, task }) => {
  const response = await axios.put(`${API_URL}/${id}`, task);
  return response.data;
});

export const deleteTask = createAsyncThunk('tasks/deleteTask', async (id) => {
  await axios.delete(`${API_URL}/${id}`);
  return id;
});

const taskSlice = createSlice({
  name: 'tasks',
  initialState: { tasks: [], status: 'idle', error: null },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchTasks.fulfilled, (state, action) => {
        state.tasks = action.payload;
      })
      .addCase(addTask.fulfilled, (state, action) => {
        state.tasks.push(action.payload);
      })
      .addCase(updateTask.fulfilled, (state, action) => {
        const index = state.tasks.findIndex((task) => task.id === action.payload.id);
        if (index !== -1) {
          state.tasks[index] = action.payload;
        }
      })
      .addCase(deleteTask.fulfilled, (state, action) => {
        state.tasks = state.tasks.filter((task) => task.id !== action.payload);
      });
  },
});

export default taskSlice.reducer;

// Redux Store (src/redux/store.js)
import { configureStore } from '@reduxjs/toolkit';
import taskReducer from './taskSlice';

export const store = configureStore({
  reducer: {
    tasks: taskReducer,
  },
});

// Components
// TaskList.js
import React, { useEffect } from 'react';
import { useSelector, useDispatch } from 'react-redux';
import { fetchTasks, deleteTask } from '../redux/taskSlice';

const TaskList = ({ onEdit }) => {
  const dispatch = useDispatch();
  const { tasks, status } = useSelector((state) => state.tasks);

  useEffect(() => {
    if (status === 'idle') {
      dispatch(fetchTasks());
    }
  }, [status, dispatch]);

  const handleDelete = (id) => {
    dispatch(deleteTask(id));
  };

  return (
    <div className="task-list">
      <h2>Task List</h2>
      <ul>
        {tasks.map((task) => (
          <li key={task.id}>
            <h3>{task.title}</h3>
            <p>{task.description}</p>
            <p>Status: {task.completed ? 'Completed' : 'Pending'}</p>
            <button onClick={() => onEdit(task)}>Edit</button>
            <button onClick={() => handleDelete(task.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default TaskList;

// AddTaskForm.js
import React, { useState } from 'react';
import { useDispatch } from 'react-redux';
import { addTask } from '../redux/taskSlice';

const AddTaskForm = () => {
  const [title, setTitle] = useState('');
  const [description, setDescription] = useState('');
  const dispatch = useDispatch();

  const handleSubmit = (e) => {
    e.preventDefault();
    if (!title.trim()) return;
    dispatch(addTask({ title, description, completed: false }));
    setTitle('');
    setDescription('');
  };

  return (
    <form onSubmit={handleSubmit}>
      <h2>Add Task</h2>
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        placeholder="Task Title"
        required
      />
      <textarea
        value={description}
        onChange={(e) => setDescription(e.target.value)}
        placeholder="Task Description"
      />
      <button type="submit">Add Task</button>
    </form>
  );
};

export default AddTaskForm;

// EditTaskForm.js
import React, { useState } from 'react';
import { useDispatch } from 'react-redux';
import { updateTask } from '../redux/taskSlice';

const EditTaskForm = ({ task, onCancel }) => {
  const [title, setTitle] = useState(task.title);
  const [description, setDescription] = useState(task.description);
  const [completed, setCompleted] = useState(task.completed);
  const dispatch = useDispatch();

  const handleSubmit = (e) => {
    e.preventDefault();
    dispatch(updateTask({ id: task.id, task: { title, description, completed } }));
    onCancel();
  };

  return (
    <form onSubmit={handleSubmit}>
      <h2>Edit Task</h2>
      <input
        type="text"
        value={title}
        onChange={(e) => setTitle(e.target.value)}
        required
      />
      <textarea
        value={description}
        onChange={(e) => setDescription(e.target.value)}
      />
      <label>
        <input
          type="checkbox"
          checked={completed}
          onChange={(e) => setCompleted(e.target.checked)}
        />
        Completed
      </label>
      <button type="submit">Save</button>
      <button type="button" onClick={onCancel}>Cancel</button>
    </form>
  );
};

export default EditTaskForm;

// App.js
import React, { useState } from 'react';
import { Provider } from 'react-redux';
import { store } from './redux/store';
import TaskList from './components/TaskList';
import AddTaskForm from './components/AddTaskForm';
import EditTaskForm from './components/EditTaskForm';

const App = () => {
  const [editingTask, setEditingTask] = useState(null);

  return (
    <Provider store={store}>
      <div className="app">
        <h1>Task Manager</h1>
        {editingTask ? (
          <EditTaskForm task={editingTask} onCancel={() => setEditingTask(null)} />
        ) : (
          <AddTaskForm />
        )}
        <TaskList onEdit={(task) => setEditingTask(task)} />
      </div>
    </Provider>
  );
};

export default App;

```

