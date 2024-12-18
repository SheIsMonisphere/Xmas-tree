<template>
  <div class="catalog-page">
    <div class="catalog-header">
      <h1>Nos Sapins de Noël</h1>

      <div class="filters-wrapper">
        <div class="filters">
          <div class="filter-group">
            <label for="category">Catégorie</label>
            <select id="category" v-model="filters.category">
              <option value="">Toutes les catégories</option>
              <option value="Premium">Premium</option>
              <option value="Nordic">Nordic</option>
              <option value="Classique">Classique</option>
              <option value="Enneigé">Enneigé</option>
              <option value="Naturel">Naturel</option>
            </select>
          </div>

          <div class="filter-group">
            <label for="sort">Tri</label>
            <select id="sort" v-model="filters.sort">
              <option value="">Par défaut</option>
              <option value="asc">Prix croissant</option>
              <option value="desc">Prix décroissant</option>
            </select>
          </div>
        </div>
      </div>
    </div>

    <div class="products-grid">
      <div v-for="product in filteredProducts" :key="product.id" class="product-card">
        <div class="product-image">
          <span v-if="product.discount" class="discount">{{ product.discount }}</span>
          <img :src="product.image" :alt="product.name">
          <div class="quick-actions">
            <button class="favorite-btn" @click="toggleFavorite(product.id)" :class="{ 'is-favorite': isInFavorites(product.id) }">
              <i class="fas fa-heart"></i>
            </button>
          </div>
        </div>

        <div class="product-info">
          <h3>{{ product.name }}</h3>
          <p class="description">{{ product.description }}</p>

          <div class="pricing-section">
            <div class="price-display" v-if="selectedSizes[product.id]">
              <span class="current-price">{{ formatPrice(product.prices[selectedSizes[product.id]]) }}€</span>
              <span class="discount-label" v-if="product.discount">
                {{ calculateOriginalPrice(product.prices[selectedSizes[product.id]], product.discount) }}€
              </span>
            </div>
          </div>

          <div class="size-selector">
            <label for="size">Taille</label>
            <div class="size-buttons">
              <button v-for="(height, size) in product.sizes" :key="size" class="size-btn" :class="{ 'selected': selectedSizes[product.id] === size }" @click="selectSize(product.id, size)">
                {{ size }} ({{ height }}) - {{ formatPrice(product.prices[size]) }}€
              </button>
            </div>
          </div>

          <button class="add-to-cart" @click="addToCart(product)" :disabled="!selectedSizes[product.id]">
            <i class="fas fa-shopping-cart"></i>
            Ajouter au panier
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import { useStore } from 'vuex'
import { useFavorites } from '@/composables/useFavorites'

const store = useStore()
const { isInFavorites, toggleFavorite } = useFavorites()
const selectedSizes = ref({})
const filters = ref({
  category: '',
  sort: ''
})

const filteredProducts = computed(() => {
  return store.getters.filteredProducts
})

const formatPrice = (price) => {
  return parseFloat(price).toFixed(2)
}

const calculateOriginalPrice = (price, discount) => {
  const discountValue = parseInt(discount.replace('-', '').replace('%', ''))
  const originalPrice = price / (1 - discountValue/100)
  return formatPrice(originalPrice)
}

const selectSize = (productId, size) => {
  selectedSizes.value[productId] = size
}

const addToCart = (product) => {
  if (selectedSizes.value[product.id]) {
    store.dispatch('addToCart', {
      product,
      selectedSize: selectedSizes.value[product.id],
      quantity: 1
    })
  }
}

watch(filters, (newFilters) => {
  store.dispatch('updateFilters', newFilters)
}, { deep: true })
</script>

<style scoped>
.catalog-page {
  padding: 2rem;
  max-width: 1400px;
  margin: 0 auto;
}

.catalog-header {
  margin-bottom: 2rem;
}

h1 {
  color: #0B3B24;
  font-size: 2.5rem;
  text-align: center;
  margin-bottom: 2rem;
}

.filters-wrapper {
  background: white;
  padding: 1rem;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.05);
  margin-bottom: 2rem;
}

.filters {
  display: flex;
  gap: 1.5rem;
  justify-content: center;
}

.filter-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.filter-group label {
  font-size: 0.9rem;
  color: #666;
  font-weight: 500;
}

.filters select {
  padding: 0.75rem;
  border: 1px solid #ddd;
  border-radius: 6px;
  min-width: 200px;
  appearance: none;
  background: url("data:image/svg+xml,...") no-repeat right 0.75rem center/8px 10px;
  background-color: white;
  cursor: pointer;
  transition: border-color 0.3s;
}

.filters select:hover {
  border-color: #0B3B24;
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 2rem;
}

.product-card {
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.product-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.15);
}

.product-image {
  position: relative;
  padding-top: 100%;
  background: #f8f8f8;
}

.product-image img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: contain;
  padding: 1rem;
}

.quick-actions {
  position: absolute;
  top: 10px;
  right: 10px;
  z-index: 2;
}

.favorite-btn {
  background: white;
  border: none;
  width: 36px;
  height: 36px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  transition: all 0.3s ease;
}

.favorite-btn i {
  color: #ddd;
  transition: color 0.3s ease;
}

.favorite-btn.is-favorite i {
  color: #c41e3a;
}

.discount {
  position: absolute;
  top: 10px;
  left: 10px;
  background: #c41e3a;
  color: white;
  padding: 0.5rem 0.75rem;
  border-radius: 6px;
  font-weight: 500;
  z-index: 2;
}

.product-info {
  padding: 1.5rem;
}

.product-info h3 {
  font-size: 1.1rem;
  color: #333;
  margin-bottom: 0.5rem;
}

.description {
  color: #666;
  font-size: 0.9rem;
  margin-bottom: 1rem;
  line-height: 1.4;
}

.pricing-section {
  margin: 1rem 0;
}

.price-display {
  display: flex;
  gap: 0.75rem;
  align-items: center;
}

.current-price {
  font-size: 1.25rem;
  font-weight: 600;
  color: #c41e3a;
}

.discount-label {
  color: #999;
  text-decoration: line-through;
  font-size: 0.9rem;
}

.size-selector {
  margin-bottom: 1.5rem;
}

.size-selector label {
  display: block;
  margin-bottom: 0.5rem;
  color: #666;
  font-size: 0.9rem;
}

.size-buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.size-btn {
  padding: 0.5rem 1rem;
  border: 1px solid #ddd;
  border-radius: 6px;
  background-color: #f8f8f8;
  cursor: pointer;
  transition: all 0.3s ease;
}

.size-btn.selected {
  border-color: #0B3B24;
  background-color: #0B3B24;
  color: white;
}

.add-to-cart {
  width: 100%;
  padding: 1rem;
  background: #0B3B24;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 500;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
  transition: background-color 0.3s;
}

.add-to-cart:hover {
  background: #0a2e1c;
}

.add-to-cart:disabled {
  background: #ddd;
  cursor: not-allowed;
}

@media (max-width: 768px) {
  .catalog-page {
    padding: 1rem;
  }

  .filters {
    flex-direction: column;
    gap: 1rem;
  }

  .products-grid {
    grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  }

  h1 {
    font-size: 2rem;
  }
}
</style>