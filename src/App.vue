<template>
  <div class="header"><h1>Модуль электронный помощник</h1></div>
  <div class="chat-container">
    <div class="chat-bot">
      <div class="chat-header">
        <h2>Электронный помощник</h2>
      </div>
      <div class="chat-body">
        <div v-for="(message, index) in messages" :key="index" class="message">
          <div v-if="message.user" class="user-message">{{ message.text }}</div>
          <div v-else class="bot-message">{{ message.text }}</div>
        </div>
      </div>
      <div class="chat-input">
        <input
          v-model="inputText"
          @keyup.enter="sendMessage"
          placeholder="Введите сообщение..."
        />
        <button @click="sendMessage">Отправить</button>
      </div>
      <div v-if="showDatePicker" class="date-picker">
        <vue-datepicker
          v-model="selectedDate"
          @update:modelValue="handleDateSelect"
        />
      </div>
    </div>
    <div class="help-panel">
      <h2>Подсказки по использованию</h2>
      <ul>
        <li>Введите "старт", чтобы начать диалог.</li>
        <li>Введите "Какие есть отели?", чтобы увидеть список отелей.</li>
        <li>Выберите отель, чтобы увидеть доступные номера.</li>
        <li>Выберите номер, чтобы выбрать дату бронирования.</li>
        <li>Введите "посмотреть свою бронь", чтобы увидеть вашу бронь.</li>
        <li>Введите "отменить бронь", чтобы отменить вашу бронь.</li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue';
import VueDatepicker from 'vue3-datepicker';

const inputText = ref('');
const messages = reactive([]);
const hotels = {
  'Отель Континенталь': ['Номер 101', 'Номер 102', 'Номер 103'],
  'Отель Москва': ['Номер 201', 'Номер 202', 'Номер 203'],
  'Отель Победа': ['Номер 301', 'Номер 302', 'Номер 303'],
};
const booking = reactive({
  hotel: '',
  room: '',
  date: '',
});
const showDatePicker = ref(false);
const selectedDate = ref(null);

const sendMessage = () => {
  if (inputText.value.trim() === '') return;

  messages.push({ user: true, text: inputText.value });
  processMessage(inputText.value);
  inputText.value = '';
};

const processMessage = (message) => {
  if (message.toLowerCase() === 'старт') {
    messages.push({
      user: false,
      text: 'Привет! Я ваш электронный помощник. Чем могу помочь?',
    });
  } else if (message.toLowerCase() === 'какие есть отели?') {
    messages.push({
      user: false,
      text:
        'У нас есть следующие отели: ' +
        Object.keys(hotels).join(', ') +
        '. Какой отель вас интересует? (введите "Отель ...")',
    });
  } else if (Object.keys(hotels).includes(message)) {
    booking.hotel = message;
    saveBookingToSessionStorage();
    messages.push({
      user: false,
      text:
        'Отличный выбор! Показаны свободные номера в отеле ' +
        message +
        ': ' +
        hotels[message].join(', ') +
        '. Какой номер вас интересует? (введите "Номер ...")',
    });
  } else if (hotels[booking.hotel]?.includes(message)) {
    booking.room = message;
    saveBookingToSessionStorage();
    messages.push({ user: false, text: 'Выберите дату бронирования:' });
    showDatePicker.value = true;
  } else if (message.toLowerCase() === 'посмотреть свою бронь') {
    if (booking.hotel && booking.room && booking.date) {
      messages.push({
        user: false,
        text:
          'Ваша бронь: Отель - ' +
          booking.hotel +
          ', Номер - ' +
          booking.room +
          ', Дата - ' +
          booking.date,
      });
    } else {
      messages.push({
        user: false,
        text: 'У вас нет забронированных номеров.',
      });
    }
  } else if (message.toLowerCase() === 'отменить бронь') {
    booking.hotel = '';
    booking.room = '';
    booking.date = '';
    saveBookingToSessionStorage();
    messages.push({ user: false, text: 'Ваша бронь успешно отменена.' });
  } else {
    messages.push({
      user: false,
      text: 'Извините, я не понимаю ваш запрос. Пожалуйста, попробуйте еще раз.',
    });
  }
};

const handleDateSelect = (date) => {
  if (date) {
    booking.date = date.toISOString().split('T')[0];
    saveBookingToSessionStorage();
    messages.push({
      user: false,
      text:
        'Номер ' +
        booking.room +
        ' в отеле ' +
        booking.hotel +
        ' на дату ' +
        booking.date +
        ' успешно забронирован. Вы можете посмотреть свою бронь или отменить её.',
    });
    showDatePicker.value = false;
  }
};

const saveBookingToSessionStorage = () => {
  sessionStorage.setItem('booking', JSON.stringify(booking));
};

onMounted(() => {
  const storedBooking = sessionStorage.getItem('booking');
  if (storedBooking) {
    Object.assign(booking, JSON.parse(storedBooking));
  }
});
</script>

<style scoped>
.header {
  height: 100px;
  text-align: center;
  font-size: large;
  background-color: #007bff;
  color: white;
  padding-top: 20px;
  margin-bottom: 20px;
}

.chat-container {
  display: flex;
  justify-content: space-between;
  max-width: 800px;
  margin: 0 auto;
}

.chat-bot {
  width: 60%;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.chat-header {
  background-color: #007bff;
  color: white;
  padding: 10px;
  text-align: center;
}

.chat-body {
  height: 300px;
  overflow-y: auto;
  padding: 10px;
  background-color: #f9f9f9;
}

.message {
  margin-bottom: 10px;
}

.user-message {
  text-align: right;
  background-color: #dcf8c6;
  padding: 5px;
  border-radius: 5px;
}

.bot-message {
  text-align: left;
  background-color: #e5e5ea;
  padding: 5px;
  border-radius: 5px;
}

.chat-input {
  display: flex;
  padding: 10px;
  background-color: #f1f1f1;
}

.chat-input input {
  flex: 1;
  padding: 5px;
  border: 1px solid #ccc;
  border-radius: 3px;
}

.chat-input button {
  margin-left: 10px;
  padding: 5px 10px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 3px;
  cursor: pointer;
}

.date-picker {
  padding: 10px;
  background-color: #f1f1f1;
}

.help-panel {
  width: 35%;
  padding: 10px;
  background-color: #f9f9f9;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.help-panel h2 {
  margin-bottom: 10px;
}

.help-panel ul {
  list-style-type: disc;
  padding-left: 20px;
}
</style>
