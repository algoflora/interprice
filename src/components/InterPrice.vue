<template>
  <div class="flex flex-col font-sans mx-auto w-10/12">
    <div class="mx-8 flex flex-row items-center">
      <buttons-group @change="currencies = $event" :values="currencies" class="mr-8"/>
      <buttons-group @change="selectedYears = $event" :values="selectedYears" multiple class="mr-8"/>
      <buttons-group @change="displays = $event" :values="displays" class=""/>
    </div>
    <div class="mx-8 my-5 w-1/3 flex flex-row justify-start items-center">
      <input placeholder="Filter by company name..." v-model="filterString"
             class="w-full p-1 text-lg outline-none rounded border border-blue-500"/>
    </div>
    <div class="mx-5">
      <table class="w-full tracking-tighter leading-8">
        <thead>
        <tr>
          <th colspan="3"></th>
          <th v-for="year in tableYears"
              :key="year"
              colspan="2"
              class="border-b border-gray-400 w-max m-2"
              style="border-left: white 10px solid; border-right: white 10px solid;"
          >{{ year }}
          </th>
        </tr>
        <tr class="text-gray-400">
          <th class="w-8 border-b-2 border-gray-400"></th>
          <th class="w-32 border-b-2 border-gray-400">
            <div class="cursor-pointer flex flex-row items-center justify-start" @click="changeSort('DateSent')">
              <span>DATE SENT</span>
              <span class="text-black ml-1"
                    :class="{'text-opacity-25': sort.field !== 'DateSent'}"
              >{{ sort.order.DateSent > 0 ? '▼' : '	▲' }}</span>
            </div>
          </th>
          <th class="w-auto border-b-2 border-gray-400">
            <div class="cursor-pointer flex flex-row items-center justify-start" @click="changeSort('Company')">
              <span>COMPANY</span>
              <span class="text-black ml-1"
                    :class="{'text-opacity-25': sort.field !== 'Company'}"
              >{{ sort.order.Company > 0 ? '▼' : '	▲' }}</span>
            </div>
          </th>
          <template v-for="year in tableYears">
            <th v-for="couponType in $options.couponTypesList" :key="year + couponType"
                class="w-16 border-b-2 border-gray-400 tracking-tight">{{ couponType }}
            </th>
          </template>
        </tr>
        </thead>
        <tbody>
        <template v-for="(company, companyIndex) in sortedCompanies">
          <tr :key="'row'+display+company.Company">
            <td :class="{'border-b border-gray-200': companyIndex !== sortedCompanies.length - 1}">
              <template v-if="company.Quote !== null">
                <div v-if="companiesExpanded[company.Id]" @click="$set(companiesExpanded, company.Id, false)"
                     class="cursor-pointer">
                  <svg class="w-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
                       stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/>
                  </svg>
                </div>
                <div v-else @click="$set(companiesExpanded, company.Id, true)" class="cursor-pointer">
                  <svg class="w-4" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24"
                       stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"/>
                  </svg>
                </div>
              </template>
            </td>
            <td class="text-left" :class="{'border-b border-gray-200': companyIndex !== sortedCompanies.length - 1}">
              <template v-if="company.Quote !== null">
                <span>{{ formatDate(company.DateSent) }}</span>
              </template>
            </td>
            <td class="text-left" :class="{'border-b border-gray-200': companyIndex !== sortedCompanies.length - 1}">
              <span :class="company.Quote != null ? 'font-semibold' : 'font-light'">{{ company.Company }}</span>
            </td>
            <template v-for="year in tableYears">
              <template v-for="couponType in $options.couponTypesList">
                <td :key="year+couponType"
                    :class="{
                  'bg-yellow-100': isCellMinimum(company.Id, year, couponType),
                  'border-b border-gray-200': companyIndex !== sortedCompanies.length - 1
                }">
                  <template v-if="company.Quote !== null">
                    <div
                        :id="year+couponType"
                        class="mx-1 w-auto h-full flex items-center justify-center"
                    >
                      <span class="text-sm">{{ getCellData(company.Id, year, couponType) }}</span>
                    </div>
                  </template>
                </td>
              </template>
            </template>
          </tr>
          <template v-if="companiesExpanded[company.Id]">
            <tr v-for="additionalDisplay in $options.displaysList.filter(d => d !== display)"
                :key="'row'+additionalDisplay+company.Company">
              <td class="border-b border-gray-200"></td>
              <td class="border-b border-gray-200"></td>
              <td class="text-left border-b border-gray-200">{{ additionalDisplay }}</td>
              <template v-for="year in tableYears">
                <template v-for="couponType in $options.couponTypesList">
                  <td :key="year+couponType" class="border-b border-gray-200">
                    <div class="w-full h-full flex items-center justify-center">
                      <span class="text-sm">{{ getCellData(company.Id, year, couponType, additionalDisplay) }}</span>
                    </div>
                  </td>
                </template>
              </template>
            </tr>
          </template>
        </template>
        <tr>
          <td class="border-l border-t border-b border-gray-400"></td>
          <td class="border-t border-b border-gray-400"></td>
          <td class="text-left border-t border-b border-gray-400">Average by {{ display }}</td>
          <template v-for="year in tableYears">
            <template v-for="couponType in $options.couponTypesList">
              <td class="border-t border-b last:border-r border-gray-400" :key="year+couponType">
                <div class="w-full h-full flex items-center justify-center">
                  <span class="text-sm">{{ getAverage(year, couponType) }}</span>
                </div>
              </td>
            </template>
          </template>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import ButtonsGroup from '@/components/InterPrice/ButtonsGroup'
