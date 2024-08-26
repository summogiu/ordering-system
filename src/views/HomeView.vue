<template>
  <div class="frame">
    <div class="header-box">
      <span class="title-en">Good Drink</span><span class="title-zh"> 好飲</span>
    </div>
    <div class="main-box">
      <div class="item-list-box">
        <button type="button" class="type-btn" @click="typeDrinkIsOpen = !typeDrinkIsOpen">飲品
          <transition name="typeBtn">
            <font-awesome-icon :icon="['fas', 'plus']" class="btn-icon" v-if="!typeDrinkIsOpen"/>
            <font-awesome-icon :icon="['fas', 'minus']" class="btn-icon" v-else />
          </transition>
        </button>
        <ul class="item-list" :class="[typeDrinkIsOpen ? 'open' : '']">
          <li v-for="item in products" :key="item.name" @click="selectProduct(item)" :class="[isFocus === item ? 'focus' : '']">
            {{ item.name }}
          </li>
        </ul>
      </div>
      <div class="customized-content-box">
        <div class="customized" :class="[isFocus ? 'noDisabled' : '']">
          <label for="productNum" class="customized-label">數量</label>
          <div class="customized-item">
            <input type="number" min="1" id="productNum" class="product-num" v-model="nowCustomize.qty">杯
          </div>
          <label class="customized-label">冰塊</label>
          <div class="customized-item">
            <label v-for="item,i in iceType" :key="i" :for="item">
              <input type="radio" name="ice" :id="item" :value="item" v-model="nowCustomize.ice" :disabled="iceLocked && item !== nowCustomize.ice">
              {{ item }}
            </label>
          </div>
          <label class="customized-label">甜度</label>
          <div class="customized-item">
            <label v-for="item,i in sugarType" :key="i" :for="item">
              <input type="radio" name="sugar" :id="item" :value="item" v-model="nowCustomize.sugar" :disabled="sugarLocked && item !== nowCustomize.sugar">
              {{ item }}
            </label>
          </div>
          <label class="customized-label">加料</label>
          <div class="customized-item">
            <label v-for="item,i in toppingsType" :key="i" :for="item">
              <input type="checkbox" name="toppings" :id="item" :value="item" v-model="nowCustomize.toppings">
              {{ item }}
            </label>
          </div>
          <label for="ps" class="customized-label">備註</label>
          <textarea class="ps" id="ps" cols="30" rows="10" placeholder="備註內容" v-model="nowCustomize.ps"></textarea>
          <div class="customized-btn-box">
            <button type="button" class="customized-btn remake-btn" @click="remakeCustomize">重製</button>
            <button type="button" class="customized-btn add-btn" @click="addToOrder">加入</button>
          </div>
        </div>
        <div class="current-order">
          <table>
            <thead>
              <tr>
                <th>品項</th>
                <th>冰塊</th>
                <th>甜度</th>
                <th>加料</th>
                <th>單價</th>
                <th>數量</th>
                <th>小計</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item,i in buyingList" :key="i">
                <td>{{ item.name }}</td>
                <td>{{ item.ice }}</td>
                <td>{{ item.sugar }}</td>
                <td>{{ item.toppings && item.toppings.join(',') }}</td>
                <td>{{ item.price }}</td>
                <td>{{ item.qty }}</td>
                <td>{{ item.subtotal }}</td>
              </tr>
            </tbody>
          </table>
          <p class="total">共{{ total }}元</p>
          <button type="button" class="create-order-btn" :class="[buyingList.length === 0 ? 'disabled' : '']" @click="createOrder" :disabled="buyingList.length === 0">{{ buyingList.length !== 0 ? '建立訂單' : '請先加入品項' }}</button>
        </div>
      </div>
    </div>
    <div class="order-box">
      <p class="no-order-tip" v-if="EstablishedOrder.length === 0">目前尚無訂單</p>
      <div class="order" v-for="item in EstablishedOrder" :key="item.id">
        <table>
          <thead>
            <tr>
              <th>品項</th>
              <th>冰塊</th>
              <th>甜度</th>
              <th>加料</th>
              <th>單價</th>
              <th>數量</th>
              <th>小計</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="product,i in item.itemlist" :key="i">
              <td>{{ product.name }}</td>
              <td>{{ product.ice }}</td>
              <td>{{ product.sugar }}</td>
              <td>{{ product.toppings && product.toppings.join(',') }}</td>
              <td>{{ product.price }}</td>
              <td>{{ product.qty }}</td>
              <td>{{ product.subtotal }}</td>
            </tr>
          </tbody>
        </table>
        <div class="other-data">
          <p>訂單成立時間：{{ item.time }}</p>
          <p>餐點數：{{ item.totalQty }}</p>
          <p>訂單編號：{{ item.id }}</p>
          <p>共{{ item.total }}元</p>
          <p>付款狀態：{{ item.isPay ? '已付款' : '未付款' }}</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import TheWelcome from '../components/TheWelcome.vue'
