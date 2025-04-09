<template>
    <IodPopup ref="popup" :title max-width="700px" blur="0">
        <HeFlex is="form" padding="1.5rem" gap="2.5rem" @submit.prevent="save">
            <ErrorAlert :errors="form.errors" />
            
            <div class="grid grid-cols-2 gap-4">
                <IodInput type="text" v-model="form.name" label="Name" ref="nameInput"/>
                <IodInput type="text" v-model="form.kennel" label="Zwinger">
                    <template #right>
                        <IodIoggle v-model="form.kennel_name_first" v-tooltip="'Zwingername folgt zuerst'"/>
                    </template>
                </IodInput>

                <IodSelect v-model="form.sex" label="Geschlecht" :options="[
                    {value: 'male', text: 'Rüde'},
                    {value: 'female', text: 'Hündin'},
                    {value: 'unknown', text: 'Unbekannt'},
                ]"/>
                <IodInput type="text" v-model="form.breed" label="Rasse"/>
                
                <IodInput type="text" v-model="form.chip_number" label="Chipnummer"/>
                <IodInput type="text" v-model="form.studbook_number" label="Zuchtbuchnummer"/>
            </div>
            
            <div class="grid grid-cols-2 gap-4">
                <IodInput type="date" v-model="form.birthdate" placeholder="Geburtsdatum"/>
                <IodInput type="text" v-model="form.size" label="Größe (cm)"/>
                
                <IodInput type="text" v-model="form.hair_type" label="Haartyp"/>
                <IodInput type="text" v-model="form.hair_color" label="Haarfarbe"/>
            </div>
            
            <div class="grid grid-cols-2 gap-4">
                <button class="parent-button father" type="button" @click="animalSearchPopup.open({sex: 'male'}, (item) => form.father = item)" v-tooltip="'Klicken zum Auswählen'">
                    <div class="bar"><IodIcon icon="male"/><span>Vater</span></div>
                    <div class="info-wrapper">
                        <template v-if="form.father">
                            <b>{{ form.father.full_name }}</b>
                            <span>{{ form.father.breed }}</span>
                        </template>
                        <i v-else>Vater Auswählen</i>
                    </div>
                </button>
                <button class="parent-button mother" type="button" @click="animalSearchPopup.open({sex: 'female'}, (item) => form.mother = item)" v-tooltip="'Klicken zum Auswählen'">
                    <div class="bar"><IodIcon icon="female"/><span>Mutter</span></div>
                    <div class="info-wrapper">
                        <template v-if="form.mother">
                            <b>{{ form.mother.full_name }}</b>
                            <span>{{ form.mother.breed }}</span>
                        </template>
                        <i v-else>Mutter Auswählen</i>
                    </div>
                </button>
            </div>
            
            <div class="grid grid-cols-1 gap-4">
                <div class="flex vertical background-soft radius-m full-width">
                    <TextEditor simplified v-show="tab === 'notes'" v-model="form.note"/>
                    <TextEditor simplified v-show="tab === 'gen1'" v-model="form.awards.gen1"/>
                    <TextEditor simplified v-show="tab === 'gen2'" v-model="form.awards.gen2"/>
                    <TextEditor simplified v-show="tab === 'gen3'" v-model="form.awards.gen3"/>
                    <TextEditor simplified v-show="tab === 'gen4'" v-model="form.awards.gen4"/>
                    <div class="flex border-top" style="padding: .5rem 1rem" v-show="['gen2', 'gen3', 'gen4'].includes(tab)">
                        <div class="spacer"></div>
                        <IodButton
                            type="button"
                            size="small"
                            variant="contained"
                            label="von voriger Genaration übernehmen"
                            :disabled="!textFromPreviousGeneration"
                            @click="copyTextFromPreviousGeneration"
                        />
                    </div>
                </div>
            </div>

            <HeFlex horizontal gap="1rem">
                <IodButton type="button" class="flex-1" corner="pill" variant="contained" label="Abbrechen" @click="popup.close()" :loading="form.processing" />
                <IodButton type="submit" class="flex-1" corner="pill" variant="filled" :label="saveButtonLabel" :loading="form.processing" />
            </HeFlex>
        </HeFlex>
    </IodPopup>

    <GencestorSearchAnimal ref="animalSearchPopup"/>
</template>

