<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <canvas width="800px" height="500px" id="histogramme">
    </canvas>


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
            if (-operation.amount > range.start && -operation.amount <= range.end) {
              amount += -operation.amount;
            }
          });
          bins.push( {range: range, amount: amount} );
        }

        );

        return bins;
      },
      getMaximumExpense(money_operations) {
        var max = -money_operations[0].amount;
        money_operations.forEach(function(money_operation) {
          max = Math.max(max,-money_operation.amount);
        });
        return max;
      },
      //bins is an array of {range: range, amount: amount}
      getBiggestBinAmout(bins) {
        var max = bins[0].amount;
        bins.forEach(function(bin) {
          max = Math.max(max,bin.amount);
        });
        return max;
      },
      generateBinRanges(maximumAmount, binSize) {
        var price_ranges_array = [ ]
        for (var i=0;i<maximumAmount; i+=binSize) {
          price_ranges_array.push({start:i, end:i+binSize})
        }
        return price_ranges_array;
      },
      orderMoneyOperationByMonths(money_operations){
        var monthlyMoneyOperation = {};
        money_operations.forEach(function(money_operation)
        {

        });
      },
      drawHistogram(bins) {

      var canvas = document.getElementById("histogramme");
      var ctx = canvas.getContext("2d");
      ctx.fillStyle = "#FFFFFF";
      ctx.fillRect(0,0, canvas.width, canvas.height);

      var width = canvas.width;
      var height = canvas.height;
      var xHistoStart = 100;
      var histoWidth = 500
      var xHistoEnd = width - 100;

      //draw inner background of histo
      ctx.fillStyle = "#999999";
      ctx.fillRect(xHistoStart,0, histoWidth, canvas.height);

      ctx.fillStyle = "#000000";
      var finalEnd = bins[bins.length-1].range.end;
      var maximumBinAmount = this.getBiggestBinAmout(bins);
      bins.forEach(function (bin)
        {
          var xStart = bin.range.start * histoWidth / finalEnd;
          var xEnd = bin.range.end * histoWidth / finalEnd;
          var y = bin.amount * height / maximumBinAmount;
          var yStart = height-y;
          ctx.fillRect(xStart + xHistoStart,yStart, xEnd-xStart, y);
        }
      );
      var ctx_text = canvas.getContext("2d");
      ctx_text.font = '12px serif';
      var valueStep =250;
      for (var i=valueStep;i < maximumBinAmount; i+=valueStep)
      {
        var textY = i * height / maximumBinAmount;
        ctx_text.fillText(`${i}`,xHistoStart-50,height-textY);
      }
      }
    },
    mounted() {

      axios({
        method: "get",
        url: `http://${location.hostname}:3000/money_operations`
      })
      .then(result => {
        const money_operations = result.data;
        console.log(money_operations[0]);
        var maximumAmount=this.getMaximumExpense(money_operations);

        var price_ranges_array = this.generateBinRanges(maximumAmount, 25);

        var bins=this.orderByPriceRange(price_ranges_array, money_operations);

        this.drawHistogram(bins);

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
