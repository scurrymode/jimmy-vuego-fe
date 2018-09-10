<template>
  <v-app id="inspire">
    <v-navigation-drawer
      v-model="drawer"
      fixed
      app
    >
      <v-list dense>
        <img src="../assets/Vuego-logo.png">
        <v-list-tile @click="movePage('inventory')">
          <v-list-tile-content>
            <v-list-tile-title>Inventory</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
        <v-list-tile @click="movePage('commission')">
          <v-list-tile-content>
            <v-list-tile-title>Commission</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
        <v-list-tile @click="movePage('market')">
          <v-list-tile-content>
            <v-list-tile-title>Manage Market</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
        <v-list-tile @click="movePage('customer')">
          <v-list-tile-content>
            <v-list-tile-title>Manage Customer</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
        <v-list-tile @click="movePage('report')">
          <v-list-tile-content>
            <v-list-tile-title>Report Setting</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
        <v-list-tile @click="movePage('signOut')">
          <v-list-tile-content>
            <v-list-tile-title>Sign Out</v-list-tile-title>
          </v-list-tile-content>
        </v-list-tile>
      </v-list>
    </v-navigation-drawer>
    <v-toolbar color="indigo" dark fixed app>
      <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
      <v-toolbar-title>Inventory List</v-toolbar-title>
    </v-toolbar>
    <v-content>
      <v-container fluid fill-height>
        <v-layout
          justify-center
        >
          <v-flex text-xs-center>
            <input placeholder="Filter: Filter by Model, Make, YEAR, etc..." style="width:80%; border:solid 1px blue"/>
            <v-btn>Update Filter</v-btn>
            <v-data-table
            :headers="headers"
            :items="inventories"
            hide-actions
            class="elevation-1"
            >
              <template slot="items" slot-scope="props">
                <td ><v-checkbox v-model="checkbox[props.index]"></v-checkbox></td>
                <td>{{ props.index+1 }}</td>
                <td>{{ props.item.vin }}</td>
                <td class="text-xs-right">{{ props.item.model }}</td>
                <td class="text-xs-right">{{ props.item.make }}</td>
                <td class="text-xs-right">{{ props.item.year }}</td>
                <td class="text-xs-right">{{ props.item.msrp }}</td>
                <td class="text-xs-right">{{ props.item.status }}</td>
                <td class="text-xs-right">{{ props.item.booked }}</td>
                <td class="text-xs-right">{{ props.item.listed }}</td>
              </template>
            </v-data-table>
            <div class="text-xs-center">
            <template>
                <v-dialog
                  v-model="dialog"
                  width="500"
                >
                  <v-btn
                    slot="activator"
                    color="green lighten-2"
                    dark
                  >
                    +
                  </v-btn>
                  <v-card>
                    <v-form id="form">
                    <v-card-title
                      class="headline grey lighten-2"
                      primary-title
                    >
                      Add Inventory
                    </v-card-title>
                    <v-card-text>
                      <v-flex xs12 sm6 md3>
                        <v-text-field
                          label="VIN#"
                          id="vin"
                          v-model="vin"
                        ></v-text-field>
                      </v-flex>
                      <v-flex xs12 sm6 md3>
                        <v-text-field
                          label="Model"
                          id="model"
                          v-model="model"
                        ></v-text-field>
                      </v-flex>
                      <v-flex xs12 sm6 md3>
                        <v-text-field
                          label="Make"
                          id="make"
                          v-model="make"
                        ></v-text-field>
                      </v-flex>
                      <v-flex xs12 sm6 md3>
                        <v-text-field
                          label="Year"
                          id="year"
                          v-model="year"
                        ></v-text-field>
                      </v-flex>
                      <v-flex xs12 sm6 md3>
                        <v-text-field
                          label="MSRP"
                          id="msrp"
                          v-model="msrp"
                        ></v-text-field>
                      </v-flex>
                      <v-flex xs6 sm3 md1>
                        <v-text-field
                          label="Booked"
                          id="booked"
                          v-model="booked"
                        ></v-text-field>
                      </v-flex>
                      <v-flex xs6 sm3 md1>
                        <v-text-field
                          label="Listed"
                          id="listed"
                          v-model="listed"
                        ></v-text-field>
                      </v-flex>
                    </v-card-text>

                    <v-divider></v-divider>

                    <v-card-actions>
                      <v-spacer></v-spacer>
                      <v-btn
                        color="primary"
                        flat
                        @click="submit"
                      >
                        SAVE
                      </v-btn>
                      <v-btn
                        color="danger"
                        flat
                        @click="cancel"
                      >
                        CANCEL
                      </v-btn>
                    </v-card-actions>
                    </v-form>
                  </v-card>
                </v-dialog>
            </template>
            <v-btn @click="deleteInventory" color="red lighten-2" dark>-</v-btn>
            <input type="file">
            <v-btn @click="createPdf">pdf</v-btn>
            </div>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import axios from 'axios'

