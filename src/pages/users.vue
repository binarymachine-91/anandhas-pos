<script>
import axios  from "axios"
import DataTable from 'primevue/datatable'
import Column from 'primevue/column'
import InputText from "primevue/inputtext"
import  Toast  from 'primevue/toast'
import ContextMenu from 'primevue/contextmenu'
import { FilterMatchMode } from "primevue/api"


export default {
  components: { DataTable, Column, Toast, ContextMenu, InputText },
  // eslint-disable-next-line vue/component-api-style
  data() {
    return {
      formState: {
        name: '',
        department: '',
      },
      user_table: [],
      selectedUser: null,
      editingRows: [],
      filters: {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        name: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
      },
      menuModel: [
        { label: 'Delete', icon: 'pi pi-fw pi-times', command: () => this.deleteUser(this.selectedUser) },
      ],
    }
  },
  // eslint-disable-next-line vue/component-api-style
  mounted() {
    this.get_user()
  },
  // eslint-disable-next-line vue/component-api-style
  methods: {
    onRowContextMenu(event) {
      this.$refs.cm.show(event.originalEvent)
    },
    editUser(event) {
      axios.post("https://anandhas-api-server.onrender.com/update_staff", event.newData)
        .then(response => {
          this.get_user()
          this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
          this.formState.name = ''
          this.formState.department = ''
        })
    },
    deleteUser(menu) {
      axios.delete(`https://anandhas-api-server.onrender.com/delete_staff/${menu.sid}`)
        .then(response => {
          this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
          this.get_user() // Refresh the table after deletion
          this.formState.name = ''
          this.formState.department = ''
        })

    },
    onFinish(){
      this.create_user()
    },
    onFinishFailed(){
      console.log("user create failed")
    },
    get_user() {
      try {
        const response =  axios.get("https://anandhas-api-server.onrender.com/get_staff/-1")
          .then(response => {
            this.user_table = response.data
          })
      } catch (error) {
        console.error(error)
      }
    },
    create_user() {
      axios.post("https://anandhas-api-server.onrender.com/create_staff", { "name": this.formState.name, "department": this.formState.department })
        .then(response => {
          this.get_user()
          this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
          this.formState.name = ''
          this.formState.department = ''
        })
    },
  },
}
</script>

<template>
  <a-row>
    <Toast />
    <!-- create Menu ! -->
    <a-col :span="8">
      <a-card title="Create New User" :bordered="false" >
        <a-form
          :model="formState"
          name="basic"
          :label-col="{ span: 8 }"
          :wrapper-col="{ span: 16 }"
          autocomplete="off"
          @finish="onFinish"
          @finishFailed="onFinishFailed"
        >
          <a-form-item
            label="Name"
            name="name"
            :rules="[{ required: true, message: 'Please input your name!' }]"
          >
            <a-input style="width:250px" v-model:value="formState.name" />
          </a-form-item>

          <a-form-item
            label="Department"
            name="department"
            :rules="[{ required: true, message: 'Please select your department!' }]"
          >
            <a-select default-value="admin" v-model:value="this.formState.department" style="width: 150px">
              <a-select-option value="kitchen">
                Kitchen
              </a-select-option>
              <a-select-option value="admin">
                Admin
              </a-select-option>
            </a-select>
          </a-form-item>

          <a-form-item :wrapper-col="{ offset: 8, span: 16 }">
            <a-button type="primary" style="color:white" html-type="submit">Create</a-button>
          </a-form-item>
        </a-form>
      </a-card>
    </a-col>
    <!-- Display Menu ! -->
    <a-col :span="14" style="margin-left: 10px">
      <a-card title="Available Menu " :bordered="false" >
        <ContextMenu ref="cm" :model="menuModel" />
        <DataTable
          v-model:editingRows="editingRows"
          v-model:filters="filters"
          :value="this.user_table"
          showGridlines
          class="p-datatable-sm"
          contextMenu v-model:contextMenuSelection="selectedUser"
          @rowContextmenu="onRowContextMenu"
          editMode="row"
          dataKey="sid"
          paginator :rows="10"
          filterDisplay="row"
          :globalFilterFields="['sname']"
          @row-edit-save="editUser"
          tableClass="editable-cells-table"
          tableStyle="min-width: 50rem">
          <template #header>
            <div class="flex justify-content-end">
                    <span class="p-input-icon-left">
                        <i class="pi pi-search" />
                        <InputText v-model="filters['global'].value" placeholder="User Search" />
                    </span>
            </div>
          </template>
          <template #empty> No User found. </template>
          <Column field="sid" header="ID"></Column>
          <Column field="sname" header="Name">
            <template #editor="{ data, field }">
              <InputText v-model="data[field]" />
            </template>
          </Column>
          <Column field="department" header="Department"></Column>
          <Column :rowEditor="true" style=" min-width: 8rem" bodyStyle="text-align:center"></Column>
        </DataTable>
      </a-card>
    </a-col>
  </a-row>



</template>

<style lang="scss" scoped>
::v-deep(.editable-cells-table td.p-cell-editing) {
  padding-top: 0.6rem;
  padding-bottom: 0.6rem;
}
</style>
