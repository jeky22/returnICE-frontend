<template>
  <div>
    <div v-if = "this.isLoading === false" class = "loading">
      <img src = '../../assets/loading.gif'>
    </div>
    <div v-if = "this.isLoading == true">
      <div id = 'menu_div'>
          <button v-on:click = "add_function" class = "append_btn"></button>
          <button v-on:click = "delete_function" class = "delete_btn" ></button>
      </div>
      <div>
        <modals-container v-on:loadData="loadData"/>
      </div>
      <vue-good-table
        class = "my-table"
        @on-selected-rows-change="selectionChanged"
        :columns="columns"
        :rows="rows"
        :select-options="{
          enabled: true,
          disableSelectInfo: true,
        }"
        styleClass="vgt-table striped"/>
      </div>
  </div>
</template>
<script>
import 'vue-good-table/dist/vue-good-table.css'
import { VueGoodTable } from 'vue-good-table'
import addMenu from './add_menu.vue'
import axios from 'axios'

export default {
  name: 'menuTable',
  components: {
    VueGoodTable
  },
  data: function () {
    return {
      isLoading: false,
      selectList: [],
      columns: [
        {
          label: '메뉴 이름',
          field: 'name'
        },
        {
          label: '가격',
          field: 'price'
        },
        {
          label: '메뉴정보',
          field: 'info'
        },
        {
          label: '평균 평점',
          field: 'score'
        }
      ],
      rows: [],
      menu_data: []
    }
  },
  beforeCreate () {
    axios.get('/api/sellers/product', {
    }).then((res) => {
      for (let i = 0; i < res.data.menu.length; i++) {
        this.rows.push({ id: i + 1, name: res.data.menu[i].menuName, price: res.data.menu[i].price, info: res.data.menu[i].info, score: res.data.menu[i].avgScore })
        this.menu_data.push({ id: i + 1, menuId: res.data.menu[i].menuId })
      }
      this.isLoading = true
    })
  },
  methods: {
    loadData: function () {
      axios.get('/api/sellers/product', {
      }).then((res) => {
        this.menu_data = []
        this.rows = []
        for (let i = 0; i < res.data.menu.length; i++) {
          this.rows.push({ id: i + 1, name: res.data.menu[i].menuName, price: res.data.menu[i].price, info: res.data.menu[i].info, score: res.data.menu[i].avgScore })
          this.menu_data.push({ id: i + 1, menuId: res.data.menu[i].menuId })
        }
        this.isLoading = true
      })
    },
    selectionChanged: function (params) {
      this.selectList = params.selectedRows
    },
    delete_function: async function () {
      for (let i = 0; i < this.selectList.length; i++) {
        for (let j = 0; j < this.menu_data.length; j++) {
          if (this.menu_data[j].id === this.selectList[i].id) {
            console.log(this.menu_data[j].menuId)
            await axios.delete(`/api/sellers/product/menu/${this.menu_data[j].menuId}`, {}).then((res) => {
              if (res.data.success === false) {
                alert('구독권에 포함되어 있어 삭제 할 수 없습니다.')
              }
            })
          }
        }
      }
      this.loadData()
    },
    add_function: function () {
      this.$modal.show(addMenu, {
        modal: this.$modal
      }, {
        name: 'add_menu',
        width: '330px',
        height: '250px',
        draggable: false
      })
    }
  }
}
</script>
<style src = "./menu_table.css" scoped>
</style>
