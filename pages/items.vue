<template>
    <div>
        <div class="controls">
            <input 
                type="text" 
                v-model="searchQuery" 
                placeholder="Suche" 
                class="search-input"
            />
            <NuxtLink to="/create-item" class="btn">
                Artikel ➕
            </NuxtLink>

            <!-- Toggle button to switch views -->
            <button class="btn view-toggle" @click="showTable = !showTable">
                {{ showTable ? "🖼️" : "📊" }}
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
                                class="input-cell value"
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
                                    🔍
                                </NuxtLink>
                                <div class="tb-btn">
                                <NuxtLink @click="deleteItem(item)" class="btn">
                                    🚮
                                </NuxtLink>
                            </div>
                            </div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<style scoped>
.controls {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    margin-bottom: 1rem;
}

.search-input {
    flex-grow: 1;
    padding: 14px;
    font-size: 1rem;
}

.view-toggle {
    margin-left: auto;
}

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
    font-size: 1rem;
}

.gid {
    text-align: center;
}

.tb-btn {
    display: flex;
    justify-content: center;
}

.tb-btn .btn {
    background-color: var(--accent1);
    color: white;
    padding: 12px 20px;
    font-size: 14px;
}

.table-view th, .table-view td {
    padding: 5px 12px;
    border: 1px solid #ffffff00;
}

/* Spezifische Spaltenbreiten anpassen */
.table-view th:nth-child(2), /* Name */
.table-view td:nth-child(2) {
    width: 40%;
}

.table-view th:nth-child(3), /* Anzahl */
.table-view td:nth-child(3) {
    width: 10%;
}

.table-view th:nth-child(4), /* Preis */
.table-view td:nth-child(4) {
    width: 10%;
}

.table-view th:nth-child(5), /* Lagerort */
.table-view td:nth-child(5) {
    width: 25%;
}

.table-view th:nth-child(6), /* Speichern */
.table-view td:nth-child(6) {
    width: 10%;
}
</style>

<script setup>
import { ref, computed } from 'vue';

const client = useSupabaseClient();
const searchQuery = ref("");
const showTable = ref(true);

const { data: items, refresh: refreshItems} = await useAsyncData('items', async () => {
    const { data } = await client.from('items').select('*');
    return data || [];
});

const filteredItems = computed(() => {
    if (!searchQuery.value.trim()) {
        return items.value;
    }
    return items.value.filter(item => 
        item.name?.toLowerCase().includes(searchQuery.value.toLowerCase())
    );
});

const saveItem = async (item) => {
    const { error } = await client.from('items').upsert([item], { returning: 'minimal' });
    if (error) {
        console.error("Fehler beim Speichern:", error);
    }
};

const deleteItem = async (item) => {
    const { error } = await client.from('items').delete().eq('id', item.id);
    if (error) {
        console.error("Fehler beim Löschen:", error);
    }
    refreshItems();
};
</script>
