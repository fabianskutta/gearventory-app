<template>
    <div class="container">
      <p>
        <strong>Name: </strong>
        <input type="text" v-model="item.name" />
      </p>
  
      <p>
        <strong>Beschreibung: </strong>
        <input type="text" v-model="item.description" placeholder="Keine Beschreibung" />
      </p>
  
      <p>
        <strong>Wert in €: </strong>
        <input type="number" step="0.1" v-model="item.value" placeholder="Nicht angegeben" />
      </p>
  
      <p>
        <strong>Lagerort: </strong>
        <input type="text" v-model="item.storage_location" placeholder="Unbekannt" />
      </p>
  
      <p>
        <strong>Artikelnummer: </strong>
        <input type="text" v-model="item.gid" :readonly="true" />
      </p>
  
      <p>
        <strong>Menge: </strong>
        <input type="number" v-model="item.quantity" placeholder="0" />
      </p>
  
      <button class="btn" @click="createItem">Erstellen</button>
    </div>
  </template>
  
  <script setup>
const client = useSupabaseClient();
const user = useSupabaseUser();
const router = useRouter();

const item = ref({
  name: '',
  description: '',
  value: 0.00,
  storage_location: '',
  gid: '',
  quantity: 0,
  bulk_item: true,
});

async function getNextGid() {
  const { data, error } = await client
    .from('items')
    .select('gid')
    .order('gid', { ascending: false })
    .limit(1);

  if (error) {
    console.error('Fehler beim Abrufen der letzten Artikelnummer:', error);
    return '0001'; // Standardwert falls ein Fehler passiert
  }

  const nextGid = data && data.length > 0 ? parseInt(data[0].gid) + 1 : 1;

  // Formatiere die GID immer als 4-stellige Zahl mit führenden Nullen
  return nextGid.toString().padStart(4, '0');
}


onMounted(async () => {
  item.value.gid = await getNextGid();
});

async function createItem() {
  // Stelle sicher, dass die GID als String mit führenden Nullen formatiert wird
  const formattedGid = item.value.gid.padStart(4, '0'); 
  
  const { data, error } = await client
    .from('items')
    .insert([{
      name: item.value.name,
      description: item.value.description,
      value: item.value.value,
      storage_location: item.value.storage_location,
      gid: formattedGid,  // Speichere die GID immer als String mit führenden Nullen
      quantity: item.value.quantity,
      bulk_item: item.value.bulk_item,
      user_id: user.value.id,
    }]).select();

  if (error) {
    console.error('Fehler beim Erstellen des Items:', error);
  } else {
    console.log('Neues Item wurde erfolgreich erstellt!', data);
    router.push('/items');
  }
}
</script>

  
  <style scoped>
  .container {
    max-width: 800px;
    margin: 0 auto;
    border-radius: 12px;
    background-color: var(--background2);
    display: flex;
    flex-direction: column;
    padding: 20px;
  }
  p {
    margin: 10px 0;
    color: var(--text1);
    font-size: 1.1rem;
  }
  input, label, button {
    padding: 10px;
    font-size: 1rem;
    width: 100%;
  }
  
  input[type="number"], input[type="text"], input[type="checkbox"] {
    font-size: 1.1rem;
    width: 100%;
    padding: 8px;
    margin-top: 8px;
  }
  input[type="checkbox"] {
    width: auto;
  }
  
  .individual-items {
    margin-top: 20px;
    padding-top: 10px;
    border-top: 1px solid #ccc;
  }
  .individual-items h3 {
    margin-bottom: 10px;
    font-size: 1.2rem;
  }
  ul {
    list-style-type: none;
    padding-left: 0;
  }
  li {
    margin: 8px 0;
    font-size: 1rem;
  }
  </style>
  