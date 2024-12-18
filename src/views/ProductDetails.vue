<template>
  <div class="product-details">
    <div class="product-container">
      <!-- Images Section -->
      <div class="product-images">
        <div class="main-image-container">
          <img 
            :src="selectedImage" 
            :alt="product.name" 
            class="main-image"
            :class="{ 'zoomed': isZoomed }"
            @mouseover="isZoomed = true"
            @mouseleave="isZoomed = false"
          >
        </div>
        <div class="thumbnail-gallery">
          <div v-for="(image, index) in productImages" 
               :key="index" 
               class="thumbnail-wrapper"
               :class="{ active: selectedImageIndex === index }"
               @click="selectImage(index)">
            <img :src="image" :alt="`${product.name} - vue ${index + 1}`">
          </div>
        </div>
      </div>

      <!-- Product Info Section -->
      <div class="product-info">
        <h1>{{ product.name }}</h1>
        <p class="price">{{ selectedPrice }}€</p>

        <!-- Size Selection -->
        <div class="size-selection">
          <h3>Hauteur</h3>
          <div class="size-options">
            <button 
              v-for="(height, size) in product?.sizes" 
              :key="size"
              :class="{ 
                active: selectedSize === size,
                'out-of-stock': stocksParTaille[size] === 0
              }"
              @click="selectedSize = size"
              :disabled="stocksParTaille[size] === 0"
            >
              {{ height }}
              <span class="stock-info" :class="{ 'low-stock': stocksParTaille[size] <= 2 }">
                ({{ stocksParTaille[size] }} {{ stocksParTaille[size] <= 1 ? 'unité' : 'unités' }})
              </span>
            </button>
          </div>
        </div>

        <!-- Quantity -->
        <div class="quantity-selector">
          <label>Quantité</label>
          <div class="quantity-controls">
            <button 
              class="quantity-btn"
              @click="decrementQuantity" 
              :disabled="quantity <= 1"
            >
              <i class="fas fa-minus"></i>
            </button>
            <input 
              type="number" 
              v-model="quantity" 
              :max="stockDisponible"
              min="1"
              @input="validateQuantity"
              readonly
            >
            <button 
              class="quantity-btn"
              @click="incrementQuantity"
              :disabled="quantity >= stockDisponible"
            >
              <i class="fas fa-plus"></i>
            </button>
          </div>
          <p v-if="quantity >= stockDisponible" class="stock-warning">
            Stock limité à {{ stockDisponible }} {{ stockDisponible <= 1 ? 'unité' : 'unités' }}
          </p>
        </div>

        <div class="actions">
          <button 
            class="add-to-cart-btn" 
            @click="addToCart"
            :disabled="stocksParTaille[selectedSize] === 0"
          >
            {{ stocksParTaille[selectedSize] === 0 ? 'Rupture de stock' : 'Ajouter au panier' }}
          </button>
          <button 
            class="favorite-btn"
            @click="toggleFavorite(product.id, selectedSize)"
            :class="{ 'is-favorite': isInFavorites(product.id, selectedSize) }"
            >
              <i class="fas fa-heart"></i>
          </button>
        </div>

        <div class="product-details-tabs">
          <details>
            <summary>Description</summary>
            <p>{{ product.description }}</p>
          </details>
        </div>
      </div>
    </div>

    <!-- Utilisation du composant ProductRecommended -->
    <ProductRecommended :current-product-id="product?.id" />
  </div>
</template>

<!-- ProductDetails.vue -->
<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { useStore } from 'vuex'
import { useFavorites } from '@/composables/useFavorites'

import ProductRecommended from '@/components/ProductRecommended.vue'

// Import des images
import formeImage from '@/assets/images/2SAPIN/Forme.jpg'
import imageA from '@/assets/images/2SAPIN/A.webp'
import imageB from '@/assets/images/2SAPIN/B.webp'

const route = useRoute()
const store = useStore()
const { toggleFavorite, isInFavorites } = useFavorites()

const emit = defineEmits(['openCart'])

const generateRandomStock = () => Math.floor(Math.random() * 5) + 0

const selectedSize = ref('Moyen')
const quantity = ref(1)
const selectedImageIndex = ref(0)
const isZoomed = ref(false)

const stocksParTaille = ref({
  'Petit': generateRandomStock(),
  'Moyen': generateRandomStock(),
  'Grand': generateRandomStock()
})

const stockDisponible = computed(() => stocksParTaille.value[selectedSize.value])

const product = computed(() => 
  store.state.products.find(p => p.id === parseInt(route.params.id))
)

const selectedPrice = computed(() => 
  product.value?.prices[selectedSize.value]
)

