<template>
  <div class="container">
    <div class="controls">
      <div class="controls-left">
        {{ item.gid }}&nbsp;-&nbsp;<strong>{{ item.name }}</strong>
      </div>
      <div class="controls-right">
        <NuxtLink  class="btn close" @click="close">Schließen</NuxtLink>
        <NuxtLink  class="btn" @click="saveItem">Speichern</NuxtLink>
      </div>
    </div>
    <div class="tabs">
      <button
        :class="{ active: currentTab === 'data' }"
        @click="setCurrentTab('data')"
        class="tab-btn"
      >
        Daten
      </button>
      <button v-if="item.bulk_item === false"
        :class="{ active: currentTab === 'serials' }"
        @click="setCurrentTab('serials')"
        class="tab-btn"
      >
        Seriennummer
      </button>
      <button v-if="item.bulk_item === true"
        :class="{ active: currentTab === 'stock' }"
        @click="setCurrentTab('stock')"
        class="tab-btn"
      > Bestand
      </button>
      <button
        :class="{ active: currentTab === 'accessories' }"
        @click="setCurrentTab('accessories')"
        class="tab-btn"
      > Zubehör
      </button>
    </div>

    <div class="tab-container">
      <div v-if="currentTab === 'data'" class="tab-content">
        <div class="grid">
          <p class="gid-field">
            <strong>ID: </strong>
            <input type="text" v-model="item.gid"/>
          </p>
          <p class="name-field">
            <strong>Name: </strong>
            <input type="text" v-model="item.name"/>
          </p>
        </div>

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
          <strong>Bestandsberechnungsmethode: </strong>
          <select v-model="item.bulk_item">
            <option value="true">Massenartikel</option>
            <option value="false">Seriennummern</option>
          </select>
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
    </div>
  </div>
</template>

<script setup>
import QrcodeVue from 'vue-qrcode';
const client = useSupabaseClient();
const { id } = useRoute().params;
const router = useRouter();

const item = ref({});
const currentTab = ref('data');

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

function close() {
  navigateTo(`/items`);
}

function setCurrentTab(tab) {
  currentTab.value = tab;
}
</script>

<style scoped>
.container {
    display: flex;
    flex-direction: column;
    height: calc(100vh - 40px);
}

.controls {
    display: flex;
    justify-content: space-between;
    gap: 1rem;
    margin-bottom: 1rem;

    .close {
      background-color: #ffffff49;
    }
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
  gap: 10px;
}

.tab-container {
  display: flex;
  justify-content: center;
}

.tab-content {
    width: 100%;
    background-color: var(--background2);
    border-radius: 12px;
    padding: 10px 20px 10px 20px;
    max-width: 800px;
}

.tab-btn {
  background-color: #ffffff49;
  color: white;
  padding: 10px 15px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  text-decoration: none;
  font-size: 1rem;
  white-space: nowrap;
  font-weight: 500;
  transition: background-color 0.3s ease, border-color 0.3s ease;
  font-family: 'Inter', sans-serif;
}

.tab-btn.active {
  background-color: var(--accent1);
}

.gid-field {
  width: 100px;
}

.name-field {
  width: 100%;
}

input[type="text"],
input[type="number"],
textarea,
select {
  font-size: 1rem;
  width: 100%;
  padding: 8px;
  margin-top: 8px;
}

.grid {
  display: flex;
  gap: 20px;
}

</style>
