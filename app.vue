<template>
  <div class="app-container">
    <Nav v-if="showNav" />

    <div class="content-container">
      <NuxtPage />
    </div>
  </div>
</template>

<script setup>
import { useRoute } from 'vue-router';
import { ref, watch, onMounted } from 'vue';

const route = useRoute();
const showNav = ref(true);

const checkRoute = () => {
  showNav.value = !['/login', '/register'].includes(route.path);
};

onMounted(() => {
  checkRoute();
});

watch(
  () => route.path,
  checkRoute
);
</script>


<style>
.app-container {
  display: flex;
  height: 100vh;
}

.content-container {
  flex: 1;
  padding: 20px;
  overflow-y: auto;
}
</style>
