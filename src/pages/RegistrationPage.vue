<template>
  <div class="form-page">
    <h1>Регистрация</h1>

    <form @submit="handleSubmit" class="registration-form">
      <!-- Имя -->
      <div class="form-group">
        <label for="name">Имя</label>
        <input
          id="name"
          v-model="form.name"
          type="text"
          class="form-input"
          :class="{ 'input-error': errors.name }"
        />
        <span v-if="errors.name" class="error-message">{{ errors.name }}</span>
      </div>

      <!-- Email -->
      <div class="form-group">
        <label for="email">Email</label>
        <input
          id="email"
          v-model="form.email"
          type="email"
          class="form-input"
          :class="{ 'input-error': errors.email }"
        />
        <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
      </div>

      <!-- Пароль -->
      <div class="form-group">
        <label for="password">Пароль</label>
        <input
          id="password"
          v-model="form.password"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.password }"
        />
        <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
      </div>

      <!-- Подтверждение пароля -->
      <div class="form-group">
        <label for="confirmPassword">Подтверждение пароля</label>
        <input
          id="confirmPassword"
          v-model="form.confirmPassword"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.confirmPassword }"
        />
        <span v-if="errors.confirmPassword" class="error-message">{{ errors.confirmPassword }}</span>
      </div>

      <!-- Общее сообщение об ошибке -->
      <div v-if="generalError" class="general-error">{{ generalError }}</div>

      <!-- Кнопка отправки -->
      <button
        type="submit"
        class="btn-submit"
        :disabled="isLoading"
      >
        {{ isLoading ? 'Отправка...' : 'Зарегистрироваться' }}
      </button>
    </form>

    <!-- Сообщение об успехе -->
    <div v-if="successMessage" class="success-message">
      {{ successMessage }}
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import {reactive } from 'axios'
// import axios from 'axios' // Раскомментируйте, если установили axios

const isLoading = ref(false)
const successMessage = ref('')
const generalError = ref(null)

// Данные формы
const form = reactive({
  name: '',
  email: '',
  password: '',
  confirmPassword: ''
})

// Ошибки валидации
const errors = reactive({
  name: null,
  email: null,
  password: null,
  confirmPassword: null
})

// Простая валидация
const validate = () => {
  // Сброс ошибок
  Object.keys(errors).forEach(key => (errors[key] = null))
  let hasErrors = false

  // Имя: не пустое, длина >= 2
  if (!form.name || form.name.trim().length < 2) {
    errors.name = 'Имя должно быть не короче 2 символов.'
    hasErrors = true
  }

  // Email: не пустой, простой формат (исправлена регулярка из документа)
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!form.email || !emailRegex.test(form.email)) {
    errors.email = 'Введите корректный email.'
    hasErrors = true
  }

  // Пароль: не пустой, длина >= 6
  if (!form.password || form.password.length < 6) {
    errors.password = 'Пароль должен быть не короче 6 символов.'
    hasErrors = true
  }

  // Подтверждение пароля: совпадает с паролем
  if (form.password !== form.confirmPassword) {
    errors.confirmPassword = 'Пароли не совпадают.'
    hasErrors = true
  } else if (!form.confirmPassword && form.password) {
    // Если пароль заполнен, а подтверждение пустое
    errors.confirmPassword = 'Подтвердите пароль.'
    hasErrors = true
  }

  return !hasErrors
}

const handleSubmit = async (event) => {
  event.preventDefault() // предотвращаем перезагрузку страницы
  successMessage.value = ''
  generalError.value = null

  if (!validate()) {
    return // если есть ошибки валидации – не отправляем
  }

  isLoading.value = true

  try {
    const payload = {
      name: form.name.trim(),
      email: form.email.trim(),
      password: form.password,
      confirmPassword: form.confirmPassword 
    }

    // Вариант 1: fetch (нативно)
    // Исправлена опечатка в URL из документа (было typecode, стало typicode)
    const response = await fetch('https://jsonplaceholder.typicode.com/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(payload)
    })

    if (!response.ok) {
      // Исправлена опечатка с кавычками в строке из документа
      throw new Error(`Ошибка сервера: ${response.status}`)
    }

    const data = await response.json()
    successMessage.value = 'Регистрация успешно отправлена (демо). Данные: ' + JSON.stringify(data, null, 2)
    
  } catch (err) {
    // Исправлена опечатка в слове catch (в документе было просто catc)
    generalError.value = err instanceof Error ? err.message : 'Произошла ошибка при отправке.'
  } finally {
    isLoading.value = false
  }
}
</script>

<style scoped>
/* Базовые стили для формы (Пункт 6 из задания) */
.form-page {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  font-family: sans-serif;
}

.registration-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.form-input {
  padding: 0.5rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 1rem;
}

.input-error {
  border-color: red;
  outline-color: red;
}

.error-message {
  color: red;
  font-size: 0.85rem;
}

.general-error {
  color: white;
  background-color: #dc3545;
  padding: 0.5rem;
  border-radius: 4px;
  font-size: 0.9rem;
}

.btn-submit {
  padding: 0.75rem;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1rem;
  font-weight: bold;
}

.btn-submit:disabled {
  background-color: #a0d8b3;
  cursor: not-allowed;
}

.success-message {
  margin-top: 1rem;
  padding: 1rem;
  background-color: #d4edda;
  color: #155724;
  border-radius: 4px;
  word-break: break-all; /* Чтобы длинный JSON не ломал верстку */
}
</style>