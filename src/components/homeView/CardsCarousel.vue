<template>
  <section class="py-18 five-percent">
    <div class="flex flex-col justify-center align-center pb-8" v-motion-fade-slide>
      <h2 class="text-center dark-headline">Selected trader cards</h2>
      <p class="text-center dark-text">A few of the many Pokemon cards you can collect</p>
    </div>

    <!-- loading and error message -->
    <div v-if="loading" class="text-center">Loading...</div>
    <div v-else-if="error" class="text-center text-red-500">There's an error.</div>

    <!-- swiper cards -->
    <div v-else v-motion-fade-slide>
      <div class="swiper" v-motion-fade-slide>
        <div class="swiper-wrapper">
          <div v-for="card in limitedCards" :key="card.id" class="swiper-slide">
            <img :src="card.images.small" alt="Pokemon card" />
          </div>
        </div>

        <!-- arrows to scroll -->
        <div class="swiper-button-prev"></div>
        <div class="swiper-button-next"></div>
      </div>
    </div>

    <div class="flex justify-center align-center pt-12">
      <RouterLink to="/auth"
        ><button class="btn-1" v-motion-fade-slide>Start your collection now</button></RouterLink
      >
    </div>
  </section>
</template>

<script setup lang="ts">
import { onMounted, computed } from 'vue'
// import swiper carousel
import { watch } from 'vue'
import { nextTick } from 'vue'
import { useSwiper } from '../swiperCarousel/SwiperCarousel'
import { useCards } from '../../modules/useCards'

// cards fetching
const { loading, error, cards, fetchCards } = useCards()

onMounted(() => {
  fetchCards()
})

// only display 15 random cards
const limitedCards = computed(() => {
  return cards.value || []
})

// initialize Swiper after its loaded
watch(loading, async (isLoading) => {
  if (!isLoading) {
    await nextTick()
    // initialize Swiper
    useSwiper()
  }
})
</script>

<style scoped>
/* swiper */
.swiper {
  width: 100%;
  max-width: 1000px;
  margin: 0 auto;
  position: relative;
}

.swiper-wrapper {
  display: flex;
}

.swiper-slide {
  position: relative;
  width: auto;
  border-radius: 3px;
  transition: transform 0.3s ease;
  border-radius: 10px;
}

.swiper-slide-active {
  transform: scale(1.5);
}

.swiper-slide img {
  display: block;
  width: 100%;
  height: 100%;
  object-fit: cover;
  border-radius: inherit;
  user-select: none;
}

.swiper-button-prev,
.swiper-button-next {
  z-index: 10;
  color: var(--light-text);
  text-shadow: 0 0 5px rgba(0, 0, 0, 0.5);
}
</style>
