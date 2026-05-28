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
      :items="localItems"
      :items-per-page="10"
      class="elevation-1 rounded-lg"
      hover
      density="compact"
    >
      <template v-slot:item="{ item, index }">
        <tr>
          <td v-for="header in headers" :key="header.key" :class="{ 'error-cell': isError(item, header.key) }">
            <div 
              v-if="editingCell.index !== index || editingCell.key !== header.key"
              @click="startEdit(index, header.key, item[header.key])"
              class="editable-cell"
            >
              {{ item[header.key] || ' ' }}
            </div>
            <v-text-field
              v-else
              v-model="editingValue"
              density="compact"
              hide-details
              variant="underlined"
              autofocus
              @blur="saveEdit(index, header.key)"
              @keyup.enter="saveEdit(index, header.key)"
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

const emit = defineEmits(['update:items'])

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

const editingCell = ref({ index: -1, key: null })
const editingValue = ref('')

const startEdit = (index, key, value) => {
  editingCell.value = { index, key }
  editingValue.value = value
}

const saveEdit = (index, key) => {
  if (editingCell.value.index === index && editingCell.value.key === key) {
    localItems.value[index][key] = editingValue.value
    editingCell.value = { index: -1, key: null }
    emit('update:items', localItems.value)
  }
}

const cancelEdit = () => {
  editingCell.value = { index: -1, key: null }
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
  localItems.value.unshift(newRow)
  emit('update:items', localItems.value)
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
