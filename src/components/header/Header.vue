<template>
  <div class="hed">
    <div class="CentFlex">
      <div class="Conteyer">
        <router-link to="/" class="logo">
          <img class="LogoImg" src="/logoPNG.jpg" alt="">
        </router-link>
      </div>
      <div class="Maga">
        <div class="CenterCaps">
          <div class="navig">
            <router-link to="/Companies">О  компании</router-link>
            <router-link to="">Как купить</router-link>
            <router-link to="">Оплата и доставка</router-link>
            <router-link to="/Guarantee">Гарантия</router-link>
            <router-link to="/Contacts">Контакты</router-link>
          </div>
          <div class="CenterInfa">
            <div class="nn1">
              <router-link to="/address" class="address">
                <img src="/src/assets/Header-img/pin.png" alt="" width="35px" height="35px">
              </router-link>
            </div>
            <div class="nn2">
              <router-link to="/basket" class="basket">
                <img src="/src/assets/Header-img/Cart1.svg" alt="" width="40px" height="40px">
              </router-link>
            </div>
            <div class="nn3">
              <router-link v-if="isAuthenticated" to="/Profile" class="reg">
                <div v-if="userAvatar" class="avatar-container">
                  <img :src="userAvatar" alt="Avatar" class="avatar" />
                </div>
                <img v-else src="/src/assets/Header-img/User.svg" alt="" width="40px" height="40px">
              </router-link>
              <router-link v-else to="/reg" class="reg">
                <img src="/src/assets/Header-img/User.svg" alt="" width="40px" height="40px">
              </router-link>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Блок индикатора страницы, отображается только если не на главной странице -->
    <div v-if="$route.path !== '/'" class="page-indicator">
      <h1 class="page-title">{{ currentPageTitle }}</h1>
      <div class="breadcrumb">Home > {{ currentPageTitle }}</div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Header',
  data() {
    return {
      searchQuery: '', // Строка поиска
      userAvatar: null, // Аватар пользователя
    };
  },
  computed: {
    // Проверка авторизации пользователя
    isAuthenticated() {
      return localStorage.getItem('isAuthenticated') === 'true';
    },
    // Имя пользователя
    userName() {
      const user = JSON.parse(localStorage.getItem('currentUser')) || {};
      return user.name || '';
    },
    // Вычисляемое свойство для названия текущей страницы
    currentPageTitle() {
      // Сопоставление путей маршрутов с названиями страниц
      const routeTitles = {
        '/Profile': 'Профиль',
        '/basket': 'Корзина',
        '/address': 'Адрес',
        '/reg': 'Регистрация',
        '/terms': 'Условия',
        '/product/:id': 'Детали продукта',
        '/card-replenishment': 'Пополнение карты',
        '/legal-information': 'Юридическая информация',
        '/data-protection': 'Защита данных',
        '/Guarantee': 'Гарантия',
        '/Contacts': 'Контакты',
        '/Companies': 'О компании'
      };
      
      // Получаем текущий путь и возвращаем соответствующее название
      const path = this.$route.path;
      if (path.startsWith('/product/')) {
        return 'Детали продукта';
      }
      return routeTitles[path] || 'Неизвестная страница';
    }
  },
  mounted() {
    this.loadUserAvatar(); // Загружаем аватар при монтировании компонента
  },
  methods: {
    // Обработчик поиска
    onSearch() {
      this.$emit('update-search', this.searchQuery);
    },
    // Загрузка аватара пользователя из localStorage
    loadUserAvatar() {
      const users = JSON.parse(localStorage.getItem('users')) || [];
      const userEmail = localStorage.getItem('userEmail');
      const user = users.find(user => user.email === userEmail);
      if (user && user.avatar) {
        this.userAvatar = user.avatar;
      }
    }
  },
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Lato:wght@300&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Jost:wght@300&display=swap');

/* Стили для навигационных ссылок */
.hhjaa a {
  height: 50px;
  margin-top: 20px;
  font-size: 20px;
  font-family: 'Lato', sans-serif;
  font-weight: 400;
}

