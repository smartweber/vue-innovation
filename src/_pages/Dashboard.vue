<template>
  <div class="container">
    <section class="section">
      <div class="tile box">
        <form @submit.prevent="validateForm('addProductForm')" class="columns column is-multiline is-12" data-vv-scope="addProductForm">
          <div class="title">Add a Food</div>

          <div class="column is-12 field">
            <label class="label">Fridge Name</label>
            <p class="control has-icon has-icon-right">
              <span :class="{'select': true, 'is-danger': errors.has('addProductForm.fride_name') }">
                <select v-validate="'required|not_in:Choose'" name="fride_name" v-model="frideName">
                  <option v-bind:value="'Fridge1'">
                    Fridge1
                  </option>
                  <option v-bind:value="'Fridge2'">
                    Fridge2
                  </option>
                </select>
              </span>
              <i v-show="errors.has('addProductForm.fride_name')" class="fa fa-warning"></i>
              <span v-show="errors.has('addProductForm.fride_name')" class="help is-danger">{{ errors.first('addProductForm.fride_name') }}</span>
            </p>
          </div>

          <div class="column is-12 field">
            <label class="label">Food Name</label>
            <p class="control has-icon has-icon-right">
              <input name="product_name" v-validate="'required|min:2'" :class="{'input': true, 'is-danger': errors.has('addProductForm.product_name') }" type="text" placeholder="Product Name" v-model="productName">
              <i v-show="errors.has('addProductForm.product_name')" class="fa fa-warning"></i>
              <span v-show="errors.has('addProductForm.product_name')" class="help is-danger">{{ errors.first('addProductForm.product_name') }}</span>
            </p>
          </div>

          <div class="column is-12 field">
            <label class="label">Stock</label>
            <p class="control has-icon has-icon-right">
              <input name="stock" v-validate="'required|min_value:1'" :class="{'input': true, 'is-danger': errors.has('addProductForm.stock') }" type="number" placeholder="Stock" v-model="stock">
              <i v-show="errors.has('addProductForm.stock')" class="fa fa-warning"></i>
              <span v-show="errors.has('addProductForm.stock')" class="help is-danger">{{ errors.first('addProductForm.stock') }}</span>
            </p>
          </div>

          <div class="column is-12 field">
            <p class="control">
              <button class="button is-primary" type="submit" name="button">Add</button>
              <button class="button is-danger" type="button" name="button" @click="errors.clear('addProductForm')">Clear</button>
            </p>
          </div>
        </form>
      </div>
    </section>

    <section class="section">
      <div class="box">
        <div class="field is-12">
          <p class="control">
            <button class="button is-primary modal-button" data-target="modal-ter" aria-haspopup="true"  v-on:click="onMovementModal()">Movement</button>
          </p>
        </div>
        <div class="field has-addons is-12">
          <p class="control">
            <input class="input" type="text" placeholder="Search" v-model="searchQuery">
          </p>
          <p class="control">
            <a class="button is-static">
              Search
            </a>
          </p>
        </div>

        <div class="is-12">
          <innovation-grid
            :data="gridData"
            :columns="gridColumns"
            :filter-key="searchQuery"
            @consume="onConsume">
          </innovation-grid>
        </div>
      </div>
    </section>

    <div :class="{'modal': true, 'is-active':  isComponentModalActive}">
      <div class="modal-background"></div>
      <div class="modal-card">
        <header class="modal-card-head">
          <p class="modal-card-title">Move groceries between Fridge 1 and Fridge2</p>
          <button class="delete" aria-label="close"></button>
        </header>
        <section class="modal-card-body">
          <div class="columns">
            <div class="column">
              <div class="select is-multiple">
                <select multiple v-model="selectedFridge1">
                  <option v-for="product in fridgeList1" v-bind:value="product.id" :key="product.id">
                    {{ product.name }}
                  </option>
                </select>
              </div>
            </div>
            <div class="column action">
              <button class="button is-default" name="button" @click.prevent="onMoveToFride2">
                Left
              </button>
              <button class="button is-default" name="button" @click.prevent="onMoveToFride1">
                Right
              </button>
            </div>
            <div class="column">
              <div class="select is-multiple">
                <select multiple v-model="selectedFridge2">
                  <option v-for="product in fridgeList2" v-bind:value="product.id" :key="product.id">
                    {{ product.name }}
                  </option>
                </select>
              </div>
            </div>
          </div>
        </section>
        <footer class="modal-card-foot">
          <button class="button is-success" @click.prevent="onApply">Apply</button>
          <button class="button" v-on:click="isComponentModalActive = false">Cancel</button>
        </footer>
      </div>
    </div>
  </div>
