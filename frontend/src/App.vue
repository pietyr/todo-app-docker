<template>
  <div class="min-h-screen bg-slate-50 py-8 px-4 sm:px-6 lg:px-8">
    <!-- Header -->
    <header class="max-w-7xl mx-auto mb-8 text-center sm:text-left sm:flex sm:items-center sm:justify-between border-b border-slate-200 pb-5">
      <div>
        <h1 class="text-3xl font-extrabold text-slate-900 tracking-tight sm:text-4xl">
          Tygodniowa Lista Zadań
        </h1>
        <p class="mt-2 text-sm text-slate-500">
          Zarządzanie zadaniami w podziale na dni tygodnia.
        </p>
      </div>
      <div class="mt-4 sm:mt-0 flex justify-center space-x-3">
        <button 
          @click="openAddModal()" 
          class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition-colors"
        >
          <svg class="h-5 w-5 mr-1.5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
          </svg>
          Nowe Zadanie
        </button>
      </div>
    </header>

    <!-- Error/Loading notifications -->
    <div class="max-w-7xl mx-auto">
      <div v-if="error" class="mb-4 bg-red-50 border-l-4 border-red-400 p-4 rounded-md">
        <div class="flex">
          <div class="flex-shrink-0">
            <svg class="h-5 w-5 text-red-400" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd" />
            </svg>
          </div>
          <div class="ml-3">
            <p class="text-sm text-red-700 font-medium">Błąd komunikacji z API: {{ error }}</p>
          </div>
        </div>
      </div>

      <div v-if="loading" class="text-center py-12">
        <svg class="animate-spin h-10 w-10 text-indigo-600 mx-auto" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.121 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
        </svg>
        <p class="mt-2 text-sm text-slate-500">Pobieranie zadań z bazy danych...</p>
      </div>
    </div>

    <!-- Weekly Board Grid -->
    <main class="max-w-7xl mx-auto">
      <div v-if="!loading" class="grid grid-cols-1 md:grid-cols-5 gap-6">
        
        <!-- Day Column -->
        <div 
          v-for="day in weekDays" 
          :key="day.id" 
          class="bg-white rounded-xl shadow-sm border border-slate-200 flex flex-col min-h-[450px]"
        >
          <!-- Column Header -->
          <div class="p-4 border-b border-slate-100 flex items-center justify-between rounded-t-xl" :class="day.bgHeader">
            <h2 class="font-bold text-slate-800 flex items-center">
              {{ day.name }}
            </h2>
            <span class="bg-white/80 backdrop-blur text-slate-700 text-xs font-semibold px-2 py-0.5 rounded-full border border-slate-200/50">
              {{ getTasksByDay(day.id).length }}
            </span>
          </div>

          <!-- Tasks List -->
          <div class="p-3 flex-1 overflow-y-auto space-y-3">
            <div 
              v-if="getTasksByDay(day.id).length === 0" 
              class="h-full flex flex-col items-center justify-center text-center p-6 text-slate-400"
            >
              <svg class="h-8 w-8 text-slate-300 stroke-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-6 9l2 2 4-4" />
              </svg>
              <p class="text-xs mt-2 font-medium">Brak zadań</p>
              <button 
                @click="openAddModal(day.id)" 
                class="mt-2 text-xs text-indigo-600 hover:text-indigo-800 font-semibold focus:outline-none transition-colors"
              >
                Dodaj pierwsze
              </button>
            </div>

            <!-- Task Card -->
            <div 
              v-for="task in getTasksByDay(day.id)" 
              :key="task.id" 
              class="group relative bg-slate-50 hover:bg-white rounded-lg p-3 border border-slate-200/60 hover:border-indigo-200 shadow-sm hover:shadow transition-all duration-150"
              :class="{ 'opacity-60 bg-slate-100 border-slate-200': task.done }"
            >
              <!-- Completed Checkbox & Title -->
              <div class="flex items-start">
                <button 
                  @click="toggleTaskDone(task)"
                  class="mt-1 flex-shrink-0 h-4 w-4 rounded-full border border-slate-300 flex items-center justify-center focus:outline-none focus:ring-2 focus:ring-offset-1 focus:ring-indigo-500 transition-all"
                  :class="task.done ? 'bg-indigo-600 border-indigo-600' : 'bg-white hover:border-indigo-400'"
                >
                  <svg v-if="task.done" class="h-2.5 w-2.5 text-white" viewBox="0 0 20 20" fill="currentColor">
                    <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd" />
                  </svg>
                </button>
                <div class="ml-2.5 flex-1 pr-6">
                  <h3 
                    class="text-sm font-semibold text-slate-800 leading-tight break-words"
                    :class="{ 'line-through text-slate-400': task.done }"
                  >
                    {{ task.title }}
                  </h3>
                  <p 
                    v-if="task.description" 
                    class="text-xs text-slate-500 mt-1 whitespace-pre-wrap break-words leading-normal"
                    :class="{ 'line-through text-slate-400': task.done }"
                  >
                    {{ task.description }}
                  </p>
                </div>
              </div>

              <!-- Action Buttons (edit/delete) -->
              <div class="absolute right-2 top-2 flex space-x-1 sm:opacity-0 group-hover:opacity-100 transition-opacity">
                <!-- Edit button -->
                <button 
                  @click="openEditModal(task)"
                  class="p-1 rounded text-slate-400 hover:text-indigo-600 hover:bg-indigo-50 focus:outline-none transition-all"
                  title="Edytuj zadanie"
                >
                  <svg class="h-3.5 w-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
                  </svg>
                </button>
                <!-- Delete button -->
                <button 
                  @click="deleteTask(task.id)"
                  class="p-1 rounded text-slate-400 hover:text-red-600 hover:bg-red-50 focus:outline-none transition-all"
                  title="Usuń zadanie"
                >
                  <svg class="h-3.5 w-3.5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                  </svg>
                </button>
              </div>
            </div>
          </div>
        </div>

      </div>
    </main>

    <!-- Modal Form (Add / Edit) -->
    <div 
      v-if="modal.show" 
      class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 z-50 transition-all duration-200"
      @click.self="closeModal()"
    >
      <div class="bg-white rounded-xl shadow-xl border border-slate-200 max-w-md w-full overflow-hidden transform transition-all scale-100">
        <!-- Modal Header -->
        <div class="px-6 py-4 border-b border-slate-100 flex items-center justify-between bg-slate-50">
          <h3 class="text-lg font-bold text-slate-900">
            {{ modal.isEdit ? 'Edytuj Zadanie' : 'Dodaj Nowe Zadanie' }}
          </h3>
          <button 
            @click="closeModal()" 
            class="rounded-md text-slate-400 hover:text-slate-500 hover:bg-slate-100 p-1.5 focus:outline-none focus:ring-2 focus:ring-indigo-500"
          >
            <span class="sr-only">Zamknij</span>
            <svg class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>
        </div>

        <!-- Modal Form -->
        <form @submit.prevent="submitForm()" class="p-6 space-y-4">
          <!-- Title -->
          <div>
            <label for="title" class="block text-sm font-semibold text-slate-700">Tytuł zadania *</label>
            <input 
              v-model="form.title" 
              type="text" 
              id="title" 
              required 
              placeholder="np. Przygotować prezentację z Dockera" 
              class="mt-1.5 block w-full rounded-md border-slate-300 px-3 py-2 text-sm text-slate-900 shadow-sm border focus:border-indigo-500 focus:ring-indigo-500 focus:outline-none"
              ref="titleInput"
            />
          </div>

          <!-- Description -->
          <div>
            <label for="description" class="block text-sm font-semibold text-slate-700">Opis (opcjonalnie)</label>
            <textarea 
              v-model="form.description" 
              id="description" 
              rows="3" 
              placeholder="np. Opisać architekturę, wolumeny oraz mechanizm sieciowy..." 
              class="mt-1.5 block w-full rounded-md border-slate-300 px-3 py-2 text-sm text-slate-900 shadow-sm border focus:border-indigo-500 focus:ring-indigo-500 focus:outline-none resize-none"
            ></textarea>
          </div>

          <!-- Day of Week Selection -->
          <div>
            <label for="day_of_week" class="block text-sm font-semibold text-slate-700">Dzień tygodnia *</label>
            <select 
              v-model="form.day_of_week" 
              id="day_of_week" 
              required
              class="mt-1.5 block w-full rounded-md border-slate-300 px-3 py-2 text-sm text-slate-900 shadow-sm border focus:border-indigo-500 focus:ring-indigo-500 focus:outline-none bg-white"
            >
              <option v-for="day in weekDays" :key="day.id" :value="day.id">
                {{ day.name }}
              </option>
            </select>
          </div>

          <!-- Done Checkbox (only shown during edit) -->
          <div v-if="modal.isEdit" class="flex items-center">
            <input 
              v-model="form.done" 
              type="checkbox" 
              id="form_done" 
              class="h-4 w-4 rounded text-indigo-600 focus:ring-indigo-500 border-slate-300"
            />
            <label for="form_done" class="ml-2 block text-sm font-medium text-slate-700">Oznacz jako wykonane</label>
          </div>

          <!-- Actions -->
          <div class="mt-5 sm:mt-6 flex justify-end space-x-3 border-t border-slate-100 pt-4">
            <button 
              type="button" 
              @click="closeModal()" 
              class="inline-flex justify-center px-4 py-2 border border-slate-300 text-sm font-medium rounded-md text-slate-700 bg-white hover:bg-slate-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition-colors"
            >
              Anuluj
            </button>
            <button 
              type="submit" 
              :disabled="saving"
              class="inline-flex justify-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
            >
              <span v-if="saving">Zapisywanie...</span>
              <span v-else>Zapisz</span>
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

