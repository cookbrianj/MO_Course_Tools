<template>
  <v-app>
    <v-app-bar color="primary" elevation="2">
      <v-app-bar-title class="text-h5 font-weight-medium">MOSIS Data Reconciliation</v-app-bar-title>
    </v-app-bar>

    <v-main class="bg-grey-lighten-4">
      <v-container fluid class="px-6 py-6">
        <!-- Upload Section -->
        <v-row class="mb-6">
          <v-col cols="12" md="4">
            <v-card class="elevation-1 rounded-lg pa-4">
              <h3 class="text-subtitle-1 font-weight-bold mb-2">1. October Course Assignment</h3>
              <v-file-input
                v-model="files.octCourse"
                accept=".csv,.txt,.tsv"
                label="Upload File"
                variant="outlined"
                density="compact"
                hide-details
                prepend-icon="mdi-file-document-outline"
                @update:modelValue="handleFileUpload($event, 'octCourse')"
              ></v-file-input>
            </v-card>
          </v-col>
          <v-col cols="12" md="4">
            <v-card class="elevation-1 rounded-lg pa-4">
              <h3 class="text-subtitle-1 font-weight-bold mb-2">2. October Student Assignment</h3>
              <v-file-input
                v-model="files.octStudent"
                accept=".csv,.txt,.tsv"
                label="Upload File"
                variant="outlined"
                density="compact"
                hide-details
                prepend-icon="mdi-account-group-outline"
                @update:modelValue="handleFileUpload($event, 'octStudent')"
              ></v-file-input>
            </v-card>
          </v-col>
          <v-col cols="12" md="4">
            <v-card class="elevation-1 rounded-lg pa-4">
              <h3 class="text-subtitle-1 font-weight-bold mb-2">3. June Student Course Completion</h3>
              <v-file-input
                v-model="files.juneCompletion"
                accept=".csv,.txt,.tsv"
                label="Upload File"
                variant="outlined"
                density="compact"
                hide-details
                prepend-icon="mdi-check-decagram-outline"
                @update:modelValue="handleFileUpload($event, 'juneCompletion')"
              ></v-file-input>
            </v-card>
          </v-col>
        </v-row>

        <!-- Dashboard Tabs -->
        <v-card class="elevation-1 rounded-lg">
          <v-tabs v-model="activeTab" color="primary" bg-color="white">
            <v-tab value="missingAssignment">
              <v-icon start>mdi-alert-circle-outline</v-icon>
              Missing from Assignment
              <v-badge v-if="missingFromAssignment.length" :content="missingFromAssignment.length" color="error" inline></v-badge>
            </v-tab>
            <v-tab value="missingCompletion">
              <v-icon start>mdi-alert-circle-outline</v-icon>
              Missing from Completion
              <v-badge v-if="missingFromCompletion.length" :content="missingFromCompletion.length" color="warning" inline></v-badge>
            </v-tab>
            <v-tab value="octCourse">October Course Data</v-tab>
            <v-tab value="octStudent">October Student Data</v-tab>
            <v-tab value="juneCompletion">June Completion Data</v-tab>
          </v-tabs>

          <v-card-text class="bg-grey-lighten-5 pt-4">
            <v-window v-model="activeTab">
              
              <v-window-item value="missingAssignment">
                <v-alert v-if="!datasets.juneCompletion.length || !datasets.octStudent.length" type="info" variant="tonal" class="mb-4">
                  Please upload both October Student Assignment and June Course Completion files to view this report.
                </v-alert>
                <DataTable 
                  v-else
                  title="Missing from October Assignment" 
                  :items="missingFromAssignment" 
                  filename="MissingFromAssignment.csv"
                  :errorCheck="() => true" 
                />
              </v-window-item>

              <v-window-item value="missingCompletion">
                <v-alert v-if="!datasets.juneCompletion.length || !datasets.octStudent.length" type="info" variant="tonal" class="mb-4">
                  Please upload both October Student Assignment and June Course Completion files to view this report.
                </v-alert>
                <DataTable 
                  v-else
                  title="Missing from June Completion" 
                  :items="missingFromCompletion" 
                  filename="MissingFromCompletion.csv"
                  :errorCheck="() => true" 
                />
              </v-window-item>

              <v-window-item value="octCourse">
                <DataTable 
                  title="October Course Assignment" 
                  :items="datasets.octCourse" 
                  @update:items="datasets.octCourse = $event"
                  filename="OctoberCourse_Edited.csv"
                />
              </v-window-item>

              <v-window-item value="octStudent">
                <DataTable 
                  title="October Student Assignment" 
                  :items="datasets.octStudent" 
                  @update:items="datasets.octStudent = $event"
                  filename="OctoberStudent_Edited.csv"
                />
              </v-window-item>

              <v-window-item value="juneCompletion">
                <DataTable 
                  title="June Student Course Completion" 
                  :items="datasets.juneCompletion" 
                  @update:items="datasets.juneCompletion = $event"
                  filename="JuneCompletion_Edited.csv"
                />
              </v-window-item>

            </v-window>
          </v-card-text>
        </v-card>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, reactive, computed } from 'vue'