import { ref, computed } from 'vue'

const iceType = ref(['正常冰', '少冰', '微冰', '去冰', '熱'])
const sugarType = ref(['全糖', '七分', '半糖', '三分', '無糖'])
const toppingsType = ref (['珍珠', '粉條', '小粉圓', '椰果', '芋頭'])
const products = ref([
  {
    name: '珍珠鮮奶茶',
    engName: 'Pearl Milk Tea',
    price: 60,
    defaults: {
      toppings: ['珍珠'],
      sugar: '',
      ice: '',
    }
  },
  {
    name: '椰果鮮奶茶',
    engName: 'Coconut Milk Tea',
    price: 60,
    defaults: {
      toppings: ['椰果'],
      sugar: '',
      ice: '',
    }
  },
  {
    name: '鮮奶茶',
    engName: 'Milk Tea',
    price: 50,
    defaults: {
      toppings: [''],
      sugar: '',
      ice: '',
    }
  },
  {
    name: '古意冬瓜茶 (糖固定)',
    engName: 'Winter Melon Drink',
    price: 30,
    defaults: {
      toppings: [''],
      sugar: '全糖',
      ice: '',
    }
  },
  {
    name: '蜜香紅茶',
    engName: 'Black Tea',
    price: 30,
    defaults: {
      toppings: [''],
      sugar: '',
      ice: '',
    }
  },
  {
    name: '包種青茶',
    engName: 'Black Tea',
    price: 35,
    defaults: {
      toppings: [''],
      sugar: '',
      ice: '',
    }
  },
  {
    name: '檸檬烏龍',
    engName: 'Lemon Oolong Tea',
    price: 55,
    defaults: {
      toppings: [''],
      sugar: '',
      ice: '',
    }
  },
  {
    name: '薑母茶 (熱飲)',
    engName: 'Ginger Tea',
    price: 55,
    defaults: {
      toppings: [''],
      sugar: '',
      ice: '熱',
    }
  },
  {
    name: '青草茶',
    engName: 'Herbal Tea',
    price: 35,
    defaults: {
      toppings: [''],
      sugar: '',
      ice: '',
    }
  },
  {
    name: '金桔檸檬',
    engName: 'Kumquat Lemonade',
    price: 40,
    defaults: {
      toppings: [''],
      sugar: '',
      ice: '',
    }
  },
  {
    name: '柳澄青茶',
    engName: 'Orange Mountain Tea',
    price: 45,
    defaults: {
      toppings: [''],
      sugar: '',
      ice: '',
    }
  }
])

const typeDrinkIsOpen = ref(false)

const nowCustomize = ref({
  name: '',
  price: 0,
  ice: '正常冰',
  sugar: '半糖',
  toppings: [],
  qty: 1,
  ps: '',
  subtotal: 0
})
const buyingList = ref([])

