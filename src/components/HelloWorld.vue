<template>
  <v-form>
    <v-container>
      <v-layout row wrap>
        <v-flex sm12 lg6>
          <v-text-field outline label="title" v-model="title"></v-text-field>
        </v-flex>

        <v-flex sm12 lg6>
          <v-select
            :items="items"
            item-text="text"
            item-value="val"
            v-model="category_id"
            label="categoryid"
            outline
          ></v-select>
        </v-flex>

        <v-flex sm12 lg6>
          <v-rating v-model="difficulty" background-color="orange lighten-3" color="orange" x-large></v-rating>
        </v-flex>

        <v-flex xs12>
          <v-textarea outline label="description" v-model="description"></v-textarea>
        </v-flex>
      </v-layout>
      <v-tabs color="cyan" dark slider-color="yellow" class="white elevation-1" show-arrows>
        <v-tab v-for="n in numTestCase" :key="n" ripple>case {{ n }}</v-tab>
        <v-tab ripple @click="addArrayTestcase" :key="9999" :disabled="idProblem==-1">Add+</v-tab>
        <v-tab-item v-for="n in numTestCase" :key="n">
          <v-layout row wrap>
            <v-flex xs12>
              <v-btn color="success" @click="updateTestcase(n-1)">Update this testcase</v-btn>
              <v-btn color="error" @click="deleteTestcase(n-1)">Delete this testcase</v-btn>
            </v-flex>
            <v-flex xs6>
              <v-textarea outline label="INPUT" v-model="testcase[n-1].in"></v-textarea>
            </v-flex>
            <v-flex xs6>
              <v-textarea outline label="OUTPUT" v-model="testcase[n-1].out"></v-textarea>
            </v-flex>
          </v-layout>
        </v-tab-item>
        <v-tab-item :key="9999">Add new test case finish {{id}}</v-tab-item>
      </v-tabs>
      <v-layout row wrap class="pt-5">
        <v-flex xs12>
          <v-btn color="success" @click="addData">{{idProblem==-1?"Submit":"Update"}} Data</v-btn>
          <v-btn color="warning" @click="clear">Reset (New problems)</v-btn>
        </v-flex>
      </v-layout>
    </v-container>
  </v-form>
</template>

<script>
import { connect } from "./../config";

var serialize = function(obj) {
  var str = [];
  for (var p in obj)
    if (obj.hasOwnProperty(p)) {
      str.push(encodeURIComponent(p) + "=" + encodeURIComponent(obj[p]));
    }
  return str.join("&");
};

let getData = function(url, data, type) {
  var queryString = serialize(data);

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
  props: {
    id: {
      type: Number,
      default: -1
    }
  },
  data() {
    return {
      numTestCase: 0,
      title: "",
      category_id: "",
      difficulty: 1,
      description: "",
      testcase: [],
      idProblem: -1,
      items: [
        { text: "Basic", val: "1" },
        { text: "Condition", val: "2" },
        { text: "Loop", val: "3" },
        { text: "Function", val: "4" }
      ]
    };
  },
  methods: {
    updateTestcase: function(index) {
      let url = `${connect}/problems/${this.idProblem}/testcases/${index}`;
      let data = {
        input: this.testcase[index].in,
        output: this.testcase[index].out
      };
      getData(url, data, "PUT")
        .then(response => {
          console.log("Success:", response);
        })
        .catch(error => console.error("Error:", error));
    },
    deleteTestcase: function(index) {
      let url = `${connect}/problems/${this.idProblem}/testcases/${index}`;
      getData(url, {}, "DELETE")
        .then(response => {
          console.log("Success:", response);
          this.testcase.splice(index, 1);
          this.numTestCase--;
        })
        .catch(error => console.error("Error:", error));
    },
    addArrayTestcase: function() {
      this.testcase.push({ in: "", out: "" });

      let url = `${connect}/problems/${this.idProblem}/testcases`;
      let data = {
        input: this.testcase[this.numTestCase].in,
        output: this.testcase[this.numTestCase].out
      };
      getData(url, data, "POST")
        .then(response => {
          console.log("Success:", response);
        })
        .catch(error => console.error("Error:", error));

      this.numTestCase++;
    },
    addData: function() {
      let url =
        `${connect}/problems` +
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
          let json = JSON.parse(response);
          if (this.idProblem == -1) this.idProblem = json.id;
        })
        .catch(error => console.error("Error:", error));
    },
    addTestcase: function() {},
    clear: function() {
      this.numTestCase = 0;
      this.title = "";
      this.category_id = "";
      this.difficulty = 1;
      this.description = "";
      this.testcase = [];
      this.idProblem = -1;
    }
  },
  watch: {
    // whenever question changes, this function will run
    id: function() {
      if (this.id != -1) {
        fetch(`${connect}/problems/${this.id}`, {
          method: "GET"
        })
          .then(res => res.json())
          .then(response => {
            console.log("Success:", response);
            this.problem = response;
            this.title = response.title;
            this.category_id = response.category_id;
            this.difficulty = response.difficulty;
            this.description = response.description;
            this.idProblem = response.id;
          })
          .catch(error => console.error("Error:", error));

        fetch(`${connect}/problems/${this.id}/testcases`, {
          method: "GET"
        })
          .then(res => res.json())
          .then(response => {
            console.log("Success:", response);
            this.testcase = [];
            this.numTestCase = response.length;
            for (let i = 0; i < response.length; i++) {
              this.testcase.push({
                in: response[i].input,
                out: response[i].output
              });
            }
          })
          .catch(error => console.error("Error:", error));
      } else {
        this.clear();
      }
    }
  }
};
</script>

<style>
textarea {
  font-family: monospace;
}
</style>
