<template>
  <div class="hello">
    <GChart type="AreaChart" :data="turnipData" :options="graphOptions">
    </GChart>
  </div>
</template>

<script>
import { GChart } from "vue-google-charts";
export default {
  name: "MainPage",
  components: {
    GChart
  },
  data: function() {
    return {
      turnipData: [["Day"]],
      graphOptions: {
        title: "Get Rich Quick",
        vAxis: { minValue: 0 }
      },
      idToUser: {
        UM762HAKZ: "Willard",
        U0108CJ77JN: "Hayden",
        ULXAU74AV: "Davis",
        ULWJ61PQ9: "Shoeman",
        UM532106A: "Max"
      },
      orderedUsers: [
        "UM762HAKZ",
        "U0108CJ77JN",
        "ULXAU74AV",
        "ULWJ61PQ9",
        "UM532106A"
      ]
    };
  },
  mounted() {
    fetch("https://turniprofit.firebaseio.com/prices.json").then(response => {
      response.json().then(res => {
        console.log(res);
        for (let userId in this.idToUser) {
          this.turnipData[0].push(this.idToUser[userId]);
        }
        let days = [];
        for (let day in res) {
          let obj = res[day];
          obj["date"] = day;
          days.push(obj);
        }
        days.sort((a, b) => {
          if (a.date > b.date) return 1;
          else return -1;
        });
        console.log("Sorted Days: ", days);
        for (let i = 0; i < days.length; i++) {
          let date = days[i].date;
          if ("morning" in days[i]) {
            let mornPrices = [date + " M"];
            for (let j = 0; j < this.orderedUsers.length; j++) {
              let userId = this.orderedUsers[j];
              if (userId in days[i].morning) {
                mornPrices.push(parseInt(days[i].morning[userId].price));
              } else {
                mornPrices.push(0);
              }
            }
            this.turnipData.push(mornPrices);
            console.log("Morning Prices: ", mornPrices);
          }
          if ("evening" in days[i]) {
            let evePrices = [date + " E"];
            for (let j = 0; j < this.orderedUsers.length; j++) {
              let userId = this.orderedUsers[j];
              if (userId in days[i].evening) {
                evePrices.push(parseInt(days[i].evening[userId].price));
              } else {
                evePrices.push(0);
              }
            }
            this.turnipData.push(evePrices);
            console.log("Evening Prices: ", evePrices);
          }
        }
      });
    });
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
