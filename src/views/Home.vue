<template>
  <v-container>
    <v-row justify="center">
      <v-col        
        cols="10"
        md="8"
        >
            <secure_table
            v-for="(table,index) in filledTables"
            :key="index"            
            :table_name="table"
            :table_rules="table_rules(table)"
            :groups_list="groups_list"           
            @edit-rule="editRule"
            @add-rule="addRule"
            @delete-rule="deleteRule"       
            ></secure_table>

      </v-col>
    </v-row>
  </v-container>
</template>

<script>
// @ is an alias to /src
import secure_table from '@/components/secure_table.vue'

export default {
  name: 'home',
  components: {   
    secure_table
  },

  data(){
    return{
      tables:'',
      rules:[]
    }
  },

  methods:{
    initialize() {
      this.tables = this.$mokeServices.getTables();
      this.rules = this.$mokeServices.getDefinitions()   
    },

    table_rules(table_name) {
      return this.rules.filter(table => table.table_name == table_name)
    },    

    editRule(editedRule){
      const editedIndex = this.rules.findIndex(rule => rule.id == editedRule.id);      
      //Object.assign(this.rules[editedIndex], editedRule)
      this.rules.splice(editedIndex,1,editedRule);
      console.log(`Regla ${editedRule} actualizada correctamente`);
    },

    addRule(newRule){
      this.rules.push(newRule)
    },

    deleteRule(ruleId){
      const toDeleteIndex = this.rules.findIndex(rule => rule.id == ruleId);
      this.rules.splice(toDeleteIndex,1)
    }

  },

  computed:{
    //para saber si la tabla tiene elementos, en caso contrario no se renderiza
    filledTables(){
      return this.tables.filter(table => this.table_rules(table).length)
    },

    //para computar los roles existentes en las tablas y adicionarlos al select del adicionar
    groups_list(){
      let groups = [];
      this.rules.forEach( function(el){        
          if(groups.indexOf(el.groups) == -1 && el.groups){
            groups.push(el.groups);
          }              
      });
      return groups
    }
  },

  created(){
    this.initialize();    
  }

}
</script>
