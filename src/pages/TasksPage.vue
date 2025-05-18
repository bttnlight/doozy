<template>
  <q-page class="bg-white">
    <div class="tasks-container">
      <!-- Header -->
      <div class="header-section">
        <h3 class="section-title">Your Full To-do List</h3>
        <div class="section-divider"></div>
      </div>

      <!-- Column Headers -->
      <div class="column-headers">
        <div class="col-span-1"></div>
        <div class="col-span-5"></div>

        <div
          class="column-label col-span-3 cursor-pointer flex items-center gap-8"
          @click="toggleSortByDate"
        >
          <span>Date</span>
          <q-icon
            v-if="sortState !== 'none'"
            :name="sortState === 'asc' ? 'arrow_upward' : 'arrow_downward'"
            size="16px"
          />
        </div>

        <div
          class="column-label col-span-2 relative cursor-pointer flex items-center gap-2"
          @click="categoryFilterMenu = false"
        >
          <span> Category </span>

          <q-icon name="arrow_drop_down" size="20px" class="q-ml-xs" />

          <q-menu v-model="categoryFilterMenu" anchor="bottom middle" self="top middle">
            <q-list class="manrope" style="min-width: 140px">
              <q-item
                v-for="option in ['All', 'Work', 'Personal', 'Finance', 'Education', 'Health']"
                :key="option"
                clickable
                v-close-popup
                @click="selectedCategory = option"
                :class="{ 'bg-grey-3': selectedCategory === option }"
              >
                <q-item-section>{{ option }}</q-item-section>
              </q-item>
            </q-list>
          </q-menu>
        </div>

        <div class="col-span-1"></div>
      </div>

      <!-- Add Task Button -->
      <div class="add-task-button task-row" @click="openNewTaskModal" style="cursor: pointer">
        <div class="col-span-1 flex items-center justify-center">
          <q-icon name="add" color="gray" />
        </div>
        <div class="col-span-5 task-title manrope text-gray-500">Add New Task</div>
        <div class="col-span-6"></div>
      </div>

      <!-- Task List -->
      <div class="task-list">
        <div
          v-for="(task, index) in sortedFilteredTasks"
          :key="task.id ?? index"
          class="task-row"
          @click="editTask(task)"
        >
          <div class="col-span-1 flex items-center justify-center">
            <q-checkbox
              :model-value="task.completed"
              @update:model-value="(val) => updateTaskCompletion(task, val)"
              color="black"
              @click.stop
            />
          </div>
          <div
            class="col-span-5 task-title manrope"
            :class="{ 'line-through text-gray-400': task.completed }"
          >
            {{ task.title }}
          </div>
          <div class="col-span-3 task-meta manrope">
            {{ task.dueDate }}
          </div>
          <div class="col-span-2 task-meta manrope">
            {{ task.category }}
          </div>
          <div class="col-span-1 text-right">
            <q-btn flat round color="gray" icon="more_horiz" size="sm" @click.stop>
              <q-menu>
                <q-list style="min-width: 100px">
                  <q-item clickable v-close-popup @click.stop="deleteTask(index)">
                    <q-item-section class="manrope">Delete</q-item-section>
                  </q-item>
                </q-list>
              </q-menu>
            </q-btn>
          </div>
        </div>

        <div v-if="tasks.length === 0" class="empty-state manrope">
          No tasks found. Add a new task to get started!
        </div>
      </div>
    </div>

    <TaskModal
      v-model="newTaskModal"
      :modelTask="newTask"
      :categoryOptions="categoryOptions"
      :isEditing="editingIndex >= 0"
      @save="handleTaskSave"
    />
  </q-page>
</template>

<script lang="ts">
import TaskModal from 'components/TaskModal.vue';
import { defineComponent } from 'vue';
import axios from 'axios';

const API_URL = 'https://jsonplaceholder.typicode.com/todos';

type Task = {
  id?: number;
  title: string;
  dueDate: string;
  category: string;
  description: string;
  completed: boolean;
};

interface ApiTask {
  userId: number;
  id: number;
  title: string;
  completed: boolean;
}

