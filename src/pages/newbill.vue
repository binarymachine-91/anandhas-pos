<script>
import axios  from "axios"
import DataTable from 'primevue/datatable'
import Column from 'primevue/column'
import InputText from "primevue/inputtext"
import  Toast  from 'primevue/toast'
import ContextMenu from 'primevue/contextmenu'
import { FilterMatchMode } from "primevue/api"
import _ from "lodash"
import moment from "moment"
import locale from 'ant-design-vue/es/date-picker/locale/en_US'

export default {
  components: { DataTable, Column, Toast, ContextMenu, InputText },
  // eslint-disable-next-line vue/component-api-style
  data() {
    return {
      locale,
      user_data: [],
      menu_data: [],
      date_reset: {
        o_date: true,
        d_date: true,
      },
      selectedMenu: null,
      formState: {
        name: '',
        mobile: '',
        address: '',
        bill_no: '',
        billing_staff: '',
        delivery_staff: '',
        order_date: '',
        delivery_date: '',
      },
      formStateItem: {
        menu: '',
        qty: 1,
      },
      menu_table: [],
      editingRows: [],
      filters: {
        global: { value: null, matchMode: FilterMatchMode.CONTAINS },
        name: { value: null, matchMode: FilterMatchMode.STARTS_WITH },
      },
      menuModel: [
        { label: 'Delete', icon: 'pi pi-fw pi-times', command: () => this.deleteMenu(this.selectedMenu) },
      ],
    }
  },
  // eslint-disable-next-line vue/component-api-style
  mounted() {
    this.get_user()
    this.get_menu()
    this.get_bill_no()
  },
  // eslint-disable-next-line vue/component-api-style
  methods: {
    get_user() {
      try {
        axios.get("https://anandhas-api-server.onrender.com/get_staff/-1")
          .then(response => {
            this.user_data = response.data
          })
      } catch (error) {
        console.error(error)
      }
    },
    get_bill_no() {
      try {
        axios.get("https://anandhas-api-server.onrender.com/get_bill_no")
          .then(response => {
            this.formState.bill_no = response.data
          })
      } catch (error) {
        console.error(error)
      }
    },
    onRowContextMenu(event) {
      this.$refs.cm.show(event.originalEvent)
    },
    deleteMenu(menu) {
      var fil = _.filter(this.menu_table, o => {
        return o.sid != menu.sid
      })

      fil = fil.map((item, index) => ({ ...item, sid: index + 1 }))

      this.menu_table = fil
    },
    save_bill() {
      if (this.menu_table.length !== 0)
      {
        console.log(this.menu_table)
        this.formState.items = this.menu_table
        this.formState.o_date = moment(new Date(this.formState.order_date)).format('DDMMYYYY')
        this.formState.d_date = moment(new Date(this.formState.delivery_date)).format('DDMMYYYYHHmm')
        console.log(this.formState)
        axios.post("https://anandhas-api-server.onrender.com/save_bill", this.formState)
          .then(response => {
            this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
            this.formState.name = ''
            this.formState.mobile = ''
            this.formState.address = ''
            this.formState.bill_no = ''
            this.formState.billing_staff = ''
            this.formState.delivery_staff = ''
            this.formState.order_date = ''
            this.formState.delivery_date = ''
            this.formState.items = ''
            this.menu_table = []
            this.formStateItem.menu = ''
          })
      }
      else {
        this.$toast.add({ severity: 'error', summary: 'Info', detail: "Please add menu ", life: 3000 })
      }
    },
    onFinish(){
      this.save_bill()
    },
    onFinishFailed(){
      console.log("failed")
    },
    onFinishItem(){
      if (_.isEmpty(this.menu_table)) {
        this.menu_table.push({
          'sid': 1,
          'item': _.filter(this.menu_data, o =>{
            return o.mid === this.formStateItem.menu
          })[0]['name'],
          'qty': this.formStateItem.qty,
          'iid': this.formStateItem.menu
        })
        this.formStateItem.menu = ''
        this.formStateItem.qty = 1
      }
      else {
        var last_index = _.orderBy(this.menu_table, 'sid', 'desc')[0]['sid']
        this.menu_table.push({
          'sid': last_index + 1,
          'item': _.filter(this.menu_data, o =>{
            return o.mid === this.formStateItem.menu
          })[0]['name'],
          'qty': this.formStateItem.qty,
          'iid': this.formStateItem.menu,
        })
        this.formStateItem.menu = ''
        this.formStateItem.qty = 1
      }

    },
    onFinishFailedItem(){
      console.log("failed")
    },
    get_menu() {
      try {
        const response = axios.get("https://anandhas-api-server.onrender.com/get_menu/-1")
          .then(response => {
            this.menu_data = response.data
          })
      } catch (error) {
        console.error(error)
      }
    },
    create_menu() {
      axios.post("https://anandhas-api-server.onrender.com/create_menu", { "name": this.formState.menu })
        .then(response => {
          this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
          this.get_menu()
          this.menu = ''
        })
    },
    update_menu(data) {
      axios.post("https://anandhas-api-server.onrender.com/update_menu", data)
        .then(response => {
          this.$toast.add({ severity: 'success', summary: 'Info', detail: response.data, life: 3000 })
          this.get_menu()
        })
    },
  },
}
</script>

