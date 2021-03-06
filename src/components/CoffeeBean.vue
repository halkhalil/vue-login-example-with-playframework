<template>
  <div>
    <app-header></app-header>
    <b-table striped hover :items="coffeeBeans" :fields="fields">
      <template slot="name" slot-scope="row">
        <div v-if="!row.item.editable">
          <div v-text="row.item.name" @click="row.item.editable = true" />
        </div>
        <div v-if="row.item.editable">
          <b-form-input id="name"
                       type="text"
                       v-model="form.name"
                       required
                       @change="row.item.changed=true;"
                       :placeholder="`${row.item.name}`">
          </b-form-input>
        </div>
      </template>
      <template slot="kind" slot-scope="row">
        <div v-if="!row.item.editable">
          <div v-text="row.item.kind" @click="row.item.editable = true" />
        </div>
        <div v-if="row.item.editable">
          <b-row class="mb-2">
            <v-autocomplete :items="coffeeKinds"
              v-model="form.kind"
              :get-label="getLabel"
              :component-item='template'
              @change="row.item.changed=true;"
              @update-items="updateCoffeeKinds">
            </v-autocomplete>
          </b-row>
        </div>
      </template>
      <template slot="update" slot-scope="row">
        <b-form @submit="update">
          <div v-if="row.item.changed">
            <b-button type="submit" size="sm" variant="primary">update</b-button>
            <b-row class="sr-only">
            <b-form-input id="cid"
                          v-bind:value="`${form.cid = row.item.id}`">
            </b-form-input>
            <b-form-input id="coffee_shop_id"
                          v-bind:value="`${form.coffee_shop_id = row.item.coffee_shop_id}`">
            </b-form-input>
            </b-row>
          </div>
          <div v-if="!row.item.changed">
          </div>
        </b-form>
      </template>
    </b-table>

    <template>
      <b-button :pressed.sync="addToggle" size="sm" class="mr-2" variant="outline-success">
        {{ addToggle ? 'Cancel' : ''}} Add new Coffee Beans
      </b-button>
      <br/>
      <br/>
      <b-card v-if="addToggle">
        <b-form @submit="create">
          <b-row class="mb-2">
          <b-form-input id="name"
                       type="text"
                       v-model="form.name"
                       required
                       placeholder='coffee name'>
          </b-form-input>
          </b-row>

          <b-row class="mb-2">
            <v-autocomplete :items="coffeeKinds" v-model="form.kind" :get-label="getLabel" :component-item='template' @update-items="updateCoffeeKinds">
            </v-autocomplete>
          </b-row>
          <b-button type="submit" size="sm" variant="primary">Create</b-button>
        </b-form>
      </b-card>
    </template>
    <app-footer></app-footer>
  </div>
</template>

<script>
import ApiClient from './utils/ApiClient'
import AppHeader from './AppHeader'
import AppFooter from './AppFooter'
import ItemTemplate from './ItemTemplate'

export default {
  data () {
    return {
      coffeeShopId: 0,
      addToggle: false,
      fields: {
        id: {label: '-', sortable: false},
        name: {label: 'name', sortable: true},
        kind: {label: 'kind', sortable: true},
        update: {label: ' ', sortable: true}
      },
      coffeeKinds: [],
      template: ItemTemplate,
      coffeeBeans: [],
      form: {
        cid: '',
        name: '',
        kind: '',
        coffee_shop_id: ''
      }
    }
  },
  components: { AppHeader, AppFooter },
  methods: {
    find: function (id) {
      ApiClient.find('/api/coffee-beans/', id, (res) => {
        console.log(res.data)
      }, (error) => {
        console.log(error)
      })
    },
    reSearch: function () {
      let targetPath = '/api/coffee-beans?coffee-shop-id=' + this.coffeeShopId

      ApiClient.search(targetPath, (response) => {
        this.coffeeBeans = response.data.map(function (bean) {
          bean.editable = false
          bean.changed = false

          return bean
        })
      }, (error) => {
        console.log(error)
        this.$router.push('/signIn')
      })
    },
    create: function () {
      let params = {
        id: 0, // dummy value
        name: this.form.name,
        kind: this.form.kind.name,
        coffee_shop_id: Number(this.coffeeShopId)
      }

      ApiClient.create('/api/coffee-beans', params, (res) => {
        this.reSearch()
        this.addToggle = false
      }, (error) => {
        console.log(error)
      })
    },
    update: function () {
      let params = {
        id: Number(this.form.cid),
        name: this.form.name,
        kind: this.form.kind.name,
        coffee_shop_id: Number(this.form.coffee_shop_id)
      }

      ApiClient.update('/api/coffee-beans', params, (res) => {
        this.reSearch()
      }, (error) => {
        console.log(error)
      })
    },
    destroy: function () {
      ApiClient.destroy('/api/coffee-beans', this.form.cid, (res) => {
        this.reSearch()
        this.addToggle = false
      }, (error) => {
        console.log(error)
      })
    },
    getLabel: function (item) {
      if (item !== null) {
        return item.name
      }
    },
    updateCoffeeKinds: function () {
      let targetPath = '/api/coffee-kinds'

      ApiClient.search(targetPath, (response) => {
        this.coffeeKinds = response.data
      }, (error) => {
        console.log(error)
        this.$router.push('/signIn')
      })
    }
  },
  created: function () {
    this.coffeeShopId = this.$route.query['coffee-shop-id']
    this.reSearch()
    this.updateCoffeeKinds()
  }
}
</script>
