<template>
  <div class="app">
    <h1>Менеджер книг</h1>

    <!-- Форма -->
    <AddBookForm @add-book="addBook" />

    <!-- Фильтры -->
    <BookFilters
      v-model:searchQuery="searchQuery"
      v-model:filter="currentFilter"
      :books="books"
    />

    <!-- Список книг -->
    <BookCard
      v-for="book in filteredBooks"
      :key="book.id"
      :book="book"
      @toggle="toggleBook(book.id)"
      @delete="deleteBook(book.id)"
      @rate="rateBook(book.id, $event)"
    />
  </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import AddBookForm from './components/AddBookForm.vue'
import BookCard from './components/BookCard.vue'
import BookFilters from './components/BookFilters.vue'

const books = ref([])

// Загрузка из localStorage
const savedBooks = localStorage.getItem('books')
if (savedBooks) {
  books.value = JSON.parse(savedBooks)
}

const currentFilter = ref('all')
const searchQuery = ref('')

const addBook = (bookData) => {
  const newBook = {
    id: Date.now(),
    ...bookData,
    completed: false,
    rating: 0
  }
  books.value.push(newBook)
}

const toggleBook = (id) => {
  const book = books.value.find(b => b.id === id)
  if (book) {
    book.completed = !book.completed
    if (!book.completed) {
      book.rating = 0
    }
  }
}

const rateBook = (id, rating) => {
  const book = books.value.find(b => b.id === id)
  if (book && book.completed) {
    book.rating = rating
  }
}

const deleteBook = (id) => {
  books.value = books.value.filter(b => b.id !== id)
}

// 👇 ВОТ ЗДЕСЬ ДОЛЖЕН БЫТЬ watch
watch(
  books,
  (newBooks) => {
    localStorage.setItem('books', JSON.stringify(newBooks))
  },
  { deep: true }
)

const filteredBooks = computed(() => {
  return books.value
    .filter(book => {
      if (currentFilter.value === 'unread') return !book.completed
      if (currentFilter.value === 'read') return book.completed
      return true
    })
    .filter(book => {
      if (!searchQuery.value) return true
      const query = searchQuery.value.toLowerCase()
      return (
        book.title.toLowerCase().includes(query) ||
        book.author.toLowerCase().includes(query)
      )
    })
})
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: Arial, sans-serif;
  background: #f5f5f5;
}

.app {
  max-width: 800px;
  margin: 40px auto;
  padding: 20px;
}
</style>