// Komunikacja z API wewnątrz sieci przeglądarki (localhost do kontenera zmapowanego na port 8000)
const API_BASE_URL = 'http://localhost:8000/api';

export default {
  name: 'App',
  data() {
    return {
      tasks: [],
      loading: true,
      saving: false,
      error: null,
      weekDays: [
        { id: 'Monday', name: 'Poniedziałek', bgHeader: 'bg-indigo-50 border-indigo-100' },
        { id: 'Tuesday', name: 'Wtorek', bgHeader: 'bg-emerald-50 border-emerald-100' },
        { id: 'Wednesday', name: 'Środa', bgHeader: 'bg-amber-50 border-amber-100' },
        { id: 'Thursday', name: 'Czwartek', bgHeader: 'bg-purple-50 border-purple-100' },
        { id: 'Friday', name: 'Piątek', bgHeader: 'bg-rose-50 border-rose-100' }
      ],
      modal: {
        show: false,
        isEdit: false,
        editId: null
      },
      form: {
        title: '',
        description: '',
        day_of_week: 'Monday',
        done: false
      }
    };
  },
  methods: {
    // Pobranie wszystkich zadań z API
    async fetchTasks() {
      this.loading = true;
      this.error = null;
      try {
        const response = await axios.get(`${API_BASE_URL}/tasks`);
        this.tasks = response.data;
      } catch (err) {
        console.error('Błąd pobierania zadań:', err);
        this.error = 'Nie udało się połączyć z API Laravela. Upewnij się, że kontenery działają poprawnie.';
      } finally {
        this.loading = false;
      }
    },

    // Filtrowanie zadań dla danego dnia tygodnia
    getTasksByDay(dayId) {
      return this.tasks.filter(task => task.day_of_week === dayId);
    },

    // Otwarcie modalu dla nowego zadania
    openAddModal(dayId = 'Monday') {
      this.modal.isEdit = false;
      this.modal.editId = null;
      this.form = {
        title: '',
        description: '',
        day_of_week: dayId,
        done: false
      };
      this.modal.show = true;
      this.$nextTick(() => {
        if (this.$refs.titleInput) {
          this.$refs.titleInput.focus();
        }
      });
    },

    // Otwarcie modalu do edycji istniejącego zadania
    openEditModal(task) {
      this.modal.isEdit = true;
      this.modal.editId = task.id;
      this.form = {
        title: task.title,
        description: task.description || '',
        day_of_week: task.day_of_week,
        done: task.done
      };
      this.modal.show = true;
    },

    // Zamknięcie modalu i czyszczenie formularza
    closeModal() {
      this.modal.show = false;
    },

    // Zapisywanie formularza (Create lub Update)
    async submitForm() {
      this.saving = true;
      this.error = null;
      try {
        if (this.modal.isEdit) {
          // Edycja zadania
          const response = await axios.put(`${API_BASE_URL}/tasks/${this.modal.editId}`, this.form);
          const index = this.tasks.findIndex(t => t.id === this.modal.editId);
          if (index !== -1) {
            this.tasks[index] = response.data;
          }
        } else {
          // Tworzenie nowego zadania
          const response = await axios.post(`${API_BASE_URL}/tasks`, this.form);
          this.tasks.push(response.data);
        }
        this.closeModal();
      } catch (err) {
        console.error('Błąd podczas zapisywania:', err);
        this.error = 'Wystąpił błąd podczas zapisywania zadania.';
      } finally {
        this.saving = false;
      }
    },

    // Szybkie przełączanie statusu Done (kliknięciem w kółko)
    async toggleTaskDone(task) {
      try {
        const updatedStatus = !task.done;
        const response = await axios.put(`${API_BASE_URL}/tasks/${task.id}`, {
          done: updatedStatus
        });
        const index = this.tasks.findIndex(t => t.id === task.id);
        if (index !== -1) {
          this.tasks[index] = response.data;
        }
      } catch (err) {
        console.error('Błąd przełączania statusu:', err);
        this.error = 'Wystąpił błąd podczas zmiany statusu zadania.';
      }
    },

    // Usuwanie zadania
    async deleteTask(id) {
      if (!confirm('Czy na pewno chcesz usunąć to zadanie?')) {
        return;
      }
      this.error = null;
      try {
        await axios.delete(`${API_BASE_URL}/tasks/${id}`);
        this.tasks = this.tasks.filter(t => t.id !== id);
      } catch (err) {
        console.error('Błąd usuwania zadania:', err);
        this.error = 'Nie udało się usunąć zadania.';
      }
    }
  },
  mounted() {
    this.fetchTasks();
  }
};
</script>