</template>

<script>
import InnovationGrid from '@/_components/innovation-grid'
import { uuid } from 'vue-uuid'

export default {
  name: 'Dashboard',
  components: { InnovationGrid },
  data () {
    // localStorage.setItem('gridData', JSON.stringify([]))
    let gridData = JSON.parse(localStorage.getItem('gridData'))
    if (!gridData) {
      gridData = []
    }
    return {
      searchQuery: '',
      frideName: '',
      productName: '',
      stock: '',
      gridColumns: ['fridge', 'name', 'stock'],
      gridData: gridData,
      isComponentModalActive: false,
      fridgeList1: [],
      fridgeList2: [],
      selectedFridge1: [],
      selectedFridge2: []
    }
  },
  methods: {
    validateForm (scope) {
      this.$validator.validateAll(scope).then((result) => {
        if (result) {
          let stock = parseInt(this.stock)
          let find = false

          for (let i = 0; i < this.gridData.length; i++) {
            if ((this.gridData[i]['fridge'] === this.frideName) && (this.gridData[i]['name'] === this.productName)) {
              this.gridData[i]['stock'] += stock
              find = true
              break
            }
          }

          if (!find) {
            this.gridData.push({
              id: uuid.v1(),
              fridge: this.frideName,
              name: this.productName,
              stock: stock
            })
            localStorage.setItem('gridData', JSON.stringify(this.gridData))
          }
        }
      })
    },
    onConsume (value) {
      for (let i = 0; i < this.gridData.length; i++) {
        if (this.gridData[i]['id'] === value.id) {
          if (this.gridData[i]['stock'] <= value.amount) {
            this.gridData.splice(i, 1)
          } else {
            this.gridData[i]['stock'] -= value.amount
          }
        }
      }

      localStorage.setItem('gridData', JSON.stringify(this.gridData))
    },
    moveProduct (id, isFrom1To2) {
      if (isFrom1To2) {
        let index = -1
        let product = null
        for (let i = 0; i < this.fridgeList1.length; i++) {
          if (this.fridgeList1[i]['id'] === id) {
            index = i
            product = this.fridgeList1[i]
            break
          }
        }

        if (index !== -1) {
          this.fridgeList1.splice(index, 1)
          this.fridgeList2.push(product)
        }
      } else {
        let index = -1
        let product = null
        for (let i = 0; i < this.fridgeList2.length; i++) {
          if (this.fridgeList2[i]['id'] === id) {
            index = i
            product = this.fridgeList2[i]
            break
          }
        }

        if (index !== -1) {
          this.fridgeList2.splice(index, 1)
          this.fridgeList1.push(product)
        }
      }
    },
    onMovementModal () {
      this.fridgeList1 = []
      this.fridgeList2 = []

      if (this.gridData && this.gridData.length > 0) {
        for (let i = 0; i < this.gridData.length; i++) {
          if (this.gridData[i]['fridge'] === 'Fridge1') {
            this.fridgeList1.push(this.gridData[i])
          } else if (this.gridData[i]['fridge'] === 'Fridge2') {
            this.fridgeList2.push(this.gridData[i])
          }
        }
      }
      this.isComponentModalActive = true
    },
    onMoveToFride2 () {
      for (let i = 0; i < this.selectedFridge1.length; i++) {
        this.moveProduct(this.selectedFridge1[i], true)
      }
    },
    onMoveToFride1 () {
      for (let i = 0; i < this.selectedFridge2.length; i++) {
        this.moveProduct(this.selectedFridge2[i], false)
      }
    },
    onApply () {
      this.gridData = []
      if (this.fridgeList1 && this.fridgeList1.length > 0) {
        for (let i = 0; i < this.fridgeList1.length; i++) {
          this.fridgeList1[i]['fridge'] = 'Fridge1'
        }
      }
      if (this.fridgeList2 && this.fridgeList2.length > 0) {
        for (let i = 0; i < this.fridgeList2.length; i++) {
          this.fridgeList2[i]['fridge'] = 'Fridge2'
        }
      }
      this.gridData.push(...this.fridgeList1)
      this.gridData.push(...this.fridgeList2)
      localStorage.setItem('gridData', JSON.stringify(this.gridData))
      this.isComponentModalActive = false
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  .select {
    width: 100%;
  }

  .select select {
    width: 100%;
  }

  table {
    width: 100%;
  }

  .action {
    display: flex;
    flex-flow: column;
  }
</style>
