<template>
  <div id="app">
    <!-- 1. Declarative Rendering -->
    <div class="container">
      <div class="header">
        <h1>{{ appTitle }}</h1>
        <p>{{ greeting }}</p>
      </div>

      <!-- 7. Computed Property - Statistics -->
      <div class="stats">
        <div class="stat-item">
          <div class="stat-number">{{ totalTodos }}</div>
          <div class="stat-label">Total Tugas</div>
        </div>
        <div class="stat-item">
          <div class="stat-number">{{ completedTodos }}</div>
          <div class="stat-label">Selesai</div>
        </div>
        <div class="stat-item">
          <div class="stat-number">{{ activeTodos }}</div>
          <div class="stat-label">Aktif</div>
        </div>
        <div class="stat-item">
          <div class="stat-number">{{ completionPercentage }}%</div>
          <div class="stat-label">Progress</div>
        </div>
      </div>

      <!-- Progress Bar -->
      <div class="progress-bar">
        <div class="progress-fill" :style="{ width: completionPercentage + '%' }"></div>
      </div>

      <!-- 4. Form Bindings -->
      <div class="form-section">
        <form @submit.prevent="addTodo">
          <div class="form-group">
            <label for="todoTitle">Judul Tugas</label>
            <input 
              type="text" 
              id="todoTitle"
              class="form-control"
              v-model="newTodo.title"
              placeholder="Masukkan tugas baru..."
              required
            >
          </div>
          
          <div class="form-group">
            <label for="todoDescription">Deskripsi (Opsional)</label>
            <textarea 
              id="todoDescription"
              class="form-control"
              v-model="newTodo.description"
              placeholder="Deskripsi tugas..."
              rows="3"
            ></textarea>
          </div>

          <div class="form-group">
            <label for="todoPriority">Prioritas</label>
            <select id="todoPriority" class="form-control" v-model="newTodo.priority">
              <option value="low">Rendah</option>
              <option value="medium">Sedang</option>
              <option value="high">Tinggi</option>
            </select>
          </div>

          <div class="form-group">
            <label for="todoDueDate">Tanggal Target</label>
            <input 
              type="date" 
              id="todoDueDate"
              class="form-control"
              v-model="newTodo.dueDate"
            >
          </div>

          <!-- 3. Event Listeners -->
          <button type="submit" class="btn btn-primary">
            ➕ Tambah Tugas
          </button>
        </form>
      </div>

      <!-- Search and Filter -->
      <div class="todo-list">
        <div class="search-box">
          <span class="search-icon">🔍</span>
          <input 
            type="text" 
            class="form-control"
            v-model="searchQuery"
            placeholder="Cari tugas..."
          >
        </div>

        <!-- Filter Buttons -->
        <div class="filters">
          <button 
            class="filter-btn"
            :class="{ active: currentFilter === 'all' }"
            @click="setFilter('all')"
          >
            Semua ({{ totalTodos }})
          </button>
          <button 
            class="filter-btn"
            :class="{ active: currentFilter === 'active' }"
            @click="setFilter('active')"
          >
            Aktif ({{ activeTodos }})
          </button>
          <button 
            class="filter-btn"
            :class="{ active: currentFilter === 'completed' }"
            @click="setFilter('completed')"
          >
            Selesai ({{ completedTodos }})
          </button>
        </div>

        <!-- 5. Conditional Rendering & 6. List Rendering -->
        <div v-if="filteredTodos.length === 0 && searchQuery === ''" class="empty-state">
          <div style="font-size: 4rem;">📝</div>
          <h3>Belum ada tugas</h3>
          <p>Tambahkan tugas pertama Anda di atas!</p>
        </div>

        <div v-else-if="filteredTodos.length === 0 && searchQuery !== ''" class="empty-state">
          <div style="font-size: 4rem;">🔍</div>
          <h3>Tidak ditemukan</h3>
          <p>Tidak ada tugas yang cocok dengan pencarian "{{ searchQuery }}"</p>
        </div>

        <!-- 8. Template Refs -->
        <div v-else ref="todoListContainer">
          <div 
            v-for="todo in filteredTodos" 
            :key="todo.id"
            class="todo-item fade-in"
            :class="{ completed: todo.completed }"
          >
            <!-- 2. Attribute Bindings -->
            <input 
              type="checkbox" 
              class="todo-checkbox"
              :checked="todo.completed"
              @change="toggleTodo(todo.id)"
            >
            
            <div class="todo-text">
              <div style="font-weight: bold;">{{ todo.title }}</div>
              <div v-if="todo.description" style="color: #6c757d; font-size: 0.9rem;">
                {{ todo.description }}
              </div>
            </div>
            
            <div 
              class="todo-priority"
              :class="`priority-${todo.priority}`"
            >
              {{ getPriorityText(todo.priority) }}
            </div>
            
            <div v-if="todo.dueDate" class="todo-date">
              📅 {{ formatDate(todo.dueDate) }}
            </div>
            
            <button 
              class="btn btn-danger"
              @click="deleteTodo(todo.id)"
            >
              🗑️
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Debug Info (untuk pembelajaran) -->
    <div class="debug-info">
      <div><strong>Vue.js Concepts Demo:</strong></div>
      <div>✅ Declarative Rendering</div>
      <div>✅ Attribute Bindings</div>
      <div>✅ Event Listeners</div>
      <div>✅ Form Bindings</div>
      <div>✅ Conditional Rendering</div>
      <div>✅ List Rendering</div>
      <div>✅ Computed Properties</div>
      <div>✅ Lifecycle & Template Refs</div>
      <div>✅ Watchers</div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch, onMounted } from 'vue'

