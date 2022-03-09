<template>
  <div>
    
    <a-row style="text-align:center;">
      <h1 class="title1 color4">Tipo de cambio Peso Mexicano</h1>
    </a-row>
    
    
    <a-row class="spacePadding alignCenter">
      <h1 class="title2">Agregar fecha de consulta</h1>
      <a-date-picker @change="onChange" :disabled-date="disabledDate" />
      <a-button type="primary" v-on:click="getData(nDate)">
        Consultar
      </a-button>
    </a-row>
    
    <vue-c3 :handler="handler"></vue-c3>
    
    <a-row class="spacePadding"> 
      <a-col :sm="24" :md="12" :lg="6" v-for="(item, index) in arrayDate" :key="index*18" v-if="index>0">
        <a-card :title="item | fecha" style="width: 100%" >
          <p v-for="(itemOne, indexOne) in countryCoin" :key="indexOne*2" v-if="indexOne>0"> 
            <strong>{{itemOne[0]}}:</strong> $ {{itemOne[index]}}
          </p>
        </a-card>
      </a-col>
    </a-row>
      

  </div>
        
</template>

<script>
import Vue from 'vue'
import VueC3 from 'vue-c3'
import axios from 'axios'
import 'c3/c3.min.css'
import moment from 'moment';

export default {
  name: 'prueba',
  components: {
    VueC3
  },
  data: () => ({
    handler: new Vue(),
    nDate: '',
    historyRates: [],
    countryCoin: [["MXN"],["USD"],["CAD"],["EUR"],["JPY"],["CNY"]],
    arrayDate: ['x', ],
  }),
  methods: {
    onChange(date, dateString) {
      this.nDate = dateString;
    },
    addDate(dataAdd){
      this.arrayDate.push(dataAdd.date)  
      this.countryCoin.map(function(item) {item.push(dataAdd.rates['MXN']/dataAdd.rates[item[0]])});
      this.handler.$emit('dispatch', chart => {
        const options = {
          columns: [
          this.arrayDate,
          this.countryCoin[1],
          this.countryCoin[2],
          this.countryCoin[3],
          this.countryCoin[4],
          this.countryCoin[5],
        ]
        }
        chart.load(options)
      })
    },
    getData(dateInit){
      axios.get('http://data.fixer.io/api/'+dateInit+'?access_key=35d0bbdb8ffc032abd4b5c75e8bef2e8').then((result) => {
        this.historyRates = result.data
           this.addDate(result.data)
      })
    },
    disabledDate(current) {
      return current && current > moment().endOf('day');
    },
  },
  computed: {
    options() {
      return {
        data: {
          x: 'x',
          columns: [
            this.arrayDate,
          ],
        },
        axis: {
          x: {
            type: 'timeseries',
            text: 'Fecha',
            tick: {
                format: '%d-%m-%Y'
            }
          },
          y: {
            label: {
              text: 'MXN',
              position: 'outer-middle'
            },
            tick: {
              
               format: function (d) { return "$" + d; }
            }
          }
        },
        tooltip: {
          format: {
            title(d) {
              return `Fecha ${moment(d).format("DD-MM-YYYY")}`
            },
            value(value, ratio, id) {
              return `$ ${value}`
            }
          }
        },
        grid: {
          x: {
            show: true
          },
          y: {
            show: true
          }
        },
        
      }
    }
  },
  mounted(current) {
    this.handler.$emit('init', this.options)
    this.getData(moment().format('YYYY-MM-DD'));
  },

  filters:{
    fecha: function(value){
      return moment(value).format('DD-MM-YYYY')

    }
  }
}
</script>
<style lang="scss">@import './scss/main.scss'</style>

