<template>
  <div class="bottom-bar">
    <div class="check-content">
      <check-button
        :is-checked="isSelectAll"
        class="checked-button"
        @click.native="checkClick"/>
      <span>全选</span>
    </div>
      <div class="price">
        合计：{{totalPrice}}
      </div>

      <div class="calculate" @click="calcClick">
        去计算({{checkLength}})
      </div>
  </div>
</template>

<script>
import CheckButton from "components/content/checkButton/CheckButton";
import { mapGetters } from 'vuex'

export default {
  name: "CartButtomBar",
  components: {
    CheckButton
  },
  computed: {
    ...mapGetters(['cartList']),
    totalPrice () {
      return '￥' + this.cartList.filter(item => {
        return item.checked
      }).reduce((preValue, item) => {
        return preValue + item.price * item.count
      },0).toFixed(2)
    },
    checkLength() {
      return this.cartList.filter(item => item.checked).length
    },
    isSelectAll() {
      if (this.cartList.length ===0) return false

      // 1.使用filter
      // return !(this.cartList.filter(item => !item.checked).length)

      // 2.使用find
      return !this.cartList.find(item => !item.checked)
    }
  },
  methods: {
    checkClick() {
      if (this.isSelectAll) {
        // 全部选中
        this.cartList.forEach(item => item.checked = false)
      } else {
        // 有一些没选中
        this.cartList.forEach(item => item.checked = true)
      }
      // 不可以简化 会互相影响
      // this.cartList.forEach(item => item.checked = !this.isSelectAll)
    },
    calcClick() {
      if (!this.isSelectAll) {
        this.$toast.show('请选择购买的商品', 2000)
      }
    }
  }
}
</script>

<style scoped>
  .bottom-bar {
    position: relative;
    display: flex;

    height: 40px;
    line-height: 40px;

    background-color: #eee;

  }

  .check-content {
    display: flex;
    align-items: center;
    margin-left: 10px;
    width: 60px
  }

  .checked-button {
    width: 20px;
    height: 20px;
    line-height: 20px;
    margin-right: 5px;
  }

  .price {
    margin-left: 20px;
    flex: 1;
  }

  .calculate {
    width: 90px;
    background-color: red;
    color: #fff;
    text-align: center;
  }
</style>
