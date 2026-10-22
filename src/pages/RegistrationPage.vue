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
          @blur="validateName"
          @input="validateName"
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
          @blur="validateEmail"
          @input="validateEmail"
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
          @blur="validatePassword"
          @input="validatePassword"
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
          @blur="validateConfirmPassword"
          @input="validateConfirmPassword"
        />
        <span v-if="errors.confirmPassword" class="error-message">{{
          errors.confirmPassword
        }}</span>
      </div>

      <!-- Общее сообщение об ошибке -->
      <div v-if="generalError" class="general-error">{{ generalError }}</div>

      <!-- Кнопка отправки -->
      <button type="submit" class="btn-submit" :disabled="isLoading">
        {{ isLoading ? "Отправка..." : "Зарегистрироваться" }}
      </button>
    </form>

    <!-- Сообщение об успехе -->
    <div v-if="successMessage" class="success-message">
      {{ successMessage }}
    </div>
  </div>
</template>

<script setup>
import { ref, reactive } from "vue";

const isLoading = ref(false);
const successMessage = ref("");
const generalError = ref(null);

// Данные формы
const form = reactive({
  name: "",
  email: "",
  password: "",
  confirmPassword: "",
});

// Ошибки валидации
const errors = reactive({
  name: null,
  email: null,
  password: null,
  confirmPassword: null,
});

// функции валидации

const validateName = () => {
  if (!form.name || form.name.trim().length < 2) {
    errors.name = "Имя должно быть не короче 2 символов.";
  } else {
    errors.name = null;
  }
};

const validateEmail = () => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!form.email || !emailRegex.test(form.email)) {
    errors.email = "Введите корректный email.";
  } else {
    errors.email = null;
  }
};

const validatePassword = () => {
  const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%^&*]).{8,}$/;
  if (!form.password || !passwordRegex.test(form.password)) {
    errors.password =
      "Пароль от 8 символов, должен содержать: строчные и прописные буквы, цифры и спецсимволы (!@#$%^&*).";
  } else {
    errors.password = null;
  }

  if (form.confirmPassword) {
    validateConfirmPassword();
  }
};

const validateConfirmPassword = () => {
  if (!form.confirmPassword) {
    errors.confirmPassword = "Подтвердите пароль.";
  } else if (form.password !== form.confirmPassword) {
    errors.confirmPassword = "Пароли не совпадают.";
  } else {
    errors.confirmPassword = null;
  }
};

// проврека на кнопку
const validate = () => {
  validateName();
  validateEmail();
  validatePassword();
  validateConfirmPassword();

  return !Object.values(errors).some((error) => error !== null);
};

// отправка формы
const handleSubmit = async (event) => {
  event.preventDefault();
  successMessage.value = "";
  generalError.value = null;

  // Запускаем полную проверку перед отправкой
  if (!validate()) {
    return;
  }

  isLoading.value = true;

  try {
    const payload = {
      name: form.name.trim(),
      email: form.email.trim(),
      password: form.password,
    };

    // Вариант с fetch (нативно, как в вашем примере)
    import axios from "axios";

    const response = await axios.post(
      "https://jsonplaceholder.typicode.com/users",
      payload,
      {
        headers: {
          "Content-Type": "application/json",
        },
      }
    );

    const data = response.data;
    successMessage.value =
      "Регистрация успешно отправлена (демо). Данные: " + JSON.stringify(data, null, 2);
    form.name = "";
    form.email = "";
    form.password = "";
    form.confirmPassword = "";
    
  } catch (err) {
    if (axios.isAxiosError(err) && err.response) {
      generalError.value = `Ошибка сервера: ${err.response.status} ${err.response.statusText}`;
    } else if (axios.isAxiosError(err) && err.request) {
      generalError.value = "Сервер не отвечает. Проверьте соединение.";
    } else {
      generalError.value =
        err instanceof Error ? err.message : "Произошла ошибка при отправке.";
    }
  } finally {
    isLoading.value = false;
  }
};
</script>

<style scoped>
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
  word-break: break-all;
}
</style>