import moment from 'moment'
import {mean, min} from 'mathjs'

export default {
  name: 'InterPrice',
  data() {
    return {
      items: [],
      currencies: {},
      selectedYears: {},
      displays: {},
      companiesExpanded: {},
      filterString: '',
      sort: {field: 'DateSent', order: {DateSent: -1, Company: -1}}
    }
  },
  props: {
    data: Object
  },
  computed: {
    quotes() {
      return this.items.map(item => item.Quote).filter(quote => quote !== null)
    },
    currenciesList() {
      return [...new Set(
          this.quotes.flatMap(quote => quote.map(quoteItem => quoteItem.Currency))
      )]
    },
    currency() {
      return Object.keys(this.currencies).find(curr => this.currencies[curr])
    },
    display() {
      return Object.keys(this.displays).find(displ => this.displays[displ])
    },
    tableYears() {
      return this.getYears().filter(year => this.selectedYears[year])
    },
    sortedCompanies() {
      const sorter = (a, b) => {
        if (a.Quote === null && b.Quote !== null) return 1
        if (a.Quote !== null && b.Quote === null) return -1

        const [aVal, bVal] = [a, b].map(x => x[this.sort.field])
        const order = this.sort.order[this.sort.field]

        if (aVal < bVal) return -order
        if (aVal > bVal) return order

        return a.Preferred > b.Preferred ? order : 0
      }

      return Array.from(this.companies).sort(sorter)
          .concat(Array.from(this.nullCompanies).sort(sorter))
    },
    filteredCompanies() {
      return this.items.filter(item => item.Company.toLowerCase().includes(this.filterString.toLowerCase()))
    },
    companies() {
      return this.filteredCompanies.filter(company => company.Quote !== null)
    },
    nullCompanies() {
      return this.filteredCompanies.filter(company => company.Quote === null)
    },
    tableData() {
      const variants = this.companies.map(company => ({company}))
          .flatMap(variant =>
              this.currenciesList.map(currency => Object.assign({currency}, variant)))
          .flatMap(variant =>
              this.getYears().map(year => Object.assign({year}, variant)))
          .flatMap(variant =>
              this.$options.couponTypesList.map(couponType => Object.assign({couponType}, variant)))
          .flatMap(variant =>
              this.$options.displaysList.map(display => Object.assign({display}, variant)))

      const groupBy = (vs, f1, f2) =>
          vs.reduce((acc, v) => {
            const k = f1(v);
            (acc[k] = acc[k] || []).push(f2(v))
            return acc
          }, {})

      const byColumnsAndDisplays = groupBy(
          variants,
          v => [v.currency, v.year, v.couponType, v.display].join('.'),
          v => {
            const values = v.company.Quote.find(quoteItem =>
                quoteItem.CouponType === v.couponType
                && quoteItem.Years === parseInt(v.year)
                && quoteItem.Currency === v.currency);

            return [v.company.Id, (values && values[v.display]) || null]
          }
      )

      Object.keys(byColumnsAndDisplays).forEach(key => {
        const byColumnAndDisplay = byColumnsAndDisplays[key]

        const values = byColumnAndDisplay.map(x => x[1]).filter(x => x !== null)
        const [minimum, average] = values.length ? [min(values), mean(values)] : [null, null]

        const byCompany = Object.fromEntries(byColumnAndDisplay)

        byColumnsAndDisplays[key] = { minimum, average, byCompany }
      })

      return (byColumnsAndDisplays)
    }
  },
  methods: {
    calculateData(data) {
      this.items = data.Items
      this.currencies = Object.fromEntries(this.currenciesList.map(currency => [currency, currency === 'USD']))
      this.displays = Object.fromEntries(this.$options.displaysList.map(display => [display, display === 'Spread']))
      this.companiesExpanded = Object.fromEntries(this.items.map(item => [item.Id, false]))
    },
    getYears() {
      const years = [...new Set(
          this.quotes
              .flatMap(quote => quote
                  .map(quoteItem => quoteItem.Years)
                  .filter(year => quote.some(q => q.Currency === this.currency && q.Years === year))
              )
      )].sort((a, b) => a - b)

      if (JSON.stringify(Object.keys(this.selectedYears)) !== JSON.stringify(years.map(y => y.toString()))) {
        this.selectedYears = Object.fromEntries(years.map(year => [year.toString(), true]))
      }
      return years
    },
    changeSort(field) {
      if (this.sort.field === field) {
        this.sort.order[this.sort.field] *= -1
      } else {
        this.sort.field = field
      }
    },
    formatDate(date) {
      return moment(date).format('DD-MMM-YY')
    },
    formatCellValue(val, display = this.display) {
      if (val === undefined || val === null) {
        return null
      }
      return display === 'Yield' ?
          val.toFixed(3) + '%' :
          (val > 0 ? '+' : '') + val.toFixed(0) + 'bp'
    },
    getCellDataSummarized(year, couponType, display = this.display) {
      const key = [this.currency, year, couponType, display].join('.')
      return  this.tableData[key]
    },
    getCellData(companyId, year, couponType, display = this.display) {
      const value = this.getCellDataSummarized(year, couponType, display).byCompany[companyId]
      return this.formatCellValue(value, display)
    },
    isCellMinimum(companyId, year, couponType, display = this.display) {
      if (couponType !== 'FIX') {
        return false
      }
      const summarized = this.getCellDataSummarized(year, couponType, display)
      return summarized.minimum !== null && summarized.minimum === summarized.byCompany[companyId]
    },
    getAverage(year, couponType) {
      const averageValue = this.getCellDataSummarized(year, couponType).average
      return this.formatCellValue(averageValue)
    }
  },
  mounted() {
    this.calculateData(this.data)
  }
  ,
  watch: {
    data(newData) {
      this.calculateData(newData)
    }
  }
  ,
  displaysList: ['Spread', 'Yield', '3MLSpread'],
  couponTypesList: ['FIX', 'FRN'],
  components: {
    ButtonsGroup
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
</style>