const iceLocked = ref(false)
const sugarLocked = ref(false)
const isFocus = ref(null)

const selectProduct = (item) => {
  nowCustomize.value.name = item.name
  nowCustomize.value.price = item.price
  nowCustomize.value.toppings = []

  iceLocked.value = false
  sugarLocked.value = false
  isFocus.value = item

  if (item.defaults.ice !== '') {
    iceLocked.value = true
    nowCustomize.value.ice = item.defaults.ice
  } else {
    nowCustomize.value.ice = '正常冰'
  }

  if (item.defaults.sugar !== '') {
    nowCustomize.value.sugar = item.defaults.sugar
    sugarLocked.value = true
  } else {
    nowCustomize.value.sugar = '半糖'
  }
}
const initCustomize = () => {
  nowCustomize.value.name = ''
  nowCustomize.value.price = 0
  nowCustomize.value.ice = '正常冰'
  nowCustomize.value.sugar = '半糖'
  nowCustomize.value.toppings = []
  nowCustomize.value.qty = 1
  nowCustomize.value.ps = ''
  nowCustomize.value.subtotal = 0
}
const remakeCustomize = () => {
  nowCustomize.value.name = isFocus.value.name
  nowCustomize.value.price = isFocus.value.price
  nowCustomize.value.ice = isFocus.value.defaults.ice || '正常冰'
  nowCustomize.value.sugar = isFocus.value.defaults.sugar || '半糖'
  nowCustomize.value.toppings = []
  nowCustomize.value.qty = 1
  nowCustomize.value.ps = ''
  nowCustomize.value.subtotal = 0
}
const addToOrder = () => {
  nowCustomize.value.subtotal = nowCustomize.value.price * nowCustomize.value.qty
  buyingList.value.push(JSON.parse(JSON.stringify(nowCustomize.value)))
  isFocus.value = null
  initCustomize()
}

const total = computed(() => {
  let total = 0
  buyingList.value.forEach(item => {
    total += item.subtotal
  })
  return total
})

const EstablishedOrder = ref([])
const getCreatetime = () => {
  const year = new Date().getFullYear()
  const month = new Date().getMonth() + 1
  const day = new Date().getDate()
  const hours = new Date().getHours()
  const minutes = new Date().getMinutes()

  return `${year}-${month}-${day} ${hours}:${minutes}`
}
const getTotalQty = () => {
  const qtys = buyingList.value.map(item => item.qty)
  return qtys.reduce((a, b) => a + b, 0)
}
const createOrder = () => {
  EstablishedOrder.value.push({
    id: new Date().getTime(),
    time: getCreatetime(),
    total: total.value,
    itemlist: JSON.parse(JSON.stringify(buyingList.value)),
    totalQty: getTotalQty(),
    isPay: false
  })
  buyingList.value = []
}
</script>

