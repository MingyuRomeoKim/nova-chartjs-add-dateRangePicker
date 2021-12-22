<template>
    <card class="p-10">
      <div class="stay-right">
        <date-picker v-show="buttonDatePick" @pick="handleDatePick" v-model="datePicker" range="true"></date-picker>
        <a @click="fillData()" class="btn-refresh" v-show="buttonRefresh">
          <i class="fas fa-sync"></i>
        </a>
        <a @click="reloadPage()" class="btn-refresh" v-show="buttonReload">
          <i class="fas fa-sync"></i>
        </a>
        <a :href="externalLink" :target="externalLinkIn" class="btn-external" v-show="btnExtLink">
          <i class="fas fa-external-link-alt"></i>
        </a>
      </div>
      <h4 class="chart-js-dashboard-title">{{ checkTitle }}</h4>
      <line-chart :chart-data="datacollection" :options="options"></line-chart>
    </card>
</template>

<style>
  @import '../../css/main.css';
</style>

<script>
  import DatePicker from 'vue2-datepicker';
  import 'vue2-datepicker/index.css';
  import LineChart from '../scatter-chart.js'

  export default {
    components: {
      LineChart,
      DatePicker
    },
    data () {
      this.card.options = this.card.options != undefined ? this.card.options : false;
      return {
        datacollection: {},
        options: {},
        buttonDatePick: this.card.options.btnDatePick,
        buttonRefresh: (this.card.options != undefined) ? this.card.options.btnRefresh : false,
        buttonReload: this.card.options.btnReload,
        btnExtLink: this.card.options.extLink != undefined ? true : false,
        externalLink: this.card.options.extLink,
        externalLinkIn: this.card.options.extLinkIn != undefined ? this.card.options.extLinkIn : '_self',
        chartTooltips: this.card.options.tooltips != undefined ? this.card.options.tooltips : undefined,
        chartLayout: this.card.options.layout != undefined ? this.card.options.layout :
          {
            padding: {
              left: 20,
              right: 20,
              top: 0,
              bottom: 10
            }
          },
        chartLegend: this.card.options.legend != undefined ? this.card.options.legend :
          {
            display: true,
            position: 'left',
            labels: {
              fontColor: '#7c858e',
              fontFamily: "'Nunito'"
            }
          },
        datePicker: new Array(),
      }
    },
    computed: {
      checkTitle() {
        return this.card.title !== undefined ? this.card.title : 'Chart JS Integration';
      }
    },
    props: [
        'card'
    ],
    mounted () {
        let date_filter = document.querySelector('input[name="date-filter"]');
        if(date_filter) {
            date_filter.addEventListener('change',(event)=> {
                if(date_filter.value.indexOf('to') != -1) {
                    this.fillData('dateRange',date_filter.value);
                }
            });
        }
      this.fillData();
    },
    methods: {
        handleDatePick(date){
            if(this.datePicker.length >= 2) {
                let returnDate = this.dateFormat(this.datePicker[0]) + ' to ' + this.dateFormat(this.datePicker[1]);
                this.fillData('dateRange',returnDate);
            }
        },
        dateFormat(date) {
            let month = date.getMonth() + 1;
            let day = date.getDate();
            let hour = date.getHours();
            let minute = date.getMinutes();
            let second = date.getSeconds();

            month = month >= 10 ? month : '0' + month;
            day = day >= 10 ? day : '0' + day;
            hour = hour >= 10 ? hour : '0' + hour;
            minute = minute >= 10 ? minute : '0' + minute;
            second = second >= 10 ? second : '0' + second;

            return date.getFullYear() + '-' + month + '-' + day + ' ' + hour + ':' + minute + ':' + second;
        },
      reloadPage(){
        window.location.reload()
      },
      fillData (filter_type = '',date_value = '') {
        this.options = {
          layout: this.chartLayout,
          legend: this.chartLegend,
          scales: {
            xAxes: [ {
              type: 'linear',
              position: 'bottom',
              ticks: {
                lineHeight: 0.8,
                fontSize: 10,
              }
            }]
          },
        };

        if(this.chartTooltips !== undefined){
          this.options.tooltips = this.chartTooltips;
          const tooltiplist = ["custom", "itemSort", "filter"];
          var z;
          for (z = 0; z < tooltiplist.length; z++) {
            if(this.options.tooltips[tooltiplist[z]] != undefined){
              if(this.options.tooltips[tooltiplist[z]].search("function") != -1){
                eval("this.options.tooltips." + tooltiplist[z] + " = " + this.options.tooltips[tooltiplist[z]]);
              }
            }
          }

          if(this.chartTooltips.callbacks !== undefined){
            const callbacklist = ["beforeTitle", "title", "afterTitle", "beforeBody", "beforeLabel", "label", "labelColor", "labelTextColor", "afterLabel", "afterBody", "beforeFooter", "footer", "afterFooter"];
            var i;
            for (i = 0; i < callbacklist.length; i++) {
              if(this.options.tooltips.callbacks[callbacklist[i]] != undefined){
                if(this.options.tooltips.callbacks[callbacklist[i]].search("function") != -1){
                  eval("this.options.tooltips.callbacks." + callbacklist[i] + " = " + this.options.tooltips.callbacks[callbacklist[i]]);
                }
              }
            }
          }
        }

        if(this.card.model == 'custom' || this.card.model == undefined){
        // Custom Data
          this.title = this.card.title,
          this.datacollection = {
            datasets: this.card.series,
          }
          this.options = this.options;
        } else {
            let params = {
                model: this.card.model,
                series: this.card.series,
                options: this.card.options,
                join: this.card.join,
                col_xaxis: this.card.col_xaxis,
                expires: 0,
            }
            if(filter_type == 'dateRange') { // Custom
                let filterValue = date_value.split('to');
                if(filterValue.length !== 0) {
                    params = {
                        model: this.card.model,
                        series: this.card.series,
                        options: this.card.options,
                        join: this.card.join,
                        col_xaxis: this.card.col_xaxis,
                        expires: 0,
                        useDateFilter:'yes', // custom data
                        start_date:filterValue[0].trim(),// custom data
                        end_date:filterValue[1].trim(),// custom data
                    }
                }
            }

        // Use Model
          Nova.request().get("/nova-vendor/coroowicaksono/check-data/endpoint/", {
            params: params,
          })
          .then(({ data }) => {
            this.datacollection = {
              labels: data.dataset.xAxis,
              datasets: data.dataset.yAxis,
            };
            this.options = this.options;
          })
          .catch(({ response }) => {
            this.$set(this, "errors", response.data.errors)
          })
        }
      },
    },
  }
</script>
