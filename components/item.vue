<template>
    <nuxt-link :to="`/item/${item.id}`" class="box-sneak">
        <div class="box-container">
            <h3>{{ item.name }}</h3>
            <div class="item-details">
                <div class="item-detail">
                    <span class="label">Wert:</span>
                    <span class="value">{{ item.value }}</span>
                </div>
                <div class="item-detail">
                    <span class="label">Menge:</span>
                    <span class="value">{{ item.quantity }}</span>
                </div>
                <div class="item-detail">
                    <span class="label">ID:</span>
                    <span class="value">{{ item.gid }}</span>
                </div>
            </div>
        </div>
    </nuxt-link>
</template>

<script setup lang="ts">
defineProps<{item: Item}>();
</script>

<style lang="scss">
.box-sneak {
    background-color: var(--background2);
    display: flex;
    width: 300px;
    padding: 1.5rem;
    justify-content: center;
    flex-direction: column;
    border: 1px solid #ffffff17;
    border-radius: 20px;
    margin: 1.5rem;
    text-decoration: none;
    transition: transform 0.2s, box-shadow 0.2s;

    &:hover {
        transform: translateY(-5px);
        box-shadow: 0px 10px 15px rgba(0, 0, 0, 0.1);
    }

    h3 {
        font-weight: 800;
        font-size: 1.5rem;
        margin: 0 0 1rem 0;
        color: var(--text1);
        text-align: center;
    }

    .item-details {
        display: flex;
        flex-direction: column;
        gap: 0.5rem;

        .item-detail {
            display: flex;
            justify-content: space-between;
            font-size: 1rem;

            .label {
                font-weight: bold;
                color: var(--text1);
            }

            .value {
                color: var(--text2);
            }
        }
    }
}
</style>
