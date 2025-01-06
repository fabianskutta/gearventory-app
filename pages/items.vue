<template>
    <div class="boxes">
    <Item v-for="item of items" :item="item" :key="item.id"/>
  </div>
</template>

<style scoped>
          .boxes {
          margin-top: 2rem;
          display: flex;
          flex-wrap: wrap;
      }
  
      body {
          margin: 2rem;
      }
      </style>

<script setup>
const client = useSupabaseClient();
const { id } = useRoute().params;


const { data: items } = await useAsyncData('items', async () => {
    const { data } = await client.from('items').select('*')
    return data
})
</script>