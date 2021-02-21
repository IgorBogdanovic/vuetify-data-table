<template lang="pug">
  v-container
    v-row
      v-col(cols)
        DataTable(
          v-show="totalItems !== null"
          :headers="headers"
          :items="items"
          :server-items-length="totalItems"
          :initial-options="{ sortBy: ['user'], sortDesc: [false] }"
          :loading="loading"
          @update="getDataFromApi($event)"
        )
        v-progress-circular(
          v-if="totalItems === null"
          width="2"
          color="rs__primary"
          indeterminate
        ).mx-auto
</template>

<script>
import DataTable from '~/components/DataTable.vue'
import sales from '~/api/sales.js'

export default {
  name: 'IndexPage',

  components: {
    DataTable
  },

  data() {
    return {
      totalItems: null,
      items: [],
      loading: false,
      headers: [
        { text: 'Name', value: 'user', align: 'start', width: '24%', formatter: user => `${user.last_name}, ${user.first_name} ${user.title}` },
        { text: 'Email', value: 'email', width: '24%' },
        { text: 'Gender', value: 'gender', width: '10%' },
        { text: 'Year', value: 'year', width: '10%' },
        { text: 'Sales', value: 'sales', width: '15%', formatter: number => new Intl.NumberFormat().format(number) },
        { text: 'Country', value: 'country', width: '17%' }
      ]
    }
  },

  methods: {
    async getDataFromApi(options) {
      this.loading = true
      const data = await this.fetchData(options)
      this.items = data.items
      this.totalItems = data.total
      this.loading = false
    },
    async fetchData(options) {  
      const { sortBy, sortDesc, page, itemsPerPage, search } = options
      let items = sales.results

      if (search) {
        items = this.searchedItems(items, search)
      }
      const total = items.length

      if (sortBy.length && sortDesc.length) {
        items = await this.sortedItems(items, sortBy, sortDesc)
      }
      if (itemsPerPage) {
        items = this.paginatedItems(items, page, itemsPerPage)
      }
      
      await this.delay(2000)
      return { items, total }
    },
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms))
    },
    searchedItems(items, search) {
      return items.filter(item => Object.values(item).join(',').toLowerCase().includes(search))
    },
    async sortedItems(items, sortBy, sortDesc) {
      let sortedItems = [...items]
      const sortByReversed = [...sortBy].reverse()
      const sortDescReversed = [...sortDesc].reverse()
      for (const index in sortByReversed) {
        sortedItems = await this.sortItems(sortedItems, sortByReversed[index], sortDescReversed[index])
      }
      return sortedItems
    },
    async sortItems(items, sortBy, sortDesc) {
      return items.sort((a, b) => {
        let sortA = a[sortBy]
        let sortB = b[sortBy]
        if (sortBy === 'user') {
          sortA = a[sortBy].last_name.toLowerCase()
          sortB = b[sortBy].last_name.toLowerCase()
        }
        if (sortBy === 'sales') {
          sortA = parseInt(a[sortBy])
          sortB = parseInt(b[sortBy])
        }
        return this.orderComparision(sortDesc, sortA, sortB)
      })
    },
    orderComparision(sortDesc, sortA, sortB) {
      if (sortDesc) {
        if (sortA < sortB) return 1
        if (sortA > sortB) return -1
        return 0
      } else {
        if (sortA < sortB) return -1
        if (sortA > sortB) return 1
        return 0
      }
    },
    paginatedItems(items, page, itemsPerPage) {
      return items.slice((page - 1) * itemsPerPage, page * itemsPerPage)
    }
  }
}
</script>

<style lang="sass" scoped>
.v-progress-circular
  position: absolute
  top: 50%
  left: 50%
</style>