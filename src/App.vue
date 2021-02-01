<template>
  <div class="container-fluid p-5 bg-light text-center my-3">
    <h1>Covid-19 Daily Tracker for India</h1>
    <h5 class="text-muted">Double click on a state code and enter dates to view the graph</h5>
  </div>

  <div class="container my-5">
    <div class="row text-center">
      <div class="col-2 text-info">
        <h5>State code</h5>
        <p id="code">
          <ul>
            <li v-for="item in codes" :key="item">
              <div @dblclick="handleClick(item)">{{ item.toUpperCase() }}</div>
            </li>
          </ul>
        </p>
      </div>
      <div class="col-2 text-warning">
        <h5>Confirmed</h5>
        <p id="confirmed">    
          <ul>
            <li v-for="item in confirmed" :key="item">
              {{ item }}
            </li>
          </ul>
        </p>
      </div>
      <div class="col-2 text-success">
        <h5>Recovered</h5>
        <p id="recovered">
          <ul>
            <li v-for="item in recovered" :key="item">
              {{ item }}
            </li>
          </ul>
        </p>
      </div>
      <div class="col-6">
        <canvas id="myChart"></canvas> 
        <p>Please select start and end date</p>
        <input type="date" id="datefrom" placeholder="Select start date" min= "2020-05-14" v-model="datefrom" @change="dateSelect()">
        <input type="date" id="dateto" placeholder="Select end date" v-model="dateto" @change="dateSelect()">
      </div>
    </div>
  </div>
</template>

<script>
var myChart;
export default {
  data() {
    return {
      codes: [],
      confirmed: [],
      recovered: [],
      confirmedset: [],
      recoveredset: [],
      confirmedline: [],
      recoveredline: [],
      myChart: {},
      datefrom: '',
      dateto: '',
      statecode: ''
    }
  },
  methods: {
    filterDate(obj) {
      let currdateObj = new Date(obj.dateymd);
      let mindateObj = new Date(this.datefrom);
      let maxdateObj = new Date(this.dateto);
      return (currdateObj >= mindateObj && currdateObj <= maxdateObj);
    },
    calculateDataSets() {
      if(this.datefrom != '' && this.dateto != '' && this.statecode != '') {

        var confirmedFiltered = this.confirmedset.filter(this.filterDate);
        var recoveredFiltered = this.recoveredset.filter(this.filterDate);

        var newConfirmedLine = [];
        var newRecoveredLine = [];
        var newLabels = [];

        for(var ele in confirmedFiltered) {
          newConfirmedLine.push(confirmedFiltered[ele][this.statecode]);
          newLabels.push(confirmedFiltered[ele]["dateymd"]); 
        }

        for(var ele in recoveredFiltered)
          newRecoveredLine.push(recoveredFiltered[ele][this.statecode]);

        this.confirmedline = newConfirmedLine;
        this.recoveredline = newRecoveredLine;

        // console.log(this.confirmedline);
        // console.log(this.recoveredline);

        myChart.data.datasets[0].data = this.confirmedline;
        myChart.data.datasets[1].data = this.recoveredline;
        myChart.data.labels = newLabels;
        myChart.update();
      }
    },
    handleClick(item) {
      this.statecode = item;
      this.calculateDataSets();
    },
    dateSelect() {
      this.calculateDataSets();
    }
  },
  mounted() {
    fetch('https://api.covid19india.org/states_daily.json')
    .then(res => res.json())
    .then(data => this.rawdata = data["states_daily"])
    .then(rawdata => {
      // console.log(rawdata);
      this.confirmed = rawdata[rawdata.length -3];
      this.recovered = rawdata[rawdata.length -2];
      
      delete this.confirmed.date;
      delete this.confirmed.dateymd;
      delete this.confirmed.status;
      delete this.recovered.date;
      delete this.recovered.dateymd;
      delete this.recovered.status;

      for(var key in this.confirmed)
        this.codes.push(key);

      function checkConfirmed(obj) {
        return obj.status == 'Confirmed'
      }

      function checkRecovered(obj) {
        return obj.status == 'Recovered'
      }

      this.confirmedset = rawdata.filter(checkConfirmed);
      this.recoveredset = rawdata.filter(checkRecovered);

    // Setting maximum selectable date on the datepicker as per API data availability
    var currentDate = new Date();
    currentDate.setDate(currentDate.getDate() - 1);
    var cDay = currentDate.getDate()
    if(cDay<10) cDay = '0'+cDay;
    var cMonth = currentDate.getMonth() + 1
    if(cMonth<10) cMonth = '0'+cMonth;
    var cYear = currentDate.getFullYear()
    var yesterday = cYear+'-'+cMonth+'-'+cDay;
    //console.log(yesterday);

    // Setting minimum selectable date on the datepicker as per API data availability
    var mindate = "2020-05-14";

    document.getElementById("datefrom").min = mindate;
    document.getElementById("dateto").min = mindate;
    document.getElementById("datefrom").max = yesterday;
    document.getElementById("dateto").max = yesterday;

    // Chart initialization 
    var ctx = document.getElementById("myChart").getContext('2d');
    myChart = new Chart(ctx, {
    type: 'line',
    data: {
        labels: [],
        datasets: [
          {
            label: 'Confirmed Cases',
            data: this.confirmedline,
            backgroundColor: '#f1c40f',
            borderColor: '#f1c40f',
            fill: false,
          },
          {
            label: 'Recovered Cases',
            data: this.recoveredline,
            backgroundColor: '#2ecc71',
            borderColor: '#2ecc71',
            fill: false
          }
          ]
    },
    options: {
        scales: {
            yAxes: [{
                ticks: {
                    beginAtZero: true
                }
            }]
        }
    }
    });

    })
    .catch(err => console.log(err.message))

  }
  
};

</script>

<style>
ul {
  list-style-type: none;
}
</style>