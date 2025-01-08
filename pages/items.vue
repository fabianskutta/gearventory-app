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
                Material ➕
            </NuxtLink>
        </div>

        <div class="table-view">
            <table>
                <thead>
                    <tr>
                        <th><input type="checkbox"></th>
                        <th>ID</th>
                        <th>Name</th>
                        <th>Menge</th>
                        <th>Preis</th>
                        <th>Lagerort</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="item in filteredItems" :key="item.id">
    <td>
        <input type="checkbox">
    </td>
    <td>
        <p class="cell">{{ item.gid }}</p>
    </td>
    <td>
        <p class="cell">{{ item.name || 'Nicht angegeben' }}</p>
    </td>
    <td>
        <p class="cell">{{ item.quantity || 0 }}</p>
    </td>
    <td>
        <p class="cell">{{ item.value ? item.value.toFixed(2) : '0.00' }} €</p>
    </td>
    <td>
        <p class="cell">{{ item.storage_location || 'Unbekannt' }}</p>
    </td>
    <td>
                            <div class="tb-btn">
                                <NuxtLink :to="`/item/${item.id}`" class="btn">
                                    öffnen
                                </NuxtLink>
                                <div class="tb-btn">
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
    border: 1px solid #ffffff00;
    font-size: 1rem;
    text-align: left;
    padding: 8px 10px;
}


.tb-btn {
    display: flex;
}

.tb-btn .btn {
    background-color: #ffffff49;
    color: white;
    padding: 10px 20px;
    font-size: 14px;
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
    padding: 10px;
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


.table-view td:nth-child(1) {
    width: 50px;
}

.table-view td:nth-child(2) {
    width: 100px;
}

.table-view td:nth-child(4) {
    width: 100px;
}

.table-view td:nth-child(5) {
    width: 120px;
}

.table-view td:nth-child(7) {
    width: 100px;
}


</style>

<script setup>
import { ref, computed } from 'vue';

const client = useSupabaseClient();
const searchQuery = ref("");

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


const deleteItem = async (item) => {
    const { error } = await client.from('items').delete().eq('id', item.id);
    if (error) {
        console.error("Fehler beim Löschen:", error);
    }
    refreshItems();
};
</script>
