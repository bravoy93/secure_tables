<template>
  <v-card>
    <v-app-bar
        dark
        :class="methodColor(edited_method.method)"
        no-gutters
        elevation="1">
      <v-card-title>
        <span class="headline">Method "{{edited_method.method}}"</span>
      </v-card-title>
    </v-app-bar>

    <v-card-text class="pb-0">
      <v-container>
        <v-row>
          <v-col cols="12">
            <v-text-field class="mr-2" disabled :value="edited_method.tableName" label="Table Name"></v-text-field>
          </v-col>
          <v-col cols="12">
            <v-row
            v-for="(group, index) of groups_list"
            :key="index"
            >
              <v-col cols class="py-0">
                <v-checkbox
                dense          
                :label="group"                
                :value="group"
                v-model="available_groups"
                :color="methodColor(edited_method.method)"
                :disabled="edited_method.groups[index] && edited_method.groups[index].locked || false "
                multiple
                @change="remove_active(group)"
                ></v-checkbox>
              </v-col>
              
              <v-col cols class="py-0">
                <v-switch
                  :label=" active_groups.includes(group) && 'Active' || 'Not active'"
                  v-model="active_groups"
                  :value="group"  
                  :color="methodColor(edited_method.method)"
                  :disabled="edited_method.groups[index] && edited_method.groups[index].locked || !available_groups.includes(group) || false "
                ></v-switch>      
              </v-col>

              <v-col cols class="py-0 d-flex flex-row align-center">
                <v-icon
                small
                :class="['mdi', edited_method.groups[index] && edited_method.groups[index].locked ? 'mdi-lock':'mdi-lock-open-variant']"
                :color="edited_method.groups[index] && edited_method.groups[index].locked ? 'grey lighten-1' : methodColor(edited_method.method) "                
                ></v-icon>
                <span 
                  :class="['subtitle-1' , 'ml-1' , edited_method.groups[index] && edited_method.groups[index].locked && 'text--disabled' ]"                  
                  >
                    {{edited_method.groups[index] && edited_method.groups[index].locked ? 'Locked':'Unlocked'}}
                </span>                
              </v-col> 
              
            </v-row>
            Avaibles groups:{{available_groups}} Active groups: {{active_groups}} Locked Groups {{locked_groups}}
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
      dialog: Boolean
    },
    data() {
      return {        
        available_groups:[],
        active_groups:[],
        locked_groups:[],
        edited_method:{},
        edited_rule: {},      
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
      
      remove_active(group){
        let index = this.active_groups.indexOf(group);        
        index > -1 &&  this.active_groups.splice(index,1)
      },

      close(){        
        this.$emit('close');
        setTimeout(() => {
          this.role_properties_update()
        }, 300);
        
      },

      save(){             
        for(let grp of this.available_groups) { //se iteran los grupos disponibles pues son los posibles editados o agregados
          const group = this.toEditMethod.groups.find( gr => gr.group == grp); //se crea una constante con las propiedades del grupo iterado
               
          if(group){//si grupo tiene valor entonces el grupo esta disponible en el metodo editado por ende se esta editando el mismo
            if(group.is_active == this.active_groups.includes(grp)){ //se comprueba si ha cambiado su propiedad active que es la unica editable, de ser falso solo continua iterando
              //se esta editando y no se ha cambiado nada
              continue;        
            }else{
              //se esta editando y si cambio el elemento
              Object.assign(this.edited_rule,{//se asigna a edited_rule las propiedades del grupo cambiado con su nuevo valor para luego remplazarlo en la lista del metodo
                id: group.id,
                table_name: this.edited_method.tableName,
                method: this.edited_method.method,
                groups: grp,
                is_active: this.active_groups.includes(grp),
                locked: this.locked_groups.includes(grp)
              });
              this.$emit('edit-rule',this.edited_rule)//se envia a su padre el evento para que lo actualice
              Object.assign(this.edited_rule,{})
            }            
          }else{
            //No existe, se esta adicionando
            Object.assign(this.edited_rule,{//se asigna a edited_rule las propiedades del nuevo grupo para adicionarlo a la lista del metodo
                id: Date.now(),
                table_name: this.edited_method.tableName,
                method: this.edited_method.method,
                groups: grp,
                is_active: this.active_groups.includes(grp),
                locked: this.locked_groups.includes(grp)
                });
            this.$emit('add-rule',this.edited_rule)//se envia a su padre el evento para que lo adicione
            Object.assign(this.edited_rule,{})
            }
          Object.assign(this.edited_rule,{})
        }//termina adicinar/editar y se eliminan los que no estan marcados
        for(let grp of this.toEditMethod.groups) { //se iteran los grupos que habian anteriormente par comprobar si estan en available groups, si no es asi es que furon desmarcados por tanto se elimina
                         
          if(this.available_groups.includes(grp.group)){//se comprueba si el grupo sigue siendo disponible, caso contrario se elimina
            continue;
          }else{
            //No existe, por tanto se elimina            
            this.$emit('delete-rule',grp.id)//se envia a su padre el evento para que lo elimine            
            }
        }      
        //Finalmente termina
      },

      role_properties_update(){        
        Object.assign(this.edited_method, this.toEditMethod);
        this.available_groups = [];
        this.active_groups = [];
        this.locked_groups = [];
        for(let group of this.edited_method.groups){
          this.available_groups.push(group.group);
          if(group.is_active) this.active_groups.push(group.group);
          if(group.locked) this.locked_groups.push(group.group)
        }
        // for(let group of this.toEditMethod.groups){
        //   group.is_active && this.active_groups.push(group.group)
        // }
        
        //this.toEditMethod.groups.forEach(group => this.available_groups.push(group.group));    
        //this.toEditMethod.groups.forEach(group => group.is_active && this.active_groups.push(group.group));
        
      }
    },

    created(){
      this.role_properties_update()
      // Object.assign(this.edited_method, this.toEditMethod);
      // for(let group of this.toEditMethod.groups){
      //     this.available_groups.push(group.group);
      //     if(group.is_active) this.active_groups.push(group.group);
      //     if(group.locked) this.locked_groups.push(group.group)
      // }
      //this.toEditMethod.groups.forEach(group => this.available_groups.push(group.group))
    },  

    watch: { 
      toEditMethod() {//si cambia el valor del metodo editado se cambian los grupos disponibles a los del nuevo metodo
       
       this.role_properties_update();
       
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