export default {
  name: 'App',
  setup() {
    // Reactive data
    const appTitle = ref('To-Do List Manager')
    const greeting = ref('Kelola tugas Anda dengan mudah!')
    const todos = ref([])
    const searchQuery = ref('')
    const currentFilter = ref('all')
    const todoListContainer = ref(null)

    // Form data
    const newTodo = ref({
      title: '',
      description: '',
      priority: 'medium',
      dueDate: ''
    })

    // 7. Computed Properties
    const totalTodos = computed(() => todos.value.length)
    
    const completedTodos = computed(() => 
      todos.value.filter(todo => todo.completed).length
    )
    
    const activeTodos = computed(() => 
      todos.value.filter(todo => !todo.completed).length
    )
    
    const completionPercentage = computed(() => 
      totalTodos.value === 0 ? 0 : Math.round((completedTodos.value / totalTodos.value) * 100)
    )

    const filteredTodos = computed(() => {
      let filtered = todos.value

      // Filter by status
      if (currentFilter.value === 'active') {
        filtered = filtered.filter(todo => !todo.completed)
      } else if (currentFilter.value === 'completed') {
        filtered = filtered.filter(todo => todo.completed)
      }

      // Filter by search query
      if (searchQuery.value) {
        filtered = filtered.filter(todo => 
          todo.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
          todo.description.toLowerCase().includes(searchQuery.value.toLowerCase())
        )
      }

      // Sort by priority and completion status
      return filtered.sort((a, b) => {
        if (a.completed !== b.completed) {
          return a.completed - b.completed
        }
        const priorityOrder = { high: 3, medium: 2, low: 1 }
        return priorityOrder[b.priority] - priorityOrder[a.priority]
      })
    })

    // Methods
    const addTodo = () => {
      if (newTodo.value.title.trim()) {
        const todo = {
          id: Date.now(),
          title: newTodo.value.title,
          description: newTodo.value.description,
          priority: newTodo.value.priority,
          dueDate: newTodo.value.dueDate,
          completed: false,
          createdAt: new Date().toISOString()
        }
        
        todos.value.push(todo)
        
        // Reset form
        newTodo.value = {
          title: '',
          description: '',
          priority: 'medium',
          dueDate: ''
        }
      }
    }

    const toggleTodo = (id) => {
      const todo = todos.value.find(t => t.id === id)
      if (todo) {
        todo.completed = !todo.completed
      }
    }

    const deleteTodo = (id) => {
      todos.value = todos.value.filter(t => t.id !== id)
    }

    const setFilter = (filter) => {
      currentFilter.value = filter
    }

    const getPriorityText = (priority) => {
      const priorityMap = {
        high: 'Tinggi',
        medium: 'Sedang',
        low: 'Rendah'
      }
      return priorityMap[priority]
    }

    const formatDate = (dateString) => {
      if (!dateString) return ''
      const date = new Date(dateString)
      return date.toLocaleDateString('id-ID')
    }

    // Utility functions
    const loadTodosFromStorage = () => {
      try {
        const savedTodos = localStorage.getItem('todos')
        if (savedTodos) {
          todos.value = JSON.parse(savedTodos)
        }
      } catch (error) {
        console.error('Error loading todos from localStorage:', error)
      }
    }

    const saveTodosToStorage = () => {
      try {
        localStorage.setItem('todos', JSON.stringify(todos.value))
      } catch (error) {
        console.error('Error saving todos to localStorage:', error)
      }
    }

    // 9. Watchers
    watch(searchQuery, (newQuery, oldQuery) => {
      console.log(`Pencarian berubah dari "${oldQuery}" ke "${newQuery}"`)
    })

    watch(completionPercentage, (newPercentage) => {
      if (newPercentage === 100 && totalTodos.value > 0) {
        setTimeout(() => {
          alert('🎉 Selamat! Semua tugas telah selesai!')
        }, 500)
      }
    })

    watch(todos, (newTodos) => {
      // Auto-save to localStorage
      saveTodosToStorage()
      console.log('Todos updated:', newTodos.length, 'items')
    }, { deep: true })

    // 8. Lifecycle Hooks
    onMounted(() => {
      console.log('Aplikasi To-Do List berhasil dimuat!')
      
      // Load todos from localStorage
      loadTodosFromStorage()
      
      // If no saved todos, load sample data
      if (todos.value.length === 0) {
        const sampleTodos = [
          {
            id: 1,
            title: 'Belajar Vue.js',
            description: 'Pelajari konsep-konsep dasar Vue.js',
            priority: 'high',
            dueDate: '2024-12-31',
            completed: false,
            createdAt: new Date().toISOString()
          },
          {
            id: 2,
            title: 'Buat project portofolio',
            description: 'Kembangkan website portofolio personal',
            priority: 'medium',
            dueDate: '2024-12-25',
            completed: true,
            createdAt: new Date().toISOString()
          }
        ]
        
        todos.value = sampleTodos
      }
      
      // Update greeting based on time
      const hour = new Date().getHours()
      if (hour < 12) {
        greeting.value = 'Selamat pagi! Mari mulai hari dengan produktif!'
      } else if (hour < 17) {
        greeting.value = 'Selamat siang! Tetap semangat menyelesaikan tugas!'
      } else {
        greeting.value = 'Selamat malam! Jangan lupa istirahat yang cukup!'
      }
    })

    // Export functions and data for use in template
    return {
      // Data
      appTitle,
      greeting,
      todos,
      newTodo,
      searchQuery,
      currentFilter,
      todoListContainer,
      
      // Computed
      totalTodos,
      completedTodos,
      activeTodos,
      completionPercentage,
      filteredTodos,
      
      // Methods
      addTodo,
      toggleTodo,
      deleteTodo,
      setFilter,
      getPriorityText,
      formatDate
    }
  }
}
</script>

<style scoped>
/* Component-specific styles dapat ditambahkan di sini */
</style>