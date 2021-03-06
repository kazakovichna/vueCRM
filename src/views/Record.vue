<template>
  <div class="record">
    <div class="app-page">

      <div>
        <div class="page-title">
          <h3>{{ 'Record_New' | localize }}</h3>
        </div>

        <Loader v-if="loading"/>

        <p class="center" v-else-if="!categories.length">
          {{ 'Record_No_Cat' | localize }}
          <router-link to="/categories">{{ 'Record_Add_Cat' | localize }}</router-link>
        </p>

        <form class="form" v-else @submit.prevent="handleSubmit">
          <div class="input-field" >
            <select ref="select" v-model="category">
              <option
                v-for="c in categories"
                :key="c.id"
                :value="c.id"
              >
                {{c.title}}
              </option>
            </select>
            <label>{{ 'Record_Choose_Cat' | localize }}</label>
          </div>

          <p>
            <label>
              <input
                class="with-gap"
                name="type"
                type="radio"
                value="income"
                v-model="type"
              />
              <span>{{ 'Record_Income' | localize }}</span>
            </label>
          </p>

          <p>
            <label>
              <input
                class="with-gap"
                name="type"
                type="radio"
                value="outcome"
                v-model="type"
              />
              <span>{{ 'Record_Outcome' | localize }}</span>
            </label>
          </p>

          <div class="input-field">
            <input
              id="amount"
              type="number"
              v-model.number="amount"
              :class="{ invalid: ($v.amount.$dirty && !$v.amount.required) || ($v.amount.$dirty && !$v.amount.minValue) }"
            >
            <label for="amount">{{ 'Record_Amount' | localize }}</label>
            <span class="helper-text invalid"
                  v-if="$v.amount.$dirty && !$v.amount.required"
            >{{ 'Record_Add_Amount' | localize }}</span>
            <span class="helper-text invalid"
                  v-if="$v.amount.$dirty && !$v.amount.minValue"
            >{{ 'Record_Amount_Er' | localize }} {{$v.amount.$params.minValue.min}}</span>
          </div>

          <div class="input-field">
            <input
              id="description"
              type="text"
              v-model="description"
              :class="{ invalid: $v.description.$dirty && !$v.description.required}"
            >
            <label for="description">{{ 'Record_Description' | localize }}</label>
            <span
              class="helper-text invalid"
              v-if="$v.description.$dirty && !$v.description.required"
            >{{ 'Record_Description_Er' | localize }}</span>
          </div>

          <button class="btn waves-effect waves-light" type="submit">
            {{ 'Record_Btn' | localize }}
            <i class="material-icons right">send</i>
          </button>
        </form>
      </div>

    </div>
  </div>
</template>

<script>
import { required, minValue } from 'vuelidate/lib/validators'
import { mapGetters } from 'vuex'
import localizeFilter from '@/filters/localize.filter'

export default {
  name: 'Record',
  metaInfo() {
    return {
      title: this.$title('Record_New')
    }
  },
  data: () => ({
    loading: true,
    select: null,
    categories: [],
    category: null,
    type: 'outcome',
    amount: 1,
    description: ''
  }),
  async mounted () {
    this.categories = await this.$store.dispatch('fetchCategories')
    this.loading = false

    if (this.categories.length) {
      this.category = this.categories[0].id
    }

    setTimeout(() => {
      this.select =window.M.FormSelect.init(this.$refs.select)
      M.updateTextFields()
    }, 0)
  },
  destroyed () {
    if (this.select && this.select.destroy) {
      this.select.destroy()
    }
  },
  validations: {
    amount: {
      required,
      minValue: minValue(10)
    },
    description: {
      required
    }
  },
  computed: {
    ...mapGetters([
      'info'
    ]),
    canCreateRecord () {
      if (this.type === 'income') {
        return true
      } else {
        return this.info.bill >= this.amount
      }
    }
  },
  methods: {
    async handleSubmit () {
      if (this.$v.$invalid) {
        this.$v.$touch()
        return {}
      }
      if (this.canCreateRecord) {
        try {
          await this.$store.dispatch('createRecord', {
            categoryId: this.category,
            amount: this.amount,
            description: this.description,
            type: this.type,
            date: new Date().toJSON()
          })
          const bill = this.type === 'income'
            ? this.info.bill + this.amount
            : this.info.bill - this.amount

          await this.$store.dispatch('updateInfo', {bill })
          this.$message(localizeFilter('Record_Created'))
          this.$v.$reset()
          this.amount = 1
          this.description = ''
        } catch (e) {}
      } else {
        this.$message(`${localizeFilter('Record_Not_Enough')} (${this.amount - this.info.bill})`)
      }

    }
  }
}
</script>

<style scoped>

</style>
