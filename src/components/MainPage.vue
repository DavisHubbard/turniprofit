<template>
  <div class="hello">
    <apexchart
      width="1900"
      height="800"
      type="line"
      :options="options"
      :series="series"
    ></apexchart>
  </div>
</template>

<script>
import VueApexCharts from "vue-apexcharts";
export default {
  name: "MainPage",
  components: {
    apexchart: VueApexCharts
  },
  data: function() {
    return {
      bot: null,
      turnipData: {},
      idToUser: {
        UM762HAKZ: "Willard",
        U0108CJ77JN: "Hayden",
        ULXAU74AV: "Davis",
        ULWJ61PQ9: "Shoeman",
        UM532106A: "Max",
        U010T1GNHP1: "Laura",
        UM5RY5UDU: "Adam",
        UN4N71EMB: "Alex Wong",
        ULT0JB92P: "Alex J",
        URGUP061Y: "Drew",
        U010F1DSQ6P: "Bo"
      },
      options: {
        chart: {
          zoom: { enabled: false },
          animations: { enabled: false }
        },
        title: {
          text: "Turnup for Turnips",
          align: "left"
        }
      },
      series: []
    };
  },
  mounted() {
    // const Slack = require("slack");
    // const token = this.TOKEN;
    // this.bot = new Slack({ token });
    // this.bot.conversations
    //   .members({ channel: "C010KUBA9V5" })
    //   .then(this.buildUserList)
    //   .catch(console.error);
    this.getPriceData();
  },
  methods: {
    getPriceData: function() {
      fetch("https://turniprofit.firebaseio.com/prices.json")
        .then(response => {
          response.json().then(res => {
            for (let userId in this.idToUser) {
              this.turnipData[userId] = [];
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
            let xDays = [];
            console.log("Sorted Days: ", days);
            for (let i = 0; i < days.length; i++) {
              let date = days[i].date;
              if ("morning" in days[i]) {
                let mornName = date + " M";
                xDays.push(mornName);
                for (let userId in this.idToUser) {
                  if (userId in days[i].morning) {
                    this.turnipData[userId].push(
                      parseInt(days[i].morning[userId].price)
                    );
                  } else {
                    this.turnipData[userId].push(null);
                  }
                }
              }
              if ("evening" in days[i]) {
                let eveName = date + " E";
                xDays.push(eveName);
                for (let userId in this.idToUser) {
                  if (userId in days[i].evening) {
                    this.turnipData[userId].push(
                      parseInt(days[i].evening[userId].price)
                    );
                  } else {
                    this.turnipData[userId].push(null);
                  }
                }
              }
              this.options = {
                ...this.options,
                ...{ xaxis: { categories: xDays } }
              };
            }
            for (let userId in this.turnipData) {
              this.series.push({
                name: this.idToUser[userId],
                data: this.turnipData[userId]
              });
            }
          });
        })
        .catch(err => console.log("ERROR: ", err));
    },
    buildUserList: function(res) {
      for (let i = 0; i < res.members.length; i++) {
        this.bot.users.info({ user: res.members[i] }).then(resp => {
          console.log(resp);
          if (resp.user.real_name !== "turniprofit") {
            this.idToUser[resp.user.id] = resp.user.real_name;
          }
        });
      }
      this.getPriceData();
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped></style>
