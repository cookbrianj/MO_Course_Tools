<template>
  <v-app>
    <v-app-bar color="primary" elevation="2">
      <v-app-bar-title class="text-h5 font-weight-medium">MOSIS Data Reconciliation</v-app-bar-title>
    </v-app-bar>

    <v-main class="bg-grey-lighten-4">
      <v-container fluid class="px-6 py-6">
        <!-- Upload Section -->
        <v-row class="mb-6">
          <v-col cols="12" md="3">
            <v-card class="elevation-1 rounded-lg pa-4">
              <h3 class="text-subtitle-1 font-weight-bold mb-2" title="October Course Assignment">1. Oct Course Assign</h3>
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
          <v-col cols="12" md="3">
            <v-card class="elevation-1 rounded-lg pa-4">
              <h3 class="text-subtitle-1 font-weight-bold mb-2" title="October Student Assignment">2. Oct Student Assign</h3>
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
          <v-col cols="12" md="3">
            <v-card class="elevation-1 rounded-lg pa-4">
              <h3 class="text-subtitle-1 font-weight-bold mb-2" title="June Student Course Completion">3. June Completion</h3>
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
          <v-col cols="12" md="3">
            <v-card class="elevation-1 rounded-lg pa-4">
              <h3 class="text-subtitle-1 font-weight-bold mb-2" title="October Student Core">4. Oct Student Core</h3>
              <v-file-input
                v-model="files.octStudentCore"
                accept=".csv,.txt,.tsv"
                label="Upload File"
                variant="outlined"
                density="compact"
                hide-details
                prepend-icon="mdi-account-details-outline"
                @update:modelValue="handleFileUpload($event, 'octStudentCore')"
              ></v-file-input>
            </v-card>
          </v-col>
        </v-row>

        <!-- Dashboard Tabs -->
        <div class="d-flex justify-space-between align-center mb-4">
          <h2 class="text-h6 font-weight-medium text-grey-darken-3">Reconciliation Dashboard</h2>
          <div class="d-flex align-center">
            <v-select
              v-model="selectedGrades"
              :items="availableGrades"
              label="Filter by Grade (empty = all)"
              multiple
              chips
              closable-chips
              variant="outlined"
              density="compact"
              hide-details
              style="min-width: 300px; max-width: 500px;"
            ></v-select>
            <v-btn color="primary" prepend-icon="mdi-refresh" @click="revalidate" elevation="2" class="ml-4">
              Revalidate
            </v-btn>
          </div>
        </div>
        <v-card class="elevation-1 rounded-lg">
          <v-tabs v-model="activeTab" color="primary" bg-color="white">
            <v-tab value="missingAssignment">
              <v-icon start>mdi-alert-circle-outline</v-icon>
              Missing from Assignment
              <v-badge v-if="filteredMissingAssignmentCount" :content="filteredMissingAssignmentCount" color="error" inline></v-badge>
            </v-tab>
            <v-tab value="missingCompletion">
              <v-icon start>mdi-alert-circle-outline</v-icon>
              Missing from Completion
              <v-badge v-if="filteredMissingCompletionCount" :content="filteredMissingCompletionCount" color="warning" inline></v-badge>
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
                  :externalFilter="row => isRowVisible(row, 'juneCompletion')"
                  @add-row="datasets.juneCompletion.unshift($event)"
                  show-actions
                >
                  <template v-slot:bulk-actions>
                    <v-btn color="info" variant="elevated" size="small" prepend-icon="mdi-auto-fix" @click="reconstructAll" class="mr-4">
                      Reconstruct All Possible
                    </v-btn>
                  </template>
                  <template v-slot:item-actions="{ item }">
                    <v-btn icon="mdi-auto-fix" color="info" variant="text" size="small" @click="reconstructSingle(item)" title="Auto-Reconstruct"></v-btn>
                  </template>
                </DataTable>
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
                  :externalFilter="row => isRowVisible(row, 'octStudent')"
                  @add-row="datasets.octStudent.unshift($event)"
                />
              </v-window-item>

              <v-window-item value="octCourse">
                <DataTable 
                  title="October Course Assignment" 
                  :items="datasets.octCourse" 
                  :externalFilter="row => isRowVisible(row, 'octCourse')"
                  @update:items="datasets.octCourse = $event"
                  @add-row="datasets.octCourse.unshift($event)"
                  filename="OctoberCourse_Edited.csv"
                />
              </v-window-item>

              <v-window-item value="octStudent">
                <DataTable 
                  title="October Student Assignment" 
                  :items="datasets.octStudent" 
                  :externalFilter="row => isRowVisible(row, 'octStudent')"
                  @update:items="datasets.octStudent = $event"
                  @add-row="datasets.octStudent.unshift($event)"
                  filename="OctoberStudent_Edited.csv"
                />
              </v-window-item>

              <v-window-item value="juneCompletion">
                <DataTable 
                  title="June Student Course Completion" 
                  :items="datasets.juneCompletion" 
                  :externalFilter="row => isRowVisible(row, 'juneCompletion')"
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

    <v-snackbar v-model="snackbar.show" :color="snackbar.color" :timeout="4000">
      {{ snackbar.text }}
      <template v-slot:actions>
        <v-btn color="white" variant="text" @click="snackbar.show = false">Close</v-btn>
      </template>
    </v-snackbar>
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
  juneCompletion: null,
  octStudentCore: null
})