<script lang="ts" setup>
    import dayjs from 'dayjs'



    const template = {
        id: null,
        chip_number: '',
        studbook_number: '',
        name: '',
        kennel: '',
        kennel_name_first: false,
        awards: {
            gen1: '',
            gen2: '',
            gen3: '',
            gen4: '',
        },
        note: '',
        birthdate: '',
        breed: '',
        sex: 'female',
        size: '',
        hair_type: '',
        hair_color: '',
        mother: null,
        father: null,
        created_at: null,
        updated_at: null,
    }

    const form = useForm(template)
    const popup = ref(null)
    const nameInput = ref(null)
    const searchAnimalPopup = ref(null)
    const successCallback = ref(null)
    const tab = ref('notes')

    const title = computed(() => form.id ? 'Tier bearbeiten' : 'Neues Tier erstellen')
    const saveButtonLabel = computed(() => form.id ? 'Speichern' : 'Tier erstellen')



    function open(item, callback = null) {
        form.reset()
        popup.value.open()
        successCallback.value = callback

        switch (typeof item)
        {
            case 'string': fetchItem(item); break
            case 'number': fetchItem(item); break
            case 'object': openItem(item); break
            default: openItem(); break
        }

        nextTick(() => nameInput.value.focus())
    }

    async function fetchItem(id) {
        form.processing = true

        try {
            let response = await axios.get(apiRoute('/api/gencestor/animals/{id}', { id }))
            openItem(response?.data?.data)
        }
        catch (error) {
            popup.value.close()
        }
        
        form.processing = false
    }

    function openItem(item = {}) {
        item = {...template, ...item}

        form.id = item.id
        form.chip_number = item.chip_number || ''
        form.studbook_number = item.studbook_number || ''
        form.name = item.name || ''
        form.kennel = item.kennel || ''
        form.kennel_name_first = item.kennel_name_first || false
        form.awards = {...item.awards}
        form.note = item.note || ''
        form.birthdate = item.birthdate ? dayjs(item.birthdate).format('YYYY-MM-DD') : ''
        form.breed = item.breed || ''
        form.sex = item.sex || 'unknown'
        form.size = item.size || ''
        form.hair_type = item.hair_type || ''
        form.hair_color = item.hair_color || ''
        form.mother = item.mother
        form.father = item.father
        form.created_at = item.created_at
        form.updated_at = item.updated_at
    }



    const save = form.id ? update : store

    function store() {
        form
        .transform((data) => {
            return {
                ...data,
                mother_id: data.mother?.id || null,
                father_id: data.father?.id || null,
            }
        })
        .post(apiRoute('admin.gencestor.animals.store'), {
            onSuccess: (data) => {
                done(data?.props?.flash?.success || null)
            },
        })
    }

    function update() {
        form
        .transform((data) => {
            return {
                ...data,
                mother_id: data.mother?.id || null,
                father_id: data.father?.id || null,
            }
        })
        .put(route('admin.gencestor.animals.update', form.id), {
            onSuccess: (data) => {
                done(data?.props?.flash?.success || null)
            },
        })
    }

    const done = (id) => {
        popup.value.close()
        successCallback.value?.(id)
    }



    // START: Handle generation text copy
    const textFromPreviousGeneration = computed(() => {
        switch (tab.value)
        {
            case 'gen2': return form.awards.gen1 || '';
            case 'gen3': return form.awards.gen2 || form.awards.gen1 || '';
            case 'gen4': return form.awards.gen3 || form.awards.gen2 || form.awards.gen1 || '';
            default: return '';
        }
    })

    const copyTextFromPreviousGeneration = () => {
        if (tab.value === 'notes') return

        form.awards[tab.value] = textFromPreviousGeneration.value
    }



    defineExpose({
        open
    })
</script>

<style lang="sass" scoped>
    .parent-button
        border-radius: var(--radius-m)
        background: var(--color-background-soft)
        border: 1px solid transparent
        display: flex
        flex-direction: column
        gap: 0
        justify-content: flex-start
        align-items: stretch
        overflow: hidden
        transition: all 100ms ease-in-out
        border: 1px solid transparent

        &.mother
            .bar
                color: var(--color-background)
                background: #e84393

        &.father
            .bar
                color: var(--color-background)
                background: #3867d6

        &:hover
            border: 1px solid var(--color-background-soft)
            background: var(--color-background)
            box-shadow: var(--shadow-elevation-low)

        .info-wrapper
            display: flex
            flex-direction: column
            gap: 0
            line-height: 1.5rem
            padding: .5rem 1rem

        .bar
            height: 2rem
            display: flex
            align-items: center
            color: var(--color-background)
            background: var(--color-text)
            padding: 0 .75rem
            font-size: .8rem
            font-weight: bold
            gap: .5rem

            .icon
                font-size: 1.25rem
                font-weight: lighter

    .popup-footer
        background: var(--color-background-soft)
        border-radius: 0 0 var(--radius-m) var(--radius-m)
        padding: 1rem
</style>