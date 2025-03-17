<template>
  <div class="product-detail" v-if="product">
    <h2>{{ product.name }}</h2>
    <div class="tov">
      <div class="onn">
        <img :src="currentImage" :alt="product.name" @error="imageError" v-if="!imageFailed" />
        <img v-else src="https://s3.amazonaws.com/images.ecwid.com/images/50826040/2645201940.jpg" alt="Запасное изображение" />
        <div class="thumbnail-container" v-if="![17, 18, 19].includes(Number(product.id))">
          <img
            v-for="(phone, index) in productImages"
            :key="index"
            :src="phone.img"
            :title="phone.color"
            alt="Миниатюра"
            class="thumbnail"
            @click="changeImage(phone.img)"
          />
        </div>
      </div>
      <div class="onn2">
        <p><strong></strong> {{ product.description }}</p>
        <div class="color-options" v-if="![17, 18, 19].includes(Number(product.id))">
          <button
            v-for="(phone, index) in productImages"
            :key="index"
            :style="{ backgroundColor: phone.color }"
            :title="phone.color"
            class="color-button"
            @click="changeImage(phone.img)"
          ></button>
        </div>
        <div class="ret">
          <p><strong></strong>  {{ product.currency }} {{ product.price }}.00</p>
          <p class="p3"><strong>★</strong> {{ product.rating }} / 5</p>
          
        </div>
        <div class="cart-controls">
          <button 
            :class="buttonClass" 
            @click="handleCartAction" 
            class="btn-cart"
          >
            {{ buttonText }}
          </button>
          <div v-if="isInCart" class="quantity-controls">
            <button @click="decreaseQuantity" class="quantity-button">-</button>
            <span class="quantity-text">{{ quantity }}</span>
            <button @click="increaseQuantity" class="quantity-button">+</button>
          </div>
        </div>
      </div>
    </div>
    <div class="info-section">
      <div v-if="isAdmin && isEditing">
        <h3>Редактировать товар</h3>
        <input v-model="editProduct.name" placeholder="Название" required />
        <input v-model="editProduct.description" placeholder="Описание" required />
        <input v-model.number="editProduct.price" type="number" placeholder="Цена" required />
        <input v-model="editProduct.currency" placeholder="Валюта" required />
        <input v-model="editProduct.image" placeholder="URL изображения" required />
        <input v-model="editProduct.brand" placeholder="Бренд" required />
        <input v-model="editProduct.category" placeholder="Категория" required />
        <div class="edit-buttons">
          <button @click="saveProduct" class="save-button">Сохранить</button>
          <button @click="isEditing = false" class="cancel-button">Отмена</button>
        </div>
      </div>
      <div v-else>
        <p><strong>Бренд:</strong> {{ product.brand || 'Не указан' }}</p>
        <p><strong>Категория:</strong> {{ product.category || 'Не указана' }}</p>
        <p><strong>Объем памяти:</strong> {{ product.volume.gb1 || '0' }} ГБ, {{ product.volume.gb2 || '0' }} ГБ, {{ product.volume.gb3 || '0' }} ГБ</p>
        <button @click="goBack" class="back-button">Назад</button>
        <button v-if="isAdmin" @click="toggleEditMode" class="edit-button">Редактировать</button>
      </div>
    </div>
    <div class="comments-section">
      <h3>Комментарии</h3>
      <div v-if="comments.length === 0" class="no-comments">
        <p>Комментариев пока нет. Будьте первым!</p>
      </div>
      <ul v-else class="comment-list">
        <li v-for="comment in comments" :key="comment.id" class="comment">
          <strong>{{ comment.user }}</strong>: {{ comment.text }}
          <span class="comment-date">{{ formatDate(comment.date) }}</span>
        </li>
      </ul>
      <div class="add-comment">
        <textarea v-model="newComment" placeholder="Оставьте ваш комментарий..." rows="3"></textarea>
        <button @click="addComment" class="submit-comment">Отправить</button>
      </div>
    </div>
  </div>
  <div v-else>
    <p>Данные товара не найдены.</p>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  data() {
    return {
      product: null,
      imageFailed: false,
      currentImage: '',
      quantity: 0,
      userRating: 0,
      hoverRating: 0,
      comments: [],
      newComment: '',
      cart: [],
      userEmail: localStorage.getItem('userEmail') || null,
      isEditing: false,
      editProduct: {},
    };
  },
  computed: {
    productImages() {
      const images = [
        this.product?.phone1,
        this.product?.phone2,
        this.product?.phone3,
        this.product?.phone4,
      ].filter(phone => phone && phone.img);
      return images.length > 0 ? images : [{ img: 'https://via.placeholder.com/150', color: 'default' }];
    },
    isInCart() {
      return !!this.cart.find(item => item.id === this.product?.id);
    },
    buttonText() {
      return this.isInCart ? `В корзине (${this.quantity})` : 'Добавить в корзину';
    },
    buttonClass() {
      return this.isInCart ? 'btn-in-cart' : 'btn-add-to-cart';
    },
    isAdmin() {
      return localStorage.getItem('isAdmin') === 'true';
    },
    isAuthenticated() {
      return !!this.userEmail;
    }
  },
  created() {
    this.loadProduct();
    this.loadUserRating();
    this.loadComments();
    this.loadCart();
  },
  methods: {
    loadProduct() {
      const productId = this.$route.params.id;
      const storedProduct = localStorage.getItem('currentProduct');
      if (storedProduct) {
        this.product = JSON.parse(storedProduct);
        this.currentImage = this.product.phone1?.img || 'https://via.placeholder.com/150';
        this.editProduct = { ...this.product };
      } else {
        this.fetchProductFromAPI(productId);
      }
    },
    async fetchProductFromAPI(productId) {
      try {
        const response = await fetch(`https://67bc472bed4861e07b39e4ca.mockapi.io/product/${productId}`);
        if (!response.ok) throw new Error("Продукт не найден");
        const product = await response.json();
        this.product = {
          id: product.id,
          name: product.name,
          description: product.description,
          brand: product.brand,
          category: product.category,
          price: product.price,
          currency: product.currency,
          volume: product.volume || { gb1: "0", gb2: "0", gb3: "0" },
          rating: product.rating,
          phone1: product.phone1 || { img: 'https://via.placeholder.com/150', color: 'default' },
          phone2: product.phone2,
          phone3: product.phone3,
          phone4: product.phone4,
        };
        this.currentImage = this.product.phone1.img;
        this.editProduct = { ...this.product };
        localStorage.setItem('currentProduct', JSON.stringify(this.product));
      } catch (error) {
        console.error('Ошибка загрузки продукта:', error);
        this.product = null;
      }
    },
    imageError() {
      this.imageFailed = true;
    },
    changeImage(img) {
      this.currentImage = img;
    },
    loadCart() {
      if (!this.userEmail) {
        this.cart = [];
        return;
      }
      const cart = JSON.parse(localStorage.getItem('cart')) || {};
      this.cart = cart[this.userEmail] || [];
      const productInCart = this.cart.find(item => item.id === this.product?.id);
      this.quantity = productInCart ? productInCart.quantity : 0;
    },
    handleCartAction() {
      if (!this.isAuthenticated) {
        this.$swal('Ошибка', 'Пожалуйста, авторизуйтесь, чтобы добавлять товары в корзину.', 'error');
        return;
      }
      
      const productInCartIndex = this.cart.findIndex((item) => item.id === this.product.id);
      if (productInCartIndex !== -1) {
        this.$router.push('/basket');
      } else {
        const newItem = { ...this.product, quantity: 1 };
        this.cart.push(newItem);
        this.quantity = 1;
        this.saveCart();
        this.$swal('Успех', 'Товар добавлен в корзину!', 'success');
      }
    },
    increaseQuantity() {
      if (!this.isAuthenticated) {
        this.$swal('Ошибка', 'Пожалуйста, авторизуйтесь.', 'error');
        return;
      }
      const productInCart = this.cart.find((item) => item.id === this.product.id);
      if (productInCart) {
        productInCart.quantity += 1;
        this.quantity = productInCart.quantity;
        this.saveCart();
      }
    },
    decreaseQuantity() {
      if (!this.isAuthenticated) {
        this.$swal('Ошибка', 'Пожалуйста, авторизуйтесь.', 'error');
        return;
      }
      const productInCartIndex = this.cart.findIndex((item) => item.id === this.product.id);
      if (productInCartIndex !== -1) {
        const productInCart = this.cart[productInCartIndex];
        if (productInCart.quantity > 1) {
          productInCart.quantity -= 1;
          this.quantity = productInCart.quantity;
        } else {
          this.cart.splice(productInCartIndex, 1);
          this.quantity = 0;
        }
        this.saveCart();
      }
    },
    saveCart() {
      const cart = JSON.parse(localStorage.getItem('cart')) || {};
      cart[this.userEmail] = this.cart;
      localStorage.setItem('cart', JSON.stringify(cart));
    },
    loadUserRating() {
      this.userRating = this.product?.rating || 0;
    },
    rateProduct(rating) {
      if (!this.isAuthenticated) {
        this.$swal('Ошибка', 'Пожалуйста, авторизуйтесь, чтобы оставить рейтинг.', 'error');
        return;
      }
      this.userRating = rating;
      this.product.rating = rating;
    },
    loadComments() {
      const storedComments = localStorage.getItem(`comments_${this.$route.params.id}`);
      this.comments = storedComments ? JSON.parse(storedComments) : [];
    },
    addComment() {
      if (!this.isAuthenticated) {
        this.$swal('Ошибка', 'Пожалуйста, авторизуйтесь, чтобы оставить комментарий.', 'error');
        return;
      }
      if (!this.newComment.trim()) {
        this.$swal('Ошибка', 'Комментарий не может быть пустым.', 'error');
        return;
      }
      const comment = {
        id: Date.now(),
        user: this.userEmail,
        text: this.newComment,
        date: new Date().toISOString(),
      };
      this.comments.push(comment);
      localStorage.setItem(`comments_${this.$route.params.id}`, JSON.stringify(this.comments));
      this.newComment = '';
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString('ru-RU', {
        day: '2-digit',
        month: '2-digit',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
      });
    },
    goBack() {
      this.$router.go(-1);
    },
    toggleEditMode() {
      this.isEditing = !this.isEditing;
      if (!this.isEditing) {
        this.editProduct = { ...this.product };
      }
    },
    async saveProduct() {
      try {
        const response = await axios.put(
          `https://67bc472bed4861e07b39e4ca.mockapi.io/product/${this.product.id}`,
          {
            name: this.editProduct.name,
            description: this.editProduct.description,
            price: this.editProduct.price,
            currency: this.editProduct.currency,
            phone1: { img: this.editProduct.image },
            brand: this.editProduct.brand,
            category: this.editProduct.category,
          }
        );
        this.product = {
          ...this.product,
          name: response.data.name,
          description: response.data.description,
          price: response.data.price,
          currency: response.data.currency,
          image: response.data.phone1.img,
          brand: response.data.brand,
          category: response.data.category,
        };
        localStorage.setItem('currentProduct', JSON.stringify(this.product));
        const adminProducts = JSON.parse(localStorage.getItem('adminProducts')) || [];
        const index = adminProducts.findIndex(p => p.id === this.product.id);
        if (index !== -1) {
          adminProducts[index] = this.product;
        } else {
          adminProducts.push(this.product);
        }
        localStorage.setItem('adminProducts', JSON.stringify(adminProducts));
        this.isEditing = false;
        this.$swal('Успех', 'Товар успешно отредактирован!', 'success');
      } catch (error) {
        console.error('Ошибка при сохранении:', error);
        this.$swal('Ошибка', 'Не удалось обновить товар в API. Сохранено локально.', 'warning');
        this.product = { ...this.editProduct };
        localStorage.setItem('currentProduct', JSON.stringify(this.product));
        const adminProducts = JSON.parse(localStorage.getItem('adminProducts')) || [];
        const index = adminProducts.findIndex(p => p.id === this.product.id);
        if (index !== -1) {
          adminProducts[index] = this.product;
        } else {
          adminProducts.push(this.product);
        }
        localStorage.setItem('adminProducts', JSON.stringify(adminProducts));
        this.isEditing = false;
      }
    },
  },
};
</script>

