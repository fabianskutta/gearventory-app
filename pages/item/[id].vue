<template>
  <div class="container">
    <p>
      <strong>Artikelnummer: </strong>
      {{ item.gid }}
    </p>
    <br><br>
    <qrcode-vue
      :value="item.gid"
      :size="100"
      class="qr-code"
    ></qrcode-vue>

    <p>
      <strong>Name: </strong>
      <input type="text" v-model="item.name" />
    </p>

    <p>
  <strong>Beschreibung: </strong>
  <textarea 
    v-model="item.description" 
    placeholder="Keine Beschreibung" 
    rows="5" 
    cols="10" 
    style="resize: none; overflow-y: auto;">
  </textarea>
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
      <strong>Menge: </strong>
      <input type="number" v-model="item.quantity" placeholder="0" />
    </p>

    <button class="btn" @click="saveItem">Speichern</button>
    <!--<button class="btn" @click="deleteItem()">Löschen</button>-->

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

<script setup>
import QrcodeVue from 'vue-qrcode';
const client = useSupabaseClient();
const { id } = useRoute().params;
const router = useRouter();

const item = ref({});
const { data: devices } = await useAsyncData('devices', async () => {
  const { data } = await client.from('devices').select('*').eq('item_id', id);
  return data;
});

const { data } = await useAsyncData('items', async () => {
  const { data, error } = await client.from('items').select('*').eq('id', id).single();
  if (error) {
    console.error("Fehler beim Abrufen des Items:", error);
    return null;
  }
  item.value = data;
});

async function saveItem() {
  const { data, error } = await client.from('items').update({
    name: item.value.name,
    description: item.value.description,
    value: item.value.value,
    storage_location: item.value.storage_location,
    quantity: item.value.quantity,
  }).eq('id', id).select();

  if (error) {
    console.error("Fehler beim Speichern der Änderungen:", error);
  } else {
    console.log("Änderungen wurden erfolgreich gespeichert!", data);
    router.push('/items');
  }
}

async function deleteItem() {
  const { error } = await client.from('items').delete().eq('id', id);

  if (error) {
    console.error("Fehler beim Löschen des Items:", error);
  } else {
    console.log("Item wurde erfolgreich gelöscht!");
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
    position: relative;
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

  .qr-code {
    position: absolute;
    top: 20px;
    right: 20px;
    max-width: 70px;
    border-radius: 8px;
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
