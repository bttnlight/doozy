<template>
  <q-page class="tasks-page">
    <div class="tasks-container">
      <!-- Header -->
      <div class="header-section">
        <div class="header-content">
          <h2 class="page-title manrope"><span class="gradient-text">Doozy</span> Tasks</h2>
          <div class="header-actions" :class="{ 'q-gutter-y-sm': $q.screen.lt.md }">
            <q-input
              v-model="searchQuery"
              outlined
              dense
              placeholder="Search tasks..."
              class="search-input manrope"
            >
              <template v-slot:prepend>
                <q-icon name="search" />
              </template>
              <template v-slot:append v-if="searchQuery">
                <q-icon name="close" class="cursor-pointer" @click="searchQuery = ''" />
              </template>
            </q-input>
            <q-btn
              color="primary"
              icon="add"
              label="New Task"
              class="new-task-btn manrope"
              @click="openNewTaskModal"
            />
          </div>
        </div>
        <div class="filters-row" :class="{ 'q-gutter-y-sm': $q.screen.lt.md }">
          <div class="filter-chips">
            <q-scroll-area
              horizontal
              style="height: 42px; width: 100%"
              :thumb-style="{ display: 'none' }"
              v-if="$q.screen.lt.md"
            >
              <div class="row no-wrap q-px-sm">
                <q-chip
                  v-for="category in ['All', 'Work', 'Personal', 'Finance', 'Education', 'Health']"
                  :key="category"
                  :selected="selectedCategory === category"
                  clickable
                  @click="selectedCategory = category"
                  :color="selectedCategory === category ? getCategoryColor(category) : 'grey-3'"
                  text-color="white"
                  class="filter-chip manrope q-mx-xs"
                  :class="{ 'selected-chip': selectedCategory === category }"
                >
                  {{ category }}
                </q-chip>
              </div>
            </q-scroll-area>

            <template v-if="!$q.screen.lt.md">
              <q-chip
                v-for="category in ['All', 'Work', 'Personal', 'Finance', 'Education', 'Health']"
                :key="category"
                :selected="selectedCategory === category"
                clickable
                @click="selectedCategory = category"
                :color="selectedCategory === category ? getCategoryColor(category) : 'grey-3'"
                text-color="white"
                class="filter-chip manrope"
                :class="{ 'selected-chip': selectedCategory === category }"
              >
                {{ category }}
              </q-chip>
            </template>
          </div>
          <div class="sort-dropdown">
            <q-btn-dropdown
              color="grey-7"
              flat
              dense
              label="Sort"
              icon="sort"
              class="sort-btn manrope"
              :class="{ 'full-width': $q.screen.lt.md }"
            >
              <q-list>
                <q-item clickable v-close-popup @click="setSortBy('dueDate', 'asc')">
                  <q-item-section>
                    <q-item-label>Date (Earliest first)</q-item-label>
                  </q-item-section>
                  <q-item-section avatar v-if="sortField === 'dueDate' && sortOrder === 'asc'">
                    <q-icon name="check" />
                  </q-item-section>
                </q-item>
                <q-item clickable v-close-popup @click="setSortBy('dueDate', 'desc')">
                  <q-item-section>
                    <q-item-label>Date (Latest first)</q-item-label>
                  </q-item-section>
                  <q-item-section avatar v-if="sortField === 'dueDate' && sortOrder === 'desc'">
                    <q-icon name="check" />
                  </q-item-section>
                </q-item>
                <q-item clickable v-close-popup @click="setSortBy('title', 'asc')">
                  <q-item-section>
                    <q-item-label>Title (A-Z)</q-item-label>
                  </q-item-section>
                  <q-item-section avatar v-if="sortField === 'title' && sortOrder === 'asc'">
                    <q-icon name="check" />
                  </q-item-section>
                </q-item>
                <q-item clickable v-close-popup @click="setSortBy('priority', 'desc')">
                  <q-item-section>
                    <q-item-label>Priority (High-Low)</q-item-label>
                  </q-item-section>
                  <q-item-section avatar v-if="sortField === 'priority' && sortOrder === 'desc'">
                    <q-icon name="check" />
                  </q-item-section>
                </q-item>
              </q-list>
            </q-btn-dropdown>
          </div>
        </div>
      </div>

      <!-- Task List -->
      <div class="task-list-container">
        <div class="task-list-header">
          <div class="task-header-item status-col">Status</div>
          <div class="task-header-item title-col">Task</div>
          <div class="task-header-item date-col">Due Date</div>
          <div class="task-header-item category-col">Category</div>
          <div class="task-header-item priority-col">Priority</div>
          <div class="task-header-item actions-col"></div>
        </div>

        <div class="task-list">
          <div
            v-for="(task, index) in sortedFilteredTasks"
            :key="task.id ?? index"
            class="task-card"
            :class="{ 'completed-task': task.completed }"
          >
            <div class="task-status status-col">
              <q-checkbox
                :model-value="task.completed"
                @update:model-value="(val) => updateTaskCompletion(task, val)"
                color="primary"
                size="md"
                class="task-checkbox"
              />
            </div>
            <div class="task-content title-col" @click="editTask(task)">
              <div class="task-title manrope" :class="{ 'completed-title': task.completed }">
                {{ task.title }}
              </div>
              <div class="task-description" v-if="task.description">
                {{ truncateDescription(task.description) }}
              </div>
            </div>
            <div class="task-date date-col" @click="editTask(task)">
              <q-chip
                outline
                size="sm"
                :color="isOverdue(task.dueDate) ? 'negative' : 'grey'"
                class="date-chip"
              >
                <q-icon name="event" size="xs" class="q-mr-xs" />
                {{ formatDate(task.dueDate) }}
              </q-chip>
            </div>
            <div class="task-category category-col" @click="editTask(task)">
              <q-chip
                size="sm"
                :color="getCategoryColor(task.category)"
                text-color="white"
                class="category-chip"
              >
                {{ task.category }}
              </q-chip>
            </div>
            <div class="task-priority priority-col" @click="editTask(task)">
              <q-badge :color="getPriorityColor(task.priority)" class="priority-badge">
                {{ task.priority || 'Medium' }}
              </q-badge>
            </div>
            <div class="task-actions actions-col">
              <q-btn flat round color="grey-7" icon="more_vert" size="sm">
                <q-menu anchor="bottom right" self="top right">
                  <q-list style="min-width: 150px">
                    <q-item clickable v-close-popup @click="editTask(task)">
                      <q-item-section avatar>
                        <q-icon name="edit" />
                      </q-item-section>
                      <q-item-section class="manrope">Edit</q-item-section>
                    </q-item>
                    <q-item clickable v-close-popup @click="duplicateTask(task)">
                      <q-item-section avatar>
                        <q-icon name="content_copy" />
                      </q-item-section>
                      <q-item-section class="manrope">Duplicate</q-item-section>
                    </q-item>
                    <q-separator />
                    <q-item
                      clickable
                      v-close-popup
                      @click="deleteTask(index)"
                      class="text-negative"
                    >
                      <q-item-section avatar>
                        <q-icon name="delete" color="negative" />
                      </q-item-section>
                      <q-item-section class="manrope">Delete</q-item-section>
                    </q-item>
                  </q-list>
                </q-menu>
              </q-btn>
            </div>
          </div>

          <div v-if="sortedFilteredTasks.length === 0" class="empty-state manrope">
            <q-icon name="task_alt" size="4rem" color="grey-4" />
            <p v-if="searchQuery">No tasks found matching "{{ searchQuery }}"</p>
            <p v-else-if="selectedCategory !== 'All'">
              No tasks in the "{{ selectedCategory }}" category
            </p>
            <p v-else>No tasks found. Add a new task to get started!</p>
            <q-btn color="primary" label="Add Task" @click="openNewTaskModal" class="q-mt-md" />
          </div>
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
import { useQuasar } from 'quasar';