<template>
  <Toast />
  <a-row :gutter="20">
    <a-col :span="10">
      <a-card title="New Bill" :bordered="false">
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
            label="Bill No"
            name="bill_no"
          >
            <a-input v-model:value="formState.bill_no" disabled style="width: 250px"/>
          </a-form-item>

          <a-form-item
            label="Name"
            name="name"
            :rules="[{ required: true, message: 'Please enter customer name!' }]"
          >
            <a-input v-model:value="formState.name" placeholder="Customer Name" style="width: 250px"/>
          </a-form-item>
          <a-form-item
            label="Mobile"
            name="mobile"
            :rules="[{ required: true, message: 'Please enter customer mobile no!' }]"
          >
            <a-input v-model:value="formState.mobile" placeholder="Mobile" style="width: 250px"/>
          </a-form-item>
          <a-form-item
            label="Address"
            name="address"
            :rules="[{ required: true, message: 'Please enter address!' }]"
          >
            <a-textarea v-model:value="formState.address" placeholder="Address" allow-clear />
          </a-form-item>
          <a-form-item
            label="Bill By"
            name="billing_staff"
            :rules="[{ required: true, message: 'Please select billing staff!' }]"
          >
            <a-select
              show-search
              v-model:value="formState.billing_staff"
              placeholder="Billing Staff"
              style="width: 200px"
              :default-active-first-option="true"
              :show-arrow="true"
              :filter-option="true"
              :not-found-content="null"
            >
              <a-select-option v-for="d in this.user_data" :key="d.sid">
                {{ d.sname }}
              </a-select-option>
            </a-select>
          </a-form-item>
          <a-form-item
            label="Delivery By"
            name="delivery_staff"
            :rules="[{ required: true, message: 'Please select delivery address!' }]"
          >
            <a-select
              show-search
              v-model:value="formState.delivery_staff"
              placeholder="Delivery Staff"
              style="width: 200px"
              :default-active-first-option="true"
              :show-arrow="true"
              :filter-option="true"
              :not-found-content="null"
            >
              <a-select-option v-for="d in this.user_data" :key="d.sid">
                {{ d.sname }}
              </a-select-option>
            </a-select>
          </a-form-item>
          <a-form-item
            label="Order Date"
            name="order_date"
            v-if="date_reset.o_date"
            :rules="[{ required: true, message: 'Please select order date!' }]"
          >
            <a-date-picker format="DD-MM-YYYY"  :locale="locale" v-model:value="formState.order_date" placeholder="Order Date"/>
          </a-form-item>
          <a-form-item
            label="Delivery Date"
            name="delivery_date"
            v-if="date_reset.d_date"
            :rules="[{ required: true, message: 'Please select delivery date!' }]"
          >
            <a-date-picker show-time :locale="locale" format="DD-MM-YYYY HH:mm" v-model:value="formState.delivery_date" placeholder="Delivery Date"/>
          </a-form-item>
          <a-form-item :wrapper-col="{ offset: 8, span: 16 }">
            <a-button type="primary" style="color:white" html-type="submit">Save Bill</a-button>
          </a-form-item>
        </a-form>
      </a-card>
    </a-col>

    <a-col :span="14">
      <a-card title="Items" :bordered="false">
        <a-row style="margin-bottom:20px">
          <a-col :span="24">
            <a-form
              layout="inline"
              :model="formStateItem"
              @finish="onFinishItem"
              @finishFailed="onFinishFailedItem"
            >
              <a-form-item
                label="Menu Name"
                name="menu"
                :rules="[{ required: true, message: 'Please select menu!' }]"
              >
                <a-select
                  v-model:value="formStateItem.menu"
                  placeholder="select menu"
                  style="width: 200px"
                  :default-active-first-option="true"
                  :show-arrow="true"
                  :filter-option="true"
                  :not-found-content="null"
                >
                  <a-select-option v-for="d in this.menu_data" :key="d.mid">
                    {{ d.name }}
                  </a-select-option>
                </a-select>
              </a-form-item>

              <a-form-item
                label="Qty"
                name="qty"
                :rules="[{ required: true, message: 'Please select qty!' }]"
              >
                <a-input-number v-model:value="formStateItem.qty" :min=1 :max=500 placeholder="Qty">
                </a-input-number>
              </a-form-item>
              <a-form-item>
                <a-button
                  type="primary"
                  html-type="submit"
                  style="color:white"
                >
                  Add to Bill
                </a-button>
              </a-form-item>
            </a-form>
          </a-col>

        </a-row>
        <ContextMenu ref="cm" :model="menuModel" />
        <DataTable
          v-model:editingRows="editingRows"
          v-model:filters="filters"
          :value="this.menu_table"
          showGridlines
          class="p-datatable-sm"
          contextMenu v-model:contextMenuSelection="selectedMenu"
          @rowContextmenu="onRowContextMenu"
          editMode="row"
          dataKey="sid"
          paginator :rows="10"
          filterDisplay="row"
          :globalFilterFields="['item']"
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
          <template #empty> No menu found. </template>
          <Column field="sid" header="ID"></Column>
          <Column field="item" header="Menu">
            <template #editor="{ data, field }">
              <InputText v-model="data[field]" />
            </template>
          </Column>
          <Column field="qty" header="Qty">
            <template #editor="{ data, field }">
              <InputText v-model="data[field]" />
            </template>
          </Column>
        </DataTable>
      </a-card>

    </a-col>
  </a-row>
</template>

<style scoped lang="scss">

</style>
