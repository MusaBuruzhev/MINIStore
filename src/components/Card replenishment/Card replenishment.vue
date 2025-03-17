//Card replenishment.vue
<template>
  <div class="card-replenishment">
    <h2>Пополнение баланса</h2>
    <div class="payment-methods">
      <div class="payment-method sbp" @click="selectMethod('СБП')">
        <img src="/src/components/Bank/e45f1b4ff1d7934d218ba1861abe9a0c.jpg" alt="СБП" />
      </div>
      <div class="payment-method tinkoff" @click="selectMethod('Тинькофф')">
        <img src="/src/components/Bank/a172d800-7679-11ea-8287-c488c87d1c68.png" alt="Тинькофф" />
      </div>
      <div class="payment-method vtb" @click="selectMethod('ВТБ')">
        <img src="/src/components/Bank/1200x630wa.png" alt="ВТБ" />

      </div>
      <div class="payment-method ozon" @click="selectMethod('Озон Банк')">
        <img src="/src/components/Bank/6528604949.jpg" alt="Озон Банк" />
      </div>
    </div>
    <div class="selected-method">
      Выбранный способ: <strong>{{ selectedMethod || 'не выбран' }}</strong>
    </div>
    <form @submit.prevent="submitCardReplenishment">
      <div class="form-group">
        <label for="cardholderName">Имя и фамилия</label>
        <input
          v-model="cardholderName"
          type="text"
          id="cardholderName"
          placeholder="Введите имя и фамилию"
          required
        />
      </div>
      <div class="form-group">
        <label for="cardNumber">Номер карты</label>
        <input
          v-model="cardNumber"
          type="text"
          id="cardNumber"
          placeholder="Введите номер карты"
          required
          minlength="13"
          maxlength="18"
          oninput="this.value = this.value.replace(/[^0-9]/g, '');"
        />
      </div>
      <div class="form-group">
        <label for="expiryDate">Срок действия</label>
        <input
          v-model="expiryDate"
          type="text"
          id="expiryDate"
          placeholder="MM/YY"
          required
          minlength="5"
          maxlength="5"
          oninput="this.value = this.value.replace(/[^0-9]/g, '').replace(/^(\d{2})(\d{0,2})/, '$1/$2');"
        />
      </div>
      <div class="form-group">
        <label for="cvv">CVV</label>
        <input
          v-model="cvv"
          type="text"
          id="cvv"
          placeholder="CVV"
          required
          minlength="3"
          maxlength="3"
          oninput="this.value = this.value.replace(/[^0-9]/g, '');"
        />
      </div>
      <div class="form-group">
        <label for="amount">Сумма пополнения</label>
        <input
          v-model="amount"
          type="number"
          id="amount"
          placeholder="От 500 до 100000 руб"
          required
          min="500"
          max="100000"
          oninput="this.value = this.value.slice(0, 6);"
          class="no-arrows"
        />
      </div>
      <label class="checkbox-label">
        <input
          type="checkbox"
          v-model="saveData"
          class="custom-checkbox"
        />
        <span>Сохранить данные для повторного пополнения</span>
      </label>
      <div class="mor">
        <button type="submit">Пополнить</button>
        <button class="back-button" @click="goBack">Назад</button>
      </div>
    </form>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cardholderName: '',
      cardNumber: '',
      expiryDate: '',
      cvv: '',
      amount: null,
      selectedMethod: '',
      saveData: false,
    };
  },
  methods: {
    selectMethod(method) {
      this.selectedMethod = method;
    },
    validateExpiryDate(date) {
      if (!date.includes('/')) {
        return false;
      }

      const [month, year] = date.split('/');

      if (!/^\d+$/.test(month) || !/^\d+$/.test(year)) {
        return false;
      }

      const cardMonth = parseInt(month, 10);
      const cardYear = parseInt(year, 10);

      if (cardMonth < 1 || cardMonth > 12) {
        return false;
      }

      if (cardYear < 0 || cardYear > 99) {
        return false;
      }

      return true;
    },
    async submitCardReplenishment() {
      if (this.amount < 500 || this.amount > 100000) {
        this.$swal({
          icon: 'error',
          title: 'Ошибка',
          text: 'Сумма пополнения должна быть от 500 до 100000 рублей',
        });
        return;
      }
      if (!this.validateExpiryDate(this.expiryDate)) {
        this.$swal({
          icon: 'error',
          title: 'Ошибка',
          text: 'Некорректный срок действия карты',
        });
        return;
      }
      const users = JSON.parse(localStorage.getItem('users')) || [];
      const userEmail = localStorage.getItem('userEmail');
      const userIndex = users.findIndex(user => user.email === userEmail);
      if (userIndex !== -1) {
        users[userIndex].balance = (users[userIndex].balance || 0) + Number(this.amount);
        localStorage.setItem('users', JSON.stringify(users));
        localStorage.setItem('currentUser', JSON.stringify(users[userIndex]));
        await this.$swal({
          icon: 'success',
          title: 'Успех!',
          text: 'Вы успешно пополнили баланс!',
        });
        if (this.saveData) {
          const savedData = {
            cardholderName: this.cardholderName,
            cardNumber: this.cardNumber,
            expiryDate: this.expiryDate,
            cvv: this.cvv,
          };
          localStorage.setItem('savedPaymentData', JSON.stringify(savedData));
        }
        this.goBack();
      }
    },
    goBack() {
      this.$router.push('/profile');
    },
    saveFormData() {
      const formData = {
        cardholderName: this.cardholderName,
        cardNumber: this.cardNumber,
        expiryDate: this.expiryDate,
        cvv: this.cvv,
        amount: this.amount,
        selectedMethod: this.selectedMethod,
        saveData: this.saveData,
      };
      localStorage.setItem('tempFormData', JSON.stringify(formData));
    },
  },
  mounted() {
    const savedData = JSON.parse(localStorage.getItem('savedPaymentData')) || {};
    this.cardholderName = savedData.cardholderName || '';
    this.cardNumber = savedData.cardNumber || '';
    this.expiryDate = savedData.expiryDate || '';
    this.cvv = savedData.cvv || '';
    const tempFormData = JSON.parse(localStorage.getItem('tempFormData')) || {};
    this.cardholderName = tempFormData.cardholderName || this.cardholderName;
    this.cardNumber = tempFormData.cardNumber || this.cardNumber;
    this.expiryDate = tempFormData.expiryDate || this.expiryDate;
    this.cvv = tempFormData.cvv || this.cvv;
    this.amount = tempFormData.amount || this.amount;
    this.selectedMethod = tempFormData.selectedMethod || this.selectedMethod;
    this.saveData = tempFormData.saveData || this.saveData;
  },
  beforeUnmount() {
    this.saveFormData();
  },
};
</script>


