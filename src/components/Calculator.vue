<template>
  <div class="hello">
    <h1>{{ msg }}</h1>

    <div class="col-md-4 m-3">
      <table class="table table-bordered">
        <tbody>
          <tr>
            <td class="output" colspan="4">
              {{ output || 0}}
            </td>
          </tr>
          <tr>
            <td v-on:click="calculatePercentage">%</td>
            <td class="math-signs" @click="calculateSquareRoot">√</td>
            <td class="clear-signs" v-on:click="clearEntry">CE</td>
            <td class="clear-signs" v-on:click="clearField">C</td>
          </tr>
          <tr>
            <td class="output" v-on:click="getNumber('7')">7</td>
            <td class="output" v-on:click="getNumber('8')">8</td>
            <td class="output" v-on:click="getNumber('9')">9</td>
            <td class="math-signs" @click="processOutput('subtract')">-</td>
          </tr>
          <tr>
            <td class="output" v-on:click="getNumber('4')">4</td>
            <td class="output"  v-on:click="getNumber('5')">5</td>
            <td class="output" v-on:click="getNumber('6')">6</td>
            <td class="math-signs" @click="processOutput('divide')"><i class="fa-solid fa-divide"></i></td>
           
          </tr>
          <tr>
            <td class="output" v-on:click="getNumber('1')">1</td>
            <td class="output" v-on:click="getNumber('2')">2</td>
            <td class="output" v-on:click="getNumber('3')">3</td>
            <td class="math-signs" @click="processOutput('multiply')">x</td>
          </tr>
          <tr>
            <td class="output" v-on:click="getDot()">.</td>
            <td class="output" v-on:click="getNumber('0')">0</td>
            <td class="math-signs" @click="updateOutput">=</td>
            <td class="math-signs" @click="processOutput('add')">+</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
export default {
  name: 'CalculatorApp',
  props: {
    msg: String
  },
  data(){
    return {
      output:'',
      previousValue: null,
      operationFired: false,
      operationPerformed: false,
    }
  },
  methods:{
    clearField(){
      this.output = '';
    },
    clearEntry(){
      if (this.operationPerformed) {
        this.output = '';
      } else {
        this.output = this.output.slice(0, -1);
      }
    },
    calculateSquareRoot() {
      this.output = Math.sqrt(parseFloat(this.output));
      this.operationPerformed = true;
    },
    calculatePercentage(){
      this.output = parseFloat(this.output)/100;
    },
    getNumber(number){
      if(this.operationFired){
        this.output ='';
        this.operationFired = false;
      }
      this.operationPerformed = false;
      this.output = `${this.output}${number}`;
    },
    getDot(){
      if(this.output.indexOf('.') === -1){
        this.output = this.output+'.';
      }
    },
    processOutput(string){
      if(string == 'add'){
        this.operation = (a, b) => {
          return parseFloat(a) + parseFloat(b);
        }
      } else if (string =='subtract'){
        this.operation = (a, b) => {
          return parseFloat(a) - parseFloat(b);
        }
      } else if (string =='divide'){
        this.operation = (a, b) => {
          return parseFloat(a) / parseFloat(b);
        }
      } else if (string =='multiply'){
        this.operation = (a, b) => {
          return parseFloat(a) * parseFloat(b);
        }
      }
      this.operationPerformed = true;
      this.previousValue = this.output;
      this.operationFired = true;
    },
    updateOutput() {
      if (this.operation && this.previousValue !== null) {
        this.output = `${this.operation(this.previousValue, this.output)}`
        this.operationPerformed = false;
      } else {
        this.output = this.output || this.previousValue || '';
      }
    }
  }
}
</script>

<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}

.output {
  background-color: #333;
  color: #fff;
}

.bg-green {
  background-color: #42b983;
  color: #333;
}

.math-signs {
  background-color: #fff;
  color: #333;
}

.math-signs:active {
  background-color: #333;
  color: #fff;
}

.clear-signs {
  background-color: #42b983;
  color: #333;
}

.clear-signs:active {
  background-color: #333;
  color: #fff;
}
</style>
