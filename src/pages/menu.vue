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
        menu: '',
      },
      menu_table: [],
      selectedProduct: null,
      editingRows: [],
      filters: {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        name: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
      },
      menuModel: [
        { label: 'Delete', icon: 'pi pi-fw pi-times', command: () => this.deleteMenu(this.selectedProduct) },
      ],
    }
  },
  // eslint-disable-next-line vue/component-api-style
  mounted() {
    this.get_menu()
  },
  // eslint-disable-next-line vue/component-api-style
  methods: {
    onRowContextMenu(event) {
      this.$refs.cm.show(event.originalEvent)
    },
    onRowEditSave(event) {
      this.update_menu({
        'mid': event.newData.mid,
        'name': event.newData.name,
      })
    },
    deleteMenu(menu) {
      axios.delete("http://localhost:4242/delete_menu/" + menu.mid)
        .then(response => {
          this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
          this.get_menu()
        })

    },
    onFinish(){
      this.create_menu()
    },
    onFinishFailed(){
      console.log("failed")
    },
    get_menu() {
      try {
        const response = axios.get("http://localhost:4242/get_menu/-1")
          .then(response => {
            if(Array.isArray(response.data)) {
              this.menu_table = response.data
            }
            else {
              this.$toast.add({ severity: 'error', summary: 'Error', detail: 'Error on Server!', life: 3000 })
            }

          })
      } catch (error) {
        console.error(error)
      }
    },
    create_menu() {
      axios.post("http://localhost:4242/create_menu", { "name": this.formState.menu })
        .then(response => {
          this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
          this.get_menu()
          this.formState.menu = ''
        })
    },
    update_menu(data) {
      axios.post("http://localhost:4242/update_menu", data)
        .then(response => {
          this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
          this.get_menu()
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
      <a-card title="Create New Menu" :bordered="false" >
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
            label="Menu"
            name="menu"
            :rules="[{ required: true, message: 'Please input your menu name!' }]"
          >
            <a-input v-model:value="formState.menu" />
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
          :value="this.menu_table"
          showGridlines
          class="p-datatable-sm"
          contextMenu v-model:contextMenuSelection="selectedProduct"
          @rowContextmenu="onRowContextMenu"
          editMode="row"
          dataKey="mid"
          paginator :rows="10"
          filterDisplay="row"
          :globalFilterFields="['name']"
          @row-edit-save="onRowEditSave"
          tableClass="editable-cells-table"
          tableStyle="min-width: 50rem">
          <template #header>
            <div class="flex justify-content-end">
                    <span class="p-input-icon-left">
                        <i class="pi pi-search" />
                        <InputText v-model="filters['global'].value" placeholder="Menu Search" />
                    </span>
            </div>
          </template>
          <template #empty> No menu found. </template>
          <Column field="mid" header="ID"></Column>
          <Column field="name" header="Name">
            <template #editor="{ data, field }">
              <InputText v-model="data[field]" />
            </template>
          </Column>
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
