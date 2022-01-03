<template>
  <div class="list-container">
    <List />
    <VList />
  </div>
</template>

<script>
import List from './components/normal-list.vue';
import VList from './components/virtaul-list.vue';
import './style.css';

export default {
  components: {
    List,
    VList,
  },
}
</script>
