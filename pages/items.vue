<template>
    <div class="container">
        <!-- Steuerungselemente (fixiert) -->
        <div class="controls">
            <p class="selected-counter">
                {{ selectedItems.length }}
                ausgewählt
            </p>

            <input 
                type="text" 
                v-model="searchQuery" 
                placeholder="Suche" 
                class="search-input"
            />
            
            <NuxtLink 
                :disabled="selectedItems.length !== 1" 
                class="btn" 
                :class="{ disabled: selectedItems.length !== 1 }"
                @click="editItem"
            >
                Bearbeiten
            </NuxtLink>
            
            <NuxtLink 
                :disabled="selectedItems.length === 0" 
                class="btn" 
                :class="{ disabled: selectedItems.length === 0 }"
                @click="deleteSelectedItems"
            >
                Löschen
            </NuxtLink>

            <NuxtLink to="/create-item" class="btn">
                Material hinzufügen
            </NuxtLink>
        </div>

        <!-- Tabellenansicht -->
        <div class="table-view">
            <table>
                <thead class="sticky-header">
                    <tr>
                        <th><input type="checkbox" @change="toggleAll"></th>
                        <th @click="sortTable('gid')" class="sortable-header">
                            ID <span v-if="sortColumn === 'gid'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                        </th>
                        <th @click="sortTable('name')" class="sortable-header">
                            Name <span v-if="sortColumn === 'name'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                        </th>
                        <th @click="sortTable('quantity')" class="sortable-header">
                            Menge <span v-if="sortColumn === 'quantity'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                        </th>
                        <th @click="sortTable('value')" class="sortable-header">
                            Preis <span v-if="sortColumn === 'value'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                        </th>
                        <th @click="sortTable('storage_location')" class="sortable-header">
                            Lagerort <span v-if="sortColumn === 'storage_location'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                        </th>
                        <th @click="sortTable('category')" class="sortable-header">
                            Kategorie <span v-if="sortColumn === 'category'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                        </th>
                        <th @click="sortTable('type')" class="sortable-header">
                            Materialtyp <span v-if="sortColumn === 'type'">{{ sortOrder === 'asc' ? '▲' : '▼' }}</span>
                        </th>
                        <th></th>
                    </tr>
                </thead>
                <tbody class="table-body">
                    <tr v-for="item in filteredItems" :key="item.id">
                        <td>
                            <input 
                                type="checkbox" 
                                :value="item.id" 
                                v-model="selectedItems" 
                            />
                        </td>
                        <td><p class="cell">{{ item.gid }}</p></td>
                        <td><p class="cell">{{ item.name || 'Nicht angegeben' }}</p></td>
                        <td><p class="cell">{{ item.quantity || 0 }}</p></td>
                        <td><p class="cell">{{ item.value ? item.value.toFixed(2) : '0.00' }} €</p></td>
                        <td><p class="cell">{{ item.storage_location || 'Unbekannt' }}</p></td>
                        <td><p class="cell">{{ item.category || 'Unbekannt' }}</p></td>
                        <td><p class="cell">{{ item.type || 'Unbekannt' }}</p></td>
                        <td>
                            <div class="tb-btn">
                                <NuxtLink :to="`/item/${item.id}`" class="btn">öffnen</NuxtLink>
                            </div>
                        </td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const client = useSupabaseClient();
const searchQuery = ref("");
const selectedItems = ref([]);
const sortColumn = ref('gid');
const sortOrder = ref('asc');

const { data: items, refresh: refreshItems } = await useAsyncData('items', async () => {
    const { data } = await client.from('items').select('*');
    return data || [];
});

const filteredItems = computed(() => {
    let filtered = items.value;
    if (searchQuery.value.trim()) {
        filtered = filtered.filter(item => 
            item.name?.toLowerCase().includes(searchQuery.value.toLowerCase())
        );
    }
    if (sortColumn.value) {
        filtered = filtered.sort((a, b) => {
            if (a[sortColumn.value] > b[sortColumn.value]) return sortOrder.value === 'asc' ? 1 : -1;
            if (a[sortColumn.value] < b[sortColumn.value]) return sortOrder.value === 'asc' ? -1 : 1;
            return 0;
        });
    }
    return filtered;
});

const deleteSelectedItems = async () => {
    if (selectedItems.value.length === 0) return;

    const { error } = await client
        .from('items')
        .delete()
        .in('id', selectedItems.value);

    if (error) {
        console.error("Fehler beim Löschen:", error);
        return;
    }

    selectedItems.value = [];
    refreshItems();
};

const editItem = () => {
    if (selectedItems.value.length !== 1) return;
    const itemId = selectedItems.value[0];
    navigateTo(`/item/${itemId}`);
};

const toggleAll = (event) => {
    if (event.target.checked) {
        selectedItems.value = filteredItems.value.map(item => item.id);
    } else {
        selectedItems.value = [];
    }
};

const sortTable = (column) => {
    if (sortColumn.value === column) {
        sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc';
    } else {
        sortColumn.value = column;
        sortOrder.value = 'asc';
    }
};
</script>

<style scoped>
.container {
    display: flex;
    flex-direction: column;
    height: calc(100vh - 40px);
}

.controls {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    margin-bottom: 1rem;
}

.selected-counter {
    background-color: var(--background2);
    padding: 0.75rem 1rem;
    border-radius: 6px;
    font-size: 1rem;
    color: var(--text);
    white-space: nowrap;
    display: inline-block;
    margin: 0;
}

.search-input {
    flex-grow: 1;
    padding: 10px;
    font-size: 1rem;
}

.table-view {
    width: 100%;
    background-color: var(--background2);
    border-radius: 12px;
    padding: 0 10px 10px 10px;
    height: 100%;
    overflow-y: auto;
}

.table-view table {
    width: 100%;
    border-collapse: collapse;
}

.table-view th, .table-view td {
    border-bottom: 1px solid #ffffff17;
    text-align: left;
    padding: 9px 10px;
    font-size: 0.9rem;
    p {
        font-size: 0.9rem;
    }
}

.table-view th {
    padding: 15px 10px;
}

/* Fixiere den Tabellenkopf */
.sticky-header {
    position: sticky;
    top: 0;
    background-color: var(--background2);
    z-index: 1;
}

.sortable-header {
    cursor: pointer;
    transition: background-color 0.2s ease;
}

.sortable-header:hover {
    background-color: var(--background1);
}

.tb-btn {
    display: flex;
}

.tb-btn .btn {
    background-color: #ffffff49;
    color: white;
    padding: 5px 10px;
    font-size: 0.9rem;
    font-weight: 400;
}

.btn {
    cursor: pointer;
    padding: 10px 15px;
    border: none;
    border-radius: 6px;
    background-color: var(--accent1);
    color: white;
    font-size: 1rem;
    font-weight: 500;
    transition: opacity 0.2s ease-in-out;
}

.btn.disabled {
    background-color: #888888;
    cursor: not-allowed;
    opacity: 0.6;
}

.cell {
    margin: 0;
}

input[type="checkbox"] {
    appearance: none;
    width: 5px;
    height: 5px;
    margin: 0;
    border: 1px solid #ffffff49;
    border-radius: 4px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    outline: none;
    background-color: var(--background2);
    transition: background-color 0.3s ease, border-color 0.3s ease;
    padding: 8px;
}

input[type="checkbox"]:checked {
    background-color: var(--accent1);
    border-color: var(--accent1);
}

input[type="checkbox"]:checked::before {
    content: "✔";
    font-size: 14px;
    color: #fff;
    display: block;
    text-align: center;
    line-height: 20px;
}
</style>