const productImages = computed(() => {
  if (!product.value) return []
  
  const mainImage = product.value.image
  const secondaryImage = product.value.id % 2 === 0 ? imageA : imageB

  return [
    mainImage,
    formeImage,
    secondaryImage
  ]
})

const selectedImage = computed(() => 
  productImages.value[selectedImageIndex.value] || product.value?.image
)

// Ajout de onMounted pour gérer le scroll
onMounted(() => {
  window.scrollTo({ top: 0, behavior: 'smooth' })
})

const selectImage = (index) => {
  selectedImageIndex.value = index
}

const decrementQuantity = () => {
  if (quantity.value > 1) {
    quantity.value--
  }
}

const incrementQuantity = () => {
  if (quantity.value < stockDisponible.value) {
    quantity.value++
  }
}

const validateQuantity = () => {
  let value = parseInt(quantity.value)
  if (isNaN(value) || value < 1) {
    quantity.value = 1
  } else if (value > stockDisponible.value) {
    quantity.value = stockDisponible.value
  }
}

const addToCart = () => {
  if (product.value && selectedSize.value && quantity.value <= stockDisponible.value) {
    store.dispatch('addToCart', {
      product: product.value,
      selectedSize: selectedSize.value,
      quantity: quantity.value
    })
    emit('openCart')
  }
}
</script>

<style scoped>
.product-details {
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 2rem;
}

.product-container {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 4rem;
}

.product-images {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.main-image-container {
  position: relative;
  overflow: hidden;
  border-radius: 8px;
  aspect-ratio: 1;
}

.main-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
  transition: transform 0.3s ease;
}

.main-image.zoomed {
  transform: scale(1.2);
}

.thumbnail-gallery {
  display: flex;
  gap: 1rem;
  justify-content: center;
  margin-top: 1rem;
}

.thumbnail-wrapper {
  width: 80px;
  height: 80px;
  border: 2px solid #ddd;
  border-radius: 4px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s ease;
}

.thumbnail-wrapper.active {
  border-color: #c41e3a;
}

.thumbnail-wrapper:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.thumbnail-wrapper img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.thumbnail-wrapper:hover img {
  transform: scale(1.1);
}

.product-info h1 {
  font-size: 2rem;
  margin-bottom: 1rem;
}

.price {
  font-size: 1.5rem;
  color: #c41e3a;
  font-weight: bold;
  margin-bottom: 2rem;
}

.size-selection {
  margin-bottom: 2rem;
}

.size-options {
  display: flex;
  gap: 1rem;
  margin-top: 0.5rem;
}

.size-options button {
  padding: 0.5rem 1rem;
  border: 1px solid #ddd;
  background: none;
  cursor: pointer;
  border-radius: 4px;
  transition: all 0.3s ease;
}

.size-options button.active {
  border-color: #c41e3a;
  color: #c41e3a;
}

.quantity-selector {
  margin-bottom: 2rem;
}

.quantity-controls {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-top: 0.5rem;
  max-width: 150px;
}

.quantity-btn {
  width: 36px;
  height: 36px;
  border: 1px solid #ddd;
  background: white;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s ease;
}

.quantity-btn:hover:not(:disabled) {
  background: #f5f5f5;
  border-color: #c41e3a;
}

.quantity-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.quantity-controls input {
  width: 60px;
  height: 36px;
  text-align: center;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 1rem;
  -moz-appearance: textfield;
  appearance: textfield;
}

.quantity-controls input::-webkit-outer-spin-button,
.quantity-controls input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.stock-warning {
  color: #c41e3a;
  font-size: 0.9rem;
  margin-top: 0.5rem;
}

.actions {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
}

.add-to-cart-btn {
  flex: 1;
  padding: 1rem;
  background: #c41e3a;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.add-to-cart-btn:hover:not(:disabled) {
  background-color: #a01830;
}

.add-to-cart-btn:disabled {
  background-color: #ddd;
  cursor: not-allowed;
}

.favorite-btn {
  width: 50px;
  height: 50px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.favorite-btn i {
  font-size: 1.2rem;
  color: #ddd;
  transition: color 0.3s ease;
}

.favorite-btn.is-favorite i {
  color: #c41e3a;
}

.favorite-btn:hover {
  border-color: #c41e3a;
}

.favorite-btn:hover i {
  color: #c41e3a;
}

details {
  border-bottom: 1px solid #ddd;
  padding: 1rem 0;
}

summary {
  cursor: pointer;
  font-weight: bold;
}

@media (max-width: 768px) {
  .product-container {
    grid-template-columns: 1fr;
    gap: 2rem;
  }

  .thumbnail-wrapper {
    width: 60px;
    height: 60px;
  }
  
  .thumbnail-gallery {
    justify-content: center;
  }
}
</style>
