<template>
  <v-card>
    <v-app-bar
        dark
        :class="methodColor(toEditMethod.method)"
        no-gutters
        elevation="2">
      <v-card-title>
        <span class="headline">Method "{{toEditMethod.method}}"</span>
      </v-card-title>
    </v-app-bar>

    <v-card-text class="pb-0">
      <v-container>
        <v-row>
          <v-col cols="12">
            <v-text-field class="mr-2" disabled :value="toEditMethod.tableName" label="Table Name"></v-text-field>
          </v-col>
          <v-col cols="12">
            <v-row
            v-for="(group, index) of groups_list"
            :key="group"
            >
              <v-col cols class="py-0">
                <v-checkbox
                dense          
                :label="group"                
                :value="group"
                v-model="available_groups"
                :disabled="toEditMethod.groups[index] && toEditMethod.groups[index].locked || false "
                multiple
                ></v-checkbox>
              </v-col>
              
              <!-- <v-col cols class="py-0">
              <v-switch
              :label="group.is_active ? 'Active' : 'Not active'"
              :v-model="editedMethod.groups[index].is_active"
              :value="group.is_active"
              :multiple="true" 
              ></v-switch>      
              </v-col> -->

              <v-col cols class="py-0 d-flex flex-row align-center">
                <!-- <v-icon
                small           
                :class="['mdi',toEditMethod.locked? 'mdi-lock':'mdi-lock-open-variant']"
                :color="editedMethod.locked? '':'success'"
                ></v-icon>
                <span class="subtitle-1 ml-1">{{editedMethod.locked? 'Locked':'Unlocked'}}</span> -->
                {{`${toEditMethod.groups[index] ? toEditMethod.groups[index].locked : ''}`}}
              </v-col> 
              
            </v-row>
            {{available_groups}}
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
      toEditMethod: {},
      groups_list:{},
      table_method_rules: {},
      save: {},
      dialog: Boolean
    },
    data() {
      return {
        editedMethod:{},
        available_groups:[],
        active_groups:[],
        locked_groups:[]
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
      },      

      close(){
        //this.toEditMethod.     
        console.log(this.available_groups);
        this.$emit('close')
      }
    },

    created(){
      // let editedMethod = {
      //   tableName: this.toEditMethod.tableName,
      //   method: this.toEditMethod.method,
      //   groups: [],
      //   activeGroups: "",
      //   lockedGroups: ""
      // };
      // this.groups_list.forEach(el => { //se crea un grupo por defecto para cada rol para poderlo renderizar en el dialog y guardar sus propiedades en caso de adicionarlo
      //   let group = {
      //     id: Date.now(),
      //     group: el,
      //     is_active: false,
      //     locked: false,
      //   };

      //   let groupExist = this.toEditMethod.groups.find(grp => grp.group == el);
      //   if(groupExist){//se comprueba si existe dentro del method editado ese grupo
      //     console.log(groupExist)
      //     Object.assign(group,groupExist)
      //   }
      //   editedMethod.groups.push(group)
      // });
      // Object.assign(this.editedMethod, editedMethod)
      this.toEditMethod.groups.forEach(group => this.available_groups.push(group.group))
    },  

    watch: { 
      toEditMethod() {//si cambia el valor del metodo editado se cambian los grupos disponibles a los del nuevo metodo
       this.available_groups = [];
       this.toEditMethod.groups.forEach(group => this.available_groups.push(group.group));       
      },

      dialog(val) {
        val || this.close();
      }
    },
    
  }
</script>
<style>
  * {    
    user-select: none
  }
</style>