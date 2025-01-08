<template>
  <div class="container">
    <!-- Steuerungselemente (fixiert) -->
    <div class="controls">
      <div class="controls-left">
        <strong>Name: </strong>{{ item.name }}
      </div>
      <div class="controls-right">
        <button class="btn" @click="close">Schließen</button>
        <button class="btn" @click="saveItem">Speichern</button>
      </div>
    </div>

    <!-- Tabs -->
    <div class="tabs">
      <button
        :class="{ active: currentTab === 'data' }"
        @click="setCurrentTab('data')"
        class="tab-btn"
      >
        Daten
      </button>
      <button
        :class="{ active: currentTab === 'serials' }"
        @click="setCurrentTab('serials')"
        class="tab-btn"
      >
        Seriennummer
      </button>
    </div>

    <!-- Daten Tab -->
    <div v-if="currentTab === 'data'" class="tab-content">
      <p><strong>Artikelnummer: </strong>{{ item.gid }}</p>
      <qrcode-vue :value="item.gid" :size="100" class="qr-code"></qrcode-vue>

      <p>
        <strong>Name: </strong>
        <input type="text" v-model="item.name" />
      </p>

      <p>
        <strong>Beschreibung: </strong>
        <textarea v-model="item.description" placeholder="Keine Beschreibung" rows="5" cols="10" style="resize: none; overflow-y: auto;"></textarea>
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

      <p>
        <strong>Kategorie: </strong>
        <input type="text" v-model="item.category" placeholder="Unbekannt" />
      </p>

      <p>
        <strong>Typ: </strong>
        <input type="text" v-model="item.type" placeholder="Unbekannt" />
      </p>
    </div>

    <!-- Seriennummer Tab -->
    <div v-if="currentTab === 'serials'" class="tab-content">
      <p>Keine Seriennummern verfügbar (Der Abschnitt bleibt noch leer).</p>
    </div>

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
const currentTab = ref('data'); // Aktueller Tab (Daten oder Seriennummer)

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

function setCurrentTab(tab) {
  currentTab.value = tab;
}

async function saveItem() {
  const { data, error } = await client.from('items').update({
    name: item.value.name,
    description: item.value.description,
    value: item.value.value,
    storage_location: item.value.storage_location,
    quantity: item.value.quantity,
    category: item.value.category,
    type: item.value.type,
  }).eq('id', id).select();

  if (error) {
    console.error("Fehler beim Speichern der Änderungen:", error);
  } else {
    console.log("Änderungen wurden erfolgreich gespeichert!", data);
    router.push('/items');
  }
}

function close() {
  router.push('/items');
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

.controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.controls-left,
.controls-right {
  display: flex;
  align-items: center;
}

.controls-left strong {
  font-size: 1.1rem;
}

.controls-right .btn {
  margin-left: 10px;
}

.tabs {
  display: flex;
  margin-bottom: 20px;
}

.tab-btn {
  padding: 10px 20px;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.3s;
}

.tab-btn.active {
  background-color: var(--accent1);
  color: white;
}

.tab-content {
  margin-bottom: 20px;
}

.qr-code {
  position: absolute;
  top: 20px;
  right: 20px;
  max-width: 70px;
  border-radius: 8px;
}

input[type="text"],
input[type="number"],
textarea {
  font-size: 1.1rem;
  width: 100%;
  padding: 8px;
  margin-top: 8px;
}

textarea {
  resize: none;
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

button.btn {
  padding: 10px 20px;
  font-size: 1rem;
  cursor: pointer;
  background-color: var(--accent1);
  color: white;
  border: none;
  border-radius: 6px;
  transition: opacity 0.2s ease-in-out;
}

button.btn:hover {
  opacity: 0.9;
}
</style>