<style scoped>
.mor {
  margin-top: 15px;
}
.card-replenishment {
  max-width: 500px;
  margin: 0 auto;
  padding: 25px;
  border: 2px solid #e0e0e0;
  border-radius: 20px;
  background-color: #ffffff;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
  text-align: left;
}

.card-replenishment h2 {
  font-size: 24px;
  font-weight: 600;
  margin-bottom: 20px;
  color: #333;
  text-align: center;
}

.card-replenishment {
  max-width: 500px;
  margin: 0 auto;
  padding: 25px;
  border: 2px solid #e0e0e0;
  border-radius: 20px;
  background-color: #ffffff;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.1);
  text-align: left;
}

.card-replenishment h2 {
  font-size: 24px;
  font-weight: 600;
  margin-bottom: 20px;
  color: #333;
  text-align: center;
}

.payment-methods {
  display: grid;
  grid-template-columns: repeat(2, 1fr); /* Две колонки */
  gap: 15px; /* Расстояние между элементами */
  margin-bottom: 25px;
}

.payment-method {
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  border: 2px solid #e0e0e0;
  border-radius: 15px;
  background-color: transparent; /* Прозрачный фон */
  text-align: center;
  cursor: pointer;
  transition: all 0.3s ease;
  height: 150px; /* Увеличиваем высоту блока */
}

.payment-method:hover {
  transform: translateY(-5px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.1);
}

.payment-method img {
  max-width: 100%; /* Ограничение по ширине */
  max-height: 100%; /* Ограничение по высоте */
  object-fit: contain; /* Сохраняет пропорции изображения */
  width: 80%; /* Увеличиваем размер картинки */
  height: 80%; /* Увеличиваем размер картинки */
}

/* Убираем цветовые стили для отдельных банков */
.payment-method.sbp,
.payment-method.tinkoff,
.payment-method.vtb,
.payment-method.ozon {
  background-color: transparent;
  border-color: #e0e0e0;
  color: inherit;
}

.selected-method {
  margin-bottom: 20px;
  font-size: 16px;
  text-align: center;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-size: 14px;
  font-weight: 500;
  color: #555;
}

.form-group input {
  width: 100%;
  padding: 12px;
  border: 2px solid #e0e0e0;
  border-radius: 10px;
  font-size: 16px;
  transition: border-color 0.3s ease;
}

.form-group input:focus {
  border-color: #007bff;
  outline: none;
}

.save-data {
  margin-bottom: 20px;
}

.checkbox-label {
  display: flex;
  align-items: center;
  font-size: 16px;
  color: #555;
  cursor: pointer;
}

.custom-checkbox {
  width: 20px;
  height: 20px;
  margin-right: 10px;
  cursor: pointer;
}

.no-arrows::-webkit-inner-spin-button,
.no-arrows::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}


button {
  display: inline-block;
  padding: 12px 24px;
  border: none;
  border-radius: 10px;
  background-color: #007bff;
  color: white;
  font-size: 16px;
  font-weight: 500;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
  margin-right: 10px;
}

button:hover {
  background-color: #0056b3;
  transform: translateY(-2px);
}

button:active {
  transform: translateY(0);
}

.back-button {
  background-color: #6c757d;
}

.back-button:hover {
  background-color: #5a6268;
}
</style>