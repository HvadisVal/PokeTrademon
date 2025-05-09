<template>
    <div class="card-selector">
      <!-- ✅ Selected Cards Preview -->
      <div
        v-if="mode === 'select' && props.selectedCards && Object.keys(props.selectedCards).length > 0"
        class="mb-6"
      >
        <h3 class="text-md font-semibold text-gray-800 mb-2">Selected Cards:</h3>
        <div class="flex flex-wrap gap-3">
          <div
            v-for="(qty, cardId) in props.selectedCards"
            :key="cardId"
            class="relative border rounded p-2 bg-white shadow-sm"
          >
            <span
              class="absolute -top-2 -right-2 bg-red-500 text-white rounded-full w-5 h-5 text-xs flex items-center justify-center cursor-pointer"
              @click="emit('card-removed', cardId)"
            >×</span>
            <div class="text-sm font-medium text-gray-800">ID: {{ cardId }}</div>
            <div class="text-xs text-gray-600">Qty: {{ qty }}</div>
          </div>
        </div>
      </div>
      <div class="mb-4">
  <button
    class="text-sm text-blue-600 hover:underline"
    @click="showGrid = !showGrid"
  >
    {{ showGrid ? 'Hide card list' : 'Show card list' }}
  </button>
</div>

  
      <!-- 🔍 Search & Filters -->
      <div v-if="showGrid">
      <div class="mb-4 flex flex-col md:flex-row gap-4">
        <input
          type="text"
          placeholder="Search Pokémon..."
          class="p-2 border rounded white-bg round-corner dark-text"
          v-model="searchQuery"
          @input="fetchFilteredCards"
        />
  
        <select
          v-model="selectedSupertype"
          @change="fetchFilteredCards"
          class="p-2 border rounded white-bg round-corner dark-text"
        >
          <option value="">All Supertypes</option>
          <option value="Pokémon">Pokémon</option>
          <option value="Trainer">Trainer</option>
          <option value="Energy">Energy</option>
        </select>
  
        <select
          v-model="sortBy"
          @change="fetchFilteredCards"
          class="p-2 border rounded white-bg round-corner dark-text"
        >
          <option value="">Sort by</option>
          <option value="name">Name (A–Z)</option>
          <option value="-name">Name (Z–A)</option>
          <option value="hp">HP (Low–High)</option>
          <option value="-hp">HP (High–Low)</option>
          <option value="rarity">Rarity</option>
          <option value="set.name">Set Name</option>
        </select>
      </div>
  
      <!-- 🃏 Card Grid -->
      <div>
        <div v-if="loading">Loading...</div>
        <div v-else-if="error">{{ error }}</div>
        <div v-else-if="!cards.length">No cards found.</div>
        <div v-else class="flex flex-wrap -mx-2">
          <div
            v-for="card in cards"
            :key="card.id"
            class="w-full sm:w-1/2 md:w-1/3 lg:w-1/4 p-2 cursor-pointer relative"
          >
            <div class="p-4 rounded-lg shadow hover:shadow-lg transition bg-white">
              <img :src="card.images.small" :alt="card.name" class="w-full" />
  
              <!-- Optional Quantity badge -->
              <span
                v-if="mode === 'select' && props.selectedCards?.[card.id]"
                class="absolute top-3 left-3 bg-blue-600 text-white text-xs px-2 py-0.5 rounded-full"
              >
                x{{ props.selectedCards[card.id] }}
              </span>
  
              <div class="mt-2 flex justify-between items-center">
                <button class="btn-1" @click="handleCardClick(card)">
                  {{ mode === 'select' ? 'Select Card' : 'See Card' }}
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
  
      <!-- ⏭ Pagination -->
      <div class="flex justify-center gap-4 mt-6" v-if="cards.length > 0">
        <button class="btn-1" @click="prevPage" :disabled="page <= 1">Prev</button>
        <span>Page {{ page }} of {{ totalPages }}</span>
        <button class="btn-1" @click="nextPage" :disabled="page >= totalPages">Next</button>
      </div>
    </div>
  
      <!-- 👁 Modal Preview (only in view mode) -->
      <CardModal
        v-if="mode === 'view' && showModal && selectedCard"
        :visible="showModal"
        :card="selectedCard"
        @close="showModal = false"
      />
    </div>

  </template>
  
  <script setup lang="ts">
  import { ref, onMounted, watch, computed } from 'vue';
  import CardModal from './CardModal.vue';
  import { useCards } from '../modules/useCards';
  import type { PokemonCard } from '../interfaces/card';
  
  const props = defineProps<{
    mode?: 'select' | 'view',
    selectedCards?: Record<string, number>
  }>();
  
  const emit = defineEmits<{
    (e: 'card-selected', card: PokemonCard): void,
    (e: 'card-removed', cardId: string): void
  }>();
  
  const mode = props.mode || 'view';
  const showGrid = ref(true);

  
  const { loading, error, cards, fetchCards, totalCount } = useCards();
  
  const searchQuery = ref('');
  const selectedSupertype = ref('');
  const sortBy = ref('');
  const page = ref(1);
  const pageSize = 20;
  
  const selectedCard = ref<PokemonCard | null>(null);
  const showModal = ref(false);
  
  const totalPages = computed(() => Math.ceil(totalCount.value / pageSize));
  
  const fetchFilteredCards = () => {
    fetchCards(searchQuery.value, selectedSupertype.value, page.value, sortBy.value);
  };
  
  const nextPage = () => {
    if (page.value < totalPages.value) {
      page.value++;
      fetchFilteredCards();
    }
  };
  
  const prevPage = () => {
    if (page.value > 1) {
      page.value--;
      fetchFilteredCards();
    }
  };
  
  const handleCardClick = (card: PokemonCard) => {
    if (mode === 'select') {
      // Toggle support (optional)
      if (props.selectedCards?.[card.id]) {
        emit('card-removed', card.id);
      } else {
        emit('card-selected', card);
      }
    } else {
      selectedCard.value = card;
      showModal.value = true;
    }
  };
  
  onMounted(() => {
    fetchFilteredCards();
  });
  
  watch([searchQuery, selectedSupertype], () => {
    page.value = 1;
    fetchFilteredCards();
  });
  
  watch(() => props.mode, (newMode) => {
    if (newMode === 'select') {
      showModal.value = false;
      selectedCard.value = null;
    }
  });
  </script>
  
  <style scoped>
  /* Customize as needed */
  </style>
  