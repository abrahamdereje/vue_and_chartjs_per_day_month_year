<template>
<div class="content">
    <div class="container">
      <div class="Search__container">
        <input
          v-model="packagee"
          class="Search__input"
          @keyup.enter="requestData"
          placeholder="npm package name"
          type="search" name="search"
        >
        <button class="Search__button" @click="requestData">Find</button>

      </div>
      <div class="Search__settings" >
        <datepicker input-class="Search__input" placeholder="Start Date" v-model="periodStart" name="start-date" v-on:selected="validateDataRequest()"></datepicker>
        <datepicker input-class="Search__input" placeholder="End Date" v-model="periodEnd" name="end-date" v-on:selected="validateDataRequest()"></datepicker>
      </div>

      <div class="error-message" v-if="showError">
       {{ errorMessage }}
      </div>
      <hr>
      <div v-if="loading" class="loading">
        🔧  Building Charts ...
        <div class="sk-cube-grid">
        <div class="sk-cube sk-cube1"></div>
        <div class="sk-cube sk-cube2"></div>
        <div class="sk-cube sk-cube3"></div>
        <div class="sk-cube sk-cube4"></div>
        <div class="sk-cube sk-cube5"></div>
        <div class="sk-cube sk-cube6"></div>
        <div class="sk-cube sk-cube7"></div>
        <div class="sk-cube sk-cube8"></div>
        <div class="sk-cube sk-cube9"></div>
      </div>
      </div>

    <div class="Chart__container" v-if="loaded">
        <div class="Chart__title">
          <h2>Downloads per Day <span>{{ formattedPeriod }}</span></h2>

        </div>
        <hr>
        <div class="Chart__content">
          <line-chart chart-id="line-daily" v-if="loaded" :chart-data="downloads" :chart-labels="labels" />
        </div>
      </div>

      <div class="Chart__container" v-if="loaded">
        <div class="Chart__title">
          <h2>Downloads per Week <span>{{ formattedPeriod }}</span></h2>

        </div>
        <hr>

        <div class="Chart__content">
          <line-chart chart-id="line-weekly" v-if="loaded" :chart-data="downloadsWeek" :chart-labels="labelsWeek" />
        </div>
      </div>

      <div class="Chart__container" v-if="loaded">
        <div class="Chart__title">
          <h2>Downloads per Month <span>{{ formattedPeriod }}</span></h2>

        </div>
        <hr>
        <div class="Chart__content">
          <line-chart v-if="loaded" chart-id="line-monthly" :chart-data="downloadsMonth" :chart-labels="labelsMonth" />
        </div>
      </div>

      <div class="Chart__container" v-if="loaded">
        <div class="Chart__title">
          <h2>Downloads per Year <span>{{ formattedPeriod }}</span></h2>

        </div>
        <hr>
        <div class="Chart__content">
          <line-chart v-if="loaded" chart-id="line-yearly" :chart-data="downloadsYear" :chart-labels="labelsYear" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src

import axios from 'axios'


  import LineChart from '@/components/LineChart'


  import {
    dateToYear,
    dateToMonth,
    dateToWeek,
    dateToDay,
    dateBeautify
  } from '../utils/dateFormatter'
import Datepicker from 'vuejs-datepicker'
import { removeDuplicate, groupData } from '../utils/downloadFormatter.js'
export default {
  name: 'HomeView',

    components: {
      LineChart,
      Datepicker,

    },
    props: {},
    data() {
      return {
        packagee: null,
        packageName: '',
        loaded: false,
        loading: false,
        downloads: [],
        downloadsYear: [],
        downloadsMonth: [],
        downloadsWeek: [],
        labels: [],
        labelsYear: [],
        labelsMonth: [],
        labelsWeek: [],
        showError: false,
        showSettings: false,
        errorMessage: 'Please enter a package name',
        periodStart: '',
        periodEnd: new Date(),
        rawData: '',
        totalDownloads: ''
      }
    },
    mounted () {
      if (this.$route.params.packagee) {
        this.packagee = this.$route.params.packagee
        this.requestData()
      }
    },
    computed: {
      _endDate () {
        return dateToDay(this.periodEnd)
      },
      _startDate () {
        return dateToDay(this.periodStart)
      },
      period () {
        return this.periodStart ? `${this._startDate}:${this._endDate}` : 'last-month'
      },
      formattedPeriod () {
        return this.periodStart ? `${dateBeautify(this._startDate)} - ${dateBeautify(this._endDate)}` : 'last-month'
      }
    },
    methods: {
      resetState () {
        this.loaded = false
        this.showError = false
      },
      requestData () {
        if (this.packagee === null || this.packagee === '' || this.packagee === 'undefined') {
          this.showError = true
          return
        }
        this.resetState()
        this.loading = true
        axios.get(`https://api.npmjs.org/downloads/range/${this.period}/${this.packagee}`)
          .then(response => {
            this.rawData = response.data.downloads
            this.downloads = response.data.downloads.map(entry => entry.downloads)
            this.labels = response.data.downloads.map(entry => entry.day)
            this.packageName = response.data.packagee
            this.totalDownloads = this.downloads.reduce((total, download) => total + download)
            this.setURL()
            this.groupDataByDate()
            this.loaded = true
            this.loading = false
          })
          .catch(err => {
            this.errorMessage = err.response.data.error
            this.showError = true
          })
      },
      validateDataRequest () {
        console.log('ValidateData')
        if (this.packageName !== '' && this.periodStart !== '') {
          this.requestData()
        }
      },
      groupDataByDate () {
        this.formatYear()
        this.formatMonth()
        this.formatWeek()
      },
      formatYear () {
        this.labelsYear = this.rawData
          .map(entry => dateToYear(entry.day))
          .reduce(removeDuplicate, [])
           this.downloadsYear = groupData(this.rawData, dateToYear)
      },
      formatMonth () {
        this.labelsMonth = this.rawData
          .map(entry => dateToMonth(entry.day))
          .reduce(removeDuplicate, [])
        this.downloadsMonth = groupData(this.rawData, dateToMonth)
      },
      formatWeek () {
        this.labelsWeek = this.rawData
          .map(entry => dateToWeek(entry.day))
          .reduce(removeDuplicate, [])
        this.downloadsWeek = groupData(this.rawData, dateToWeek)
      },
      setURL () {
        history.pushState({ info: `npm-stats ${this.packagee}` }, this.packagee, `/#/${this.packagee}`)

      }
    }

  }

</script>