<template>
    <NuxtLayout name="auth-default" :scope :pageTitle="IPM.options.pageTitle" color="#3c9fff">
        <HeCard>
            <IodTable
                class="top-16"
                :scope
                :columns="tableColumns"
                :actions="tableActions"
                :filterSettings="tableFilters"
                :items="IPM.items"
                :loading="IPM.processing"
                v-model:selection="IPM.selection"
                v-model:filter="IPM.modelFilter"
                v-model:sort="IPM.modelSort"
                v-model:pagination="IPM.modelPagination"
                v-model:columnSettings="IPM.modelColumnSettings"
                @request:refresh="IPM.fetch()"
            >
                <template #right>
                    <IodButton type="button" corner="pill" variant="filled" label="Neues Tier" icon-right="add" @click="newAnimalPopup.open(null)"/>
                </template>
            </IodTable>
        </HeCard>

        <!-- <DialogCsvImport ref="importPopup" :fields="importFields" @import="importUsers" /> -->
        <GencestorNewAnimal ref="newAnimalPopup" />
    </NuxtLayout>
</template>

<script lang="ts" setup>
    import type { FilterSetting } from '~/components/Iod/IodTable.vue'
    import { FieldGroup, Field } from '~/classes/import/CsvImport'
    import TableColumnBuilder from '~/classes/Builder/TableColumnBuilder'
    import { toast } from 'vue3-toastify'
    
    const dayjs = useDayjs()
    const NuxtLink = defineNuxtLink({})
    const scope = 'view_admin_gencestor_animals_index'



    const newAnimalPopup = ref()
    const IPM = useItemPageManager({
        scope,
        pageTitle: 'Tier Verwaltung',
        routes: {
            fetch: '/api/gencestor/animals',
            duplicate: '/api/gencestor/animals/:id/duplicate',
            delete: '/api/gencestor/animals',
        },
    })
    
    const tableColumnBuilder = new TableColumnBuilder()
    const tableColumns = [
        tableColumnBuilder.new().name('id').label('ID').width(70).build(),
        tableColumnBuilder.new().name('name').label('Name').transform((value: string | null, item: any) => ({
            text: value || '-',
            tooltip: value || '-',
            image: item.avatar,
            icon: 'person',
        })).build(),
        tableColumnBuilder.new().name('organisation').label('Organisation').build(),
        tableColumnBuilder.new().name('customer_id').label('Kunden-Nr.').build(),
        tableColumnBuilder.new().name('employee_id').label('Personal-Nr.').build(),
        tableColumnBuilder.new().name('member_id').label('Mitglieds-Nr.').build(),
        tableColumnBuilder.new().name('username').label('Nutzername').build(),
        tableColumnBuilder.new().name('email').label('Email').build(),
        tableColumnBuilder.new().name('phone').label('Telefon').build(),
        tableColumnBuilder.new().name('main_address').label('Hauptadresse').sortable(false).transform((value: any) => stringFromAddress(value) || null).build(),
        tableColumnBuilder.new().name('billing_address').label('Rechnungsadresse').sortable(false).transform((value: any) => stringFromAddress(value) || null).build(),
        tableColumnBuilder.new().name('shipping_address').label('Lieferadresse').sortable(false).transform((value: any) => stringFromAddress(value) || null).build(),
        tableColumnBuilder.new().name('roles').label('Rollen').sortable(false).transform((value: any[]) => value?.map(e => e.name)?.join(', ') || null).build(),
        tableColumnBuilder.new().name('is_admin').label('Berechtigungslevel').sortable(false).transform((value: boolean | null, item: any) => {
            if (item?.is_admin) return { text: 'Admin', icon: 'shield', color: 'var(--color-info)', }
            if (item?.has_elevated_permissions) return { text: 'Erweitert', icon: 'check_circle', color: 'var(--color-info)', }
            return { text: 'Basis', icon: 'key', color: 'var(--color-text-soft)', }
        }).build(),
        tableColumnBuilder.new().name('email_verified_at').label('Email bestätigt').transform((value: string | null) => ({
            text: value ? dayjs(value).fromNow() : 'Ausstehend',
            tooltip: value ? dayjs(value).format('DD.MM.YYYY HH:mm') : 'Ausstehend',
            icon: value ? 'check' : 'close',
            color: value ? 'var(--color-success)' : 'var(--color-error)',
        })).build(),
        tableColumnBuilder.new().name('phone_verified_at').label('Telefon bestätigt').transform((value: string | null, item: any) => ({
            text: value ? dayjs(value).fromNow() : 'Ausstehend',
            tooltip: value ? dayjs(value).format('DD.MM.YYYY HH:mm') : 'Ausstehend',
            icon: value ? 'check' : 'close',
            color: value ? 'var(--color-success)' : 'var(--color-error)',
        })).build(),
        tableColumnBuilder.new().name('enabled_at').label('Freigegeben').transform((value: string | null, item: any) => ({
            text: value ? dayjs(value).fromNow() : 'Ausstehend',
            tooltip: value ? dayjs(value).format('DD.MM.YYYY HH:mm') : 'Ausstehend',
            icon: value ? 'check' : 'close',
            color: value ? 'var(--color-success)' : 'var(--color-error)',
        })).build(),
        tableColumnBuilder.new().name('blocked_at').label('Gesperrt').transform((value: string | null, item: any) => ({
            text: value ? dayjs(value).fromNow() : 'Aktiv',
            tooltip: value ? `${dayjs(value).format('DD.MM.YYYY HH:mm')} - Grund: ${item.block_reason || 'Nicht angegeben'}` : 'Nein',
            icon: value ? 'do_not_disturb_on' : '',
            color: value ? 'var(--color-error)' : 'var(--color-info)',
        })).build(),
        tableColumnBuilder.new().name('has_tfa_enabled').label('Zweiter Faktor').transform((value: boolean | null, item: any) => ({
            text: value ? 'Aktiv' : 'Inaktiv',
            tooltip: value ? 'Aktiv' : 'Inaktiv',
            icon: value ? 'check' : 'close',
            color: value ? 'var(--color-success)' : 'var(--color-error)',
        })).build(),
        tableColumnBuilder.new().name('last_login_at').label('Zuletzt eingeloggt').transform((value: string | null) => ({
            text: value ? dayjs(value).fromNow() : 'Nie',
            tooltip: value ? dayjs(value).format('DD.MM.YYYY HH:mm') : 'Nie',
        })).build(),
        tableColumnBuilder.new().name('created_at').label('Registriert').transform((value: string | null) => ({
            text: value ? dayjs(value).fromNow() : '-',
            tooltip: value ? dayjs(value).format('DD.MM.YYYY HH:mm') : '-',
        })).build(),
    ]

    const tableActions = [
        {
            icon: 'refresh',
            text: 'Aktualisieren',
            scope: ['global'],
            run: () => IPM.fetch(),
        },
        {
            icon: 'output_circle',
            text: 'Importieren',
            scope: ['global'],
            run: () => importPopup.value.select(),
        },
        // {
        //     icon: 'input_circle',
        //     text: 'Exportieren',
        //     scope: ['global'],
        //     run: () => importPopup.value.select(),
        // },
        // {
        //     icon: 'send',
        //     text: 'Nachricht versenden',
        //     scope: ['global'],
        //     run: (context: string, items: any) => sendEmailPopup.open(items),
        // },
        {
            icon: 'edit',
            text: 'Bearbeiten',
            scope: ['individual', 'row'],
            run: (context: string, items: any) => navigateTo(`/users/editor/${items[0]}`),
        },
        // {
        //     icon: 'content_copy',
        //     text: 'Duplizieren',
        //     scope: ['individual'],
        //     run: (context: string, items: (number | string)[]) => IPM.duplicate(items[0]),
        // },
        {
            icon: 'deselect',
            text: 'Auswahl abwählen',
            scope: ['selection'],
            run: (context: string, items: any) => IPM.deselectAll(),
        },
        {
            icon: 'delete',
            text: 'Löschen',
            color: 'var(--color-error)',
            scope: ['selection', 'individual'],
            run: (context: string, items: any) => IPM.delete(items),
        },
    ]

    const tableFilters = ref<FilterSetting[]>([
        {
            name: 'roles',
            label: 'Rollen',
            type: 'select',
            multiple: true,
            values: computed(() => IPM.availableFilterValues['roles']?.map((item: any) => ({ value: item, text: item })) || []),
        },
        {
            name: 'permission_levels',
            label: 'Berechtigungslevel',
            type: 'select',
            multiple: true,
            values: [
                { value: 'admin', text: 'Admin' },
                { value: 'elevated', text: 'Erweitert' },
            ],
        },
        {
            name: 'email_verified',
            label: 'Email bestätigt',
            type: 'select',
            multiple: false,
            values: [
                { value: 'pending', text: 'Ausstehend' },
                { value: 'active', text: 'Bestätigt' },
            ],
        },
        {
            name: 'enabled',
            label: 'Freigegeben',
            type: 'select',
            multiple: false,
            values: [
                { value: 'pending', text: 'Ausstehend' },
                { value: 'active', text: 'Freigegeben' },
            ],
        },
        {
            name: 'tfa_enabled',
            label: 'Zweiter Faktor',
            type: 'select',
            multiple: false,
            values: [
                { value: 'inactive', text: 'Inaktiv' },
                { value: 'active', text: 'Aktiv' },
            ],
        },
        {
            name: 'deleted',
            label: 'Gelöscht',
            type: 'select',
            multiple: false,
            values: [
                { value: 'deleted', text: 'Ja' },
            ],
        }
    ])



    // START: Import
    const importPopup = ref()
    const importFields = ref([
        new FieldGroup('Allgemeines', [
            new Field('username', 'Nutzername'),
            new Field('email', 'Email'),
            new Field('password', 'Passwort'),
            new Field('roles', 'Rollen'),
        ]),
        new FieldGroup('Vollständiger Name', [
            new Field('salutation', 'Anrede'),
            new Field('prefix', 'Titel'),
            new Field('firstname', 'Vorname'),
            new Field('middlename', 'Zweiter Vorname'),
            new Field('lastname', 'Nachname'),
            new Field('suffix', 'Suffix'),
            new Field('legalname', 'Rechtlicher Name'),
            new Field('nickname', 'Spitzname'),
        ]),
        new FieldGroup('Identifikation', [
            new Field('customer_id', 'Kunden-Nr.'),
            new Field('employee_id', 'Personal-Nr.'),
            new Field('member_id', 'Mitglieds-Nr.'),
        ]),
        new FieldGroup('Organisation', [
            new Field('organisation', 'Organisation'),
            new Field('department', 'Abteilung'),
            new Field('job_title', 'Position'),
        ]),
    ])

    async function importUsers(data: any[])
    {
        await useAxios().post('/api/users/import', {items: data})
        toast.success('Nutzer wurden importiert')
        IPM.fetch()
    }
    // END: Import
</script>

<style lang="sass" scoped></style>