<style scoped>
.product-detail {
  margin: 50px auto;
  padding: 50px;
  max-width: 75%;
  background: linear-gradient(135deg, #ffffff 0%, #f8f9fa 100%);
  border-radius: 24px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.06), 0 5px 15px rgba(0, 0, 0, 0.02);
  border: 1px solid rgba(236, 240, 241, 0.5);
}

.product-detail h2 {
  font-size: 40px;
  margin-bottom: 35px;
  color: #1a2a44;
  font-weight: 800;
  text-align: center;
  letter-spacing: 0.8px;
  text-transform: uppercase;
}

.tov {
  display: flex;
  gap: 50px;
  padding-bottom: 40px;
}

.onn{
  max-width: 40%;
}
.onn2 {
  width: 50%;
}

.onn {
  background: #fdfdfd;
  padding: 30px;
  border-radius: 20px;
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.04);
  transition: transform 0.4s cubic-bezier(0.25, 0.8, 0.25, 1), box-shadow 0.3s ease;
  display: flex
;
    flex-wrap: wrap;
    justify-content: center;
}

.onn:hover {
  transform: translateY(-10px);
  box-shadow: 0 12px 35px rgba(0, 0, 0, 0.07);
}

.product-detail img {
  max-width: 100%;
  height: auto;
  border-radius: 14px;
  transition: transform 0.4s cubic-bezier(0.25, 0.8, 0.25, 1);
}

