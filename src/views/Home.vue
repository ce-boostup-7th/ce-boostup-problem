<template>
  <v-form>
    <v-container>
      <v-layout column>
        <v-flex xs12 sm6>
          <v-text-field label="user"></v-text-field>
        </v-flex>
        <v-flex xs12 sm6>
          <v-text-field label="pass"></v-text-field>
        </v-flex>
        <v-flex xs12 sm6>
          <v-btn color="success" block v-on:click="login">Success</v-btn>
        </v-flex>
      </v-layout>
    </v-container>
  </v-form>
</template>

<script>
import { connect } from "./../config";

export default {
  methods: {
    login: function() {
      let url = `http://${connect}/login`;
      let data = {
        username: "motor",
        password: "rolas"
      };
      var queryString = Object.keys(data)
        .map(key => key + "=" + data[key])
        .join("&");

      fetch(url, {
        method: "POST", // *GET, POST, PUT, DELETE, etc.
        mode: "cors", // no-cors, cors, *same-origin
        cache: "no-cache", // *default, no-cache, reload, force-cache, only-if-cached
        credentials: "same-origin", // include, *same-origin, omit
        headers: {
          // "Content-Type": "application/json"
          "Content-Type": "application/x-www-form-urlencoded"
        },
        redirect: "follow", // manual, *follow, error
        referrer: "no-referrer", // no-referrer, *client
        body: queryString // body data type must match "Content-Type" header
      })
        .then(res => res.text())
        .then(response => {
          console.log("Success:", response);
          this.$router.push("about");
        })
        .catch(error => console.error("Error:", error));
    }
  }
};
</script>
