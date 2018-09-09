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
                <td ><v-checkbox @change="checkboxChaged(props.item.no)"></v-checkbox></td>
                <td>{{ props.item.no }}</td>
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
                    <v-form>
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
                        @click="dialog = false"
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

/* eslint-disable */
var jsPDF = require('jspdf');
require('jspdf-autotable');

export default {
  data: () => ({
    dialog:false,
    drawer: null,
    headers: [
      { text: '', align: 'left', value: 'check', sortable:false},
      { text: 'No', align: 'left', value: 'no'},
      { text: 'Vin#', value: 'vin' },
      { text: 'Model', value: 'model' },
      { text: 'Make', value: 'make' },
      { text: 'Year', value: 'year' },
      { text: 'MSRP', value: 'msrp' },
      { text: 'Status', value: 'status' },
      { text: 'Booked', value: 'booked' },
      { text: 'Listed', value: 'listed' },
    ],
    inventories: [
      {
        no: 1,
        vin: 'Frozen Yogurt',
        model: 159,
        make: 6.0,
        year: 24,
        msrp: 4.0,
        status: '1%',
        booked: '1%',
        listed: '1%'
      }
    ],
    checkbox:[

    ],
    no: 1,
    vin: 'Frozen Yogurt',
    model: 159,
    make: 6.0,
    year: 24,
    msrp: 4.0,
    status: '1%',
    booked: '1%',
    listed: '1%'

  }),
  props: {
    source: String
  },
  mounted () {
    axios({
        method: 'get',
        url: 'http://127.0.0.1:8081/inventory',
        config: { headers: {'Content-Type': 'multipart/form-data' }}
      }).then(response => {
        this.inventories = response.data.Data
      })
  },
  methods: {
    checkboxChaged (no) {

    },
    submit () {
      var bodyFormData = new FormData();
      bodyFormData.set('vin', this.vin);
      bodyFormData.set('model', this.model);
      bodyFormData.set('make', this.make);
      bodyFormData.set('year', this.year);
      bodyFormData.set('msrp', this.msrp);
      bodyFormData.set('status', this.status);
      bodyFormData.set('booked', this.booked);
      bodyFormData.set('listed', this.listed);

      axios({
        method: 'post',
        url: 'http://127.0.0.1:8081/inventory/add',
        data: bodyFormData,
        config: { headers: {'Content-Type': 'multipart/form-data' }}
      }).then(response => {console.log(response)})

      this.$refs.form.reset();
      this.dialog = false;
    },
    deleteInventory () {

    },
    movePage (dest) {
      if(dest == 'inventory') this.$router.replace(dest);
    },
    createPdf () {
      var columns = [
        {title: 'ID', dataKey: 'id'},
        {title: 'Name', dataKey: 'name'},
        {title: 'Country', dataKey: 'country'}
      ]
      var rows = [
        {'id': 1, 'name': 'Shaw', 'country': 'Tanzania'},
        {'id': 2, 'name': 'Nelson', 'country': 'Kazakhstan'},
        {'id': 3, 'name': 'Garcia', 'country': 'Madagascar'}
      ]
      var doc = new jsPDF('p', 'pt')
      doc.autoTable(columns, rows, {
        styles: {fillColor: [100, 255, 255]},
        columnStyles: {
          id: {fillColor: 255}
        },
        margin: {top: 60},
        addPageContent: function (data) {
          doc.text('Header', 40, 30)
        }
      })
      var string = doc.output('datauristring');
      var iframe = "<iframe width='100%' height='100%' src='" + string + "'></iframe>"
      var x = window.open();
      x.document.open();
      x.document.write(iframe);
      x.document.close();
    }
  }
}
</script>
