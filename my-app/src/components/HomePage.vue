<template>
  <v-container>

    <v-tabs v-model="tab">
      <v-tab key="home">Home</v-tab>
      <v-tab key="list">Persons List</v-tab>
      <v-tab key="table">Persons Table</v-tab>
    </v-tabs>

  <v-tabs-items v-model="tab">
    <v-tab-item>
      Home page
    </v-tab-item>
    <v-tab-item key="list">
      <v-card>
        <v-card-title>
          Person info
          <v-spacer></v-spacer>
          <v-btn icon @click="createFormVisible = true">
            <v-icon color="green">mdi-plus</v-icon>
          </v-btn>
          <v-btn icon @click="getPersons()">
            <v-icon>mdi-refresh</v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-progress-linear
            v-if="loading"
            indeterminate
            color="yellow darken-2"
          ></v-progress-linear>
          <v-list>
            <v-list-item
              v-for="person of persons"
              :key="person.name"
            >
            <v-list-item-avatar>
                <v-img :src="person.avatar"></v-img>
              </v-list-item-avatar>
              <v-list-item-content>
                <v-list-item-title v-text="person.name"></v-list-item-title>
              </v-list-item-content>
              <v-list-item-action>
                <v-btn icon @click="getPerson(person.id); updateFormVisible = true">
                  <v-icon color="blue">mdi-pencil</v-icon>
                </v-btn>
                <v-btn icon @click="confirmDialogVisible = true; personToDelete = person.id">
                  <v-icon color="red">mdi-close</v-icon>
                </v-btn>
              </v-list-item-action>
              
            </v-list-item>
          </v-list>
          
        </v-card-text>
      </v-card>
    </v-tab-item>
    <v-tab-item key="table">
      <v-data-table :headers="headers" :items="persons"></v-data-table>
    </v-tab-item>
  </v-tabs-items>

    <v-snackbar 
      v-model="snackbarVisible" 
      color="green"
      right
    >
      Success
      <template v-slot:action="{ attrs }">
        <v-btn

          text
          v-bind="attrs"
          @click="snackbar = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
    

    


    <v-dialog v-model="createFormVisible">
      <v-card>
        <v-card-title>
          Add new person
          <v-spacer></v-spacer>
          <v-btn icon @click="createFormVisible = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-text-field outlined v-model="newPerson"></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn @click="createPerson()" color="success">Create</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>


    <v-dialog v-model="updateFormVisible">
      <v-card>
        <v-card-title>
          Update person
          <v-spacer></v-spacer>
          <v-btn icon @click="updateFormVisible = false">
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>
        <v-card-text>
          <v-text-field outlined v-model="selectedPerson.name"></v-text-field>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn @click="updatePerson()" color="success">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

  <v-dialog
      v-model="confirmDialogVisible"
      persistent
      max-width="290"
    >
      <v-card>
        <v-card-title class="text-h5">
          Delete persons?
        </v-card-title>
        <v-card-text>
          Delete operations is ireversible
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn
            color="gray darken-1"
            text
            @click="confirmDialogVisible = false"
          >
            Cancel
          </v-btn>
          <v-btn
            color="green darken-1"
            text
            @click="removePerson(personToDelete); confirmDialogVisible = false"
          >
            Confirm
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
    
  </v-container>
</template>

<script>
  export default {
    name: 'HomePage',

    data: () => ({
      persons: [],
      newPerson: "",
      loading: false,
      createFormVisible: false,
      updateFormVisible: false,
      confirmDialogVisible: false,
      snackbarVisible: false,
      personToDelete: null,
      selectedPerson: {
        name: null
      },
      headers: [
          {
            text: 'ID',
            align: 'start',
            sortable: false,
            value: 'id',
          },
          {
            text: 'Name',
            align: 'start',
            sortable: false,
            value: 'name',
          }
        ],
        tab: "list"
    }),
    methods: {
      async getPersons(){
        this.loading = true;
        const response = await fetch(`http://localhost:3000/persons`);
        this.persons = await response.json();
        this.loading = false;
      },

      async getPerson(id){
        const response = await fetch(`http://localhost:3000/persons/${id}`);
        this.selectedPerson = await response.json();
      },

      async createPerson(){
        const employeeObject = {
          id: this.persons.length + 1,
          name: this.newPerson,
          avatar: "https://cdn.vuetifyjs.com/images/lists/4.jpg"
        };
        await fetch('http://localhost:3000/persons', {
          method: 'POST', // or 'PUT'
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(employeeObject),
        })

        this.newPerson = "";
        this.createFormVisible = false;
        this.snackbarVisible = true;
        await this.getPersons();
      },
      async updatePerson(){
        const employeeObject = {
          id: this.selectedPerson.id,
          name: this.selectedPerson.name,
          avatar: this.selectedPerson.avatar
        };
        await fetch(`http://localhost:3000/persons/${this.selectedPerson.id}`, {
          method: 'PUT', // or 'PUT'
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(employeeObject),
        })

        this.newPerson = "";
        this.updateFormVisible = false;
        this.snackbarVisible = true;
        await this.getPersons();
      },
      async removePerson(id){
        await fetch(`http://localhost:3000/persons/${id}`, {
          method: 'DELETE', // or 'PUT'
          headers: {
            'Content-Type': 'application/json',
          }
        });
        await this.getPersons();
      }
    },
    async mounted() {
      await this.getPersons();
    }
  }
</script>
