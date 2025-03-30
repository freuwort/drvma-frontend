<template>
    <IodPopup ref="popup" title="Neue Ahnentafel erstellen" max-width="500px" blur="0">
        <HeFlex is="form" padding="1.5rem" gap="2.5rem" @submit.prevent="store">
            <ErrorAlert :errors="form.errors" />
            
            <div class="flex flex-col gap-4">
                <IodInput label="Zwingername" required v-model="form.kennel"/>
                <IodInput label="Wurfname" required v-model="form.title"/>
            </div>

            <HeFlex horizontal gap="1rem">
                <IodButton type="button" class="flex-1" corner="pill" variant="contained" label="Abbrechen" @click="popup.close()" :loading="form.processing" />
                <IodButton type="submit" class="flex-1" corner="pill" variant="filled" label="Erstellen" :loading="form.processing" />
            </HeFlex>
        </HeFlex>
    </IodPopup>
</template>

<script lang="ts" setup>
    import { toast } from 'vue3-toastify'

    const popup = ref()
    const form = useForm({
        kennel: '',
        title: '',
    })
    
    // START: API functions
    defineExpose({ open })
    
    function open() {
        form.reset()
        popup.value.open()
    }
    
    function store() {
        form.post(apiRoute('/api/gencestor/pedigrees'), {
            onSuccess(response: any)
            {
                navigateTo(apiRoute('/gencestor/pedigrees/editor/:id', { id: response.data?.id }))
                popup.value.close()
            },
        })
    }
    // END: API functions
</script>

<style lang="sass" scoped>
</style>