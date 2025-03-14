<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { FlashcardDatabase } from '../services/database';
import type { Flashcard } from '../types';

const db = new FlashcardDatabase();
const flashcards = ref<Flashcard[]>([]);
const newFront = ref('');
const newBack = ref('');
const editingCard = ref<Flashcard | null>(null);

onMounted(async () => {
  await loadFlashcards();
});

async function loadFlashcards() {
  flashcards.value = await db.flashcards.toArray();
}

async function addFlashcard() {
  if (!newFront.value.trim() || !newBack.value.trim()) return;
  
  await db.addFlashcard(newFront.value, newBack.value);
  newFront.value = '';
  newBack.value = '';
  await loadFlashcards();
}

async function deleteFlashcard(id: number) {
  if (!confirm('Are you sure you want to delete this flashcard?')) return;
  await db.deleteFlashcard(id);
  await loadFlashcards();
}

async function startEdit(card: Flashcard) {
  editingCard.value = { ...card };
}

async function saveEdit() {
  if (!editingCard.value) return;
  await db.updateFlashcard(
    editingCard.value.id!,
    editingCard.value.front,
    editingCard.value.back
  );
  editingCard.value = null;
  await loadFlashcards();
}

function cancelEdit() {
  editingCard.value = null;
}
</script>

<template>
  <div class="container mx-auto p-4">
    <h1 class="text-2xl font-bold mb-4">Manage Flashcards</h1>
    
    <!-- Add new flashcard -->
    <div class="card bg-base-100 shadow-xl mb-6">
      <div class="card-body">
        <h2 class="card-title">Add New Flashcard</h2>
        <div class="form-control">
          <label class="label">
            <span class="label-text">Front</span>
          </label>
          <textarea
            v-model="newFront"
            class="textarea textarea-bordered"
            placeholder="Enter the front of the card"
          ></textarea>
        </div>
        <div class="form-control">
          <label class="label">
            <span class="label-text">Back</span>
          </label>
          <textarea
            v-model="newBack"
            class="textarea textarea-bordered"
            placeholder="Enter the back of the card"
          ></textarea>
        </div>
        <div class="card-actions justify-end">
          <button class="btn btn-primary" @click="addFlashcard">Add Card</button>
        </div>
      </div>
    </div>

    <!-- Flashcard list -->
    <div class="grid gap-4">
      <div v-for="card in flashcards" :key="card.id" class="card bg-base-100 shadow-xl">
        <div class="card-body">
          <template v-if="editingCard?.id === card.id">
            <div class="form-control">
              <label class="label">
                <span class="label-text">Front</span>
              </label>
              <textarea
                v-model="editingCard.front"
                class="textarea textarea-bordered"
              ></textarea>
            </div>
            <div class="form-control">
              <label class="label">
                <span class="label-text">Back</span>
              </label>
              <textarea
                v-model="editingCard.back"
                class="textarea textarea-bordered"
              ></textarea>
            </div>
            <div class="card-actions justify-end gap-2">
              <button class="btn btn-ghost" @click="cancelEdit">Cancel</button>
              <button class="btn btn-primary" @click="saveEdit">Save</button>
            </div>
          </template>
          <template v-else>
            <div class="prose">
              <p><strong>Front:</strong> {{ card.front }}</p>
              <p><strong>Back:</strong> {{ card.back }}</p>
              <p v-if="card.lastReviewDate">
                <strong>Last Review:</strong> {{ new Date(card.lastReviewDate).toLocaleDateString() }}
              </p>
            </div>
            <div class="card-actions justify-end gap-2">
              <button class="btn btn-ghost" @click="startEdit(card)">Edit</button>
              <button class="btn btn-error" @click="deleteFlashcard(card.id!)">Delete</button>
            </div>
          </template>
        </div>
      </div>
    </div>
  </div>
</template> 