<template>
  <div class="home">
    <div class="app-page">

      <div>
        <div class="page-title">
          <h3>{{ 'Bill_Name' | localize }}</h3>

          <button class="btn waves-effect waves-light btn-small" @click="refresh">
            <i class="material-icons">refresh</i>
          </button>
        </div>

        <Loader v-if="loading" />

        <div v-else class="row">

          <HomeBill
            :rates="currency.rates"
          />

          <HomeCurrency
            :rates="currency.rates"
            :date="currency.date"
          />

        </div>
      </div>
    </div>
  </div>
</template>

<script>
import HomeBill from '@/components/HomeBill'
import HomeCurrency from '@/components/HomeCurrency'

export default {
  name: 'Home',
  metaInfo() {
    return {
      title: this.$title('Bill_Name')
    }
  },
  data () {
    return {
      loading: true,
      currency: null
    }
  },
  components: {
    HomeCurrency,
    HomeBill
  },
  methods: {
    async refresh () {
      this.loading = true
      this.currency = await this.$store.dispatch('fetchCurrency')
      this.loading = false
    }
  },
  async mounted () {
    this.currency = await this.$store.dispatch('fetchCurrency')
    this.loading = false
  }
}
</script>
