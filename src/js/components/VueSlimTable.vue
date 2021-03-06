<template>
  <div :class="wrapperClass">
    <table :class="tableClass">
      <thead>
        <tr>
          <th v-for="col in columns">{{ col.title }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in rows">
          <td v-for="col in columns">
            <component v-if="col.component" :is="col.component" :params="row" :value="row[col.key]" />
            <template v-else>{{row[col.key]}}</template>
          </td>
        </tr>

        <tr v-if="syncState == 'fetched' && rows.length === 0">
          <td :colspan="columns.length" class="text-center " :class="emptyRowsClass">
            {{ emptyRowsText }}
          </td>
        </tr>

        <template v-if="syncState == 'syncing'">
          <tr v-for="row in perPage">
            <td :colspan="columns.length" class="vue-slim-tables-loading-row">
              <div></div>
            </td>
          </tr>
        </template>
      </tbody>
    </table>
    <paginate
      v-if="rowsTotalCount > 1"
      v-model="page"
      :page-count="rowsTotalCount"
      container-class="pagination mt-3"
      page-class="page-item"
      page-link-class="page-link"
      prev-class="page-item"
      prev-link-class="page-link"
      next-class="page-item"
      next-link-class="page-link"
      prev-text="<-"
      next-text="->"
    />
  </div>
</template>

<script>
  import qs from 'qs'
  import VuejsPaginate from 'vuejs-paginate'

  export default {
    props: {
      columns: { type: Array, required: true },
      remoteUrl: String,
      perPage: { type: Number, default: 25 },
      customFilters: { type: Object, default() { return {} } },
      onFetchedCallback: Function,
      wrapperClass: String,
      tableClass: String,
      emptyRowsClass: String,
      emptyRowsText: { type: String, default: 'No records found' },
      totalRowsCountKey: { type: String, default: 'totalCount' }
    },
    data() {
      return {
        page: 1,
        rows: [],
        rowsTotalCount: 0,
        syncState: 'initial'
      }
    },
    methods: {
      fetchData(page) {
        const params = qs.stringify({
          per_page: this.perPage,
          page: page,
          ...this.customFilters
        }, { arrayFormat: 'brackets' })

        this.syncState = 'syncing'
        this.rows = []

        fetch(`${this.remoteUrl}?${params}`)
          .then(res => res.json())
          .then(res => {
            this.rows = res.rows
            this.rowsTotalCount = Math.ceil(res[this.totalRowsCountKey] / this.perPage)
            this.onFetchedCallback && this.onFetchedCallback(res)
            this.syncState = 'fetched'
          })
      },
      reload() {
        this.fetchData(this.page)
      }
    },
    watch: {
      page(val) {
        this.fetchData(val)
      },
      customFilters() {
        if (this.page === 1) {
          this.fetchData(1)
        } else {
          this.page = 1
        }
      }
    },
    components: { paginate: VuejsPaginate },
    mounted() {
      this.fetchData(this.page)
    }
  }
</script>
