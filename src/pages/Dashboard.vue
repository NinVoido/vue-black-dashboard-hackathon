<template>
  <div>

    <div class="row">
      <div class="col-12">
        <card type="chart">
          <template slot="header">
            <div class="row">
              <div class="col-sm-6" :class="isRTL ? 'text-right' : 'text-left'">
                <h2 class="card-title">Total points</h2>
              </div>
              <div class="col-sm-6">
                <div class="btn-group btn-group-toggle"
                     :class="isRTL ? 'float-left' : 'float-right'"
                     data-toggle="buttons">
                  <label v-for="(option, index) in bigLineChartCategories"
                         :key="option"
                         class="btn btn-sm btn-primary btn-simple"
                         :class="{active: bigLineChart.activeIndex === index}"
                         :id="index">
                    <input type="radio"
                           @click="initBigChart(index)"
                           name="options" autocomplete="off"
                           :checked="bigLineChart.activeIndex === index">
                    {{option}}
                  </label>
                </div>
              </div>
            </div>
          </template>
          <div class="chart-area">
            <line-chart style="height: 100%"
                        ref="bigChart"
                        v-if="bigLineChart.loaded"
                        chart-id="big-line-chart"
                        :chart-data="bigLineChart.chartData"
                        :gradient-colors="bigLineChart.gradientColors"
                        :gradient-stops="bigLineChart.gradientStops"
                        :extra-options="bigLineChart.extraOptions">
            </line-chart>
          </div>
        </card>
      </div>

      <div class="col-lg-6 col-md-12">
        <card type="tasks">
          <template slot="header">
            <h4 class="title d-inline">Top players</h4>
          </template>
          <div class="table-full-width table-responsive">
            <template>
              <base-table :data="tableLol.tableData" :columns="tableLol.columns">
                <template slot="columns">
                  <th class="text-center">#</th>
                  <th>Name</th>
                  <th>Points</th>
                </template>
                <template slot-scope="{row}">
                  <td>{{row.id}}</td>
                  <td>{{row.name}}</td>
                  <td>{{row.pts}}</td>
                </template>
              </base-table>
            </template>
          </div>
        </card>
      </div>
      <div class="col-lg-6 col-md-12">
        <card class="card" :header-classes="{'text-right': isRTL}">
        <h4 slot="header" class="card-title">Top teams</h4>
          <div class="table-responsive">
          <template>
              <base-table :data="tableKek.tableData" :columns="tableKek.columns">
                <template slot="columns">
                  <th class="text-center">#</th>
                  <th>Name</th>
                  <th>Points</th>
                </template>
                <template slot-scope="{row}">
                  <td>{{row.id}}</td>
                  <td>{{row.name}}</td>
                  <td>{{row.pts}}</td>
                </template>
              </base-table>
            </template>

          </div>
        </card>
      </div>
    </div>
  </div>
