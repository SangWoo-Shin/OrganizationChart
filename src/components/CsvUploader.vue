<template>
    <div class="p-4 bg-gradient-to-b from-blue-100 to-white min-h-screen">
      <h1 class="text-4xl font-bold text-blue-700 mb-4">Organization Chart</h1>
      <h2 class="text-l font-bold text-gray-600 mb-2">Uplaod CSV File:</h2>
      <div class="flex w-full">
        <label for="dropzone-file" class="flex flex-col items-center justify-center mb-10 w-120 h-50 border-2 border-dashed rounded-lg cursor-pointer bg-white  hover:bg-gray-100 dark:border-gray-600">
            <div class="flex flex-col items-center justify-center pt-5 pb-6">
                <svg class="w-8 h-8 mb-4 text-gray-500 dark:text-gray-400" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 20 16">
                    <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 13h3a3 3 0 0 0 0-6h-.025A5.56 5.56 0 0 0 16 6.5 5.5 5.5 0 0 0 5.207 5.021C5.137 5.017 5.071 5 5 5a4 4 0 0 0 0 8h2.167M10 15V6m0 0L8 8m2-2 2 2"/>
                </svg>
                <p class="mb-2 text-sm text-gray-500 dark:text-gray-400"><span class="font-semibold">Click to upload</span> or drag and drop</p>
                <p class="text-xs text-gray-500 dark:text-gray-400">EXCL or CSV</p>
            </div>
            <input id="dropzone-file" type="file" @change="handleFile" accept=".csv" class="hidden" />
        </label>
      </div> 

      <div v-if="hierarchy.length > 0">
        <h3 class="font-semibold text-gray-600 mb-2">Organization Chart (Tree Structure):</h3>
        <div class="p-2 rounded overflow-x-auto min-w-full text-sm">
            <div class="min-w-max">
              <TreeNode v-for="employee in hierarchy" :key="employee.employee_id" :employee="employee" />
            </div>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import Papa from 'papaparse'
  import TreeNode from './TreeNode.vue'
  
  export default {
    name: 'CsvUploader',
    components: { TreeNode },
    data() {
      return {
        employees: [],
        hierarchy: []
      }
    },
    computed: {
      formattedPreview() {
        return JSON.stringify(this.employees.slice(0,5), null, 2)
      },
      formattedHierarchy() {
        return JSON.stringify(this.hierarchy, null, 2)
      }
    },
    methods: {
      //Tree Generating Function
      createHierarchy() {
        const map = new Map()

        this.employees.forEach(employee => {
          map.set(employee.employee_id, {...employee, subordinates: []})
        })

        this.employees.forEach(employee => {
          if(employee.manager) {
            const manager = map.get(employee.manager)
            if(manager) {
              manager.subordinates.push(map.get(employee.employee_id))
            }
          }
        })

        this.hierarchy = this.employees.filter(employee => !employee.manager).map(employee => map.get(employee.employee_id))
        console.log('Hierarchy:', this.hierarchy)
      },
      handleFile(event) {
        const file = event.target.files[0]
        if (!file) return
  
        Papa.parse(file, {
          header: true,
          skipEmptyLines: true,
          complete: (results) => {
            this.employees = results.data.map(row => ({
            employee_id: row['Employee Id'],
            name: row['Name'],
            job_title: row['Job Title'],
            email: row['Email'],
            manager: row['Manager'] || null,
            salary: Number(row['Salary']) || 0,
            location: row['Location'],
            department: row['Department'],
            level: Number(row['level']) || 0
          }))
          console.log('Parsed employees:', this.employees)

          this.createHierarchy()
          }
        })
      }
    },
  }
  </script>