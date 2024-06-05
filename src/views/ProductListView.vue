<template>
  <div class="container mt-5">
    <h1>Products List</h1>
    <form @submit.prevent="onSearch">
      <div class="input-group mb-3">
        <input
          type="text"
          class="form-control"
          placeholder="Search for a product"
          v-model="query"
          required
        />
        <button class="btn btn-primary" type="submit">Search</button>
      </div>
    </form>
    <div v-if="loading">Loading...</div>
    <div v-else-if="error">{{ error }}</div>
    <div v-else>
      <div class="row">
        <div class="col-md-4" v-for="product in products" :key="product.product_id">
          <div class="card mb-4">
            <img :src="product.product_image" class="card-img-top" alt="Product Image" />
            <div class="card-body">
              <h5 class="card-title">{{ product.product_title }}</h5>
              <p class="card-text">Lowest Price: ₹{{ product.product_lowest_price }}</p>
              <router-link :to="`/product/${product.product_id}`" class="btn btn-primary"
                >View Details</router-link
              >
              <button class="btn btn-secondary mt-2" @click="addToFavorites(product)">
                Add to Favorites
              </button>
            </div>
          </div>
        </div>
      </div>
      <h2>Favorites</h2>
      <ul class="list-group">
        <li class="list-group-item" v-for="fav in favorites" :key="fav.product_id">
          {{ fav.product_title }} - ₹{{ fav.product_lowest_price }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'
import axios from 'axios'

interface Product {
  product_title: string
  can_compare: boolean
  product_lowest_price: number
  product_link: string
  product_id: string
  product_category: string
  product_sub_category: string
  product_image: string
}

export default defineComponent({
  name: 'ProductsList',
  setup() {
    const products = ref<Product[]>([])
    const loading = ref(false)
    const error = ref<string | null>(null)
    const query = ref('')
    const favorites = ref<Product[]>([])

    const fetchProducts = async () => {
      loading.value = true
      error.value = null
      try {
        const response = await axios.get(
          `https://price-api.datayuge.com/api/v1/compare/search?api_key=AuFnY4Eqaw1KQALiiK3C08ZNun74sZ7hZAx&product=${query.value}`,
          {
            headers: {
              'Content-Type': 'application/json'
            }
          }
        )
        console.log('response detaik:', response.data.data)
        products.value = response.data.data
      } catch (err) {
        error.value = 'Failed to fetch products'
      } finally {
        loading.value = false
      }
    }

    const onSearch = () => {
      fetchProducts()
    }

    const addToFavorites = (product: Product) => {
      if (!favorites.value.some((fav) => fav.product_id === product.product_id)) {
        favorites.value.push(product)
      }
    }

    return {
      products,
      loading,
      error,
      query,
      onSearch,
      favorites,
      addToFavorites
    }
  }
})
</script>

<style scoped>
.container {
  max-width: 1000px;
  margin: auto;
  padding-top: 50px;
}
.card-img-top {
  height: 200px;
  object-fit: cover;
}
</style>