import Papa from 'papaparse'
import DataTable from './components/DataTable.vue'

const activeTab = ref('missingAssignment')

const files = reactive({
  octCourse: null,
  octStudent: null,
  juneCompletion: null
})

const datasets = reactive({
  octCourse: [],
  octStudent: [],
  juneCompletion: []
})

const courseKeys = [
  'CurrentSchoolYear',
  'ReportingDistrictCode',
  'ReportingSchoolCode',
  'EDSSN',
  'PosCode',
  'CTEProgType',
  'AssignNum'
]

const studentKeys = [...courseKeys, 'StateID']

// Clean keys to handle potential whitespace or case issues from CSVs
const normalizeRow = (row) => {
  const normalized = {}
  for (const key in row) {
    // Basic normalization: trim keys
    normalized[key.trim()] = row[key]
  }
  return normalized
}

const generateKeyString = (row, keys) => {
  return keys.map(k => String(row[k] || '').trim().toLowerCase()).join('|')
}

const handleFileUpload = (file, datasetName) => {
  if (!file) {
    datasets[datasetName] = []
    return
  }

  // Handle both single file from old Vuetify vs array from newer Vuetify v-file-input
  const actualFile = Array.isArray(file) ? file[0] : file
  if (!actualFile) return

  Papa.parse(actualFile, {
    header: true,
    skipEmptyLines: true,
    complete: (results) => {
      // Normalize rows
      datasets[datasetName] = results.data.map(normalizeRow)
    }
  })
}

// Missing from Assignment: Records present in the June Course Completion file but missing matching course/student keys in the October Student files.
const missingFromAssignment = computed(() => {
  if (!datasets.juneCompletion.length || !datasets.octStudent.length) return []

  const octStudentSet = new Set(datasets.octStudent.map(row => generateKeyString(row, studentKeys)))

  return datasets.juneCompletion.filter(juneRow => {
    const juneKey = generateKeyString(juneRow, studentKeys)
    return !octStudentSet.has(juneKey)
  })
})

// Missing from Completion: Records present in the October Student Assignment files but missing from the June Course Completion file.
const missingFromCompletion = computed(() => {
  if (!datasets.juneCompletion.length || !datasets.octStudent.length) return []

  const juneCompletionSet = new Set(datasets.juneCompletion.map(row => generateKeyString(row, studentKeys)))

  return datasets.octStudent.filter(octRow => {
    const octKey = generateKeyString(octRow, studentKeys)
    return !juneCompletionSet.has(octKey)
  })
})
</script>

<style>
/* Global app styles if needed */
html {
  overflow-y: auto;
}
</style>
