<script setup lang="ts">
   import TheHeader from "./components/Header.vue"
   import Shop from "./components/Shop/Shop.vue"
   import TheFooter from "./components/Footer.vue"
   import Cart from "./components/Cart/Cart.vue"
   import data from'./data/product'
   import {reactive} from'vue'
   import type { ProductCartInterface, ProductInterface , FiltersInterface, FilterUpdate} from "./interfaces"
   import {computed} from'vue'
   import { DEFAULT_FILTERS } from './data/filters'

  
   const state = reactive<{
    products: ProductInterface[],
    cart: ProductCartInterface[],
    filters: FiltersInterface
   }>({
    products: data,
    cart: [],
    filters: {...DEFAULT_FILTERS}
    })
   

function addProductToCart(productId: number): void {
  const product =state.products.find(product => product.id === productId);
  if (product ){
    const productInCart = state.cart.find(product => product.id === productId)
     if(productInCart){
      productInCart.quantity ++
     }else{
       state.cart.push({ ...product, quantity: 1 })
      }
    }
  }


function removeProductFromCart(productId: number): void {
  const productFromCart = state.cart.find(product => product.id===productId)
  if(productFromCart) {
    if(productFromCart.quantity ===1){
      state.cart=state.cart.filter(product => product.id !== productId);
    }else{
      productFromCart.quantity--
    }
  }
}
function updateFilter(filterUpdate: FilterUpdate) {
  if (filterUpdate.search !== undefined) {
    state.filters.search = filterUpdate.search;
  } else if (filterUpdate.priceRange) {
    state.filters.priceRange = filterUpdate.priceRange;
  } else if (filterUpdate.category) {
    state.filters.category = filterUpdate.category;
  } else {
    state.filters = { ...DEFAULT_FILTERS };
  }
}
const cartEmpty = computed(() => state.cart.length === 0)

const filteredProducts =computed(() =>{
  return state.products.filter((product) => {
    if(product.title.toLocaleLowerCase().startsWith(state.filters.search.toLocaleLowerCase()) && 
    product.price >= state.filters.priceRange[0] &&
    product.price <= state.filters.priceRange[1] &&
    (product.category === state.filters.category || 
    state.filters.category === 'all')){
      return true;
    }else{
      return false;
    }
  });
});
</script>

<template>

  <div class="app-container" :class="{gridEmpty: cartEmpty}">

    <TheHeader class="header" />
    <Shop :products="filteredProducts" @add-product-to-cart="addProductToCart"
          :filters="state.filters" @update-filter="updateFilter" class="shop" />
    <Cart v-if="!cartEmpty" :cart= "state.cart" class="cart" @remove-product-from-cart="removeProductFromCart"/> 
    <TheFooter class="footer"/>
    
  </div>
</template>

<style lang="scss">
@import "./assets/scss/base.scss";
@import "./assets/scss/debug.scss";

.app-container{
  min-height: 100vh;
  display: grid;
  grid-template-areas: "header header" "shop cart" "footer footer";
  grid-template-columns: 75% 25%;
  grid-template-rows: 48px auto 48px;
}

.gridEmpty{

  grid-template-areas: "header " "shop" "footer";
  grid-template-columns: 100%;
}
.header{
  grid-area: header;
}

.shop{
  grid-area: shop;
}

.cart{
  grid-area: cart;
  background-color: white;
  border-left: var(--border
  );
}

.footer{
  grid-area: footer;
}
</style>
