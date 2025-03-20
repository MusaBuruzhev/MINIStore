<template>
  <div id="app">
    <Header v-if="showHeader" />
    <router-view />
    <Footer v-if="showFooter" />
  </div>
</template>

<script>
import Header from './components/header/Header.vue';
import Footer from './components/Footer/Footer.vue';

export default {
  name: 'App',
  components: {
    Header,
    Footer,
  },
  data() {
    return {
      showHeader: true,
      showFooter: true,
    };
  },
  methods: {
    loadChatWidget() {
      // Получаем данные пользователя из localStorage
      const users = JSON.parse(localStorage.getItem('users')) || [];
      const userEmail = localStorage.getItem('userEmail');
      const currentUser = users.find(user => user.email === userEmail) || {};

      // Создаем объект для передачи данных в виджет
      window.juswidgetVariables = {
        start: {
          id: currentUser.id || 'guest',
          name: currentUser.name || 'Гость',
          email: currentUser.email || '',
          phone: currentUser.phone || '',
        },
      };

      // Добавляем скрипт для чат-виджета динамически
      const script = document.createElement('script');
      script.src =
        'https://bot.jaicp.com/chatwidget/ATdbrIxY:891c02dcdb4c57c21183bf9fd270961a542b58b4/justwidget.js?force=true';
      script.async = true;

      // Обработчик загрузки скрипта
      script.onload = () => {
        console.log('Чат-виджет успешно загружен');
      };

      // Обработчик ошибок загрузки скрипта
      script.onerror = () => {
        console.error('Ошибка загрузки чат-виджета');
      };

      // Добавляем скрипт в DOM
      document.head.appendChild(script);
    },
  },
  mounted() {
    this.loadChatWidget();
  },
};
</script>

<style>
/* Стили для чат-виджета */
.jus-widget-container {
  font-family: Arial, sans-serif;
  background-color: #f5f5f5;
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.jus-widget-header {
  background-color: #007bff;
  color: white;
  padding: 10px;
  border-top-left-radius: 10px;
  border-top-right-radius: 10px;
}

.jus-widget-message {
  padding: 10px;
  font-size: 14px;
}
/* Перемещение кнопки выше и левее */
.jus-widget-button {
  position: fixed; /* Фиксированное положение на экране */
  bottom: 120px; /* Расстояние от нижней части экрана */
  margin-left: 20%;
  z-index: 1000; /* Убедитесь, что кнопка находится поверх других элементов */
}
</style>