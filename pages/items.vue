<template>
    <div>
        <div class="controls">
            <input 
                type="text" 
                v-model="searchQuery" 
                placeholder="Search items..." 
                class="search-input"
            />
            <NuxtLink to="/create-item" class="btn">
                Artikel erstellen
            </NuxtLink>
        </div>

        <!-- Toggle button to switch views -->
        <div class="view-toggle">
            <button class="btn" @click="showTable = !showTable">
                {{ showTable ? "Wechsle zur Kachelansicht" : "Wechsle zur Tabellenansicht" }}
            </button>
        </div>

        <!-- Items Display as Boxes -->
        <div v-if="!showTable" class="boxes">
            <Item 
                v-for="item in filteredItems" 
                :item="item" 
                :key="item.id"
            />
        </div>

        <!-- Items Display as a Table -->
        <div v-if="showTable" class="table-view">
            <table>
                <thead>
                    <tr>
                        <th>id</th>
                        <th>Name</th>
                        <th>Anzahl</th>
                        <th>Preis</th>
                        <th>Lagerort</th>
                        <th>Öffnen</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="item in filteredItems" :key="item.id">
                        <td>
                            <p class="gid">{{ item.gid }}</p>
                        </td>
                        <td>
                            <input 
                                v-model="item.name" 
                                placeholder="Nicht angegeben" 
                                class="input-cell"
                                @blur="saveItem(item)"
                            />
                        </td>
                        <td>
                            <input 
                                v-model="item.quantity" 
                                type="number" 
                                placeholder="0"
                                class="input-cell"
                                @blur="saveItem(item)"
                            />
                        </td>
                        <td>
                            <input 
                                v-model="item.value" 
                                type="number" 
                                placeholder="0.00"
                                class="input-cell"
                                @blur="saveItem(item)"
                            />
                        </td>
                        <td>
                            <input 
                                v-model="item.storage_location" 
                                type="text" 
                                placeholder="Unbekannt"
                                class="input-cell"
                                @blur="saveItem(item)"
                            />
                        </td>
                        <td>
                            <div class="tb-btn">
                                <NuxtLink :to="`/item/${item.id}`" class="btn">
                                    Öffnen
                                </NuxtLink>
                            </div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<style scoped>
.boxes {
    margin-top: 2rem;
    display: flex;
    flex-wrap: wrap;
}

.table-view {
    margin-top: 2rem;
    width: 100%;
    background-color: var(--background2);
    border-radius: 12px;
    padding: 10px;
}

.table-view table {
    width: 100%;
    border-collapse: collapse;
}

.table-view th, .table-view td {
    padding: 8px 12px;
    border: 1px solid #ffffff00;
}

.gid {
    text-align: center;
}

.tb-btn {
    display: flex;
    justify-content: center;

    .btn {
    background-color: var(--accent1);
    color: white;
    padding: 12px 20px;
    font-size: 14px
}
}
.table-view th, .table-view td {
    padding: 5px 12px;
    border: 1px solid #ffffff00;
}

/* Spezifische Spaltenbreiten anpassen */
.table-view th:nth-child(2), /* Name */
.table-view td:nth-child(2) {
    width: 40%; /* Name bekommt 30% der Gesamtbreite */
}

.table-view th:nth-child(3), /* Anzahl */
.table-view td:nth-child(3) {
    width: 10%; /* Anzahl bekommt 15% der Gesamtbreite */
}

.table-view th:nth-child(4), /* Preis */
.table-view td:nth-child(4) {
    width: 10%; /* Preis bekommt 15% der Gesamtbreite */
}

.table-view th:nth-child(5), /* Lagerort */
.table-view td:nth-child(5) {
    width: 25%; /* Lagerort bekommt 25% */
}

.table-view th:nth-child(6), /* Speichern */
.table-view td:nth-child(6) {
    width: 10%; /* Speichern bekommt 15% */
}


</style>

<script setup>
import { ref, computed } from 'vue';

// Supabase und Query-Ref.
const client = useSupabaseClient();
const searchQuery = ref(""); // Suchtext als lokale Referenz
const showTable = ref(true); // Flag für den View-Toggle

// Daten vom Backend laden
const { data: items } = await useAsyncData('items', async () => {
    const { data } = await client.from('items').select('*');
    return data || []; // Fallback, falls keine Daten vorhanden sind
});

// Computed Property für die gefilterten Items
const filteredItems = computed(() => {
    if (!searchQuery.value.trim()) {
        // Alle Items anzeigen, wenn die Suchanfrage leer ist
        return items.value;
    }
    // Filterung der Items basierend auf der Suchanfrage, wenn item.name existiert
    return items.value.filter(item => {
        return item.name?.toLowerCase().includes(searchQuery.value.toLowerCase());
    });
});

// Speichern der geänderten Daten (auf das Backend übertragen)
const saveItem = async (item) => {
    const { error } = await client.from('items').upsert([item], { returning: 'minimal' });
    if (error) {
        console.error("Fehler beim Speichern:", error);
    }
};
</script>
