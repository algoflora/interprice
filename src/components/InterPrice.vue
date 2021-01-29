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
              class="border-b border-gray-400 w-max m-2" style="border-left: white 10px solid; border-right: white 10px solid;"
          >{{ year }}</th>
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
            <th v-for="couponType in couponTypesList" :key="year + couponType" class="w-16 border-b-2 border-gray-400">{{ couponType }}</th>
          </template>
        </tr>
        </thead>
        <tbody>
        <template v-for="(company, companyIndex) in sortedTableData">
          <tr :key="'row'+display+company.Company">
            <td :class="{'border-b border-gray-200': companyIndex !== sortedTableData.length - 1}">
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
            <td class="text-left" :class="{'border-b border-gray-200': companyIndex !== sortedTableData.length - 1}">
              <template v-if="company.Quote !== null">
                <span>{{ formatDate(company.DateSent) }}</span>
              </template>
            </td>
            <td class="text-left" :class="{'border-b border-gray-200': companyIndex !== sortedTableData.length - 1}">
              <span :class="company.Quote != null ? 'font-semibold' : 'font-light'">{{ company.Company }}</span>
            </td>
            <template v-for="year in tableYears">
              <template v-for="couponType in couponTypesList">
                <td :key="year+couponType"
                    :class="{
                  'bg-yellow-100': getCellData(company, year, couponType).isMinimum,
                  'border-b border-gray-200': companyIndex !== sortedTableData.length - 1
                }">
                  <template v-if="company.Quote !== null">
                    <div
                        :id="year+couponType"
                        class="mx-1 w-auto h-full flex items-center justify-center"
                    >
                      <span class="text-sm">{{ getCellData(company, year, couponType).formatted }}</span>
                    </div>
                  </template>
                </td>
              </template>
            </template>
          </tr>
          <template v-if="companiesExpanded[company.Id]">
            <tr v-for="additionalDisplay in $options.displaysList.filter(d => d !== display)"
                :key="'row'+additionalDisplay+company.Company">
              <td></td>
              <td></td>
              <td class="text-left">{{ additionalDisplay }}</td>
              <template v-for="year in tableYears">
                <template v-for="couponType in couponTypesList">
                  <td :key="year+couponType">
                    <div class="w-full h-full flex items-center justify-center">
                      <span class="text-sm">{{ getCellData(company, year, couponType, additionalDisplay).formatted }}</span>
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
            <template v-for="couponType in couponTypesList">
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
import {largerEq, mean} from 'mathjs'

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
    couponTypesList() {
      return [...new Set(
          this.quotes.flatMap(quote => quote.map(quoteItem => quoteItem.CouponType))
      )].sort()
    },
    sortedTableData() {
      return Array.from(this.tableData).sort((a, b) => {
        if (a.Quote === null && b.Quote !== null) return 1
        if (a.Quote !== null && b.Quote === null) return -1

        const [aVal, bVal] = [a, b].map(x => x[this.sort.field])
        const order = this.sort.order[this.sort.field]

        if (aVal < bVal) return -order
        if (aVal > bVal) return order

        return a.Preferred > b.Preferred ? order : 0
      })
    },
    tableData() {
      return this.items
          .filter(item => item.Company.toLowerCase().includes(this.filterString.toLowerCase()))
          .map(item => {
                const quote = item.Quote || []
                const byCurrency = Object.fromEntries(
                    this.currenciesList.map(currency => {
                      const byYear = Object.fromEntries(
                          this.getYears().map(year => {
                            const byCouponType = Object.fromEntries(
                                this.couponTypesList.map(couponType => {
                                  const value = quote.find(quoteItem =>
                                      quoteItem.CouponType === couponType
                                      && quoteItem.Years === parseInt(year)
                                      && quoteItem.Currency === currency
                                  )
                                  const byDisplay = Object.fromEntries(
                                      this.$options.displaysList.map(display => {
                                        if (value !== undefined && value[display] !== null) {
                                          const values = this.items
                                              .filter(it => it.Id !== item.Id && it.Quote !== null)
                                              .flatMap(it =>
                                                  it.Quote
                                                      .filter(quoteItem =>
                                                          quoteItem.CouponType === couponType
                                                          && quoteItem.Years === parseInt(year)
                                                          && quoteItem.Currency === currency
                                                      )
                                                      .map(item => item[display])
                                              )
                                              .filter(n => n !== null)

                                          const formatted = this.formatVal(value[display], display)
                                          const isMinimum = couponType === 'FIX' ?
                                              values.every(n => largerEq(n, value[display])) : false

                                          return [display, {value: value[display], formatted: formatted, isMinimum: isMinimum}]
                                        } else {
                                          return [display, {value: null, formatted: '', isMinimum: false}]
                                        }
                                      })
                                  )

                                  return [couponType, byDisplay]
                                })
                            )
                            return [year, byCouponType]
                          })
                      )
                      return [currency, byYear]
                    })
                )
                return Object.assign(item, byCurrency)
              }
          )
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
    formatDate(dateSent) {
      return moment(dateSent).format('DD-MMM-YY')
    },
    formatVal(val, display = this.display) {
      return display === 'Yield' ?
          val.toFixed(3) + '%' :
          (val > 0 ? '+' : '') + val.toFixed(0) + 'bp'
    },
    getCellData(company, year, couponType, display = this.display) {
      return company[this.currency][year]
          && company[this.currency][year][couponType]
          && company[this.currency][year][couponType][display]
    },
    getAverage(year, couponType) {
      const values = this.sortedTableData
          .map(company => this.getCellData(company, year, couponType).value)
          .filter(n => n !== null)
      if (values.length > 0) {
        const average = mean(values)
        return this.formatVal(average)
      } else {
        return null
      }
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
  components: {
    ButtonsGroup
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
</style>
