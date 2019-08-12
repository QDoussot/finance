<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <ul>
    <li v-for="money_operation in money_operations">
    {{ money_operation.amount }}
    </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      money_operations: []
    }
    },
    methods: {
      orderByPriceRange(price_ranges_array, money_operations) {
        var bins=[];
        price_ranges_array.forEach(function(range) {
          var amount = 0;
          money_operations.forEach(function (operation) {
            if (-operation.amount > range.start && -operation.amount < range.end) {
              amount += -operation.amount;
            }
          });

          bins.push( {range: range, amount: amount} );
        }

        );
        return bins;
      }
    },
    mounted() {


      axios({
        method: "get",
        url: `http://${location.hostname}:3000/money_operations`
      })
      .then(result => {
        this.money_operations = result.data;
        this.money_operations.sort(function(a, b) {
        var x = -a.amount; var y = -b.amount;
         return ((x < y) ? -1 : ((x > y) ? 1 : 0));
        });


        var price_ranges_array = [ ]
        var maxExpense = this.money_operations[this.money_operations.length-1];
        var maximumAmount=-maxExpense.amount;

        var step=50;
        for (var i=10;i<maximumAmount; i+=step) {
          price_ranges_array.push({start:i-step, end:i})
        }

        var bins=this.orderByPriceRange(price_ranges_array, this.money_operations);

         }
      );


    },
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
