<template>
  <q-dialog
    :model-value="modelValue"
    @update:model-value="$emit('update:modelValue', $event)"
    persistent
  >
    <q-card class="modal-card">
      <q-card-section class="modal-header">
        <div class="text-xl font-bold manrope">
          {{ isEditing ? 'Edit Task' : 'New Task' }}
        </div>
        <q-btn flat round icon="close" v-close-popup />
      </q-card-section>

      <q-separator />

      <q-card-section>
        <div class="form-grid">
          <div class="form-group">
            <label class="form-label">Title</label>
            <q-input v-model="task.title" outlined placeholder="Title" class="form-input" />
          </div>

          <div class="grid grid-cols-2 gap-4">
            <div class="form-group">
              <label class="form-label">Due Date</label>
              <q-input
                v-model="task.dueDate"
                outlined
                placeholder="Date"
                class="form-input"
                mask="date"
              >
                <template v-slot:append>
                  <q-icon name="event" class="cursor-pointer">
                    <q-popup-proxy cover transition-show="scale" transition-hide="scale">
                      <q-date v-model="task.dueDate" mask="YYYY-MM-DD" color="grey-9" />
                    </q-popup-proxy>
                  </q-icon>
                </template>
              </q-input>
            </div>

            <div class="form-group">
              <label class="form-label">Category</label>
              <q-select
                v-model="task.category"
                :options="categoryOptions"
                outlined
                placeholder="Category"
                class="form-input"
                emit-value
                map-options
              />
            </div>
          </div>

          <div class="form-group">
            <label class="form-label">Description</label>
            <q-input
              v-model="task.description"
              outlined
              type="textarea"
              placeholder="Description"
              class="form-input"
              rows="4"
            />
          </div>
        </div>
      </q-card-section>

      <q-card-actions align="center" class="pb-4">
        <q-btn label="Save" color="grey-9" class="w-1/3 manrope" @click="onSave" />
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
      task: { ...this.modelTask },
    };
  },
  watch: {
    modelTask: {
      handler(newVal) {
        this.task = { ...newVal };
      },
      deep: true,
    },
  },
  methods: {
    onSave() {
      this.$emit('save', this.task);
      this.$emit('update:modelValue', false);
    },
  },
});
</script>

<style scoped>
/* Modal */
.modal-card {
  width: 100%;
  max-width: 28rem;
}
.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.form-grid {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.form-group {
  display: flex;
  flex-direction: column;
}
.form-label {
  font-size: 0.875rem;
  font-weight: 500;
  color: #374151;
  margin-bottom: 0.25rem;
}

.form-input {
  font-family: 'Manrope', sans-serif;
}
</style>
