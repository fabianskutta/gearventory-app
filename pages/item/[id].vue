<template>
    <div class="container">
      <h1>{{ item?.name || 'Item Details' }}</h1>
      <p><strong>Beschreibung:</strong> {{ item?.description || 'Keine Beschreibung' }}</p>
      <p><strong>Wert in €:</strong> {{ item?.value ? item.value.toFixed(2) : 'Nicht angegeben' }}</p>
      <p><strong>Lagerort:</strong> {{ item?.storage_location || 'Unbekannt' }}</p>
      <p><strong>Menge:</strong> {{ item?.quantity || 0 }}</p>
      <p><strong>Bulk-Artikel:</strong> {{ item?.bulk_item ? 'Ja' : 'Nein' }}</p>
      <p><strong>ID:</strong> {{ item?.id || 'Keine ID' }}</p>
      
      <div v-if="!item.bulk_item" class="individual-items">
        <h3>Individuelle Geräte:</h3>
        <ul>
          <li v-for="device in devices" :key="device.id">
            ID: {{ item.gid }}.{{ device.serial_number }}
          </li>
        </ul>
      </div>
    
    </div>
  </template>
  
  <style scoped>
    .container {
      max-width: 800px;
      margin: 0 auto;
      border-radius: 8px;
      background-color: var(--background1);
      display: flex;
      flex-direction: column;
    }
    h1 {
      color: var(--text1);
      margin-bottom: 10px;
    }
    p {
      margin: 5px 0;
      color: var(--text2);
    }
    .individual-items {
      margin-top: 20px;
      padding-top: 10px;
      border-top: 1px solid var(--text1);
    }
    .individual-items h3 {
      margin-bottom: 5px;
      font-size: 1.1rem;
    }
    ul {
      list-style-type: none;
      padding-left: 0;
    }
    li {
      margin: 5px 0;
    }
  </style>
  
  <script setup>
  const client = useSupabaseClient();
  const { id } = useRoute().params;
  
  const { data: item } = await useAsyncData('items', async () => {
    const { data, error } = await client.from('items').select('*').eq('id', id).single();
    if (error) {
      console.error("Fehler beim Abrufen des Items:", error);
      return null;
    }
    return data;
  });
  
  const { data: devices } = await useAsyncData('devices', async () => {
    const { data } = await client.from('devices').select('*').eq('item_id', id)
    return data;
})
  
  </script>
  