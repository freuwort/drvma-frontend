<template>
    <NuxtLayout name="auth-default" limiter="medium" :scope pageTitle="Ahnentafel Editor" color="#3c9fff">
        <HeCard is="form" @submit.prevent="update">
            <div class="flex items-center p-4 rounded-t-2xl border-b sticky top-16 z-20 bg-background">
                <IodButton :is="NuxtLink" corner="pill" label="Zur Übersicht" variant="contained" to="/gencestor/pedigrees"/>
                <HeSpacer />
                <IodButton type="submit" corner="pill" label="Speichern" :loading="form.processing" variant="filled" />
            </div>

            <HeFlex padding="1.5rem 1rem" :gap="3">
                <ErrorAlert :errors="form.errors" />



                <HeFlex :gap="1">
                    <h5 class="m-0 font-medium">Allgemeines</h5>
                    <div class="flex items-center gap-4">
                        <IodInput class="flex-1" label="Zwingername" v-model="form.kennel"/>
                        <IodIcon icon="pen_size_2"/>
                        <IodInput class="flex-1" label="Wurfname" v-model="form.title"/>
                    </div>
                    <VDropdown>
                        <IodInput type="text" label="Züchter Anschrift" icon-right="location_on" :modelValue="stringFromAddress(form.address)" readonly/>
                        <template #popper><IodAddressPicker v-model="form.address"/></template>
                    </VDropdown>
                </HeFlex>

                <HeFlex :gap="1">
                    <HeFlex horizontal :gap=".5">
                        <h5 class="m-0 font-medium flex-1">Welpen</h5>
                        <IodButton type="button" label="Alle Drucken" normal-case icon-left="print" size="s" corner="pill" variant="contained" />
                        <HeDivider vertical class="h-6"/>
                        <IodButton type="button" label="Neu" color-preset="success" normal-case icon-right="add" size="s" corner="pill" />
                    </HeFlex>
                    <IodInput label="Zwingername" v-model="form.kennel"/>
                </HeFlex>
            </HeFlex>
        </HeCard>
    </NuxtLayout>
</template>

<script lang="ts" setup>
    import { toast } from 'vue3-toastify'
    
    const NuxtLink = defineNuxtLink({})
    const scope = 'view_admin_screen_screens_show'

    

    // Params
    const id = computed(() => useRoute().params.id ?? null)



    const form = useForm({
        id: id.value,
        kennel: '',
        title: '',
        address: null,
        animals: [],
        animals_count: 0,
        created_at: '',
        updated_at: '',
    })



    // START: Server routes
    function fetch() {
        form.get(apiRoute('/api/gencestor/pedigrees/:id', { id: id.value }), {
            onSuccess(response: any)
            {
                form.defaults(response.data).reset()
            },
        })
    }

    // function store() {
    //     form
    //     .post(apiRoute('/api/gencestor/pedigrees'), {
    //         onSuccess(response: any)
    //         {
    //             form.defaults(response.data).reset()
    //             toast.success('Ahnentafel wurde erstellt')
    //             navigateTo(apiRoute('/gencestor/pedigrees/editor/:id', { id: response.data?.id }))
    //         },
    //     })
    // }

    function update() {
        form
        .patch(apiRoute('/api/gencestor/pedigrees/:id', { id: id.value }), {
            onSuccess(response: any)
            {
                form.defaults(response.data).reset()
                toast.success('Ahnentafel wurde aktualisiert')
            },
        })
    }
    // END: Server routes



    // Fetch model
    fetch()
</script>

<style lang="sass" scoped>
</style>
