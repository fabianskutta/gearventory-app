<template>
    <div class="container">
      <h1>Gesamtwert aller Artikel</h1>
      <p><strong>Wert in €: </strong>{{ totalValue.toFixed(2) }}</p>
    </div>
  </template>
  
  <script setup>
  import { onMounted, ref } from 'vue';
  const client = useSupabaseClient();
  
  const totalValue = ref(0);
  
  async function calculateTotalValue() {
    const { data, error } = await client
      .from('items')
      .select('value, quantity');
  
    if (error) {
      console.error('Fehler beim Laden der Artikel:', error);
      totalValue.value = 0;
      return;
    }
  
    totalValue.value = data.reduce((sum, item) => sum + item.value * item.quantity, 0);
  }
  
  onMounted(() => {
    calculateTotalValue();
  });
  </script>
  
  <style scoped>
  .container {
    max-width: 800px;
    margin: 0 auto;
    padding: 20px;
    border-radius: 12px;
    background-color: var(--background2);
    color: var(--text1);
    text-align: center;
    font-size: 1.2rem;
  }
  h1 {
    margin-bottom: 20px;
  }
  </style>
  