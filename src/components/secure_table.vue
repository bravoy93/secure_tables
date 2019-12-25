<template>
  <v-card class="mb-2">
    <v-app-bar
        dark
        class="primary"
        no-gutter
        elevation="2"
        @click="show = !show"
    >
      <v-toolbar-title>
        <h3>{{table_name}}</h3>
      </v-toolbar-title>

      <v-spacer></v-spacer>

      <v-icon :class="['mdi', show ? 'mdi-chevron-up' : 'mdi-chevron-down']" color="white"></v-icon>
      
      <!--Aqui comienza el dialogo-->

      <v-dialog v-model="dialog" max-width="600px">
        <edit-dialog
            :close="close"
            :edited-rule="editedRule"     
            :groups_list="groups_list"
            :methods="methods"
            :save="save"
        />
      </v-dialog>

      <!--Aqui termina el dialogo-->

    </v-app-bar>

    <!--table elements here-->
    <v-card-text v-show='show'>
      <v-data-table
          dense
          :headers="table_headers"
          :items="table_method_rules"          
          @click:row="editRule"
      >
        <template v-slot:item.method="{ item }">
          <method-tag :method="item.method"/>
        </template>

        <template v-slot:item.groups="{ item }">
          <v-chip 
          v-if='!item.groups.length'
          small
          title="There are not groups assigned to this method yet"
          >not available</v-chip>
          <v-chip v-else
            v-for="group in item.groups"
            :key="group.id"
            small
            class="mr-1 primary"
            :title="`This group is ${group.is_active ? 'active':'not active'}`"          
          >
            <v-avatar
              v-show="group.is_active"              
              left 
              class="mr-n2"
            >
              <v-icon left small dense color="white" class="pr-0">mdi-check-circle</v-icon>
            </v-avatar>
            {{group.group}}
          </v-chip>
        </template>
        
        <template v-slot:item.is_active="{ item }">
          <div @click.stop>
          <v-switch
              v-show="!item.activeGroups || item.activeGroups == 100"
              :value="item.activeGroups"
              hight-detail
              dense
              small
              :color="item.is_active ? 'primary':''"
              v-model="item.is_active"              
          ></v-switch>
          <v-progress-circular
            v-show="item.activeGroups && item.activeGroups != 100"           
            :value="item.activeGroups"
            :title="`There are ${45} groups actives`"
          ></v-progress-circular>
          </div>
        </template>        

        <template v-slot:item.locked="{ item }">
          <v-icon
              small
              :class="['mdi', item.locked? 'mdi-lock':'mdi-lock-open-variant']"
              :color="item.locked? '':'success'"
          >
          </v-icon>
        </template>

        <template v-slot:no-data>
          <v-btn color="primary">Reset</v-btn>
        </template>

      </v-data-table>
    </v-card-text>

  </v-card>
</template>

<script>
  import EditDialog from "./EditDialog";
  import MethodTag from "./MethodTag";

  export default {
    components: {MethodTag, EditDialog},
    props: {
      table_name: String,
      table_rules: Array,
      global_last_id: Number,
      groups_list: Array
    },

    data() {
      return {
        table_headers: [
          {text: 'Method', value: 'method'},
          {text: 'Groups', value: 'groups'},
          {text: 'Active', value: 'is_active'},
          {text: 'Lock', value: 'locked'},
          // {text: 'Action ', value: 'action', sortable: false},
        ],
        methods: ['GET', 'POST', 'PUT', 'DELETE'],
        dialog: false,
        show: false,
        editedIndex: -1,
        editedRule: {
          id: Date.now(),
          table_name: '',
          method: '',
          groups: '',
          is_active: '',
          locked: ''
        },
        defaultRule: {
          id: Date.now(),
          table_name: this.table_name,
          method: '',
          groups: '',
          is_active: '',
          locked: true
        }

      }
    },

    methods: {

      editRule(rule) {
        this.editedIndex = this.table_rules.indexOf(rule) //Se busca el indice de la row editada para poderlo agregar ahi mismo luego
        this.editedRule = Object.assign({}, rule) //Se ponen los datos de la fruta a editar
        this.dialog = true //se activa el modal
      },

      close() {
        this.dialog = false
        setTimeout(() => {
          this.editedRule = Object.assign({}, this.defaultRule);
          this.editedIndex = -1
        }, 300)
      },

      save() {
        if (this.editedIndex > -1 /*si es mayor se esta editando una row*/) {
          this.$emit('edit-rule', this.editedRule);
          this.close()
        } else /*es -1, o sea no se esta editando, se agrega*/ if (!this.ruleExist(this.editedRule)) {//valida que no exista una row igual
          for (let rule of this.table_rules) {
            if (rule.method == this.editedRule.method && this.editedRule.groups && !rule.groups) { //verifica si se esta agregando un method ya existente con group y lo actualiza
              let groups = this.editedRule.groups;
              this.editedRule = rule;
              this.editedRule.groups = groups;
              this.editRule(this.editedRule);
              this.close();
              return;
            } else if (rule.method == this.editedRule.method && !this.editedRule.groups) { //verifica que no se ponga un method existente asociado ya a un grupo en vacio
              this.close();
              alert("There are groups associated to this method, row wasn't saved")
              return;
            }
          }
          this.$emit('add-rule', this.editedRule);
          this.defaultRule.id = Date.now();
          this.close()
        } else alert("Operation fail!... resulting row exist, please try another config")
      },

      toggleRuleKey(rule, key) {
        this.$emit('toggle-rule-key', rule, key)
      },

      //para comprobar si ya existe una row con un metodo y group determinado, retorna el indice de la row si es true, o false caso contrario
      ruleExist(editedRule) {
        for (let rule of this.table_rules) {
          if (rule.method == editedRule.method && rule.groups == editedRule.groups)
            return true
        }
        return false
      }

    },

    computed: {
      //agrupa las rows por metodo
      table_method_rules(){
        let methodRules = []; //almacenara las nuevas rows agrupadas

        this.methods.forEach( method => { //crea un objeto con el nombre del metodo iterado y un array de los roles y sus propiedades, asociados a dicho metodo
            let newMethod = {
            method: method,
            groups: [],
            activeGroups:'',
            lockedGroups:''
          };

          this.table_rules.forEach( rule => { //cada regla de la tabla la cual coincida con el metodo iterado, se crea un objeto con sus propiedades y se adiciona al groups del metodo
            if(rule.method === method ){
              let newRule = {
                id:rule.id,
                group: rule.groups,
                is_active: rule.is_active,
                locked: rule.locked
              };
              newMethod.groups.push(newRule)
            }
          })
          //akkkkiiii me quedeeeeeeeeeeeeeeeeeeeeeeeeee
          let activeGroups = newMethod.groups.filter(grp => grp.is_active);//asigna a la propiedad activeGroup la cantidad de elementos activos de la row
          let lockedGroups = parseInt(()=> newMethod.groups.filter(grp => grp.locked).length);//asigna a la propiedad lockedGroup la cantidad de elementos bloqueados de la row
          console.log(activeGroups,lockedGroups);
          methodRules.push(newMethod)
        });
        console.log(methodRules);
        return methodRules;    
      },

      //

    },

    watch: {
      dialog(val) {
        val || this.close()
      },
    },

    created() {
      this.editedRule = Object.assign({}, this.defaultRule);
    }

  }  
</script>

<style>
  * {
    user-select: none
  }
</style>