.onn:hover img {
  transform: scale(1.06);
}

.color-options {
  display: flex;
  gap: 14px;
  margin-top: 25px;
  
}

.color-button {
  width: 38px;
  height: 38px;
  border: 2px solid rgba(200, 200, 200, 0.8);
  border-radius: 50%;
  cursor: pointer;
  position: relative;
  transition: transform 0.3s ease, border-color 0.3s ease, box-shadow 0.3s ease;
}

.color-button:hover,
.color-button:focus {
  transform: scale(1.2);
  border-color: #444;
  box-shadow: 0 0 12px rgba(0, 0, 0, 0.2), 0 0 0 4px rgba(100, 100, 100, 0.1);
  outline: none;
}

.color-button:hover::after {
  content: '';
  position: absolute;
  top: -6px;
  left: -6px;
  right: -6px;
  bottom: -6px;
  border-radius: 50%;
  background: rgba(0, 0, 0, 0.05);
  animation: pulse 1.5s infinite ease-in-out;
}

@keyframes pulse {
  0% {
    transform: scale(1);
    opacity: 0.5;
  }
  50% {
    transform: scale(1.2);
    opacity: 0.2;
  }
  100% {
    transform: scale(1);
    opacity: 0.5;
  }
}

.thumbnail-container {
  display: flex;
  margin-top: 25px;
  gap: 14px;
  justify-content: center;
}

