<template lang="pug">
#adminorders.content.mt-2.p-5
  b-table(:items="orders" :fields='fields' ref='table' stacked="md")
    template(#cell(user)='data')
      | {{ data.item.user.account}}
    template(#cell(name)='data')
      | {{ data.item.userInfo.name}}
    template(#cell(phone)='data')
      | {{ data.item.userInfo.phone}}
    template(#cell(address)='data')
      | {{ data.item.userInfo.address}}
    template(#cell(courier)='data')
      | {{ data.item.userInfo.courier}}
    template(#cell(deliveryDate)='data')
      | {{ data.item.userInfo.deliveryDate}}
    template(#cell(deliveryTime)='data')
      | {{ data.item.userInfo.deliveryTime}}
    template(#cell(date)='data')
      | {{ new Date(data.item.date).toLocaleString('zh-tw') }}
    template(#cell(products)='data')
      ul
        li(v-for='product in data.item.products' :key='product._id') {{ product.product.name }} x {{ product.quantity }}
    //- template(#cell(remark)='data')
    //-   | {{ data.item.userInfo.remark}}
    template(#cell(price)='data')
      | {{ sumPrice(data.item.products) }}
    template(#cell(orderState)='data')
      //- | {{ data.item.orderState }}
      b-btn.my-2(@click='check(data.item._id, data.index)' variant='secondary' v-b-modal.modal-state ) 編輯
      //- div.d-flex.mt-2
      //-   p 已完成:
      div(v-if='data.item.orderState === true') 已聯絡及安排出貨
  b-modal#modal-state(ref="my-modal" title='編輯訂單狀況' centered ok-variant='success' ok-title='確認' cancel-variant='danger' cancel-title='取消' @ok='submitModal')
    b-form-group
      b-form-radio(v-model='form.orderState' :value='false') 未聯絡
      b-form-radio(v-model='form.orderState' :value='true') 已聯絡及安排出貨
</template>
<style>
#adminorders{
  background: #FCFCFC;
}
#adminorders.content{
  margin-left:260px;
}
@media screen and (max-width: 992px) {
  #adminorders.content{
    margin-left:0px;
  }
}
</style>

<script>
export default {
  data () {
    return {
      orders: [],
      fields: [
        { key: '_id', label: '單號' },
        { key: 'name', label: '姓名' },
        { key: 'phone', label: '電話' },
        { key: 'address', label: '地址' },
        { key: 'courier', label: '運送' },
        { key: 'deliveryDate', label: '日期' },
        { key: 'deliveryTime', label: '時間' },
        // { key: 'user', label: '使用者' },
        { key: 'date', label: '訂購日' },
        { key: 'products', label: '商品' },
        // { key: 'remark', label: '備註' },
        { key: 'price', label: '總金額' },
        { key: 'orderState', label: '訂單狀況' }
      ],
      // options: [
      //   { text: '還沒處理', value: '123' },
      //   { text: '12', value: '34' }
      // ],
      form: {
        orderState: false
      },
      aa: '',
      bb: ''
    }
  },
  methods: {
    sumPrice (products) {
      const result = products.reduce((accumulator, currentValue) => {
        return (
          accumulator + currentValue.quantity * currentValue.product.price
        )
      }, 0)
      console.log(result)
      return new Intl.NumberFormat('en-IN').format(result)
    },
    async submitModal (event) {
      console.log(this.aa)
      console.log(this.form)
      event.preventDefault()
      try {
        await this.api.patch('/orders/' + this.aa, this.form, {
          headers: {
            authorization: 'Bearer ' + this.user.token
          }
        })
        this.orders[this.bb].orderState = this.form.orderState
        this.$refs.table.refresh()
        this.$bvModal.hide('modal-state')
      } catch (error) {
        console.log(error)
        this.$swal({
          icon: 'error',
          title: '錯誤',
          text: error.response.data.message
        })
      }
    },
    check (id, index) {
      console.log(this.aa)
      this.aa = id
      this.bb = index
    }
  },
  async created () {
    try {
      const { data } = await this.api.get('/orders/all', {
        headers: {
          authorization: 'Bearer ' + this.user.token
        }
      })
      this.orders = data.result
      console.log(this.orders)
    } catch (error) {
      this.$swal({
        icon: 'error',
        title: '失敗',
        text: '取得訂單失敗'
      })
    }
  }
}
</script>
