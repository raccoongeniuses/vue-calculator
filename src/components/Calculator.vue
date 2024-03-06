<template>
  <div 
    class="hello" 
    @keyup="handleKeyPress" 
    tabindex="0" 
    ref="calculator"
  >
    <h1>{{ msg }}</h1>

    <div class="container">
      <div class="row justify-content-center">
        <div class="col-md-4 m-3">
          <table class="table table-bordered">
            <tbody>
              <tr>
                <td class="equation" colspan="4">
                  {{ equation }}
                </td>
              </tr>
              <tr>
                <td class="output" colspan="4">
                  {{ currentResult || 0 }}
                </td>
              </tr>
              <tr>
                <td v-on:click="calculatePercentage" class="smaller-button">%</td>
                <td class="math-signs smaller-button" @click="calculateSquareRoot">√</td>
                <td class="clear-signs smaller-button" v-on:click="clearEntry">CE</td>
                <td class="clear-signs smaller-button" v-on:click="clearField">C</td>
              </tr>
              <tr>
                <td class="output smaller-button" v-on:click="getNumber('7')">7</td>
                <td class="output smaller-button" v-on:click="getNumber('8')">8</td>
                <td class="output smaller-button" v-on:click="getNumber('9')">9</td>
                <td class="math-signs smaller-button" @click="processOutput('subtract')">-</td>
              </tr>
              <tr>
                <td class="output smaller-button" v-on:click="getNumber('4')">4</td>
                <td class="output smaller-button"  v-on:click="getNumber('5')">5</td>
                <td class="output smaller-button" v-on:click="getNumber('6')">6</td>
                <td class="math-signs smaller-button" @click="processOutput('divide')"><i class="fa-solid fa-divide"></i></td>
              
              </tr>
              <tr>
                <td class="output smaller-button" v-on:click="getNumber('1')">1</td>
                <td class="output smaller-button" v-on:click="getNumber('2')">2</td>
                <td class="output smaller-button" v-on:click="getNumber('3')">3</td>
                <td class="math-signs smaller-button" @click="processOutput('multiply')">x</td>
              </tr>
              <tr>
                <td class="output smaller-button" v-on:click="getDot()">.</td>
                <td class="output smaller-button" v-on:click="getNumber('0')">0</td>
                <td class="math-signs smaller-button" @click="updateOutput">=</td>
                <td class="math-signs smaller-button" @click="processOutput('add')">+</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
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
      operations: [],
      equation: '', // equation preview
    }
  },
  mounted() {
    this.$refs.calculator.focus();
  },
  computed: {
    currentResult() {
      if (this.operations.length >= 3) {
        const tempOperations = [...this.operations]; // Make the operations array copy

        // If there's a current output, add to the tempOperations array, e.g: 2 x 3 + 3 = (6) + 3 in the equation
        if (!isNaN(parseFloat(this.output))) {
          tempOperations.push(parseFloat(this.output));
        }

        // Evaluate operations
        const result = this.evaluateOperations(tempOperations);

        return result
      }
      return this.output || '0'; // Return the output if the operation is empty
    }
  },
  methods:{
    clearField(){
      this.output = '';
      this.previousValue = null;
      this.operations =[];
      this.equation = '';
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
    handleKeyPress(event) {
      const key = event.key;
      if (key >= '0' && key <= '9') {
        this.getNumber(key);
      } else if (key === '.') {
        this.getDot();
      } else if (key === '+' || key === '-' || key === '*' || key === '/') {  // keypress events
        const operations = {
          '+': 'add',
          '-': 'subtract',
          '*': 'multiply',
          '/': 'divide',
        };
        this.processOutput(operations[key]);
      } else if (key === 'Enter') {
        this.updateOutput();
      } else if (key === 'Delete') {
        this.clearField();
      } else if (key === 'Backspace') {
        this.clearEntry();
      }
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
    
    processOutput(operation) {
      if (this.output !== '') {
        // Push output as a number to the operations array
        this.operations.push(parseFloat(this.output));
        this.equation += this.output + ' ';
        this.output = '';
      }

      const operationSymbols = {
        'add': '+',
        'subtract': '-',
        'multiply': 'x',
        'divide': '÷'
      };

      this.equation += operationSymbols[operation] + ' ';

      if (this.operations.length >= 3) {
        const result = this.evaluateOperations(this.operations);
        this.operations = [result, operation];
        this.output = result.toString();
      } else if (this.operations.length === 1) {
        this.operations.push(operation);
      }

      this.operationFired = true;
    },

    updateOutput() {
       // Update output with the final result
      if (this.output !== '') {
        this.operations.push(parseFloat(this.output));
        this.equation += this.output;
      }

      const result = this.evaluateOperations(this.operations);
      this.output = result.toString();
      this.operations = [];
      this.equation = '';
    },

    evaluateOperations(operations) {
      // Multiply and Divide goes first in an equation
      for (let i = 0; i < operations.length; i++) {
        if (operations[i] === 'multiply' || operations[i] === 'divide') {
          const result = this.calculate(operations[i - 1], operations[i + 1], operations[i]);
          operations.splice(i - 1, 3, result);
          i--;
        }
      }

      for (let i = 0; i < operations.length; i++) {
        if (operations[i] === 'add' || operations[i] === 'subtract') {
          const result = this.calculate(operations[i - 1], operations[i + 1], operations[i]);
          operations.splice(i - 1, 3, result);
          i--;
        }
      }

      return operations[0];
    },

    calculate(a, b, operation) {
      switch (operation) {
        case 'add':
          return a + b;
        case 'subtract':
          return a - b;
        case 'multiply':
          return a * b;
        case 'divide':
          return a / b;
        default:
          return b;
      }
    },
  },
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

.smaller-button {
  font-size: 12px;
  padding: 10px;
}

.equation {
  min-height: 20px;
  color: #777;
}
</style>