</template>
<script>
  import LineChart from '@/components/Charts/LineChart';
  import * as chartConfigs from '@/components/Charts/config';
  import TaskList from './Dashboard/TaskList';
  import UserTable from './Dashboard/UserTable';
  import BaseTable from '@/components/BaseTable';
  import config from '@/config';

  export default {
    components: {
      LineChart,
      TaskList,
      BaseTable,
    },
    data() {
      return {
        bigLineChart: {
          allData: [
            [],
            [],
          ],
          activeIndex: 0,
          chartData: null,
          extraOptions: {
            spanGaps: true,
            //responsive: true,
            maintainAspectRatio: false,
            scales: {
              x: {
                type: 'time',
                parsing: true,
              },
            }
          },
          gradientColors: config.colors.primaryGradient,
          gradientStops: [1, 0.4, 0],
          categories: [],
          loaded: false,
        },
        tableLol: {
          columns: ["id", "name", "pts"],
          tableData: [],
          loaded: false,
        },
        tableKek: {
          columns: ["id", "name", "pts"],
          tableData: [],
          loaded: false,
        },
        user_data: null,
        teams_data: null,
      }
    },
    computed: {
      enableRTL() {
        return this.$route.query.enableRTL;
      },
      isRTL() {
        return this.$rtl.isRTL;
      },
      bigLineChartCategories() {
        return this.$t('dashboard.chartCategories');
      }
    },
    methods: {
      initBigChart(index) {
        console.log(index);
        console.log(this.bigLineChart.allData);
        let chartData = {
          datasets: this.bigLineChart.allData[index]
        }
        //this.$refs.bigChart.updateGradients(chartData);
        this.bigLineChart.chartData = chartData;
        this.bigLineChart.activeIndex = index;
      }
    },
    async mounted() {
      console.log(this.bigLineChart.activeIndex);
      this.bigLineChart.loaded = false;
      this.tableLol.loaded = false;
      let resp = await fetch("http://192.168.14.36:8000/api/v1/scoreboard/users");
      let data = await resp.json();

      for (let usr = 0; usr < data.length; usr++) {
        for (let i = 1; i < data[usr].data.length; i++) {
          data[usr].data[i].y += data[usr].data[i - 1].y;
        }
      }

      data = data.filter(c => c.data.length != 0);

      this.user_data = data.sort(function (a, b) {
        return b.data[b.data.length - 1].y - a.data[a.data.length - 1].y
      }).map((c) => ({
        id: 0,
        name: c.label,
        pts: c.data[c.data.length - 1].y
      }));

      console.log(this.user_data[0].pts);

      let data_graph = data.map((c) => (
        {
          label: c.label,
          data: c.data,
          fill: true,
          borderColor: config.colors.primary,
          borderWidth: 2,
          borderDash: [],
          borderDashOffset: 0.0,
          pointBackgroundColor: config.colors.primary,
          pointBorderColor: 'rgba(255,255,255,0)',
          pointHoverBackgroundColor: config.colors.primary,
          pointBorderWidth: 20,
          pointHoverRadius: 4,
          pointHoverBorderWidth: 15,
          pointRadius: 4,
        }
      ));


      this.bigLineChart.allData[0] = data_graph;

      resp = await fetch("http://192.168.14.36:8000/api/v1/scoreboard/teams"); //CHANGE
      data = await resp.json();

      data = JSON.parse('[{"label":"inverse-fencing","data":[{"x":"2023-08-07T20:42:23","y":100.0}]},{"label":"courtly-repository","data":[]},{"label":"extended-focus","data":[]},{"label":"ILoveYou!","data":[{"x":"2023-08-07T20:43:03","y":50.0},{"x":"2023-08-07T21:42:03","y":200.0}]}]');


      for (let usr = 0; usr < data.length; usr++) {
        for (let i = 1; i < data[usr].data.length; i++) {
          data[usr].data[i].y += data[usr].data[i - 1].y;
        }
      }

      data = data.filter(c => c.data.length != 0);

      this.teams_data = data.sort(function (a, b) {
        return b.data[b.data.length - 1].y - a.data[a.data.length - 1].y
      }).map((c) => ({
        id: 0,
        name: c.label,
        pts: c.data[c.data.length - 1].y
      }));

      data_graph = data.filter(c => c.data.length != 0).map((c) => (
        {
          label: c.label,
          data: c.data,
          fill: true,
          borderColor: config.colors.primary,
          borderWidth: 2,
          borderDash: [],
          borderDashOffset: 0.0,
          pointBackgroundColor: config.colors.primary,
          pointBorderColor: 'rgba(255,255,255,0)',
          pointHoverBackgroundColor: config.colors.primary,
          pointBorderWidth: 20,
          pointHoverRadius: 4,
          pointHoverBorderWidth: 15,
          pointRadius: 4,
        }
      ));

      console.log(data_graph);

      this.bigLineChart.allData[1] = data_graph;

      this.i18n = this.$i18n;
      if (this.enableRTL) {
        this.i18n.locale = 'ar';
        this.$rtl.enableRTL();
      }
      this.initBigChart(0, data);
      this.bigLineChart.loaded = true;

      this.tableLol.tableData = []

      this.tableLol.tableData = this.user_data;
      for (let i = 0; i < this.user_data.length; i++) {
        this.tableLol.tableData[i].id = i + 1;
      }

      this.tableKek.tableData = []

      this.tableKek.tableData = this.teams_data;
      for (let i = 0; i < this.teams_data.length; i++) {
        this.tableKek.tableData[i].id = i + 1;
      }

      this.tableLol.loaded = true;
    },
    beforeDestroy() {
      if (this.$rtl.isRTL) {
        this.i18n.locale = 'en';
        this.$rtl.disableRTL();
      }
    }
  };
</script>
<style>
</style>
