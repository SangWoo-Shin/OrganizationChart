<template>
    <div class="tree-node">
      <div :class="['flex flex-col shadow-xl border-3 bg-gray-100 rounded-lg w-72 h-116 mx-auto', departmentColor]" @click="toggle">
        <div class="m-1.0 overflow-hidden rounded-md h-30 flex justify-center items-center">
            <img v-if="employee.photo" class="w-20 h-20 rounded-full" :src="employee.photo" alt="User Photo" />
            <img v-else class="w-20 h-20 rounded-full" src="@/assets/user.png" alt="Default User Photo" />
        </div>
        <div class="p-6 text-center">
            <h4 class="mb-1 text-xl font-semibold text-slate-800">
                {{ employee.name }}
            </h4>
            <p
            class="text-sm font-semibold text-slate-500 uppercase">
                {{ employee.job_title }}
            </p>
            <p class="text-sm text-slate-600 mt-4 font-light ">
                {{ employee.department }}
            </p>
            <p class="text-sm text-slate-600 font-light ">
                {{ employee.location }}
            </p>
            <p class="text-sm text-slate-600 font-light ">
                Layer: {{ employee.level }}
            </p>
            <p class="text-sm text-slate-600 font-light">
                Manager Cost: ${{ formatCost(managementCost) }} / year
            </p>
            <p class="text-sm text-slate-600 font-light">
                IC Cost: ${{ formatCost(icCost) }} / year
            </p>
            <p class="text-sm text-slate-600 font-light">
                Total Cost: ${{ formatCost(totalCost) }} / year
            </p>
            <p class="text-sm text-slate-600 font-light">
                Manager Cost Ratio: {{ managementCostRatio }}
            </p>
        </div>
        <div class="flex justify-center p-6 pt-2 gap-7">
            <span v-if="employee.subordinates.length > 0">
                <span class="min-w-32 rounded-md bg-slate-800 py-2 px-4 border border-transparent text-center text-sm text-white transition-all shadow-md hover:shadow-lg focus:bg-slate-700 focus:shadow-none active:bg-slate-700 hover:bg-slate-700 active:shadow-none disabled:pointer-events-none disabled:opacity-50 disabled:shadow-none">{{ expanded ? '▼' : '▶' }}&nbsp;
                {{ employee.subordinates.length }} / {{ totalSubordinates }}
                </span>
            </span>
      </div>
    </div>
  
      <div v-if="expanded && employee.subordinates.length > 1" class="horizontal-subordinates">
        <div
          v-for="subordinate in employee.subordinates"
          :key="subordinate.employee_id"
          class="tree-node-wrapper"
        >
            <TreeNode :employee="subordinate" class="tree-node-2"/>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'TreeNode',
    props: {
      employee: Object,
    },
    data() {
      return {
        expanded: false,
        descendantMemo: {}
      }
    },
    computed: {
        departmentColor() {
            const departmentBorders = {
                'Customer Support and Success': 'border-sky-300',
                'Business Intelligence': 'border-green-300',
                'Network Operations': 'border-red-300',
                'Finance and Accounting': 'border-yellow-300',
                'Software Development': 'border-purple-300',
                'User Experience (UX) Design': 'border-purple-200',
                'Sales and Business Development': 'border-blue-200',
                'Cloud Computing': 'border-purple-100',
            };
            return departmentBorders[this.employee.department] || 'border-gray-500';
        },
        totalSubordinates() {
            if (!this.descendantMemo[this.employee.employee_id]) {
                this.descendantMemo[this.employee.employee_id] = {}; // Initialize if not exists
            }
            if (this.descendantMemo[this.employee.employee_id].totalSubordinates !== undefined) {
                return this.descendantMemo[this.employee.employee_id].totalSubordinates;
            }

            const countAll = (emp) => {
                if (!emp.subordinates || emp.subordinates.length === 0) return 0;
                return emp.subordinates.length + emp.subordinates.reduce((sum, sub) => sum + countAll(sub), 0);
            };

            const result = countAll(this.employee);
            this.descendantMemo[this.employee.employee_id].totalSubordinates = result;  // Memoize the result
            return result;
        },

        managementCost() {
            if (!this.descendantMemo[this.employee.employee_id]) {
                this.descendantMemo[this.employee.employee_id] = {}; // Initialize if not exists
            }
            if (this.descendantMemo[this.employee.employee_id].managementCost !== undefined) {
                return this.descendantMemo[this.employee.employee_id].managementCost;
            }

            let total = 0;
            const calculateManagementCost = (emp) => {
                if (emp.subordinates && emp.subordinates.length > 0) {
                total += emp.salary;
                emp.subordinates.forEach(sub => calculateManagementCost(sub));
                }
            };
            calculateManagementCost(this.employee);
            this.descendantMemo[this.employee.employee_id].managementCost = total;
            return total;
        },

        icCost() {
            if (!this.descendantMemo[this.employee.employee_id]) {
                this.descendantMemo[this.employee.employee_id] = {}; // Initialize if not exists
            }
            if (this.descendantMemo[this.employee.employee_id].icCost !== undefined) {
                return this.descendantMemo[this.employee.employee_id].icCost;
            }
            let total = 0;
            const calculateICCost = (emp) => {
                if (!emp.subordinates || emp.subordinates.length === 0) {
                    total += emp.salary;
                }
                    emp.subordinates?.forEach(sub => calculateICCost(sub));
                };
                calculateICCost(this.employee);
                this.descendantMemo[this.employee.employee_id].icCost = total;
                return total;
        },

        totalCost() {
            if (!this.descendantMemo[this.employee.employee_id]) {
                this.descendantMemo[this.employee.employee_id] = {}; // Initialize if not exists
            }
            if (this.descendantMemo[this.employee.employee_id].totalCost !== undefined) {
                return this.descendantMemo[this.employee.employee_id].totalCost;
            }

            let total = this.employee.salary;
            const calculateTotalCost = (emp) => {
                emp.subordinates?.forEach(sub => {
                total += sub.salary;
                calculateTotalCost(sub);
                });
            };
            calculateTotalCost(this.employee);
            this.descendantMemo[this.employee.employee_id].totalCost = total;
            return total;
        },

        managementCostRatio() {
            const mc = this.managementCost;
            const ic = this.icCost;
            return ic > 0 ? (mc / ic).toFixed(2) : '0';
        },
    },
    methods: {
        toggle() {
        this.expanded = !this.expanded
        },
        formatCost(cost) {
            if(cost / 1000000 >= 1) {
                return (cost / 1000000).toFixed(1) + 'M'; 
            } else {
                return (cost / 1000).toFixed(1) + 'K';
            }
            
        },
    }
}
  </script>
  