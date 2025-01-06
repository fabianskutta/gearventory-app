<template>
    <div class="print-layout">
      <div 
        class="item" 
        v-for="item in items" 
        :key="item.id"
      >
        <div>
          <qrcode-vue class="qrcode" :value="item.gid" size="100" />
        </div>
        <div class="item-info">
          <h3>{{ item.name }}</h3>
          <p>GID: {{ item.gid }}</p>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue';
  import QrcodeVue from 'vue-qrcode';
  
  const client = useSupabaseClient();
  const items = ref([]);
  
  onMounted(async () => {
    const { data } = await client.from('items').select('*');
    items.value = data || [];
  });
  
  </script>
  
  <style scoped>
  .print-layout {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 2rem;
    padding: 2cm;
  }
  
  .item {
    display: flex;
    align-items: center;
    justify-content: flex-start;
    border: 0.5px solid #ccc;
    padding: 0.3rem;
  }
  
  .qrcode {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    max-width: 60px;
  }
  
  .item-info {
    flex: 2;
    padding-left: 0.4rem;
  }
  
  .item-info h3 {
    margin: 0;
    font-size: 12px;
  }
  
  .item-info p {
    margin: 0;
    font-size: 10px;
  }
  </style>
  