const datasets = reactive({
  octCourse: [],
  octStudent: [],
  juneCompletion: [],
  octStudentCore: []
})

const datasetGradeKeys = reactive({
  octCourse: null,
  octStudent: null,
  juneCompletion: null,
  octStudentCore: null
})

const selectedGrades = ref([])

const availableGrades = computed(() => {
  const grades = new Set()
  const extractGrades = (datasetName) => {
    const gradeKey = datasetGradeKeys[datasetName]
    if (!gradeKey) return
    datasets[datasetName].forEach(row => {
      const val = row[gradeKey]
      if (val) grades.add(val)
    })
  }
  extractGrades('octCourse')
  extractGrades('octStudent')
  extractGrades('juneCompletion')
  extractGrades('octStudentCore')
  return Array.from(grades).sort()
})

const isRowVisible = (row, datasetName) => {
  if (selectedGrades.value.length === 0) return true
  
  let gradeKey = datasetGradeKeys[datasetName]
  if (!gradeKey || row[gradeKey] === undefined) {
    for (const name of ['octCourse', 'octStudent', 'juneCompletion', 'octStudentCore']) {
      if (datasetGradeKeys[name] && row[datasetGradeKeys[name]] !== undefined) {
        gradeKey = datasetGradeKeys[name]
        break
      }
    }
  }

  if (gradeKey && row[gradeKey] !== undefined) {
    return selectedGrades.value.includes(row[gradeKey])
  }
  
  return true
}

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
          datasetGradeKeys[datasetName] = gradeKey
          normalizedData.forEach(row => {
            const val = String(row[gradeKey] || '').trim()
            const num = parseInt(val, 10)
            if (!isNaN(num)) {
              row[gradeKey] = String(num).padStart(2, '0')
            } else {
              row[gradeKey] = val
            }
          })
        } else {
          datasetGradeKeys[datasetName] = null
        }
      }

      datasets[datasetName] = normalizedData
      revalidate()
    }
  })
}

const missingFromAssignment = ref([])
const missingFromCompletion = ref([])

const snackbar = reactive({
  show: false,
  text: '',
  color: 'success'
})

const showMessage = (text, color = 'success') => {
  snackbar.text = text
  snackbar.color = color
  snackbar.show = true
}

const findDemographics = (stateId) => {
  if (!stateId || !datasets.octStudentCore.length) return null
  const coreData = datasets.octStudentCore
  
  const coreKeys = Object.keys(coreData[0] || {})
  const coreStateIdKey = coreKeys.find(k => k.toLowerCase().replace(/[^a-z0-9]/g, '') === 'stateid' || k.toLowerCase().replace(/[^a-z0-9]/g, '') === 'mosisid')
  
  if (!coreStateIdKey) return null
  
  const match = coreData.find(row => String(row[coreStateIdKey]).trim() === String(stateId).trim())
  if (!match) return null
  
  return match
}

