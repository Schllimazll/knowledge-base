# SRP - Single Responsibility Principle (Принцип единственной ответственности)

**Один модуль = одна зона ответственности**

Каждый программный модуль (компонент, composable, класс) должен отвечать за **единственную цель** и иметь **одну причину для изменения**. Под причиной изменения понимается изменение бизнес-логики, а не технической реализации.

## 📌 Ключевые аспекты принципа

- **Ответственность ≠ задача**

  Компонент может выполнять несколько технических задач (рендеринг, обработка кликов), но они должны относиться к одной бизнес-логической ответственности.

- **Изоляция изменений**

  Если для изменения поведения системы вам приходится править один модуль в разных несвязанных местах — это нарушение SRP.

- **Гранулярность**

  Размер модуля не главное. 200-строчный компонент, решающий одну комплексную задачу, лучше пяти микро-компонентов со скрытыми зависимостями.

## 💎 Преимущества

| Аспект | Эффект |
|--------|---------|
| Тестирование | Модули с одной ответственностью требуют меньше тест-кейсов |
| Рефакторинг | Изменения в одном модуле не вызывают "эффекта домино" в других компонентах |
| Командная работа | Чёткие границы модулей уменьшают конфликты при слиянии веток |
| Производительность | Меньшие компоненты лучше оптимизируются компилятором Vue |

## 🛠 Применение SRP во Vue 3

### 📦 Компоненты

**Хороший компонент можно описать одной фразой без союзов**

```
❌ Плохо: "Компонент отображает список пользователей И обрабатывает пагинацию И фильтрацию"
✅ Хорошо: "Компонент отображает элементы списка с виртуальным скроллом"
```

**Пример хорошей декомпозиции:**

```vue
<template>
  <div>
    <UserList :data="filteredUsers" /> <!-- Только рендеринг -->
    <UserFilters @filter="applyFilters" /> <!-- Только фильтрация -->
    <UserPagination @change="changePagination" /> <!-- Только пагинация -->
  </div>
</template>
```

### 🔧 Composables

Один composable = одна бизнес-логическая цель

```ts
// ❌ Плохо: 3 несвязанные ответственности
export const useUserManager = () => {
  // Загрузка данных (Data fetching)
  const fetchUsers = async () => {...};

  // Форматирование данных (Data transformation)
  const formatUserName = (user) =>
    `${user.firstName} ${user.lastName}`;

  // Валидация (Validation)
  const validateEmail = (email) =>
    /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);

  return { fetchUsers, formatUserName, validateEmail };
};
```

```ts
// ✅ Хорошо: Разделение на специализированные composables

// 1. Только работа с API
// useUserApi.ts
export const useUserApi = () => {
  const fetchUsers = async () => {...};
  const fetchUserById = async (id) => {...};

  return { fetchUsers, fetchUserById };
};

// 2. Только трансформация данных
// useUserFormatter.ts
export const useUserFormatter = () => {
  const formatUserName = (user) =>
    `${user.firstName} ${user.lastName}`;

  const formatRegistrationDate = (date) =>
    new Date(date).toLocaleDateString();

  return { formatUserName, formatRegistrationDate };
};

// 3. Только валидация
// useValidation.ts
export const useValidation = () => {
  const validateEmail = (email) =>
    /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);

  const validatePassword = (password) =>
    password.length >= 8;

  return { validateEmail, validatePassword };
};
```
## 🚨 Антипаттерны

### 1. **Божественный объект**

```vue
<script setup>
// Плохо: Компонент делает ВСЁ
const {
  data,
  fetchData,
  paginate,
  filter,
  sort,
  exportToExcel,
  handleErrors,
  trackAnalytics
} = useMonolithComponent();
</script>
```

Решение: декомпозиция на несколько компонентов (функций)

### 2. **Синглтон**

**Синглтон** — это паттерн, при котором в приложении существует только один экземпляр какого-то класса. Существующий синглтон гарантирует, что все новые созданные объекты будут ссылаться на него.

Допустим у нас есть класс `UserService`, который отвечает за работу с пользователями, он содержит методы для работы с пользователями а так же хранит их в своем поле. Тем самым он нарушает принцип единственной ответственности (хранит данные и выполняет логику работы с ними). В данном случае лучше создать два класса: `UserService` и `UserRepository` и отказаться от синглтона. В идеале переписать все на архитектуру сторов и использовать `useUserStore`.

### 3. **Props Hell**

**Props Hell** — это антипаттерн, при котором компонент получает множество пропов, что делает его сложным для понимания и поддержки.

```vue
<template>
  <!-- Плохо: 15+ пропсов для разных сценариев -->
  <UserCard
    :user="user"
    :show-avatar="true"
    :avatar-size="40"
    :title-style="'bold'"
    :interactive="false"
    :show-status="true"
    :status-color="'green'"
    :show-actions="false"
    :show-details="true"
    :show-badges="true"
    :badge-position="'top-right'"
    :show-social="true"
    :social-links="socialLinks"
  />
</template>
```

Решение: декомпозиция на несколько компонентов

```vue
<template>
  <!-- Хорошо: Композиция компонентов -->
  <UserProfile :user="user">
    <template #avatar>
      <UserAvatar :size="40" />
    </template>
    <template #title>
      <UserTitle>{{ user.name }}</UserTitle>
    </template>
  </UserProfile>
</template>
```
