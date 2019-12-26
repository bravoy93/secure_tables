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
        <h3>{{ table_name }}</h3>
      </v-toolbar-title>

      <v-spacer></v-spacer>

      <v-icon
        :class="['mdi', show ? 'mdi-chevron-up' : 'mdi-chevron-down']"
        color="white"
      ></v-icon>

      <!--Aqui comienza el dialogo-->

      <v-dialog v-model="dialog" max-width="600px">
        <edit-dialog
          @close="close"
          :toEditMethod="editedMethod"
          :groups_list="groups_list"
          :table_method_rules="table_method_rules"
          :dialog = "dialog"
          :save="save"
        />
      </v-dialog>

      <!--Aqui termina el dialogo-->
    </v-app-bar>

    <!--table elements here-->
    <v-card-text v-show="show">
      <v-data-table
        dense
        :headers="table_headers"
        :items="table_method_rules"
        @click:row="editMethod"
      >
        <template v-slot:item.method="{ item }">
          <method-tag :method="item.method" />
        </template>

        <template v-slot:item.groups="{ item }">
          <v-chip
            v-if="!item.groups.length"
            small
            title="There are not groups assigned to this method yet"
            >not available</v-chip
          >
          <v-chip
            v-else
            v-for="group in item.groups"
            :key="group.id"
            small
            class="mr-1 primary"
            :title="
              `This group is ${group.is_active ? 'active' : 'not active'}`
            "
          >
            <v-avatar v-show="group.is_active" left class="mr-n2">
              <v-icon left small dense color="white" class="pr-0"
                >mdi-check-circle</v-icon
              >
            </v-avatar>
            {{ group.group }}
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
              :color="item.is_active ? 'primary' : ''"
              v-model="item.is_active"
            ></v-switch>
            <v-progress-circular
              v-show="item.activeGroups && item.activeGroups != 100"
              :value="item.activeGroups"
              color="primary"
              class="my-2"
              size="32"
              width="1"
            >
              <span class="caption">{{ item.activeGroups }}%</span>
            </v-progress-circular>
          </div>
        </template>

        <template v-slot:item.locked="{ item }">
          <v-icon
            v-show="!item.lockedGroups || item.lockedGroups == 100"
            small
            :class="['mdi',item.lockedGroups ? 'mdi-lock' : 'mdi-lock-open-variant', 'ml-2']"
            :color="item.locked ? '' : 'success'"
          >
          </v-icon>
          <v-progress-circular
            v-show="item.lockedGroups && item.lockedGroups != 100"
            :value="item.lockedGroups"
            color="primary"
            class="my-2"
            width="1"
          >
            <span class="caption">{{ item.lockedGroups }}%</span>
          </v-progress-circular>
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
  components: { MethodTag, EditDialog },
  props: {
    table_name: String,
    table_rules: Array,
    global_last_id: Number,
    groups_list: Array
  },

  data() {
    return {
      table_headers: [
        { text: "Method", value: "method" },
        { text: "Groups", value: "groups" },
        { text: "Active", value: "is_active" },
        { text: "Lock", value: "locked" }
        // {text: 'Action ', value: 'action', sortable: false},
      ],
      methods: ["GET", "POST", "PUT", "DELETE"],
      dialog: false,
      show: false,
      //editedIndex: -1,
      editedMethod: {},
      editedRule: {
        id: "",
        table_name: "",
        method: "",
        groups: "",
        is_active: "",
        locked: ""
      },
      defaultRule: {
        id: "",
        table_name: this.table_name,
        method: "",
        groups: "",
        is_active: "",
        locked: true
      }
    };
  },

  methods: {
    editMethod(method_row) {
      this.editedMethod = method_row;
      this.dialog = true; //se muestra el modal
    },

    close() {
      this.dialog = false;
      // setTimeout(() => {
      //   this.editedMethod = {};
      // }, 300);
    },

    save() {
      this.$emit("edit-rule", this.editedMethod);
      this.close();
    },
    

    toggleRuleKey(rule, key) {
      this.$emit("toggle-rule-key", rule, key);
    }
  },

  computed: {
    //agrupa las rows por metodo
    table_method_rules() {
      let methodRules = []; //almacenara las nuevas rows agrupadas

      this.methods.forEach(method => {
        //crea un objeto con el nombre del metodo iterado y un array de los roles y sus propiedades, asociados a dicho metodo
        let newMethod = {
          tableName: this.table_name,
          method: method,
          groups: [],
          activeGroups: "",
          lockedGroups: ""
        };

        this.table_rules.forEach(rule => {
          //cada regla de la tabla la cual coincida con el metodo iterado, se crea un objeto con sus propiedades y se adiciona al groups del metodo
          if (rule.method === method) {
            let newRule = {
              id: rule.id,
              group: rule.groups,
              is_active: rule.is_active,
              locked: rule.locked
            };
            newMethod.groups.push(newRule);
          }
        });

        let percent = property =>
          parseInt(
            (newMethod.groups.filter(grp => grp[property]).length /
              newMethod.groups.length) *
              100
          ); //devuelve el porcentaje de las opciones marcadas

        newMethod.activeGroups = percent("is_active");
        newMethod.lockedGroups = percent("locked");

        methodRules.push(newMethod);
      });
      console.log(methodRules);
      return methodRules;
    }

    //
  },
  

  created() {
    this.editedRule = Object.assign({}, this.defaultRule);
  }
};
</script>

<style>
* {
  user-select: none;
}
</style>