const reconstructRow = (juneRow, silent = false) => {
  if (!datasets.octStudent.length) return false
  
  const juneKeys = Object.keys(juneRow)
  const stateIdKey = juneKeys.find(k => k.toLowerCase().replace(/[^a-z0-9]/g, '') === 'stateid' || k.toLowerCase().replace(/[^a-z0-9]/g, '') === 'mosisid')
  const stateId = stateIdKey ? juneRow[stateIdKey] : null
  
  if (!stateId) {
    if (!silent) showMessage('Could not find StateID in the June record.', 'error')
    return false
  }

  const coreMatch = findDemographics(stateId)
  if (!coreMatch) {
    if (!silent) showMessage(`StateID ${stateId} not found in October Student Core File.`, 'error')
    return false
  }
  
  const targetKeys = Object.keys(datasets.octStudent[0] || {})
  if (targetKeys.length === 0) {
    if (!silent) showMessage('October Student file has no columns to map to.', 'error')
    return false
  }
  
  const newRow = { _reconstructed: true }
  
  const findKey = (keys, searchTerms) => {
    return keys.find(k => {
      const cleanK = k.toLowerCase().replace(/[^a-z0-9]/g, '')
      return searchTerms.some(term => cleanK.includes(term))
    })
  }

  const targetGenderKey = findKey(targetKeys, ['gender'])
  const targetRaceKey = findKey(targetKeys, ['race', 'ethnicity'])
  const targetIepKey = findKey(targetKeys, ['iep', 'disability'])
  
  const coreGenderKey = findKey(Object.keys(coreMatch), ['gender'])
  const coreRaceKey = findKey(Object.keys(coreMatch), ['race', 'ethnicity'])
  const coreIepKey = findKey(Object.keys(coreMatch), ['iep', 'disability'])

  const targetDobKey = findKey(targetKeys, ['dob', 'dateofbirth', 'birthdate'])
  const juneDobKey = findKey(juneKeys, ['dob', 'dateofbirth', 'birthdate'])

  const targetGradeKey = datasetGradeKeys.octStudent || findKey(targetKeys, ['grade'])
  const juneGradeKey = datasetGradeKeys.juneCompletion || findKey(juneKeys, ['grade'])
  
  targetKeys.forEach(tKey => {
    const cleanTKey = tKey.toLowerCase().replace(/[^a-z0-9]/g, '')
    const isStudentKey = studentKeys.some(sk => sk.toLowerCase().replace(/[^a-z0-9]/g, '') === cleanTKey)

    if (isStudentKey) {
      const jKey = findKey(juneKeys, [cleanTKey])
      newRow[tKey] = jKey ? juneRow[jKey] : ''
    }
    else if (tKey === targetDobKey && juneDobKey) {
      newRow[tKey] = juneRow[juneDobKey]
    }
    else if (tKey === targetGradeKey && juneGradeKey) {
      newRow[tKey] = juneRow[juneGradeKey]
    }
    else if (tKey === targetGenderKey && coreGenderKey) {
      newRow[tKey] = coreMatch[coreGenderKey]
    }
    else if (tKey === targetRaceKey && coreRaceKey) {
      newRow[tKey] = coreMatch[coreRaceKey]
    }
    else if (tKey === targetIepKey && coreIepKey) {
      newRow[tKey] = coreMatch[coreIepKey]
    }
    else {
      newRow[tKey] = ''
    }
  })

  datasets.octStudent.unshift(newRow)
  return true
}

const reconstructSingle = (row) => {
  if (!datasets.octStudentCore.length) {
    showMessage('Please upload the October Student Core file first.', 'warning')
    return
  }
  const success = reconstructRow(row)
  if (success) {
    revalidate()
    showMessage('Record successfully reconstructed.', 'success')
  }
}

const reconstructAll = () => {
  if (!datasets.octStudentCore.length) {
    showMessage('Please upload the October Student Core file first.', 'warning')
    return
  }
  
  let successCount = 0
  let failCount = 0
  
  const visibleMissing = missingFromAssignment.value.filter(row => isRowVisible(row, 'juneCompletion'))
  
  visibleMissing.forEach(row => {
    if (reconstructRow(row, true)) {
      successCount++
    } else {
      failCount++
    }
  })
  
  if (successCount > 0) {
    revalidate()
    showMessage(`Successfully reconstructed ${successCount} record(s). ${failCount} failed.`, 'success')
  } else {
    showMessage('Could not reconstruct any records. Make sure the Student Core file contains matching State IDs.', 'error')
  }
}

const filteredMissingAssignmentCount = computed(() => 
  missingFromAssignment.value.filter(row => isRowVisible(row, 'juneCompletion')).length
)

const filteredMissingCompletionCount = computed(() => 
  missingFromCompletion.value.filter(row => isRowVisible(row, 'octStudent')).length
)

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
