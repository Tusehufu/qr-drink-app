<template>
    <div class="mb-4">
        <h5>Ladda upp bild</h5>

        <input type="file"
               accept="image/*"
               @change="onFileChange"
               class="form-control mb-2" />

        <div v-if="previewUrl" class="mt-3">
            <p>Förhandsgranskning:</p>
            <img :src="previewUrl" class="img-fluid rounded border" style="max-height: 300px" />
        </div>

        <button class="btn btn-primary mt-3"
                :disabled="!selectedFile"
                @click="uploadImage">
            Ladda upp
        </button>
    </div>
</template>

<script setup lang="ts">
    import { ref } from 'vue'
    import { supabase } from '../lib/supabase'

    const selectedFile = ref<File | null>(null)
    const previewUrl = ref<string | null>(null)

    function onFileChange(event: Event) {
        const files = (event.target as HTMLInputElement).files
        if (files && files.length > 0) {
            selectedFile.value = files[0]
            previewUrl.value = URL.createObjectURL(files[0])
        }
    }

    async function uploadImage() {
        if (!selectedFile.value) return

        const file = selectedFile.value
        const fileName = `${Date.now()}_${file.name}`

        const { error } = await supabase.storage
            .from('user-uploads')
            .upload(fileName, file)

        if (error) {
            alert('Fel vid uppladdning: ' + error.message)
            return
        }

        const { data } = supabase.storage
            .from('user-uploads')
            .getPublicUrl(fileName)

    }
</script>