const API_URL = 'https://jsonplaceholder.typicode.com/todos';

type Task = {
  id?: number;
  title: string;
  dueDate: string;
  category: string;
  description: string;
  completed: boolean;
  priority?: string;
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
    priority: 'medium',
  };
}

export default defineComponent({
  name: 'TasksPage',

  setup() {
    return {
      $q: useQuasar(),
    };
  },

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
      sortField: 'dueDate',
      sortOrder: 'asc',
      selectedCategory: 'All',
      searchQuery: '',
    };
  },

  computed: {
    filteredTasks(): Task[] {
      let filtered = this.tasks;

      // Filter by category
      if (this.selectedCategory !== 'All') {
        filtered = filtered.filter((task) => task.category === this.selectedCategory);
      }

      // Filter by search query
      if (this.searchQuery) {
        const query = this.searchQuery.toLowerCase();
        filtered = filtered.filter(
          (task) =>
            task.title.toLowerCase().includes(query) ||
            task.description.toLowerCase().includes(query),
        );
      }

      return filtered;
    },

    sortedFilteredTasks(): Task[] {
      const tasksToSort = [...this.filteredTasks];

      return tasksToSort.sort((a, b) => {
        if (this.sortField === 'dueDate') {
          const dateA = new Date(a.dueDate).getTime();
          const dateB = new Date(b.dueDate).getTime();
          return this.sortOrder === 'asc' ? dateA - dateB : dateB - dateA;
        } else if (this.sortField === 'title') {
          return this.sortOrder === 'asc'
            ? a.title.localeCompare(b.title)
            : b.title.localeCompare(a.title);
        } else if (this.sortField === 'priority') {
          const priorityMap: Record<string, number> = {
            high: 3,
            medium: 2,
            low: 1,
            '': 0,
          };
          const priorityA = priorityMap[a.priority || ''] || 0;
          const priorityB = priorityMap[b.priority || ''] || 0;
          return this.sortOrder === 'asc' ? priorityA - priorityB : priorityB - priorityA;
        }
        return 0;
      });
    },
  },

  components: {
    TaskModal,
  },

  methods: {
    setSortBy(field: string, order: 'asc' | 'desc') {
      this.sortField = field;
      this.sortOrder = order;
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
        this.$q.notify({
          color: 'positive',
          message: 'Task saved successfully!',
          icon: 'check_circle',
          position: 'top-right',
        });
      } catch (error) {
        this.$q.notify({
          color: 'negative',
          message: 'Failed to save task.',
          icon: 'error',
          position: 'top-right',
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
          position: 'top-right',
        });

        console.log(`Task ${task.id} completion updated to`, task.completed);
      } catch (error) {
        this.$q.notify({
          color: 'negative',
          message: `Failed to update completion status for "${task.title}".`,
          icon: 'error',
          position: 'top-right',
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

    async duplicateTask(task: Task) {
      const duplicatedTask = {
        ...task,
        title: `${task.title} (Copy)`,
        completed: false,
      };

      delete duplicatedTask.id;

      this.newTask = duplicatedTask;
      this.editingIndex = -1;
      try {
        await this.saveTask();
      } catch (error) {
        console.error('Failed to duplicate task:', error);
      }
    },

    async deleteTask(index: number) {
      const task = this.tasks[index];
      if (!task?.id || typeof task.id !== 'number' || task.id < 1 || task.id > 200) {
        console.warn('Invalid task ID for deletion:', task?.id);
        this.$q.notify({
          color: 'negative',
          message: 'Invalid task ID, cannot delete.',
          icon: 'error',
        });
        return;
      }

      try {
        const confirmed = await new Promise<boolean>((resolve) => {
          this.$q
            .dialog({
              title: 'Delete Task',
              message: `Are you sure you want to delete "${task.title}"?`,
              persistent: true,
              class: 'delete-dialog',
              ok: {
                label: 'Delete',
                color: 'negative',
              },
              cancel: {
                flat: true,
              },
            })
            .onOk(() => resolve(true))
            .onCancel(() => resolve(false));
        });

        if (confirmed) {
          await axios.delete(`${API_URL}/${task.id}`);
          this.tasks.splice(index, 1);
          this.$q.notify({
            color: 'positive',
            message: `Task "${task.title}" deleted successfully.`,
            icon: 'check_circle',
            position: 'top-right',
          });
        }
      } catch (error) {
        this.$q.notify({
          color: 'negative',
          message: `Failed to delete task "${task.title}".`,
          icon: 'error',
          position: 'top-right',
        });
        console.error('Failed to delete task:', error);
      }
    },

    async handleTaskSave(task: Task) {
      this.newTask = task;
      await this.saveTask();
    },

    async fetchTasks() {
      const categories = ['Work', 'Personal', 'Finance', 'Education', 'Health'];
      const priorities = ['low', 'medium', 'high'];

      try {
        const response = await axios.get<ApiTask[]>(`${API_URL}?_limit=10`);
        this.tasks = response.data.map((task) => {
          const randomDaysAhead = Math.floor(Math.random() * 30) - 10; // Some tasks can be overdue
          const dueDate = new Date();
          dueDate.setDate(dueDate.getDate() + randomDaysAhead);

          const randomCategory =
            categories[Math.floor(Math.random() * categories.length)] ?? 'Work';

          const randomPriority =
            priorities[Math.floor(Math.random() * priorities.length)] ?? 'medium';

          return {
            id: task.id,
            title: task.title,
            description: this.generateRandomDescription(),
            dueDate: dueDate.toISOString().slice(0, 10),
            category: randomCategory,
            completed: task.completed ?? false,
            priority: randomPriority,
          };
        });
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

    generateRandomDescription(): string {
      const descriptions = [
        'Need to complete this by the deadline.',
        'This is an important task that requires attention.',
        'Follow up on this task as soon as possible.',
        'Make sure to allocate enough time for this task.',
        'This task is part of the quarterly objectives.',
        '', // This is a valid empty string, not `undefined`
        'Remember to check with the team before completing.',
        'This is a recurring task that needs to be done regularly.',
        '', // Again, still a valid string
      ];
      return descriptions[Math.floor(Math.random() * descriptions.length)] as string;
    },

    truncateDescription(description: string) {
      if (!description) return '';

      const length = this.$q.screen.lt.sm ? 30 : this.$q.screen.lt.md ? 40 : 60;

      if (description.length <= length) return description;
      return description.substring(0, length) + '...';
    },

    formatDate(dateString: string) {
      const date = new Date(dateString);
      const today = new Date();
      const tomorrow = new Date();
      tomorrow.setDate(today.getDate() + 1);

      if (date.toDateString() === today.toDateString()) {
        return 'Today';
      } else if (date.toDateString() === tomorrow.toDateString()) {
        return 'Tomorrow';
      } else {
        return date.toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
      }
    },

    isOverdue(dateString: string) {
      const date = new Date(dateString);
      const today = new Date();
      today.setHours(0, 0, 0, 0);
      return date < today;
    },

    getCategoryColor(category: string): string {
      const colorMap: Record<string, string> = {
        Work: 'blue-6',
        Personal: 'purple-6',
        Health: 'green-6',
        Finance: 'amber-6',
        Education: 'red-6',
        All: 'primary',
      };
      return colorMap[category] || 'grey-6';
    },

    getPriorityColor(priority?: string): string {
      const colorMap: Record<string, string> = {
        high: 'negative',
        medium: 'warning',
        low: 'positive',
      };
      return colorMap[priority || 'medium'] || 'grey-6';
    },
  },
});
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Manrope:wght@400;500;600;700;800&display=swap');

.manrope {
  font-family: 'Manrope', sans-serif;
}

/* Page Layout */
.tasks-page {
  background-color: #f8fafc;
  min-height: 100vh;
}

.tasks-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem 1.5rem;
}

/* Header Section */
.header-section {
  margin-bottom: 2rem;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.page-title {
  font-size: 2rem;
  font-weight: 800;
  margin: 0;
}

.gradient-text {
  background: linear-gradient(90deg, #4f46e5, #8b5cf6, #ec4899);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.header-actions {
  display: flex;
  gap: 1rem;
  align-items: center;
}

.search-input {
  width: 250px;
}

.new-task-btn {
  border-radius: 8px;
  font-weight: 600;
}

.filters-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 1rem;
}

.filter-chips {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.filter-chip {
  font-weight: 600;
  transition: all 0.2s ease;
}

.filter-chip:not(.selected-chip) {
  background-color: #f1f5f9 !important;
  color: #64748b !important;
}

.selected-chip {
  transform: scale(1.05);
}

.sort-btn {
  font-weight: 600;
}

/* Task List */
.task-list-container {
  background-color: white;
  border-radius: 12px;
  box-shadow:
    0 4px 6px -1px rgba(0, 0, 0, 0.1),
    0 2px 4px -1px rgba(0, 0, 0, 0.06);
  overflow: hidden;
}

.task-list-header {
  display: grid;
  grid-template-columns: 60px 3fr 1fr 1fr 1fr 60px;
  gap: 0.5rem;
  padding: 1rem;
  background-color: #f8fafc;
  border-bottom: 1px solid #e2e8f0;
}

.task-header-item {
  font-size: 0.875rem;
  font-weight: 600;
  color: #64748b;
  display: flex;
  align-items: center;
}

.task-list {
  max-height: 70vh;
  overflow-y: auto;
}

.task-card {
  display: grid;
  grid-template-columns: 60px 3fr 1fr 1fr 1fr 60px;
  gap: 0.5rem;
  padding: 1rem;
  border-bottom: 1px solid #e2e8f0;
  transition: all 0.2s ease;
}

.task-card:hover {
  background-color: #f8fafc;
}

.completed-task {
  background-color: #f8fafc;
}

.task-status {
  display: flex;
  align-items: center;
  justify-content: center;
}

.task-content {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
  cursor: pointer;
}

.task-title {
  font-weight: 600;
  color: #1e293b;
}

.completed-title {
  text-decoration: line-through;
  color: #94a3b8;
}

.task-description {
  font-size: 0.875rem;
  color: #64748b;
}

.task-date,
.task-category,
.task-priority {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.date-chip,
.category-chip {
  font-size: 0.75rem;
  font-weight: 600;
}

.priority-badge {
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: capitalize;
}

.task-actions {
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Empty State */
.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 4rem 2rem;
  color: #94a3b8;
  text-align: center;
}

/* Responsive Adjustments */
@media (max-width: 1024px) {
  .tasks-container {
    padding: 1.5rem 1rem;
  }
}

@media (max-width: 768px) {
  .header-content {
    flex-direction: column;
    align-items: flex-start;
    gap: 1rem;
  }

  .header-actions {
    width: 100%;
    flex-direction: column;
  }

  .search-input {
    width: 100%;
  }

  .new-task-btn {
    width: 100%;
  }

  .filters-row {
    flex-direction: column;
    align-items: flex-start;
    gap: 1rem;
  }

  .filter-chips {
    width: 100%;
    overflow-x: auto;
    padding-bottom: 0.5rem;
    flex-wrap: nowrap;
  }

  .sort-dropdown {
    width: 100%;
  }

  .sort-btn {
    width: 100%;
  }

  .task-list-header {
    grid-template-columns: 60px 3fr 1fr 1fr 60px;
  }

  .task-card {
    grid-template-columns: 60px 3fr 1fr 1fr 60px;
  }

  .priority-col {
    display: none;
  }
}

@media (max-width: 640px) {
  .page-title {
    font-size: 1.5rem;
  }

  .task-list-header {
    grid-template-columns: 60px 3fr 1fr 60px;
  }

  .task-card {
    grid-template-columns: 60px 3fr 1fr 60px;
  }

  .category-col {
    display: none;
  }

  .date-col {
    text-align: right;
  }
}

@media (max-width: 480px) {
  .tasks-container {
    padding: 1rem 0.5rem;
  }

  .task-list-container {
    border-radius: 8px;
  }

  .task-list-header {
    padding: 0.75rem 0.5rem;
  }

  .task-card {
    padding: 0.75rem 0.5rem;
  }

  .task-title {
    font-size: 0.875rem;
  }

  .task-description {
    font-size: 0.75rem;
  }
}

/* Custom Scrollbar */
.task-list::-webkit-scrollbar {
  width: 8px;
}

.task-list::-webkit-scrollbar-track {
  background: #f1f5f9;
}

.task-list::-webkit-scrollbar-thumb {
  background-color: #cbd5e1;
  border-radius: 4px;
}

.task-list::-webkit-scrollbar-thumb:hover {
  background-color: #94a3b8;
}
</style>
