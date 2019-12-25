<template>
  <v-card>
    <v-app-bar
        dark
        :class="methodColor(editedRule.method)"
        no-gutters
        elevation="2">
      <v-card-title>
        <span class="headline">Method "{{editedRule.method}}"</span>
      </v-card-title>
    </v-app-bar>

    <v-card-text class="pb-0">
      <v-container>
        <v-row>
          <v-col cols="12">
            <v-text-field class="mr-2" disabled :value="editedRule.table_name" label="Table Name"></v-text-field>            
          </v-col>
          <v-col cols="12">
            <v-row
            v-for="group of groups_list"
            :key="group"
            >
              <v-col cols class="py-0">
                <v-checkbox
                dense
                :label="group"
                v-model="groups"
                :value="group"
                :multiple="true"
                ></v-checkbox>
              </v-col>
              
              <v-col cols class="py-0">
              <v-switch 
              :label="group" 
              v-model="groups_available" 
              :value="group" 
              :multiple="true" 
              ></v-switch>      
              </v-col>

              <v-col cols class="py-0 d-flex flex-row align-center">
                <v-icon
                small           
                :class="['mdi', editedRule.locked? 'mdi-lock':'mdi-lock-open-variant']"
                :color="editedRule.locked? '':'success'"
                ></v-icon>
                <span class="subtitle-1 ml-1">{{editedRule.locked? 'Locked':'Unlocked'}}</span>
              </v-col>
              
            </v-row>
          </v-col>
          
        </v-row>
      </v-container>
    </v-card-text>

    <v-card-actions>
      <v-spacer></v-spacer>
      <v-btn color="primary" text @click="close">Cancel</v-btn>
      <v-btn color="primary" text @click="save">Save</v-btn>
    </v-card-actions>
  </v-card>
</template>
<script>
  export default {
    name: 'edit-dialog',
    props: {
      close: {},
      editedRule: {},   
      groups_list: {},
      methods: {},
      save: {}      
    },
    data() {
      return {
        groups: [],
        groups_available: [],
      };
    },

    methods:{
      methodColor(method){
        switch(method){
          case 'GET':
            return 'success';
          case 'POST':
            return 'blue';
          case 'PUT':
            return 'warning';
          case 'DELETE':
            return 'error';
          default:
            return            
        }
      }
    }
  }
</script>
<style>
  * {    
    user-select: none
  }
</style>