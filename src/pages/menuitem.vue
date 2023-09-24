<script>
import axios  from "axios"
import _ from "lodash"
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
        item: '',
        qty: '',
      },
      menu_names: [],
      selectedMenu: '',
      menu_table: [],
      item_table: [],
      selectedItem: null,
      editingRows: [],
      filters: {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        item: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
      },
      menuModel: [
        { label: 'Delete', icon: 'pi pi-fw pi-times', command: () => this.deleteMenuItem(this.selectedItem) },
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
      var newArr = _.map(this.item_table, function(a) {
        return a.iid === event.newData.iid ? event.newData : a
      })

      this.item_table = newArr

      axios.post("https://anandhas-api-server.onrender.com/create_menu_item", {
        'menu_id': this.selectedMenu,
        'menu_items': this.item_table,
      })
        .then(response => {
          this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
          this.get_menu_items()
        })

    },
    deleteMenuItem(menu) {
      var fil = _.filter(this.item_table, o => {
        return o.iid != menu.iid
      })
      this.item_table = fil
      axios.post("https://anandhas-api-server.onrender.com/create_menu_item", {
        'menu_id': this.selectedMenu,
        'menu_items': this.item_table,
      })
        .then(response => {
          this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
          this.get_menu_items()
        })
    },
    changeMenu() {
      this.selectedMenu = _.filter(this.menu_names, o=>{
        return o.value == this.formState.menu
      })[0].value
      this.get_menu_items()
    },
    onFinish(){
      if (_.isEmpty(this.item_table)) {
        this.item_table.push({
          'iid': 1,
          'item': this.formState.item,
          'qty': this.formState.qty,
        })
      }
      else {
        var last_index = _.orderBy(this.item_table, 'iid', 'desc')[0]['iid']
        this.item_table.push({
          'iid': last_index + 1,
          'item': this.formState.item,
          'qty': this.formState.qty,
        })
      }
      this.create_menu_item()
    },
    onFinishFailed(){
      console.log("failed")
    },
    get_menu() {
      try {
        const response = axios.get("https://anandhas-api-server.onrender.com/get_menu/-1")
          .then(response => {
            this.menu_table = response.data
            var keys = { mid: 'value', name: 'label' }

            this.menu_names = this.menu_table.map(function(o) {
              return _.mapKeys(o, function(v, k) {
                return k in keys ? keys[k] : k
              })
            })

          })
      } catch (error) {
        console.error(error)
      }
    },
    get_menu_items() {
      try {
        axios.get("https://anandhas-api-server.onrender.com/get_menu_items/" + this.selectedMenu)
          .then(response => {
            this.item_table = response.data
          })
      } catch (error) {
        console.error(error)
      }
    },
    create_menu_item() {
      axios.post("https://anandhas-api-server.onrender.com/create_menu_item", {
        'menu_id': this.selectedMenu,
        'menu_items': this.item_table,
      })
        .then(response => {
          this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
          this.get_menu_items()
          this.formState.item = ''
          this.formState.qty = ''
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
      <a-card title="Create New Menu Item" :bordered="false" >
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
            label="Menu Name"
            name="menu"
            :rules="[{ required: true, message: 'Please select your menu name!' }]"
          >
            <a-select
              ref="select"
              v-model:value="formState.menu"
              style="width: 250px"
              :options="menu_names"
              @change="changeMenu"
            ></a-select>
          </a-form-item>
          <a-form-item
            label="Item Name"
            name="item"
            :rules="[{ required: true, message: 'Please input your item name!' }]"
          >
            <a-input v-model:value="formState.item" />
          </a-form-item>
          <a-form-item
            label="Qty"
            name="qty"
            :rules="[{ required: true, message: 'Please input your Qty!' }]"
          >
            <a-input style="width: 150px" v-model:value="formState.qty" />
          </a-form-item>

          <a-form-item :wrapper-col="{ offset: 8, span: 16 }">
            <a-button type="primary" style="color:white" html-type="submit">Add Item</a-button>
          </a-form-item>
        </a-form>
      </a-card>
    </a-col>
    <!-- Display Menu ! -->
    <a-col :span="14" style="margin-left: 10px">
      <a-card title="Menu Items" :bordered="false" >
        <ContextMenu ref="cm" :model="menuModel" />
        <DataTable
          v-model:editingRows="editingRows"
          v-model:filters="filters"
          :value="this.item_table"
          showGridlines
          class="p-datatable-sm"
          contextMenu v-model:contextMenuSelection="selectedItem"
          @rowContextmenu="onRowContextMenu"
          editMode="row"
          dataKey="iid"
          paginator :rows="10"
          filterDisplay="row"
          :globalFilterFields="['item']"
          @row-edit-save="onRowEditSave"
          tableClass="editable-cells-table"
          tableStyle="min-width: 50rem">
          <template #header>
            <div class="flex justify-content-end">
                    <span class="p-input-icon-left">
                        <i class="pi pi-search" />
                        <InputText v-model="filters['global'].value" placeholder="Item Search" />
                    </span>
            </div>
          </template>
          <template #empty> No Item found. </template>
          <Column field="iid" header="ID"></Column>
          <Column field="item" header="Item">
            <template #editor="{ data, field }">
              <InputText v-model="data[field]" />
            </template>
          </Column>
          <Column field="qty" header="Qty">
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
