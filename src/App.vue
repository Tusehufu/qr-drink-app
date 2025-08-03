<template>
    <div>
        <nav class="navbar navbar-dark bg-dark mb-4">
            <div class="container">
                <span class="navbar-brand">Välkommen till Maddes 30-års fest!</span>
            </div>
        </nav>

        <div class="container">
            <!-- Flikar -->
            <ul class="nav nav-tabs mb-4">
                <li class="nav-item">
                    <button class="nav-link" :class="{ active: activeTab === 'menu' }"
                            @click="activeTab = 'menu'">
                        Drinkmeny
                    </button>
                </li>
                <li class="nav-item">
                    <button class="nav-link" :class="{ active: activeTab === 'gallery' }"
                            @click="activeTab = 'gallery'">
                        Galleri
                    </button>
                </li>
            </ul>

            <!-- Innehåll -->
            <div v-if="activeTab === 'menu'">
                <DrinkList />
            </div>

            <div v-else-if="activeTab === 'gallery'">
                <ImageUpload />
                <ImageGallery />
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
    import { ref } from 'vue'
    import ImageUpload from './components/ImageUpload.vue'
    import ImageGallery from './components/ImageGallery.vue'
    import DrinkList from './components/DrinkList.vue'

    const activeTab = ref<'menu' | 'gallery'>('menu')
</script>
