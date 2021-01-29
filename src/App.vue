<template>
  <div class="container-fluid p-5 bg-light text-center my-3">
    <h1>Covid-19 Daily Tracker for India</h1>
    <h5 class="text-muted">Double click on a state's name to view it's data of last month</h5>
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
        <p class="text-center"> Dates of last 30 days </p>
      </div>
    </div>
  </div>
</template>

<script>
var myChart;
export default {
  data() {
    return {
      rawdata: [],
      codes: [],
      confirmed: [],
      recovered: [],
      confirmed30: [],
      recovered30: [],
      confirmedline: [],
      recoveredline: [],
      myChart: {}
    }
  },
  methods: {
    handleClick(item) {
      // console.log(item);

      var newConfirmedLine = [];
      var newRecoveredLine = [];

      for(var ele in this.confirmed30)
        newConfirmedLine.push(this.confirmed30[ele][item]); 

      for(var ele in this.recovered30)
        newRecoveredLine.push(this.recovered30[ele][item]);

      this.confirmedline = newConfirmedLine;
      this.recoveredline = newRecoveredLine;

      // console.log(this.confirmedline);
      // console.log(this.recoveredline);

      myChart.data.datasets[0].data = this.confirmedline;
      myChart.data.datasets[1].data = this.recoveredline;
      myChart.update();
    }
  },
  mounted() {
    fetch('https://api.covid19india.org/states_daily.json')
    .then(res => res.json())
    .then(data => this.rawdata = data["states_daily"].slice(870))
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

      rawdata.reverse();
      var i;
      for(i=0; i<91; i++) {
        if(i%3==1) this.recovered30.push(rawdata[i]);
        else if(i%3==2) this.confirmed30.push(rawdata[i]);
        }

      this.confirmed30.reverse();
      this.recovered30.reverse();

      // console.log(this.confirmed30);
      // console.log(this.recovered30);
      // console.log(this.codes);
      // console.log(this.confirmed);
      // console.log(this.recovered);
    
    for(var item in this.confirmed30)
      this.confirmedline.push(this.confirmed30[item].mh); 

    for(var item in this.recovered30)
      this.recoveredline.push(this.recovered30[item].mh);
   

    // Chart initialization 
    var ctx = document.getElementById("myChart").getContext('2d');
    myChart = new Chart(ctx, {
    type: 'line',
    data: {
        labels: ['30', '31', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24', '25', '26', '27', '28'],
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