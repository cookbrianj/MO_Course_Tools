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
        <div class="d-flex justify-space-between align-center mb-4">
          <h2 class="text-h6 font-weight-medium text-grey-darken-3">Reconciliation Dashboard</h2>
          <v-btn color="primary" prepend-icon="mdi-refresh" @click="revalidate" elevation="2">
            Revalidate
          </v-btn>
        </div>
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
                  @add-row="datasets.juneCompletion.unshift($event)"
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
                  @add-row="datasets.octStudent.unshift($event)"
                />
              </v-window-item>

              <v-window-item value="octCourse">
                <DataTable 
                  title="October Course Assignment" 
                  :items="datasets.octCourse" 
                  @update:items="datasets.octCourse = $event"
                  @add-row="datasets.octCourse.unshift($event)"
                  filename="OctoberCourse_Edited.csv"
                />
              </v-window-item>

              <v-window-item value="octStudent">
                <DataTable 
                  title="October Student Assignment" 
                  :items="datasets.octStudent" 
                  @update:items="datasets.octStudent = $event"
                  @add-row="datasets.octStudent.unshift($event)"
                  filename="OctoberStudent_Edited.csv"
                />
              </v-window-item>

              <v-window-item value="juneCompletion">
                <DataTable 
                  title="June Student Course Completion" 
                  :items="datasets.juneCompletion" 
                  @update:items="datasets.juneCompletion = $event"
                  @add-row="datasets.juneCompletion.unshift($event)"
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
    const cleanKey = key.trim()
    let val = row[key]
    if (cleanKey === 'ReportingDistrictCode' && val) {
      val = String(val).trim().padStart(6, '0')
    } else if (cleanKey === 'ReportingSchoolCode' && val) {
      val = String(val).trim().padStart(4, '0')
    }
    normalized[cleanKey] = val
  }
  return normalized
}

const generateKeyString = (row, keys) => {
  return keys.map(k => {
    let val = String(row[k] || '').trim().toLowerCase()
    if (k === 'ReportingDistrictCode' && val) {
      val = val.padStart(6, '0')
    } else if (k === 'ReportingSchoolCode' && val) {
      val = val.padStart(4, '0')
    }
    return val
  }).join('|')
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
      let normalizedData = results.data.map(normalizeRow)

      // Filter for Grade > 5 if a grade column exists
      if (normalizedData.length > 0) {
        const keys = Object.keys(normalizedData[0])
        // Clean keys to handle spaces and underscores (e.g. 'Grade Level' -> 'gradelevel')
        const cleanKeys = keys.map(k => k.toLowerCase().replace(/[^a-z0-9]/g, ''))
        
        let gradeKeyIndex = cleanKeys.findIndex(k => 
          ['grade', 'gradelevel', 'stategrade', 'studentgrade', 'currentgrade'].includes(k)
        )
        // Fallback: any column with 'grade' in it
        if (gradeKeyIndex === -1) {
          gradeKeyIndex = cleanKeys.findIndex(k => k.includes('grade') && !k.includes('upgrade'))
        }
        
        if (gradeKeyIndex !== -1) {
          const gradeKey = keys[gradeKeyIndex]
          normalizedData = normalizedData.filter(row => {
            const val = String(row[gradeKey] || '').trim()
            const num = parseInt(val, 10)
            if (!isNaN(num)) {
              if (num > 5) {
                // Zero-pad to 2 digits
                row[gradeKey] = String(num).padStart(2, '0')
                return true
              }
            }
            return false // Drop K, PK, PKP, etc.
          })
        }
      }

      datasets[datasetName] = normalizedData
      revalidate()
    }
  })
}

const missingFromAssignment = ref([])
const missingFromCompletion = ref([])

const revalidate = () => {
  if (!datasets.juneCompletion.length || !datasets.octStudent.length) {
    missingFromAssignment.value = []
    missingFromCompletion.value = []
    return
  }

  // Missing from Assignment
  const octStudentSet = new Set(datasets.octStudent.map(row => generateKeyString(row, studentKeys)))
  missingFromAssignment.value = datasets.juneCompletion.filter(juneRow => {
    const juneKey = generateKeyString(juneRow, studentKeys)
    return !octStudentSet.has(juneKey)
  })

  // Missing from Completion
  const juneCompletionSet = new Set(datasets.juneCompletion.map(row => generateKeyString(row, studentKeys)))
  missingFromCompletion.value = datasets.octStudent.filter(octRow => {
    const octKey = generateKeyString(octRow, studentKeys)
    return !juneCompletionSet.has(octKey)
  })
}
</script>

<style>
/* Global app styles if needed */
html {
  overflow-y: auto;
  font-size: 75% !important;
}
</style>