function createEmptyTask(): Task {
  return {
    title: '',
    dueDate: new Date().toISOString().slice(0, 10),
    category: '',
    description: '',
    completed: false,
  };
}

export default defineComponent({
  name: 'TasksPage',

  watch: {
    selectedCategory(newVal) {
      console.log('Category changed:', newVal);
    },
  },

  mounted() {
    this.fetchTasks().catch((error) => {
      console.error('Failed to fetch tasks on mount:', error);
    });
  },

  data() {
    return {
      tasks: [] as Task[],
      newTask: createEmptyTask(),
      newTaskModal: false,
      categoryOptions: [
        { label: 'Work', value: 'Work' },
        { label: 'Personal', value: 'Personal' },
        { label: 'Health', value: 'Health' },
        { label: 'Finance', value: 'Finance' },
        { label: 'Education', value: 'Education' },
      ],
      editingIndex: -1,
      sortState: 'none' as 'none' | 'asc' | 'desc',
      selectedCategory: 'All',
      categoryFilterMenu: false,
    };
  },

  computed: {
    filteredTasks(): Task[] {
      if (this.selectedCategory === 'All') return this.tasks;
      return this.tasks.filter((task) => task.category === this.selectedCategory);
    },

    sortedFilteredTasks(): Task[] {
      const tasksToSort = [...this.filteredTasks];
      if (this.sortState === 'asc') {
        return tasksToSort.sort(
          (a, b) => new Date(a.dueDate).getTime() - new Date(b.dueDate).getTime(),
        );
      } else if (this.sortState === 'desc') {
        return tasksToSort.sort(
          (a, b) => new Date(b.dueDate).getTime() - new Date(a.dueDate).getTime(),
        );
      }
      return tasksToSort;
    },
  },

  components: {
    TaskModal,
  },

  methods: {
    toggleSortByDate() {
      if (this.sortState === 'none') this.sortState = 'asc';
      else if (this.sortState === 'asc') this.sortState = 'desc';
      else this.sortState = 'none';
    },

    openNewTaskModal() {
      this.newTask = createEmptyTask();
      this.editingIndex = -1;
      this.newTaskModal = true;
    },

    async saveTask() {
      if (!this.newTask.title) {
        this.$q.notify({
          color: 'negative',
          message: 'Please enter a task title',
          icon: 'warning',
        });
        return;
      }

      try {
        if (this.editingIndex >= 0) {
          const taskId = this.tasks[this.editingIndex]?.id;
          if (!taskId || typeof taskId !== 'number' || taskId < 1 || taskId > 200) {
            console.error('Invalid task ID for update:', taskId);
            return;
          }

          await axios.put(`${API_URL}/${taskId}`, { ...this.newTask, id: taskId });
          this.tasks.splice(this.editingIndex, 1, { ...this.newTask, id: taskId });
        } else {
          const response = await axios.post(API_URL, this.newTask);

          const newId =
            typeof response.data.id === 'number' && response.data.id <= 200
              ? response.data.id
              : this.generateLocalId();

          this.tasks.push({ ...this.newTask, id: newId });
        }

        this.newTaskModal = false;
        this.$q.notify({ color: 'positive', message: 'Task saved successfully!' });
      } catch (error) {
        this.$q.notify({
          color: 'negative',
          message: 'Failed to save task.',
          icon: 'error',
        });
        console.error('Failed to save task:', error);
      }
    },

    async updateTaskCompletion(task: Task, completed: boolean) {
      task.completed = completed;
      if (!task.id || typeof task.id !== 'number' || task.id < 1 || task.id > 200) {
        console.warn('Skipping updateTaskCompletion due to invalid or missing task ID:', task.id);
        return;
      }

      try {
        await axios.put(`${API_URL}/${task.id}`, task);

        const index = this.tasks.findIndex((t) => t.id === task.id);
        if (index !== -1 && this.tasks[index]) {
          this.tasks[index].completed = task.completed;
        }

        this.$q.notify({
          color: 'positive',
          message: `Task "${task.title}" marked as ${completed ? 'completed' : 'incomplete'}.`,
          icon: 'check_circle',
        });

        console.log(`Task ${task.id} completion updated to`, task.completed);
      } catch (error) {
        this.$q.notify({
          color: 'negative',
          message: `Failed to update completion status for "${task.title}".`,
          icon: 'error',
        });
        console.error('Failed to update task completion:', error);
      }
    },

    editTask(task: Task) {
      this.newTask = { ...task };
      this.editingIndex = this.tasks.findIndex((t) => t.id === task.id);
      console.log('Editing task with id:', task.id, 'at index:', this.editingIndex);
      this.newTaskModal = true;
    },

    deleteTask(index: number) {
      const task = this.tasks[index];
      if (!task?.id || typeof task.id !== 'number' || task.id < 1 || task.id > 200) {
        console.warn('Invalid task ID for deletion:', task?.id);
        return;
      }

      this.$q
        .dialog({
          title: 'Confirm',
          message: 'Are you sure you want to delete this task?',
          cancel: true,
          persistent: true,
        })
        .onOk(() => {
          void (async () => {
            try {
              await axios.delete(`${API_URL}/${task.id}`);
              this.tasks.splice(index, 1);

              this.$q.notify({
                color: 'positive',
                message: `Task "${task.title}" deleted successfully.`,
                icon: 'delete',
              });
            } catch (error) {
              this.$q.notify({
                color: 'negative',
                message: `Failed to delete task "${task.title}".`,
                icon: 'error',
              });
              console.error('Failed to delete task:', error);
            }
          })();
        });
    },

    async handleTaskSave(task: Task) {
      this.newTask = task;
      await this.saveTask();
    },

    async fetchTasks() {
      try {
        const response = await axios.get<ApiTask[]>(`${API_URL}?_limit=10`);
        this.tasks = response.data.map((task) => ({
          id: task.id,
          title: task.title,
          description: '',
          dueDate: new Date().toISOString().slice(0, 10),
          category: 'Work',
          completed: task.completed ?? false,
        }));
      } catch (error) {
        console.error('Failed to fetch tasks:', error);
      }
    },

    generateLocalId() {
      const ids = this.tasks.map((t) => t.id).filter((id): id is number => id !== undefined);
      let maxId = 200;
      ids.forEach((id) => {
        if (id > maxId) maxId = id;
      });
      return maxId + 1;
    },
  },
});
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;600;700&display=swap');

