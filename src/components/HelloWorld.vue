<template>
  <v-form>
    <v-container>
      <v-layout row wrap>
        <v-flex xs12 sm6>
          <v-text-field outline label="title" v-model="title"></v-text-field>
        </v-flex>
        <v-flex xs12 sm6>
          <v-text-field outline label="categoryid" v-model="category_id"></v-text-field>
        </v-flex>
        <v-flex xs12 sm6>
          <v-text-field outline label="tags"></v-text-field>
        </v-flex>
        <v-flex xs12 sm6>
          <v-text-field outline label="difficulty" v-model="difficulty"></v-text-field>
        </v-flex>
        <v-flex xs12>
          <v-textarea outline label="description" v-model="description"></v-textarea>
        </v-flex>
      </v-layout>
      <v-tabs color="cyan" dark slider-color="yellow" class="white elevation-1">
        <v-tab v-for="n in numTestCase" :key="n" ripple>Testcase {{ n }}</v-tab>
        <v-tab ripple @click="addArrayTestcase" :key="9999">Add+</v-tab>
        <v-tab-item v-for="n in numTestCase" :key="n">
          <v-layout row wrap>
            <v-flex xs12>
              <v-btn
                color="success"
                @click="deleteTestcase(n-1)"
                :disabled="testcase[n-1].add"
              >Delete this testcase {{testcase[n-1].notadd}}</v-btn>
            </v-flex>
            <v-flex xs6>
              <v-textarea outline label="INPUT" v-model="testcase[n-1].in"></v-textarea>
            </v-flex>
            <v-flex xs6>
              <v-textarea outline label="OUTPUT" v-model="testcase[n-1].out"></v-textarea>
            </v-flex>
          </v-layout>
        </v-tab-item>
        <v-tab-item :key="9999">Add new test case finish</v-tab-item>
      </v-tabs>
      <v-layout row wrap class="pt-5">
        <v-flex xs12>
          <v-btn color="success" @click="addData">{{idProblem==-1?"Submit":"Update"}} Data</v-btn>
          <v-btn color="success" @click="addTestcase" :disabled="idProblem==-1">Add testcase</v-btn>
          <v-btn color="warning" @click="clear">Reset (New problems)</v-btn>
        </v-flex>
      </v-layout>
    </v-container>
  </v-form>
</template>

<script>
import { connect, path } from "./../config";

let getData = function(url, data, type) {
  var queryString = Object.keys(data)
    .map(key => key + "=" + data[key])
    .join("&");

  return fetch(url, {
    method: type, // *GET, POST, PUT, DELETE, etc.
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
  }).then(res => res.text());
};
export default {
  data() {
    return {
      numTestCase: 0,
      title: "",
      category_id: "",
      difficulty: "",
      description: "",
      testcase: [],
      idProblem: -1
    };
  },
  methods: {
    deleteTestcase: function(index) {
      this.testcase.splice(index, 1);
      this.numTestCase--;
    },
    addArrayTestcase: function() {
      this.testcase.push({ in: "", out: "", add: false });
      this.numTestCase++;
    },
    addData: function() {
      let url =
        `http://${connect + path}/problems` +
        (this.idProblem == -1 ? "" : "/" + this.idProblem);
      let data = {
        title: this.title,
        category_id: this.category_id,
        difficulty: this.difficulty,
        description: this.description
      };

      getData(url, data, this.idProblem == -1 ? "POST" : "PUT")
        .then(response => {
          console.log("Success:", response);
          if (this.idProblem == -1) this.idProblem = response;
        })
        .catch(error => console.error("Error:", error));
    },
    addTestcase: function() {
      if (this.idProblem == -1) return;
      if (this.saveTestcase == true) return;
      let url = `http://${connect + path}/problems/${
        this.idProblem
      }/testcases/`;

      for (let i = 0; i < this.numTestCase; i++) {
        if (this.testcase[i].add) continue;
        let data = {
          input: this.testcase[i].in,
          output: this.testcase[i].out
        };
        this.testcase[i].add = true;
        getData(url, data, "POST")
          .then(response => {
            console.log("Success:", response);
            this.$forceUpdate();
          })
          .catch(error => console.error("Error:", error));
      }
    },
    clear: function() {
      this.numTestCase = 0;
      this.title = "";
      this.category_id = "";
      this.difficulty = "";
      this.description = "";
      this.testcase = [];
      this.idProblem = -1;
    }
  }
};
</script>

<style>
textarea {
  font-family: monospace;
}
</style>
