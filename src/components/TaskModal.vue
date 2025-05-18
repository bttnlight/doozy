<template>
  <q-dialog
    :model-value="modelValue"
    @update:model-value="$emit('update:modelValue', $event)"
    persistent
    transition-show="scale"
    transition-hide="scale"
  >
    <q-card class="modal-card">
      <div class="modal-decoration"></div>
      
      <q-card-section class="modal-header">
        <div class="text-xl font-bold manrope gradient-text">
          {{ isEditing ? 'Edit Task' : 'Create New Task' }}
        </div>
        <q-btn flat round icon="close" color="grey-7" v-close-popup class="close-btn" />
      </q-card-section>

      <q-separator color="grey-3" />

      <q-card-section class="q-pt-lg">
        <div class="form-grid">
          <div class="form-group">
            <label class="form-label">
              <q-icon name="title" size="16px" class="q-mr-xs" />
              Title
            </label>
            <q-input 
              v-model="task.title" 
              outlined 
              placeholder="What needs to be done?" 
              class="form-input"
              color="primary"
              :rules="[val => !!val || 'Title is required']"
            />
          </div>

          <div class="grid-cols-2-wrapper">
            <div class="form-group">
              <label class="form-label">
                <q-icon name="event" size="16px" class="q-mr-xs" />
                Due Date
              </label>
              <q-input
                v-model="task.dueDate"
                outlined
                placeholder="Select date"
                class="form-input date-input"
                mask="date"
                color="primary"
              >
                <template v-slot:append>
                  <q-icon name="event" class="cursor-pointer">
                    <q-popup-proxy cover transition-show="scale" transition-hide="scale">
                      <q-date 
                        v-model="task.dueDate" 
                        mask="YYYY-MM-DD" 
                        color="primary" 
                        today-btn
                        minimal
                      />
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>
            </div>

            <div class="form-group">
              <label class="form-label">
                <q-icon name="label" size="16px" class="q-mr-xs" />
                Category
              </label>
              <q-select
                v-model="task.category"
                :options="categoryOptions"
                outlined
                placeholder="Select category"
                class="form-input"
                emit-value
                map-options
                color="primary"
              >
                <template v-slot:option="scope">
                  <q-item v-bind="scope.itemProps">
                    <q-item-section>
                      <q-item-label>
                        <div class="flex items-center">
                          <div class="category-dot" :class="`bg-${getCategoryColor(scope.opt.value)}`"></div>
                          {{ scope.opt.label }}
                        </div>
                      </q-item-label>
                    </q-item-section>
                  </q-item>
                </template>
                <template v-slot:selected>
                  <div class="flex items-center" v-if="task.category">
                    <div class="category-dot" :class="`bg-${getCategoryColor(task.category)}`"></div>
                    {{ getCategoryLabel(task.category) }}
                  </div>
                </template>
              </q-select>
            </div>
          </div>

          <div class="form-group">
            <label class="form-label">
              <q-icon name="description" size="16px" class="q-mr-xs" />
              Description
            </label>
            <q-input
              v-model="task.description"
              outlined
              type="textarea"
              placeholder="Add details about your task..."
              class="form-input"
              rows="4"
              color="primary"
            />
          </div>

          <div class="form-group priority-section">
            <label class="form-label">
              <q-icon name="flag" size="16px" class="q-mr-xs" />
              Priority
            </label>
            <div class="priority-buttons">
              <q-btn-toggle
                v-model="task.priority"
                toggle-color="primary"
                :options="[
                  {label: 'Low', value: 'low'},
                  {label: 'Medium', value: 'medium'},
                  {label: 'High', value: 'high'}
                ]"
                spread
                unelevated
                class="full-width"
              />
            </div>
          </div>
        </div>
      </q-card-section>

      <q-card-actions align="right" class="q-pb-md q-px-md">
        <q-btn flat label="Cancel" color="grey-7" v-close-popup class="q-mr-sm" />
        <q-btn 
          :label="isEditing ? 'Update' : 'Create'" 
          color="primary" 
          class="save-btn manrope" 
          @click="onSave" 
          :disable="!task.title"
        />
      </q-card-actions>
    </q-card>
  </q-dialog>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import type { PropType } from 'vue';

export default defineComponent({
  name: 'TaskModal',
  props: {
    modelValue: {
      type: Boolean,
      required: true,
    },
    modelTask: {
      type: Object as PropType<{
        title: string;
        dueDate: string;
        category: string;
        description: string;
        completed: boolean;
        priority?: string;
      }>,
      required: true,
    },
    categoryOptions: {
      type: Array as PropType<Array<{ label: string; value: string }>>,
      required: true,
    },
    isEditing: {
      type: Boolean,
      default: false,
    },
  },
  emits: ['update:modelValue', 'save'],
  data() {
    return {
      task: { 
        ...this.modelTask,
        priority: this.modelTask.priority || 'medium'
      },
    };
  },
  watch: {
    modelTask: {
      handler(newVal) {
        this.task = { 
          ...newVal,
          priority: newVal.priority || 'medium'
        };
      },
      deep: true,
    },
  },
  methods: {
    onSave() {
      this.$emit('save', this.task);
      this.$emit('update:modelValue', false);
    },
    getCategoryColor(category: string): string {
      const colorMap: Record<string, string> = {
        'Work': 'blue-6',
        'Personal': 'purple-6',
        'Health': 'green-6',
        'Finance': 'amber-6',
        'Education': 'red-6'
      };
      return colorMap[category] || 'grey-6';
    },
    getCategoryLabel(value: string): string {
      const option = this.categoryOptions.find(opt => opt.value === value);
      return option ? option.label : value;
    }
  },
});
</script>

<style scoped>
/* Modal */
.modal-card {
  width: 100%;
  max-width: 30rem;
  border-radius: 16px;
  overflow: hidden;
  position: relative;
}

.modal-decoration {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 6px;
  background: linear-gradient(90deg, #4F46E5, #8B5CF6, #EC4899);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 1.5rem;
}

.gradient-text {
  background: linear-gradient(90deg, #4F46E5, #8B5CF6);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  font-weight: 700;
}

.form-grid {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.grid-cols-2-wrapper {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
}

.form-group {
  display: flex;
  flex-direction: column;
}

.form-label {
  font-size: 0.875rem;
  font-weight: 600;
  color: #4B5563;
  margin-bottom: 0.5rem;
  display: flex;
  align-items: center;
}

.form-input {
  font-family: 'Manrope', sans-serif;
  border-radius: 8px;
}

.form-input :deep(.q-field__control) {
  border-radius: 8px;
}

.save-btn {
  border-radius: 8px;
  font-weight: 600;
  padding: 0 1.5rem;
}

.close-btn {
  transition: transform 0.2s;
}

.close-btn:hover {
  transform: rotate(90deg);
}

.category-dot {
  width: 10px;
  height: 10px;
  border-radius: 50%;
  margin-right: 8px;
}

.priority-buttons :deep(.q-btn) {
  border-radius: 8px;
  font-weight: 500;
}

.priority-buttons :deep(.q-btn-group) {
  border-radius: 8px;
  overflow: hidden;
}
</style>
