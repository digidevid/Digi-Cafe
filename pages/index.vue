<template>
  <div class="max-w-lg mx-auto bg-blue-100">
    <div class="py-8 px-5">
      <h1 class="text-center text-3xl font-bold mb-4">Menu DigiCafe</h1>
      <div>
        <h3 class="text-xl font-bold mb-2">Makanan</h3>
        <div v-for="(food, id) in foods" :key="id">
          <div
            class="flex justify-between items-center border-b-2 border-dashed border-yellow-600 p-2 relative z-0"
          >
            <div class="flex space-x-3 items-center">
              <div>
                <p>{{ food.name }}</p>
                <p class="text-sm text-gray-600">
                  {{ toRupiah(food.price) }}
                </p>
              </div>
              <div
                v-if="food.label && food.label.toLowerCase() === 'new'"
                class="text-white font-bold text-xs text-shadow-new animate-bounce"
              >
                {{ food.label }}
              </div>
            </div>
            <div class="flex space-x-3 items-center">
              <button @click="minus(food)" :disabled="food.quantity === 0">
                <font-awesome-icon
                  :icon="['fa', 'circle-minus']"
                  class="text-red-400 w-6 h-6"
                />
              </button>
              <p>{{ food.quantity }}</p>
              <button @click="plus(food)">
                <font-awesome-icon
                  :icon="['fa', 'circle-plus']"
                  class="text-green-400 w-6 h-6"
                />
              </button>
            </div>
            <div
              v-if="food.label && food.label.toLowerCase() === 'best seller'"
              class="absolute -z-10 top-0 left-0 w-full h-full"
            >
              <div class="absolute stamp">
                <div class="price-tag text-xs animate-pulse">
                  {{ food.label }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div>
        <h3 class="text-xl font-bold mt-4 mb-2">Minuman</h3>
        <div v-for="(drink, id) in drinks" :key="id">
          <div
            class="flex justify-between items-center border-b-2 border-dashed border-yellow-600 p-2 relative z-0"
          >
            <div class="flex space-x-3 items-center">
              <div>
                <p>{{ drink.name }}</p>
                <p class="text-sm text-gray-600">
                  {{ toRupiah(drink.price) }}
                </p>
              </div>
              <div
                v-if="drink.label && drink.label.toLowerCase() === 'new'"
                class="text-white font-bold text-xs text-shadow-new animate-bounce"
              >
                {{ drink.label }}
              </div>
            </div>
            <div class="flex space-x-3 items-center float-right">
              <button @click="minus(drink)" :disabled="drink.quantity === 0">
                <font-awesome-icon
                  :icon="['fa', 'circle-minus']"
                  class="text-red-400 w-6 h-6"
                />
              </button>
              <p>{{ drink.quantity }}</p>
              <button @click="plus(drink)">
                <font-awesome-icon
                  :icon="['fa', 'circle-plus']"
                  class="text-green-400 w-6 h-6"
                />
              </button>
            </div>
            <div
              v-if="drink.label && drink.label.toLowerCase() === 'best seller'"
              class="absolute -z-10 top-0 left-0 w-full h-full"
            >
              <div class="absolute stamp">
                <div class="price-tag text-xs animate-pulse">
                  {{ drink.label }}
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="sticky bottom-0 bg-white rounded-t-2xl">
      <div
        class="pt-4 pb-2 px-6 flex justify-between items-center text-xl font-bold"
      >
        <p>TOTAL</p>
        <p>{{ toRupiah(total) }}</p>
      </div>
      <div class="p-2 pb-0">
        <button
          class="bg-red-500 w-full py-3 text-lg font-semibold text-white rounded-md"
          @click="reset"
        >
          Hapus
        </button>
      </div>
      <div class="p-2">
        <button
          class="bg-blue-500 w-full py-3 text-lg font-semibold text-white rounded-md"
          @click="checkout"
        >
          Checkout
        </button>
      </div>
    </div>
    <ModalCheckout
      :is-show="isShowModalCheckout"
      :checkout-menus="checkoutItems"
      :total="total"
      :is-loading="isLoadingSubmit"
      @onClose="onCloseModalCheckout"
      @submitData="submitData"
    />
  </div>
</template>

<script>
import ModalCheckout from '../components/modal-checkout.vue'
import { foods, drinks } from '../constants/menu'
import { toRupiah } from '../helper/currency'
export default {
  name: 'IndexPage',
  components: {
    ModalCheckout,
  },
  data() {
    return {
      foods,
      drinks,
      total: 0,
      checkoutItems: [],
      isShowModalCheckout: false,
      isLoadingSubmit: false,
    }
  },
  methods: {
    minus(item) {
      item.quantity--
      item.totalPrice = item.price * item.quantity
      this.calculate()
    },
    plus(item) {
      item.quantity++
      item.totalPrice = item.price * item.quantity
      this.calculate()
    },
    calculate() {
      const totalFoodPrice = this.foods
        .map((item) => item.totalPrice)
        .reduce((prev, curr) => prev + curr, 0)
      const totalDrinkPrice = this.drinks
        .map((item) => item.totalPrice)
        .reduce((prev, curr) => prev + curr, 0)
      this.total = totalFoodPrice + totalDrinkPrice
    },
    reset() {
      this.foods.forEach((el) => {
        el.quantity = 0
        el.totalPrice = 0
      })
      this.drinks.forEach((el) => {
        el.quantity = 0
        el.totalPrice = 0
      })
      this.total = 0
    },
    checkout() {
      const allMenu = this.foods.concat(this.drinks)
      this.checkoutItems = allMenu.filter((item) => item.quantity > 0)
      this.isShowModalCheckout = true
    },
    onCloseModalCheckout() {
      this.isShowModalCheckout = false
    },
    submitData(data, customerName) {
      this.isLoadingSubmit = true
      const order = []
      data.forEach((el) => {
        el.customerName = customerName
        order.push(`\n${el.name} (${el.quantity}) : ${toRupiah(el.totalPrice)}`)
      })
      const whatsappText = `Halo, saya *${customerName}* mau pesan menu berikut: \n${order}\n*TOTAL : ${toRupiah(
        this.total
      )}*\n\nTerima Kasih`
      const whatsappUrl = `https://api.whatsapp.com/send?phone=6281269085794&text=${encodeURIComponent(
        whatsappText
      )}`
      const scriptUrl =
        'https://script.google.com/macros/s/AKfycbyh1TI0jEDqMR9Als0eqDKcy9puNpCsLj3FezJZCGAT9zF5AYbOo6wWlSb2K-fM64_e/exec'

      const fetches = []
      let isSuccess = true
      data.forEach((el) => {
        const newForm = new FormData()
        const today = new Date()

        const date = today.getDate()
        const month = today.getMonth() + 1
        const year = today.getFullYear()
        const fullDate = `${date}/${month}/${year}`

        const hour = today.getHours()
        const minute = today.getMinutes()
        const time = `${hour}:${minute}`

        newForm.append('Tanggal', fullDate)
        newForm.append('Jam', time)
        newForm.append('Menu', el.name)
        newForm.append('Harga', el.price)
        newForm.append('Jumlah', el.quantity)
        newForm.append('Total', el.totalPrice)
        newForm.append('Pemesan', el.customerName)

        fetches.push(
          fetch(scriptUrl, {
            method: 'POST',
            body: newForm,
            changeOrigin: true,
          }).catch((error) => {
            console.error('Error!', error.message)
            alert(error.message)
            isSuccess = false
          })
        )
      })
      Promise.all(fetches).then((res) => {
        window.location.href = whatsappUrl
        this.isShowModalCheckout = false
        this.isLoadingSubmit = false
        this.reset()
      })
    },
    toRupiah,
  },
}
</script>

<style scoped>
.text-shadow-new {
  color: rgb(233, 112, 132);
  background-color: yellow;
  text-shadow: 1.5px 1.5px blue;
  box-shadow: 2px 2px blue;
  border: 2px solid rgb(233, 112, 132);
  border-radius: 10px;
  padding: 2px 4px;
}
.stamp {
  top: 50%;
  right: 65px;
  transform: translate(-50%, -50%);
}
.price-tag {
  background: rgb(243, 97, 97);
  color: #fff;

  /* Center the price */
  align-items: center;
  display: flex;
  justify-content: center;

  /* Used to position the triangle */
  position: relative;

  /* Size */
  height: 2rem;

  /* Spacing */
  padding: 0.25rem 0.5rem;
}

/* The triangle */
.price-tag::before {
  content: '';

  border-color: transparent rgb(243, 97, 97) transparent transparent;
  border-style: solid;
  border-width: calc(2rem / 2) calc(2rem / 2) calc(2rem / 2) 0rem;

  /* Position */
  left: 0px;
  position: absolute;
  top: 0px;
  transform: translate(-100%, 0px);
}

/* The Dotted */
.price-tag::after {
  content: '';

  /* Make it like a cirle */
  background: #fff;
  border-radius: 9999rem;

  /* Position */
  left: 3px;
  position: absolute;
  top: 50%;
  transform: translate(-0.5rem, -50%);

  /* Size */
  height: 0.5rem;
  width: 0.5rem;
}
</style>