/* Основной контейнер хедера */
.hed {
  width: 80%;
  margin-left: auto;
  margin-right: auto;
  margin-bottom: 10px;
  position: relative; /* Добавляем позиционирование для правильного размещения */
}

/* Позиционирование элементов */
.nn1 {
  margin-top: 40px;
}
.nn2 {
  margin-top: 39px;
}
.nn3 {
  margin-top: 39px;
}

/* Верхняя часть */
.TopCaps {
  width: 100%;
  height: 50px;
}
.TopCaps div {
  width: 50%;
  display: flex;
  justify-content: space-between;
  height: 50px;
  align-items: center;
}
.TopCaps a {
  text-decoration: none;
  color: #000;
  font-family: 'Lato', sans-serif;
  font-weight: 400;
}

/* Центральный flex-контейнер */
.CentFlex {
  display: flex;
  width: 100%;
  height: 175px; /* Фиксируем высоту основного хедера */
  margin-top: -30px;
}

/* Контейнер логотипа */
.Conteyer {
  width: 20%;
  margin-top: 12px;
  display: flex;
  align-items: center;
}
.logo {
  width: 100%;
  height: 100%;
}

/* Центральная часть */
.CenterCaps {
  height: 100px;
  margin-top: 30px;
  display: flex;
  align-items: center;
  justify-content: flex-end;
}

/* Навигация */
.navig {
  margin-top: 20px;
  margin-right: 40px;
}
.navig a {
  text-decoration: none;
  color: #000;
  margin: 20px;
  font-size: 20px;
  font-family: 'Lato', sans-serif;
  font-weight: 400;
}

/* Поиск */
.Search {
  display: flex;
  align-items: center;
  width: 50%;
  height: 60px;
  margin-top: 20px;
  margin-left: 20px;
  position: relative;
}
.Search input {
  width: 100%;
  height: 45px;
  padding: 0 15px 0 50px;
  border: 2px solid #e0e0e0;
  border-radius: 30px;
  font-size: 18px;
  color: #333;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}
.Search button {
  position: absolute;
  top: 50%;
  left: 15px;
  transform: translateY(-50%);
  border: none;
  background: none;
  cursor: pointer;
  width: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.Search button::before {
  content: '';
  display: block;
  width: 18px;
  height: 18px;
  background: url('/public/iconPosika.png') no-repeat center center;
  background-size: contain;
}

/* Информация в центре */
.CenterInfa {
  width: 12%;
  height: 100px;
  display: flex;
  justify-content: space-between;
}
.CenterInfa div h2 {
  color: #ff5722;
  text-decoration: none;
}

/* Логотип */
.LogoImg {
  width: 70%;
  height: 42%;
  background-position: center;
  background-size: cover;
  background-repeat: no-repeat;
  margin-left: 20px;
  margin-top: 40px;
}

/* Правая часть */
.Maga {
  width: 75%;
  margin-left: auto;
}

/* Контейнер аватара */
.avatar-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}
.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
}
.username {
  margin-top: 5px;
  font-size: 14px;
  color: #333;
}

/* Стили для индикатора страницы */
.page-indicator {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  padding: 20px 0; /* Добавляем отступы сверху и снизу */
  gap: 10px;
  width: 100%; 
  height: 281px;
  background: #EDF1F3;
  flex: none;
  order: 1;
  flex-grow: 0;
  margin-top: 20px; /* Отступ от верхнего блока */
  margin-bottom: 20px; /* Отступ от нижнего контента */
}

.page-title {
  height: 95px;
  font-family: 'Jost', sans-serif;
  font-style: normal;
  font-weight: 300;
  font-size: 83px;
  line-height: 114%;
  letter-spacing: 0.02em;
  text-transform: uppercase;
  color: #272727;
  flex: none;
  order: 0;
  flex-grow: 0;
}

.breadcrumb {
  height: 22px;
  font-family: 'Lato', sans-serif;
  font-style: normal;
  font-weight: 300;
  font-size: 16px;
  line-height: 138%;
  letter-spacing: 0.03em;
  text-transform: capitalize;
  color: #3A3A3A;
  flex: none;
  order: 1;
  flex-grow: 0;
}
</style>