var JsPDF = require('jspdf')
require('jspdf-autotable')

export default {
  data: () => ({
    dialog: false,
    drawer: null,
    headers: [
      { text: '', align: 'left', value: 'check', sortable: false },
      { text: 'No', align: 'left', value: 'no', sortable: false },
      { text: 'Vin#', value: 'vin', sortable: false },
      { text: 'Model', value: 'model', sortable: false },
      { text: 'Make', value: 'make', sortable: false },
      { text: 'Year', value: 'year' },
      { text: 'MSRP', value: 'msrp' },
      { text: 'Status', value: 'status' },
      { text: 'Booked', value: 'booked' },
      { text: 'Listed', value: 'listed' }
    ],
    inventories: [],
    checkbox: [],
    vin: 'MNBUMF050FW496402',
    model: '320i',
    make: 'BMW',
    year: '2014',
    msrp: '147,000',
    status: 'ordered',
    booked: 'y',
    listed: 'n'
  }),
  mounted () {
    this.getInventory()
  },
  methods: {
    initForm () {
      this.vin = ''
      this.model = ''
      this.make = ''
      this.year = ''
      this.msrp = ''
      this.status = ''
      this.booked = ''
      this.listed = ''
    },
    getInventory () {
      axios({
        method: 'get',
        url: 'http://127.0.0.1:8081/inventory',
        config: { headers: { 'Content-Type': 'multipart/form-data' } }
      }).then(response => {
        this.inventories = response.data.Data
        // eslint-disable-next-line
        for (let i of this.inventories) {
          this.checkbox.push(false)
        }
      }).catch(error => { console.error(error) })
    },
    submit () {
      var bodyFormData = new FormData()
      bodyFormData.set('vin', this.vin)
      bodyFormData.set('model', this.model)
      bodyFormData.set('make', this.make)
      bodyFormData.set('year', this.year)
      bodyFormData.set('msrp', this.msrp)
      bodyFormData.set('status', this.status)
      bodyFormData.set('booked', this.booked)
      bodyFormData.set('listed', this.listed)
      axios({
        method: 'post',
        url: 'http://127.0.0.1:8081/inventory/add',
        data: bodyFormData,
        config: { headers: { 'Content-Type': 'multipart/form-data' } }
      }).then(response => {
        console.log(response)
        this.checkbox = []
        this.getInventory()
      }).catch(error => { console.error(error) })
      this.initForm()
      this.dialog = false
    },
    cancel () {
      this.initForm()
      this.dialog = false
    },
    deleteInventory () {
      let vins = []
      for (let [index, c] of this.checkbox.entries()) {
        if (c === true) {
          vins.push(this.inventories[index].vin)
        }
      }
      axios.post('http://127.0.0.1:8081/inventory/delete', {
        'vins': vins
      })
        .then(response => {
          console.log(response)
          this.checkbox = []
          this.getInventory()
        })
        .catch(error => { console.error(error) })
    },
    movePage (dest) {
      if (dest === 'inventory') this.$router.replace('/')
    },
    createPdf () {
      var columns = [
        { title: 'Vin#', dataKey: 'vin' },
        { title: 'Model', dataKey: 'model' },
        { title: 'Make', dataKey: 'make' },
        { title: 'Year', dataKey: 'year' },
        { title: 'MSRP', dataKey: 'msrp' },
        { title: 'Status', dataKey: 'status' },
        { title: 'Booked', dataKey: 'booked' },
        { title: 'Listed', dataKey: 'listed' }
      ]
      var rows = this.inventories
      var doc = new JsPDF('p', 'pt')
      doc.autoTable(columns, rows, {
        styles: {fillColor: [100, 255, 255]},
        columnStyles: {
          id: {fillColor: 255}
        },
        margin: {top: 60},
        addPageContent: function (data) {
          doc.text('Inventory', 40, 30)
        }
      })
      var string = doc.output('datauristring')
      var iframe = "<iframe width='100%' height='100%' src='" + string + "'></iframe>"
      var x = window.open()
      x.document.open()
      x.document.write(iframe)
      x.document.close()
    }
  }
}
</script>
