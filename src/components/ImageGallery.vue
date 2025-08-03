<template>
    <div>
        <h5>Galleri</h5>

        <!--<div v-if="loading" class="text-muted">Laddar bilder...</div>

        <div v-else-if="images.length === 0" class="text-muted">
            Inga bilder uppladdade ännu.
        </div>-->

        <div class="row">
            <div class="col-6 col-md-4 col-lg-3 mb-4" v-for="url in images" :key="url">
                <a :href="url" target="_blank" rel="noopener">
                    <img :src="url" class="img-fluid rounded shadow-sm border" />
                </a>
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
    import { onMounted, onBeforeUnmount, ref } from 'vue'
    import { supabase } from '../lib/supabase'

    const images = ref<string[]>([])
    let intervalId: ReturnType<typeof setInterval>

    const loadImages = async () => {
        const { data, error } = await supabase.storage
            .from('user-uploads')
            .list('', { limit: 100, sortBy: { column: 'created_at', order: 'desc' } })

        if (error || !data) return

        const newUrls = data.map(item =>
            supabase.storage.from('user-uploads').getPublicUrl(item.name).data.publicUrl
        )

        // Endast uppdatera om något ändrats (enkelt sätt: jämför längd eller första bild)
        if (
            newUrls.length !== images.value.length ||
            newUrls[0] !== images.value[0]
        ) {
            images.value = newUrls
        }
    }

    onMounted(async () => {
        await loadImages()

        // Starta polling var 5:e sekund
        intervalId = setInterval(loadImages, 5000)
    })

    onBeforeUnmount(() => {
        if (intervalId) clearInterval(intervalId)
    })




</script>