<style lang="scss">
  .header-box{
    background: #ffd447;
    border-bottom: 2px solid white;
    padding: 0 20px;

    .title-en{
      color: rgb(43, 43, 43);
      font-size: 40px;
      font-family:'Franklin Gothic Medium', 'Arial Narrow', Arial, sans-serif;
      
    }
    .title-zh{
      color: rgb(43, 43, 43);
      font-size: 30px;
    }
  }

  .main-box{
    display: flex;
    border-bottom: 2px solid white;

    .item-list-box{
      background: #ffd447;
      width: 200px;

      .type-btn{
        background: #ffd447;
        border: none;
        border-bottom: 2px solid rgb(43, 43, 43);
        width: 100%;
        padding: 10px 20px;
        font-size: 20px;
        text-align: start;
        cursor: pointer;
        position: relative;

        .btn-icon{
          position: absolute;
          right: 20px;
          top: 15px;
        }
      }
      .item-list{
        max-height: 0;
        overflow: hidden;
        transition: max-height .3s;

        li{
          background-color: #ffd447;
          border-bottom: 1px dashed rgb(43, 43, 43);
          padding: 10px 20px;
          cursor: pointer;

          &:hover{
            background-color: #fcdf80;
          }
        }
        li.focus{
          background: white;
        }
      }
      .item-list.open{
        max-height: 100%;
      }
    }
    .customized-content-box{
      width: calc(100% - 200px);

      .customized{
        box-shadow: 0px 5px 5px rgb(218, 218, 218);
        padding: 10px 20px;
        position: relative;

        .customized-label{
          display: block;
        }
        .product-num{
          margin-right: 5px;
          font-size: 16px;
          width: 50px;
        }
        .customized-item{
          padding: 5px 0 10px;
          margin-bottom: 10px;
          border-bottom: 1px solid rgb(226, 226, 226);

          label{
            margin-right: 10px;
          }
        }
        .ps{
          resize: none;
          width: 100%;
          height: 60px;
          padding: 10px;
          font-size: 16px;
        }
        .customized-btn-box{
          display: flex;
          justify-content: end;

          .customized-btn{
            border: 1px solid #ffd447;
            font-size: 16px;
            padding: 5px 20px;
            cursor: pointer;
          }
          .remake-btn{
            background-color: white;
            margin-right: 10px;

            &:hover{
              background-color: #f5f5f5;
            }
          }
          .add-btn{
            background-color: #ffd447;

            &:hover{
              background-color: #fcdf80;
            }
          }
        }

        &::before{
          display: flex;
          justify-content: center;
          align-items: center;
          content: '◀◀ 請先選擇品項';
          font-size: 18px;
          color: white;
          position: absolute;
          width: 100%;
          height: 100%;
          top: 0;
          left: 0;
          background: rgba($color: #000000, $alpha: 0.8);
        }
      }
      .customized.noDisabled{
        &::before{
          display: none;
        }
      }
      .current-order{
        margin-top: 30px;
        padding: 0 20px;
        display: flex;
        flex-direction: column;
        align-items: flex-end;

        table{
          width: 100%;

          thead{
            border-bottom: 1px solid rgb(43, 43, 43);
          }
          tbody{
            border-bottom: 1px solid rgb(226, 226, 226);

            tr{
              padding: 5px 0;
            }
          }
          tr{
            display: flex;
            justify-content: space-around;

            th,td{
              width: calc(100% / 7);
              margin-right: 5px;
              text-align: center;
            }
          }
        }
        .create-order-btn{
          background-color: #ffd447;
          border: none;
          width: 100%;
          font-size: 14px;
          padding: 5px 0;
          margin-bottom: 20px;
          cursor: pointer;

            &:hover{
              background-color: #fcdf80;
            }
        }
        .create-order-btn.disabled{
          background-color: #c4c4c4;
          color: #000000;
          cursor: default;

          &:hover{
              background-color: #c4c4c4;
            }
        }
      }
    }
  }

  .order-box{
    .no-order-tip{
      text-align: center;
      color: #aaaaaa;
      padding: 20px 0;
    }
    .order{
      margin-bottom: 10px;

      table{
        width: 100%;
        
        thead{
          background: #ffd447;
          padding: 5px 0;
          display: block;
        }
        tr{
          display: flex;
          justify-content: space-around;

          th,td{
            width: calc(100% / 7);
            text-align: center;
            margin-right: 5px;
          }
        }
      }
      .other-data{
        padding: 5px 20px;
        display: flex;
        flex-wrap: wrap;
        width: 100%;

        p{
          width: 50%;
        }
      }
    }
  }

// 動畫
  .typeBtn-enter-from,.typeBtn-leave-to{
    transform: rotate(0deg);
    opacity: 0;
  }
  .typeBtn-enter-active,.typeBtn-leave-active{
    transition: transform .5s, opacity .2s;
  }
  .typeBtn-enter-to,.typeBtn-leave-from{
    transform: rotate(180deg);
    opacity: 1;
  }
</style>