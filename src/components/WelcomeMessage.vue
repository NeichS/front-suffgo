<template>
    <div v-bind="$attrs">
      <h2>¡Bienvenido, {{ userName }}!</h2>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue';
  
  const userName = ref('');
  
  onMounted(() => {
    userName.value = localStorage.getItem('userName') || 'Usuario';   
  });
  </script>
