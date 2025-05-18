<template>
  <q-drawer v-model="internalModel" show-if-above :width="240">
    <div class="sidebar">
      <div class="sidebar-content">
        <div class="sidebar-header">
          <div class="menu-title">Menu</div>
          <q-btn flat dense round icon="menu" size="sm" class="menu-icon" />
        </div>

        <q-list padding class="menu-list">
          <q-item
            clickable
            v-ripple
            to="/"
            exact
            class="menu-item"
            :class="{ 'active-route': route.path === '/' }"
          >
            <q-item-section avatar>
              <q-icon name="dashboard" />
            </q-item-section>
            <q-item-section>Dashboard</q-item-section>
          </q-item>

          <q-item
            clickable
            v-ripple
            to="/tasks"
            exact
            class="menu-item"
            :class="{ 'active-route': route.path === '/tasks' }"
          >
            <q-item-section avatar>
              <q-icon name="check_box" />
            </q-item-section>
            <q-item-section>Tasks</q-item-section>
          </q-item>

          <q-item
            clickable
            v-ripple
            to="/calendar"
            exact
            class="menu-item"
            :class="{ 'active-route': route.path === '/calendar' }"
          >
            <q-item-section avatar>
              <q-icon name="calendar_today" />
            </q-item-section>
            <q-item-section>Calendar</q-item-section>
          </q-item>
        </q-list>

        <div class="logout-section">
          <q-item clickable v-ripple class="menu-item">
            <q-item-section avatar>
              <q-icon name="logout" />
            </q-item-section>
            <q-item-section>Logout</q-item-section>
          </q-item>
        </div>
      </div>
    </div>
  </q-drawer>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { useRoute } from 'vue-router';

const props = defineProps({
  modelValue: {
    type: Boolean,
    default: true,
  },
});

const emit = defineEmits(['update:modelValue']);
const route = useRoute();

const internalModel = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value),
});
</script>

<style scoped>
.sidebar {
  background-color: #d9d9d9;
  font-family: 'Manrope', sans-serif;
  border-top-right-radius: 16px;
  border-bottom-right-radius: 16px;
  overflow: hidden;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.sidebar-content {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.sidebar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px;
}

.menu-title {
  font-weight: 600;
  font-size: 16px;
}

.menu-list {
  flex-grow: 1;
}

.menu-item {
  border-radius: 0;
  color: #333;
  font-weight: 500;
}

.active-route {
  font-weight: 600;
}

.logout-section {
  margin-top: auto;
  padding-bottom: 16px;
}
</style>
