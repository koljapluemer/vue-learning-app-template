<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { FlashcardDatabase } from '../services/database';
import type { Flashcard } from '../types';
import { Rating, type Grade } from 'ts-fsrs';

const db = new FlashcardDatabase();
const currentCard = ref<Flashcard | null>(null);
const isRevealed = ref(false);
const dueCount = ref(0);

onMounted(async () => {
  await loadNextCard();
});

async function loadNextCard() {
  const dueCards = await db.getDueFlashcards();
  dueCount.value = dueCards.length;
  
  if (dueCards.length === 0) {
    currentCard.value = null;
    return;
  }
  
  // Get a random card from the due cards
  const randomIndex = Math.floor(Math.random() * dueCards.length);
  currentCard.value = dueCards[randomIndex];
  isRevealed.value = false;
}

async function revealCard() {
  isRevealed.value = true;
}

async function scoreCard(grade: Grade) {
  if (!currentCard.value) return;
  
  await db.reviewFlashcard(currentCard.value.id!, grade);
  await loadNextCard();
}
</script>

<template>
  <div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">Practice Flashcards</h1>
    
    <div v-if="currentCard" class="card bg-base-100 shadow-xl max-w-2xl mx-auto">
      <div class="card-body">
        <div class="prose max-w-none">
          <p class="text-lg">{{ currentCard.front }}</p>
          
          <div v-if="isRevealed" class="mt-4">
            <p class="text-lg">{{ currentCard.back }}</p>
          </div>
        </div>
        
        <div class="card-actions justify-center mt-4">
          <button
            v-if="!isRevealed"
            class="btn btn-primary"
            @click="revealCard"
          >
            Reveal Answer
          </button>
          
          <div v-else class="flex flex-row gap-2">
            <button class="btn btn-error" @click="scoreCard(Rating.Again)">
              Wrong
            </button>
            <button class="btn btn-warning" @click="scoreCard(Rating.Hard)">
              Hard
            </button>
            <button class="btn btn-success" @click="scoreCard(Rating.Good)">
              Correct
            </button>
            <button class="btn btn-info" @click="scoreCard(Rating.Easy)">
              Easy
            </button>
          </div>
        </div>
      </div>
    </div>
    
    <div v-else class="text-center">
      <p class="text-xl mb-4">
        {{ dueCount === 0 ? 'No cards due for review!' : 'Loading next card...' }}
      </p>
      <p v-if="dueCount === 0" class="text-gray-600">
        Come back later to practice more cards.
      </p>
    </div>
  </div>
</template> 