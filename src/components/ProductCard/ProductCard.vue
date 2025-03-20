//ProductCard
<template>
  <div class="product-card" @click="goToProductDetail">
    <div v-if="isLoading" class="skeleton-loader">
      <div class="skeleton-image"></div>
      <div class="skeleton-text"></div>
      <div class="skeleton-text short"></div>
    </div>
    <template v-else>
      <div class="image-container">
        <img
          v-if="!imageFailed"
          :src="imageUrl"
          alt="Изображение товара"
          @load="imageLoaded"
          @error="imageError"
        />
        <img v-else :src="fallbackImageUrl" alt="Запасное изображение" />
      </div>
      <div class="content">
        <div class="title-container">
          <h3>{{ product.title.length > 20 ? product.title.slice(0, 20) + '...' : product.title }}</h3>
          <p class="description">{{ product.description?.slice(0, 50) }}...</p>
        </div>
        <div class="price-container">
          <p class="price">{{ product.price }} {{ product.currency }}</p>
        </div>
      </div>
    </template>
  </div>
</template>

<script>
export default {
  props: {
    product: Object,
  },
  data() {
    return {
      isLoading: true,
      imageFailed: false,
      loadTimer: null,
      fallbackImageUrl: 'https://s3.amazonaws.com/images.ecwid.com/images/50826040/2645201940.jpg',
    };
  },
  computed: {
    imageUrl() {
      return this.product.image && this.product.image.trim() !== ''
        ? this.product.image
        : this.fallbackImageUrl;
    },
    isInCart() {
      const userEmail = localStorage.getItem('userEmail');
      if (!userEmail) return false;
      const cart = JSON.parse(localStorage.getItem('cart')) || {};
      const userCart = cart[userEmail] || [];
      return !!userCart.find(item => item.id === this.product.id);
    },
  },
  mounted() {
    this.loadTimer = setTimeout(() => {
      if (this.isLoading) {
        this.isLoading = false;
      }
    }, 1000);
    window.addEventListener('storage', this.checkCartState);
  },
  methods: {
    goToProductDetail() {
      localStorage.setItem('currentProduct', JSON.stringify(this.product));
      this.$router.push({ path: `/product/${this.product.id}` });
    },
    imageLoaded() {
      clearTimeout(this.loadTimer);
      this.isLoading = false;
    },
    imageError() {
      clearTimeout(this.loadTimer);
      this.isLoading = false;
      this.imageFailed = true;
    },
    checkCartState() {
      this.$forceUpdate();
    },
  },
  beforeUnmount() {
    clearTimeout(this.loadTimer);
    window.removeEventListener('storage', this.checkCartState);
  },
};
</script>

<style scoped>
.product-card {
  height: 264px;
  position: relative;
  background: white;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.175);
  transition: all 0.3s ease;
  cursor: pointer;
  display: flex;
  flex-direction: column;
  max-width: 260px;
}

.product-card:hover {
  transform: translateY(-1px);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.104);
}

.image-container {
  position: relative;
  width: 100%;
  height: 180px;
  overflow: hidden;
}

.product-card img {
  width: 100%;
  height: 100%;
  object-fit: contain;
  transition: transform 0.3s ease;
}

.product-card:hover img {
  
}

.content {
  padding: 1rem;
  display: flex;
  justify-content: space-between;
  flex-grow: 1;
  gap: 0.5rem;
}

.title-container {
  flex: 1;
}

.product-card h3 {
  font-size: 1rem;
  font-weight: 500;
  color: #333;
  margin: 0 0 0.3rem 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.product-card:hover h3 {
  color: #555;
}

.description {
  font-size: 0.75rem;
  color: #666;
  margin: 0;
  line-height: 1.2;
}

.price-container {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.3rem;
}

.price {
  font-size: 0.95rem;
  font-weight: 600;
  color: #007bff;
  margin: 0;
}

.skeleton-loader {
  padding: 1rem;
  flex-grow: 1;
}

.skeleton-image {
  width: 100%;
  height: 180px;
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: loading 1.5s infinite;
  border-radius: 8px;
}

.skeleton-text {
  height: 12px;
  background: #f0f0f0;
  border-radius: 4px;
  animation: loading 1.5s infinite;
  margin: 0.4rem 0;
}

.skeleton-text.short {
  width: 60%;
}

@keyframes loading {
  0% { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

@media (max-width: 768px) {
  .product-card {
    max-width: 200px;
  }
  
  .image-container {
    height: 140px;
  }
  
  .product-card h3 {
    font-size: 0.9rem;
  }
  
  .description {
    font-size: 0.7rem;
  }
  
  .price {
    font-size: 0.85rem;
  }
  
  .content {
    padding: 0.75rem;
  }
  
  .skeleton-image {
    height: 140px;
  }
  
  .skeleton-loader {
    padding: 0.75rem;
  }
}
</style>