.manrope {
  font-family: 'Manrope', sans-serif;
}

/* Layout & Structure */
.tasks-container {
  margin: 1rem;
  padding: 0;
}

.header-section {
  margin: 0;
}

.section-title {
  font-weight: bold;
  color: #000;
}

.section-divider {
  height: 1px;
  background-color: #a1a1a1;
  margin-top: 0.5rem;
  margin-bottom: 0.5rem;
}

.column-headers {
  display: grid;
  grid-template-columns: repeat(12, minmax(0, 1fr));
  gap: 1rem;
  padding: 0.25rem 1rem;
  align-items: center;
}

.column-label:hover {
  background-color: #f5f5f5;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out;
}

.header-label {
  grid-column: span 3 / span 3;
  font-size: 0.875rem;
  color: #6b7280;
}

.add-task-button {
  border-top: 1px solid #a1a1a1;
  border-bottom: 1px solid #a1a1a1;
  padding: 0.5rem 1rem;
  height: 3rem;
}

.task-list {
  border-color: #f3f4f6;
}

.task-row {
  display: grid;
  grid-template-columns: repeat(12, minmax(0, 1fr));
  gap: 1rem;
  padding: 0.25rem 1rem;
  align-items: center;
  border-bottom: 1px solid #a1a1a1;
}

.task-row:hover {
  background-color: #f5f5f5;
  cursor: pointer;
  transition: background-color 0.2s ease-in-out;
}

.task-title {
  grid-column: span 5 / span 5;
}

.task-meta {
  font-size: 0.875rem;
  color: #6b7280;
}

.empty-state {
  padding: 2rem 0;
  text-align: center;
  color: #a1a1a1;
  font-style: italic;
}
</style>
