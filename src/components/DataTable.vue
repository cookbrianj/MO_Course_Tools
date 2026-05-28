<template>
  <div class="data-table-container">
    <div class="d-flex justify-space-between align-center mb-4">
      <h3 class="text-h6 font-weight-regular">{{ title }}</h3>
      <div>
        <v-btn color="primary" variant="text" prepend-icon="mdi-plus" @click="addRow" class="mr-2">
          Add Record
        </v-btn>
        <v-btn color="success" variant="elevated" prepend-icon="mdi-download" @click="exportData">
          Save & Download
        </v-btn>
      </div>
    </div>
    
    <v-data-table
      :headers="headers"
      :items="filteredItems"
      :items-per-page="10"
      class="elevation-1 rounded-lg"
      hover
      density="compact"
    >
      <template v-slot:body.prepend>
        <tr>
          <td v-for="header in headers" :key="header.key" class="pa-1 bg-grey-lighten-4">
            <v-text-field
              v-model="filters[header.key]"
              density="compact"
              variant="outlined"
              hide-details
              placeholder="Filter..."
              bg-color="white"
            ></v-text-field>
          </td>
        </tr>
      </template>

      <template v-slot:item="{ item }">
        <tr>
          <td v-for="header in headers" :key="header.key" :class="{ 'error-cell': isError(item.raw || item, header.key) }">
            <div 
              v-if="editingCell.item !== item || editingCell.key !== header.key"
              @click="startEdit(item, header.key, (item.raw || item)[header.key])"
              class="editable-cell"
            >
              {{ (item.raw || item)[header.key] || ' ' }}
            </div>
            <v-text-field
              v-else
              v-model="editingValue"
              density="compact"
              hide-details
              variant="underlined"
              autofocus
              @blur="saveEdit(item, header.key)"
              @keyup.enter="saveEdit(item, header.key)"
              @keyup.esc="cancelEdit"
            ></v-text-field>
          </td>
        </tr>
      </template>
    </v-data-table>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import Papa from 'papaparse'

const props = defineProps({
  title: String,
  items: Array,
  filename: {
    type: String,
    default: 'export.csv'
  },
  errorCheck: {
    type: Function,
    default: () => false
  }
})

const emit = defineEmits(['update:items', 'add-row'])

const localItems = ref([])

watch(() => props.items, (newItems) => {
  localItems.value = [...newItems]
}, { immediate: true, deep: true })

const headers = computed(() => {
  if (localItems.value.length === 0) return []
  const keys = Object.keys(localItems.value[0])
  return keys.map(key => ({
    title: key,
    key: key,
    align: 'start',
    sortable: true
  }))
})

const filters = ref({})

const filteredItems = computed(() => {
  return localItems.value.filter(row => {
    for (const key in filters.value) {
      const filterValue = filters.value[key]
      if (filterValue) {
        const rowValue = String(row[key] || '').toLowerCase()
        if (!rowValue.includes(filterValue.toLowerCase())) {
          return false
        }
      }
    }
    return true
  })
})

const editingCell = ref({ item: null, key: null })
const editingValue = ref('')

const startEdit = (item, key, value) => {
  editingCell.value = { item, key }
  editingValue.value = value
}

const saveEdit = (item, key) => {
  if (editingCell.value.item === item && editingCell.value.key === key) {
    const rawItem = item.raw || item
    const idx = localItems.value.indexOf(rawItem)
    if (idx !== -1) {
      localItems.value[idx][key] = editingValue.value
    } else {
      rawItem[key] = editingValue.value
    }
    editingCell.value = { item: null, key: null }
    emit('update:items', localItems.value)
  }
}

const cancelEdit = () => {
  editingCell.value = { item: null, key: null }
}

const addRow = () => {
  const newRow = {}
  if (headers.value.length > 0) {
    headers.value.forEach(h => {
      newRow[h.key] = ''
    })
  } else {
    // Default keys if empty dataset
    const defaultKeys = ['CurrentSchoolYear', 'ReportingDistrictCode', 'ReportingSchoolCode', 'EDSSN', 'PosCode', 'CTEProgType', 'AssignNum', 'StateID']
    defaultKeys.forEach(k => newRow[k] = '')
  }
  emit('add-row', newRow)
}

const exportData = () => {
  const csv = Papa.unparse(localItems.value)
  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
  const link = document.createElement('a')
  const url = URL.createObjectURL(blob)
  link.setAttribute('href', url)
  link.setAttribute('download', props.filename)
  link.style.visibility = 'hidden'
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
}

const isError = (item, key) => {
  return props.errorCheck(item, key)
}
</script>

<style scoped>
.editable-cell {
  min-height: 24px;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  transition: background-color 0.2s;
}
.editable-cell:hover {
  background-color: rgba(0,0,0,0.05);
}
.error-cell {
  background-color: rgba(255, 82, 82, 0.1); /* Soft red for errors */
}
</style>
