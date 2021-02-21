<template lang="pug">
  v-data-table(
    :headers="headers"
    :items="items"
    :options.sync="options"
    :server-items-length="serverItemsLength"
    :search="serverItemsLength < 0 ? search : undefined"
    :loading="loading"
    multi-sort
    item-key="email"
    dense
  ).elevation-1.mt-10
    template(#top)
      v-text-field(
        prepend-icon="mdi-magnify"
        label="Search"
        v-model="search"
      ).mx-4
    template(
      v-for="header in headers.filter(header => header.hasOwnProperty('formatter'))"
      #[`item.${header.value}`]="{ header, value }"
    ) {{ header.formatter(value) }}
</template>

<script>
export default {
  props: {
    headers: {
      type: Array,
      required: true
    },
    items: {
      type: Array,
      required: true
    },
    serverItemsLength: {
      type: Number,
      default: -1
    },
    initialOptions: {
      type: Object,
      default: () => {}
    },
    loading: {
      type: Boolean,
      default: false
    }
  },

  data() {
    return {
      search: '',
      options: this.initialOptions,
      delay: null
    }
  },

  watch: {
    search () {
      if (this.serverItemsLength < 0) return
      const doneTyping = () => {
        this.options = { ...this.options, page: 1 }
      }
      this.typingDelay(500, doneTyping)
    },
    options: {
      handler(newVal) {
        this.$emit('update', { ...newVal, search: this.search.trim().toLowerCase() })
      },
      deep: true
    }
  },

  methods: {
    typingDelay(ms, doneTyping) {
      if (this.delay) clearTimeout(this.delay)
      this.delay = setTimeout(() => {
        doneTyping()
        this.delay = null
      }, ms)
    }
  }
}
</script>

<style lang="sass" scoped>
  .v-data-table
    max-width: 100%
    border: 1px solid $rs__grey

  ::v-deep thead.v-data-table-header
    background-color: lighten($rs__lilac, 15%)

  ::v-deep thead.v-data-table-header tr th
    color: $rs__black_light !important
    text-transform: uppercase

  ::v-deep tbody tr:hover
    background-color: lighten($rs__lilac, 30%) !important
</style>
