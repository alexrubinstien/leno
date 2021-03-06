<template>
  <div class="data-visualisation-tab dashboard-tab">
    <div class="row">
      <div class="col-md-6">
        <div class="chart-container" v-if="isLoaded">
          <vuestic-chart
            :options="options"
            v-bind:data="donutChartData"
            type="donut"
          ></vuestic-chart>
        </div>
      </div>
      <vuestic-pre-loader v-show="!isLoaded" class="pre-loader"></vuestic-pre-loader>
      <div class="col-md-6" v-if="isLoaded">
        <vuestic-data-table
          :apiMode="apiMode"
          :tableData="table.datas"
          :tableFields="table.fields"
          :onEachSide="onEachSide"
          :sortFunctions="table.sortFunctions"
          :perPageSelectorShown="false"
          :dataModeFilterableFields="dataModeFilterableFields"
          @onRowClicked="onRowClicked"
        />
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import BadgeColumn from 'components/users/BadgeColumn.vue'
import TableDataInfo from '@/helpers/TableDataInfo'
import Proxy from '@/proxies/Proxy'

Vue.component('badge-column', BadgeColumn)

export default {
  name: 'data-visualisation-tab',

  created () {
    this.initalization()
  },
  props: {
    endpoint: {
      type: String,
      require: true
    },
    type: {
      type: String,
      require: true
    },
  },
  data () {
    return {
      isLoaded: false,
      donutChartData: {
        labels: [],
        datasets: [{
          label: 'Population (millions)',
          backgroundColor: [],
          data: []
        }]
      },
      apiMode: false,
      table: {
        datas: {
          data: {}
        },
        fields: [],
        sortFunctions: {}
      },
      onEachSide: 1,
      dataModeFilterableFields: ['name'],
      itemsPerPage: [
        {
          value: 5
        },
        {
          value: 6
        }
      ],
      options: {},
      result: []
    }
  },
  methods: {
    async initalization () {
      const {providerId, providerAccessToken} = this.$store.getters['auth/provider']
      try {
        const {success, error, ...data} = await new Proxy(this.endpoint).submit('post', {
          providerId,
          providerAccessToken,
          ...this.parameters
        })

        if (success && data) {
          this.result = Object.values(data).pop()
          this.table = new TableDataInfo(this.result)
          this.drawChart(this.result)
          console.log('this.table', this.table)
        } else {
          this.tableData = []
          this.showToast(error)
        }
        this.isLoaded = true
      } catch (error) {
        console.log('empty table')
        this.isLoaded = true
      }
    },
    createTable (data) {
      if (data) {
        this.table = new TableDataInfo(data)
      }
    },
    drawChart (data) {
      const palette = this.$store.getters['shared/palette']
      const colorType = [palette.info, palette.warning, palette.primary, palette.fontColor, palette.success, palette.danger]
      this.donutChartData.labels = data.map(ele => {
        return ele[this.type]
      })
      let emptyFlg = true
      this.donutChartData.datasets[0].data = data.map(ele => {
        if (ele.percentage !== 0) emptyFlg = false
        return ele.percentage
      })
      this.donutChartData.datasets[0].backgroundColor = Object.keys(data).map((ele, index) => {
        return colorType[index % 7]
      })

      if (emptyFlg) {
        this.donutChartData.datasets[0].data = data.map((ele, index) => {
          if (index === 0) return 1
          else return 0
        })

        this.donutChartData.datasets[0].backgroundColor = Object.keys(data).map((ele, index) => {
          if (index === 0) return '#5A79EE'
          else { return colorType[index % 7] }
        })

        this.options = {
          tooltips: {
            enabled: false
          },
        }
      }
    },
    onRowClicked (e) {
      if (this.type !== 'employer') {
        const selectedClass = this.result.filter(ele => e.row === ele.row).pop()
        this.$router.push(`dashboard/class/${selectedClass.classDescriptionId}`)
      }
    },
    showToast () {
      this.$store.dispatch('auth/notification', {
        type: 'ERROR',
        title: 'ClassesVisualisation',
        message: 'Oops, Please try again later.'
      })
    }
  },
}
</script>

<style lang="scss" scoped>
@import '../../../sass/_variables.scss';
@import '~bootstrap/scss/functions';
@import '~bootstrap/scss/variables';
@import '~bootstrap/scss/mixins/breakpoints';

.chart-container {
  padding: 0 2rem;
  height: 24rem;
}
</style>