.thumbnail {
  width: 100px;
  height: auto;
  cursor: pointer;
  border: 2px solid rgba(200, 200, 200, 0.8);
  border-radius: 12px;
  transition: transform 0.3s ease, border-color 0.3s ease, box-shadow 0.3s ease;
}

.thumbnail:hover,
.thumbnail:focus {
  transform: scale(1.12);
  border-color: #444;
  box-shadow: 0 0 12px rgba(0, 0, 0, 0.15);
  outline: none;
}

.onn2 p {
  font-size: 18px;
  color: #5c6b80;
  line-height: 1.8;
  margin-bottom: 30px;
  font-weight: 400;
}

.ret {
  margin: 30px 0;
}

.ret p {
  font-size: 20px;
  color: #1a2a44;
  font-weight: 600;
}

.p3 {
  margin: 14px 0;
}

.star-rating {
  display: inline-flex;
  margin-top: 14px;
  cursor: pointer;
}

.star {
  color: #e0e5ec;
  font-size: 32px;
  transition: color 0.2s ease, transform 0.2s ease;
}

.star.filled {
  color: #f39c12;
}

.star:hover {
  transform: scale(1.2);
}

.btn-cart,
.back-button,
.submit-comment {
  background: linear-gradient(135deg, #95a5a6 0%, #7f8c8d 100%);
  color: #fff;
  border: none;
  border-radius: 35px;
  padding: 16px 40px;
  font-size: 18px;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.3s ease, transform 0.3s ease, box-shadow 0.3s ease;
}

.btn-cart:hover,
.back-button:hover,
.submit-comment:hover {
  background: linear-gradient(135deg, #7f8c8d 0%, #636e72 100%);
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.12);
}

.btn-in-cart {
  background: linear-gradient(135deg, #7f8c8d 0%, #636e72 100%);
}

.quantity-controls {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 25px;
}

.quantity-button {
  width: 48px;
  height: 48px;
  background: #2c3e50;
  color: #fff;
  border: none;
  border-radius: 50%;
  font-size: 24px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.3s ease, transform 0.3s ease, box-shadow 0.3s ease;
}

.quantity-button:hover {
  background: #233544;
  transform: scale(1.12);
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

.quantity-text {
  margin: 0 20px;
  font-size: 24px;
  font-weight: 700;
  color: #1a2a44;
}

.info-section {
  margin-top: 50px;
  padding: 35px;
  background: linear-gradient(135deg, #fdfdfd 0%, #f5f6f8 100%);
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.04);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.info-section:hover {
  transform: translateY(-8px);
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.07);
}

.info-section p {
  margin-bottom: 20px;
  font-size: 18px;
  color: #5c6b80;
  font-weight: 400;
}

.comments-section {
  margin-top: 60px;
  padding: 35px;
  background: linear-gradient(135deg, #fdfdfd 0%, #f5f6f8 100%);
  border-radius: 20px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.04);
}

.comments-section h3 {
  font-size: 28px;
  margin-bottom: 30px;
  color: #1a2a44;
  font-weight: 700;
}

.no-comments p {
  font-size: 18px;
  color: #95a5a6;
  text-align: center;
  font-weight: 400;
}

.comment-list {
  list-style: none;
  padding: 0;
}

.comment {
  padding: 20px;
  margin-bottom: 20px;
  background: #fff;
  border-radius: 14px;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.03);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.comment:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.06);
}

.comment strong {
  color: #1a2a44;
  font-weight: 700;
}

.comment-date {
  display: block;
  font-size: 14px;
  color: #bdc3c7;
  margin-top: 8px;
  font-weight: 300;
}

.add-comment {
  margin-top: 30px;
}

.add-comment textarea {
  width: 100%;
  padding: 20px;
  border: 2px solid rgba(200, 200, 200, 0.8);
  border-radius: 14px;
  font-size: 17px;
  resize: vertical;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

.add-comment textarea:focus {
  border-color: #636e72;
  box-shadow: 0 0 12px rgba(0, 0, 0, 0.08);
  outline: none;
}

.submit-comment {
  display: block;
  margin: 25px auto 0;
  padding: 16px 45px;
}
.info-section input {
  width: 100%;
  padding: 0.75rem;
  margin-bottom: 1rem;
  border: 2px solid rgba(200, 200, 200, 0.8);
  border-radius: 12px;
  font-size: 17px;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

.info-section input:focus {
  border-color: #636e72;
  box-shadow: 0 0 12px rgba(0, 0, 0, 0.08);
  outline: none;
}

.edit-buttons {
  display: flex;
  gap: 1rem;
  justify-content: center;
  margin-top: 1.5rem;
}

.save-button,
.cancel-button,
.edit-button {
  padding: 0.75rem 2rem;
  border: none;
  border-radius: 35px;
  color: #fff;
  font-size: 18px;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.3s ease, transform 0.3s ease, box-shadow 0.3s ease;
}

.save-button {
  background: linear-gradient(135deg, #72aec8 0%, #5a96b0 100%);
}

.save-button:hover {
  background: linear-gradient(135deg, #5a96b0 0%, #4682b4 100%);
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.12);
}

.cancel-button {
  background: linear-gradient(135deg, #e74c3c 0%, #c0392b 100%);
}

.cancel-button:hover {
  background: linear-gradient(135deg, #c0392b 0%, #a93226 100%);
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.12);
}

.edit-button {
  background: linear-gradient(135deg, #f39c12 0%, #e67e22 100%);
  margin-top: 1rem;
}

.edit-button:hover {
  background: linear-gradient(135deg, #e67e22 0%, #d35400 100%);
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.12);
}
</style>