<template>
  <div
    class="v-filter"
  >
    <div class="v-filter_list-select">
      <v-select
        v-for="select in selects"
        :key="select.id"
        :options="select.options"
        :title="select.title"
        :mode="select.mode"
      />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import VSelect from './components/v-select/v-select.vue';

const selects = ref([
  { 
    id: 1, 
    title: 'Search select', 
    mode: 'search',
    options: [
      {id: 1, content: 'option 1'},
      {id: 2, content: 'option 2'},
      {id: 3, content: 'option 3'},
      {id: 4, content: 'option 4'},
      {id: 5, content: 'option 5'},
      {id: 6, content: 'option 6'},
      {id: 7, content: 'option 7'},
      {id: 8, content: 'option 8'},
      {id: 9, content: 'option 9'}
    ]
  },
  { 
    id: 2, 
    title: 'Single select', 
    mode: 'single',
    options: [
      {id: 1, content: 'option 1'},
      {id: 2, content: 'option 2'},
      {id: 3, content: 'option 3'},
      {id: 4, content: 'option 4'},
      {id: 5, content: 'option 5'},
      {id: 6, content: 'option 6'},
      {id: 7, content: 'option 7'},
      {id: 8, content: 'option 8'},
      {id: 9, content: 'option 9'}
    ]
  },
  { 
    id: 3, 
    title: 'Multi select', 
    mode: 'multi',
    options: [
      {id: 1, content: 'option 1'},
      {id: 2, content: 'option 2'},
      {id: 3, content: 'option 3'},
      {id: 4, content: 'option 4'},
      {id: 5, content: 'option 5'},
      {id: 6, content: 'option 6'},
      {id: 7, content: 'option 7'},
      {id: 8, content: 'option 8'},
      {id: 9, content: 'option 9'}
    ]
  }
]);

</script>

<style lang="scss">
.v-filter {
  display: flex;
  flex-direction: column;
  gap: 8px;

  &_list-select {
    display: flex;
    flex-wrap: wrap;
    gap: 12px 40px;
    padding: 24px 16px;
    padding-bottom: 24px;
    background-color: var(--grey-light);
  }
}

</style>
