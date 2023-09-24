<script>
import axios from "axios"
import { FilterMatchMode } from "primevue/api"
import _ from "lodash"
import moment from "moment"
import locale from 'ant-design-vue/es/date-picker/locale/en_US'
import jsPDF from "jspdf"
import autoTable from 'jspdf-autotable'
// eslint-disable-next-line import/no-unresolved



export default {

  // eslint-disable-next-line vue/component-api-style
  data() {
    return {
      reportdata: [],
      locale,
      bill_table: [],
      details_table: [],
      selectedMenu: null,
      enabler: {
        reportButton: false,
      },
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
      selectedBill: null,
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
    this.get_bill()
  },
  // eslint-disable-next-line vue/component-api-style
  methods: {
    get_bill() {
      axios.get("http://127.0.0.1:4242/get_bills/-1")
        .then(response => {
          _.forEach(response.data, o=> {
            o.o_date = moment(o.o_date, 'DDMMYYYY').format("DD/MM/YYYY")
            o.d_date = moment(o.d_date, 'DDMMYYYYHHmm').format("DD/MM/YYYY HH:mm")
          })
          this.bill_table = response.data
          console.log(this.bill_table)
        })
    },
    onRowSelect(event) {
      axios.get("http://127.0.0.1:4242/get_bill_details/" + this.selectedBill.bid)
        .then(response => {
          this.details_table = response.data
          this.enabler.reportButton = true
        })
      this.get_report_data()
    },
    get_user() {
      try {
        axios.get("http://127.0.0.1:4242/get_staff/-1")
          .then(response => {
            this.user_data = response.data
          })
      } catch (error) {
        console.error(error)
      }
    },
    get_bill_no() {
      try {
        axios.get("http://127.0.0.1:4242/get_bill_no")
          .then(response => {
            this.formState.bill_no = response.data
          })
      } catch (error) {
        console.error(error)
      }
    },
    get_report_data(){
      axios.get("http://127.0.0.1:4242/get_report_data/" + this.selectedBill.bid)
        .then(response => {
          this.reportdata = response.data
          console.log(this.reportdata)
        })
    },
    onRowContextMenu(event) {
      this.$refs.cm.show(event.originalEvent)
    },
    generatePDF() {
      console.log(this.selectedBill)
      var tableData = this.reportdata
      var doc = new jsPDF({
        format: "a3",
      })
      doc.setFontSize(15);
      doc.text('HOTEL ANANDHAS', 130, 5)
      // Draw a horizontal line under the header
      doc.setLineWidth(0.4)
      doc.line(125, 7, 185, 7)

      // Define the header content
      var billID = "Bill ID: " + this.selectedBill.bid
      var orderDate = "Order Date: " + this.selectedBill.o_date
      var deliveryDate = "Delivery Date: " + this.selectedBill.d_date
      var billRef = "Bill Ref: " + this.selectedBill.bill_ref

      var billby = "Bill By: " + this.selectedBill.b_staff
      var deliveryby = "Delivery By: " + this.selectedBill.d_staff

      var name = "Customer Name: " + this.selectedBill.name
      var mobile = "Mobile: " + this.selectedBill.mobile
      var address = "Address: " + this.selectedBill.address

      var payment = "Payment: Unpaid"
      var delivery = "Delivery: Delivered"

// Set the font size and position for the header elements
      doc.setFontSize(10)
      doc.text(10, 12, name)
      doc.text(120, 12, payment)
      doc.text(200, 12, billID)

      doc.text(10, 16, mobile)
      doc.text(120, 16, delivery)
      doc.text(200, 16, billRef)

      doc.text(10, 20, address)
      doc.text(120, 20, billby)
      doc.text(200, 20, orderDate)

      // doc.text(10, 25, mobile)
      doc.text(120, 24, deliveryby)
      doc.text(200, 24, deliveryDate)



      doc.autoTable({
        startY: 30,
        head: [
          [
            {
              content: 'Summary Bill',
              colSpan: 6,
              styles: { halign: 'center', fillColor: [243, 66, 19] },
            },
          ],
        ],
        body: tableData,
        theme: 'grid',
      })
      var reportName = moment(this.selectedBill.d_date, 'DD/MM/YYYY HH:mm').format("DD-MM-YYYY") + "-" + this.selectedBill.name + "-" +this.selectedBill.bill_ref + ".pdf"
      console.log(reportName)
      doc.save(reportName)

    },
  },
}
</script>

<template>
  <Toast />
  <a-row :gutter="20">
    <a-col :span="14">
      <a-card title="Available Bills" :bordered="false">
        <DataTable
          v-model:editingRows="editingRows"
          v-model:filters="filters"
          :value="this.bill_table"
          showGridlines
          class="p-datatable-sm"
          dataKey="bid"
          v-model:selection="selectedBill"
          @rowSelect="onRowSelect"
          selectionMode="single"
          :metaKeySelection="false"
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
          <Column field="bid" header="ID"></Column>
          <Column field="name" header="name"></Column>
          <Column field="mobile" header="mobile"></Column>
          <Column field="address" header="address"></Column>
          <Column field="bill_ref" header="Ref"></Column>
          <Column field="b_staff" header="Bill Staff"></Column>
          <Column field="d_staff" header="Delivery Staff"></Column>
          <Column field="o_date" header="Order Date" sortable></Column>
          <Column field="d_date" header="Delivery Date" sortable></Column>
        </DataTable>
      </a-card>
    </a-col>
    <a-col :span="10">
      <a-card title="View Bill" :bordered="false">
        <Button style="margin-bottom:20px"  v-if="enabler.reportButton" @click="generatePDF" label="Print" />
        <TreeTable
          showGridlines
          class="p-treetable-sm"
          :value="details_table"
        >
          <Column field="id" style="width:30px" header="ID"></Column>
          <Column field="name" style="width:200px" header="Name" expander></Column>
          <Column field="qty" style="width:30px" header="Qty"></Column>
          <Column field="total" style="width:80px" header="Total"></Column>
        </TreeTable>
      </a-card>
    </a-col>
  </a-row>

  <a-row :gutter="20" style="margin-top: 60px" v-if="false">
    <a-col :span="12">
      <a-card title="View Bill" :bordered="false">
        <TreeTable
          showGridlines
          class="p-datatable-sm"
          :value="details_table"
        >
          <Column field="id" style="width:60px" header="ID" sortable></Column>
          <Column field="name" style="width:200px" header="Name" sortable expander></Column>
          <Column field="qty" style="width:60px" header="Qty" sortable></Column>
          <Column field="total" style="width:80px" header="Total" sortable></Column>
        </TreeTable>
      </a-card>
    </a-col>
    <a-col :span="12">

    </a-col>
  </a-row>
</template>

<style scoped lang="scss">

</style>
