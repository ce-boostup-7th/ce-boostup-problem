<template>
  <v-container>
    <v-toolbar flat color="white">
      <v-toolbar-title>Expandable Table</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-dialog v-model="dialog" max-width="800px" persistent>
        <template v-slot:activator="{ on }">
          <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
        </template>
        <v-card>
          <v-card-title>
            <span class="headline">Edit</span>
          </v-card-title>

          <v-card-text>
            <HelloWorld :id="editedIndex" />
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" flat @click="close">Cancel</v-btn>
            <!-- <v-btn color="blue darken-1" flat @click="save">Save</v-btn> -->
          </v-card-actions>
        </v-card>
      </v-dialog>
      <v-btn
        color="primary"
        dark
        @click="expand = !expand"
      >{{ expand ? 'Close' : 'Keep' }} other rows</v-btn>
    </v-toolbar>
    <v-data-table :headers="headers" :items="problem" :expand="expand" item-key="id">
      <template v-slot:items="props">
        <tr @click="props.expanded = !props.expanded">
          <td class="text-xs-right">{{ props.item.id }}</td>
          <td class="text-xs-right">{{ props.item.category_id }}</td>
          <td>{{ props.item.title }}</td>
          <td class="text-xs-right">{{ props.item.difficulty }}</td>
          <td class="text-xs-right">{{ new Date(props.item.updated_at).toLocaleString() }}</td>
          <td class="justify-center layout px-0">
            <v-icon small class="mr-2" @click="editItem(props.item)">edit</v-icon>
            <v-icon small @click="deleteItem(props.item)">delete</v-icon>
          </td>
        </tr>
      </template>
      <template v-slot:expand="props">
        <v-card flat>
          <v-card-text>{{ props.item.description }}</v-card-text>
        </v-card>
      </template>
    </v-data-table>
  </v-container>
</template>

<script>
import { connect } from "./../config";
import HelloWorld from "../components/HelloWorld";

export default {
  components: {
    HelloWorld
  },
  data() {
    return {
      dialog: false,
      expand: false,
      headers: [
        {
          text: "ID",
          align: "left",
          value: "id"
        },
        { text: "category_id", value: "category_id" },
        { text: "title", value: "title" },
        { text: "difficulty", value: "difficulty" },
        { text: "updated_at", value: "updated_at" },
        { text: "Actions", value: "name", sortable: false }
      ],
      problem: [],
      editedIndex: -1
    };
  },
  mounted() {
    this.fetch();
  },
  methods: {
    fetch() {
      fetch(`${connect}/problems`, {
        method: "GET"
      })
        .then(res => {
          if (Math.floor(res.status / 100) != 2) {
            alert("ERROR::Respond\n\n" + res.text());
            return res.status;
          }
          return res.json();
        })
        .then(response => {
          console.log("Success:", response);
          this.problem = response;
        })
        .catch(error => console.error("Error:", error));
    },

    editItem(item) {
      this.editedIndex = item.id;
      this.dialog = true;
    },

    deleteItem(item) {
      if (confirm("Are you sure you want to delete this item?")) {
        fetch(`${connect}/problems/${item.id}`, {
          method: "DELETE"
        })
          .then(res => {
            if (Math.floor(res.status / 100) != 2) {
              alert("ERROR::Respond\n\n" + res.text());
              return res.status;
            }
            return res.json();
          })
          .then(response => {
            console.log("Success:", response);
            this.fetch();
          })
          .catch(error => console.error("Error:", error));
      }
    },

    close() {
      this.dialog = false;
      this.fetch();
      setTimeout(() => {
        this.editedIndex = -1;
      }, 300);
    }
  }
};
</script>