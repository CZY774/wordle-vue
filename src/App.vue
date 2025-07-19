<!-- Perbaikan untuk App.vue -->

<!-- 1. Ubah wrapper utama untuk sticky footer -->
<template>
  <div
    class="min-h-screen bg-background text-foreground flex flex-col"
    :class="{ 'contrast-125': highContrast }"
  >
    <!-- Header - tambah responsive padding -->
    <div class="border-b border-border px-2 sm:px-4 py-3">
      <div class="max-w-md mx-auto flex justify-between items-center">
        <div class="flex gap-1 sm:gap-2">
          <button
            @click="showStats = true"
            class="p-1.5 sm:p-2 hover:bg-muted rounded transition-colors duration-200"
            :aria-label="'View statistics'"
            :title="'View statistics'"
          >
            <!-- Chart Bar Icon -->
            <svg class="w-4 h-4 sm:w-5 sm:h-5" fill="currentColor" viewBox="0 0 24 24">
              <path
                d="M3 13h2v8H3v-8zm4-6h2v14H7V7zm4-4h2v18h-2V3zm4 8h2v10h-2v-10z"
              />
            </svg>
          </button>
          <button
            @click="hardMode = !hardMode"
            class="p-1.5 sm:p-2 hover:bg-muted rounded transition-colors duration-200"
            :class="{ 'text-yellow-400': hardMode }"
            :aria-label="'Toggle hard mode'"
            :title="hardMode ? 'Switch to Normal Mode' : 'Switch to Hard Mode'"
          >
            <!-- Fire Icon -->
            <svg class="w-4 h-4 sm:w-5 sm:h-5" fill="currentColor" viewBox="0 0 24 24">
              <path
                d="M13.5.67s.74 2.65.74 4.8c0 2.06-1.35 3.73-3.41 3.73-2.07 0-3.63-1.67-3.63-3.73l.03-.36C5.21 7.51 4 10.62 4 14c0 4.42 3.58 8 8 8s8-3.58 8-8C20 8.61 17.41 3.8 13.5.67zM11.71 19c-1.78 0-3.22-1.4-3.22-3.14 0-1.62 1.05-2.76 2.81-3.12 1.77-.36 3.6-1.21 4.62-2.58.39 1.29.28 2.92-.73 4.22-1.01 1.29-2.75 1.85-4.48 1.85v2.77z"
              />
            </svg>
          </button>
        </div>

        <h1 class="text-xl sm:text-2xl font-bold tracking-wider">WORDLE</h1>

        <div class="flex gap-1 sm:gap-2">
          <button
            @click="highContrast = !highContrast"
            class="p-1.5 sm:p-2 hover:bg-muted rounded transition-colors duration-200"
            :class="{ 'text-blue-400': highContrast }"
            :aria-label="'Toggle high contrast'"
            :title="
              highContrast ? 'Toggle Normal Contrast' : 'Toggle High Contrast'
            "
          >
            <!-- Contrast Icon -->
            <svg class="w-4 h-4 sm:w-5 sm:h-5" fill="currentColor" viewBox="0 0 24 24">
              <path
                d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-1 17.93c-3.94-.49-7-3.85-7-7.93s3.05-7.44 7-7.93v15.86z"
              />
            </svg>
          </button>
          <button
            @click="startNewGame"
            class="p-1.5 sm:p-2 hover:bg-muted rounded transition-colors duration-200"
            :aria-label="'New game'"
            :title="'New game'"
          >
            <!-- Refresh Icon -->
            <svg class="w-4 h-4 sm:w-5 sm:h-5" fill="currentColor" viewBox="0 0 24 24">
              <path
                d="M17.65 6.35C16.2 4.9 14.21 4 12 4c-4.42 0-7.99 3.58-7.99 8s3.57 8 7.99 8c3.73 0 6.84-2.55 7.73-6h-2.08c-.82 2.33-3.04 4-5.65 4-3.31 0-6-2.69-6-6s2.69-6 6-6c1.66 0 3.14.69 4.22 1.78L13 11h7V4l-2.35 2.35z"
              />
            </svg>
          </button>
        </div>
      </div>
    </div>

    <!-- Main content - flex-1 untuk mengambil sisa ruang -->
    <div class="container mx-auto px-2 sm:px-4 py-4 sm:py-8 flex-1">
      <!-- Loading State -->
      <div v-if="loading" class="text-center">
        <div
          class="inline-block animate-spin rounded-full h-8 w-8 border-b-2 border-primary"
        ></div>
        <p class="mt-2 text-muted-foreground">Loading words...</p>
      </div>

      <!-- Game Board -->
      <div v-else class="max-w-md mx-auto">
        <!-- Game Grid - responsive cell size -->
        <div
          class="grid grid-rows-6 gap-1 sm:gap-2 mb-6 sm:mb-8"
          role="grid"
          :aria-label="'Wordle game board'"
        >
          <div
            v-for="(row, rowIndex) in gameBoard"
            :key="rowIndex"
            class="grid grid-cols-5 gap-1 sm:gap-2 transition-transform duration-300"
            :class="{ 'animate-shake': shakeRow === rowIndex }"
            role="row"
          >
            <div
              v-for="(cell, colIndex) in row"
              :key="colIndex"
              class="w-10 h-10 sm:w-12 sm:h-12 border-2 border-border rounded flex items-center justify-center text-base sm:text-lg font-bold uppercase transition-all duration-300 select-none"
              :class="getCellClass(rowIndex, colIndex, cell)"
              :style="{
                animationDelay: revealedRows.has(rowIndex)
                  ? `${colIndex * FLIP_DELAY}ms`
                  : '0ms',
              }"
              role="gridcell"
              :aria-label="
                cell.letter ? `${cell.letter} ${cell.status}` : 'empty cell'
              "
            >
              {{ cell.letter }}
            </div>
          </div>
        </div>

        <!-- Game Status - responsive text -->
        <div class="text-center mb-4 sm:mb-6 min-h-[60px] flex flex-col justify-center px-2">
          <div
            v-if="gameStatus === 'won'"
            class="text-green-400 animate-bounce-in"
          >
            <p class="text-lg sm:text-xl font-bold mb-2">🎉 Fantastic!</p>
            <p class="mb-3 text-sm sm:text-base">
              You guessed the word in {{ currentRow + 1 }} tries!
            </p>
            <button
              @click="shareResults"
              class="flex items-center gap-2 px-3 sm:px-4 py-2 bg-green-600 text-white rounded-lg font-bold hover:bg-green-700 transition-colors duration-200 mx-auto text-sm sm:text-base"
              :disabled="shareButtonText !== 'Share'"
            >
              <!-- Share Icon -->
              <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24">
                <path
                  d="M18 16.08c-.76 0-1.44.3-1.96.77L8.91 12.7c.05-.23.09-.46.09-.7s-.04-.47-.09-.7l7.05-4.11c.54.5 1.25.81 2.04.81 1.66 0 3-1.34 3-3s-1.34-3-3-3-3 1.34-3 3c0 .24.04.47.09.7L8.04 9.81C7.5 9.31 6.79 9 6 9c-1.66 0-3 1.34-3 3s1.34 3 3 3c.79 0 1.5-.31 2.04-.81l7.12 4.16c-.05.21-.08.43-.08.65 0 1.61 1.31 2.92 2.92 2.92 1.61 0 2.92-1.31 2.92-2.92s-1.31-2.92-2.92-2.92z"
                />
              </svg>
              {{ shareButtonText }}
            </button>
          </div>
          <div
            v-else-if="gameStatus === 'lost'"
            class="text-red-400 animate-fade-in"
          >
            <p class="text-lg sm:text-xl font-bold mb-2">💀 Game Over!</p>
            <p class="mb-3 text-sm sm:text-base">
              The word was:
              <span class="font-bold text-primary">{{ currentWord }}</span>
            </p>
            <button
              @click="shareResults"
              class="flex items-center gap-2 px-3 sm:px-4 py-2 bg-red-600 text-white rounded-lg font-bold hover:bg-red-700 transition-colors duration-200 mx-auto text-sm sm:text-base"
              :disabled="shareButtonText !== 'Share'"
            >
              <!-- Share Icon -->
              <svg class="w-4 h-4" fill="currentColor" viewBox="0 0 24 24">
                <path
                  d="M18 16.08c-.76 0-1.44.3-1.96.77L8.91 12.7c.05-.23.09-.46.09-.7s-.04-.47-.09-.7l7.05-4.11c.54.5 1.25.81 2.04.81 1.66 0 3-1.34 3-3s-1.34-3-3-3-3 1.34-3 3c0 .24.04.47.09.7L8.04 9.81C7.5 9.31 6.79 9 6 9c-1.66 0-3 1.34-3 3s1.34 3 3 3c.79 0 1.5-.31 2.04-.81l7.12 4.16c-.05.21-.08.43-.08.65 0 1.61 1.31 2.92 2.92 2.92 1.61 0 2.92-1.31 2.92-2.92s-1.31-2.92-2.92-2.92z"
                />
              </svg>
              {{ shareButtonText }}
            </button>
          </div>
          <div v-else-if="error" class="text-red-400 font-bold animate-shake text-sm sm:text-base">
            {{ error }}
          </div>
          <div
            v-else-if="gameStatus === 'playing'"
            class="text-muted-foreground"
          >
            <p class="text-sm sm:text-base">Guess the WORDLE in {{ MAX_GUESSES }} tries</p>
            <p v-if="hardMode" class="text-yellow-400 mt-1 text-xs sm:text-sm">
              🔥 Hard Mode Active: No yellow hints!
            </p>
          </div>
        </div>

        <!-- Virtual Keyboard - responsive -->
        <div class="space-y-1 sm:space-y-2 mb-6" role="group" :aria-label="'Virtual keyboard'">
          <div
            v-for="(row, index) in keyboard"
            :key="index"
            class="flex justify-center gap-1"
          >
            <button
              v-for="key in row"
              :key="key"
              @click="handleKeyPress(key)"
              class="px-1 sm:px-2 py-2 sm:py-3 rounded font-bold text-xs sm:text-sm transition-all duration-200 h-10 sm:h-12 flex items-center justify-center select-none focus:outline-none focus:ring-2 focus:ring-primary active:transform active:scale-95"
              :class="[
                getKeyClass(key),
                key === 'ENTER' || key === 'BACKSPACE' ? 'min-w-[48px] sm:min-w-[60px]' : 'min-w-[28px] sm:min-w-[32px]'
              ]"
              :disabled="gameStatus !== 'playing'"
              :aria-label="
                key === 'BACKSPACE'
                  ? 'Backspace'
                  : key === 'ENTER'
                  ? 'Enter'
                  : key
              "
            >
              <span v-if="key === 'BACKSPACE'">
                <!-- Backspace Icon -->
                <svg class="w-4 h-4 sm:w-5 sm:h-5" fill="currentColor" viewBox="0 0 24 24">
                  <path
                    d="M22 3H7c-.69 0-1.23.35-1.59.88L0 12l5.41 8.11c.36.53.9.89 1.59.89h15c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zm-3 12.59L17.59 17 14 13.41 10.41 17 9 15.59 12.59 12 9 8.41 10.41 7 14 10.59 17.59 7 19 8.41 15.41 12 19 15.59z"
                  />
                </svg>
              </span>
              <span v-else-if="key === 'ENTER'">
                <!-- Enter Icon -->
                <svg class="w-4 h-4 sm:w-5 sm:h-5" fill="currentColor" viewBox="0 0 24 24">
                  <path
                    d="M19 7v4H5.83l3.58-3.59L8 6l-6 6 6 6 1.41-1.41L5.83 13H21V7z"
                  />
                </svg>
              </span>
              <span v-else>{{ key }}</span>
            </button>
          </div>
        </div>

        <!-- Tech Stack - responsive dan compact -->
        <div class="flex flex-wrap justify-center gap-2 mb-4">
          <div
            class="flex items-center gap-1 px-2 py-1 bg-secondary/50 rounded-md text-xs"
          >
            <Zap class="w-3 h-3" />
            <span class="hidden sm:inline">Vue 3 + Vite</span>
            <span class="sm:hidden">Vue</span>
          </div>
          <div
            class="flex items-center gap-1 px-2 py-1 bg-secondary/50 rounded-md text-xs"
          >
            <Palette class="w-3 h-3" />
            <span class="hidden sm:inline">Tailwind CSS</span>
            <span class="sm:hidden">CSS</span>
          </div>
          <div
            class="flex items-center gap-1 px-2 py-1 bg-secondary/50 rounded-md text-xs"
          >
            <Code class="w-3 h-3" />
            <span class="hidden sm:inline">TypeScript</span>
            <span class="sm:hidden">TS</span>
          </div>
          <div
            class="flex items-center gap-1 px-2 py-1 bg-secondary/50 rounded-md text-xs"
          >
            <Heart class="w-3 h-3" />
            <span class="hidden sm:inline">Lucide Icons</span>
            <span class="sm:hidden">Icons</span>
          </div>
        </div>
      </div>
    </div>

    <!-- Footer - sticky di bawah -->
    <footer class="border-t border-border py-4 sm:py-6 mt-auto">
      <div class="max-w-md mx-auto px-2 sm:px-4 text-center">
        <div class="text-sm text-muted-foreground mb-3">
          <p class="mb-2">
            Created by
            <span class="font-semibold text-foreground">Cornelius Yoga</span>
          </p>
          <p class="text-xs">
            Heavily inspired by
            <a
              href="https://www.youtube.com/watch?v=5xf4_Kx7azg"
              class="italic hover:text-foreground transition-colors"
              target="_blank"
              rel="noopener noreferrer"
              >Conner Ardman feat. Clément Mihailescu</a
            >
          </p>
        </div>
        
        <!-- Links - responsive -->
        <div class="flex justify-center gap-3 sm:gap-4 text-xs text-muted-foreground">
          <a
            href="https://www.instagram.com/corneliusyoga"
            class="hover:text-foreground transition-colors duration-200 flex items-center gap-1"
            target="_blank"
          >
            <Instagram class="w-3 h-3" />
            <span class="hidden sm:inline">Instagram</span>
            <span class="sm:hidden">IG</span>
          </a>
          <a
            href="https://github.com/CZY774"
            class="hover:text-foreground transition-colors duration-200 flex items-center gap-1"
            target="_blank"
          >
            <Github class="w-3 h-3" />
            <span class="hidden sm:inline">GitHub</span>
            <span class="sm:hidden">GH</span>
          </a>
          <a
            href="https://czy.digital"
            class="hover:text-foreground transition-colors duration-200 flex items-center gap-1"
            target="_blank"
          >
            <Globe class="w-3 h-3" />
            <span class="hidden sm:inline">Portfolio</span>
            <span class="sm:hidden">Web</span>
          </a>
        </div>
      </div>
    </footer>

    <!-- Statistics Modal - responsive -->
    <div
      v-if="showStats"
      class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-2 sm:p-4"
      @click.self="showStats = false"
    >
      <div class="bg-card rounded-lg p-4 sm:p-6 max-w-sm w-full animate-fade-in max-h-[90vh] overflow-y-auto">
        <div class="flex justify-between items-center mb-4">
          <h2 class="text-lg sm:text-xl font-bold text-foreground">Statistics</h2>
          <button
            @click="showStats = false"
            class="text-muted-foreground hover:text-foreground p-1"
            :aria-label="'Close statistics'"
          >
            <!-- Close Icon -->
            <svg class="w-5 h-5 sm:w-6 sm:h-6" fill="currentColor" viewBox="0 0 24 24">
              <path
                d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"
              />
            </svg>
          </button>
        </div>

        <div class="grid grid-cols-4 gap-2 sm:gap-4 mb-6">
          <div class="text-center">
            <div class="text-xl sm:text-2xl font-bold text-foreground">
              {{ stats.gamesPlayed }}
            </div>
            <div class="text-xs text-muted-foreground">Played</div>
          </div>
          <div class="text-center">
            <div class="text-xl sm:text-2xl font-bold text-foreground">
              {{ winPercentage }}
            </div>
            <div class="text-xs text-muted-foreground">Win %</div>
          </div>
          <div class="text-center">
            <div class="text-xl sm:text-2xl font-bold text-foreground">
              {{ stats.currentStreak }}
            </div>
            <div class="text-xs text-muted-foreground">Current Streak</div>
          </div>
          <div class="text-center">
            <div class="text-xl sm:text-2xl font-bold text-foreground">
              {{ stats.maxStreak }}
            </div>
            <div class="text-xs text-muted-foreground">Max Streak</div>
          </div>
        </div>

        <div class="mb-4">
          <h3 class="text-sm font-bold text-foreground mb-2">
            GUESS DISTRIBUTION
          </h3>
          <div class="space-y-1">
            <div
              v-for="(count, index) in stats.guessDistribution"
              :key="index"
              class="flex items-center gap-2"
            >
              <div class="text-foreground text-sm w-3">{{ index + 1 }}</div>
              <div class="flex-1 bg-muted rounded h-4 sm:h-5 relative overflow-hidden">
                <div
                  class="bg-green-600 h-full rounded transition-all duration-500"
                  :style="{ width: `${(count / maxGuessCount) * 100}%` }"
                />
                <div
                  class="absolute right-1 sm:right-2 top-0 text-foreground text-xs leading-4 sm:leading-5"
                >
                  {{ count }}
                </div>
              </div>
            </div>
          </div>
        </div>

        <div class="text-center">
          <button
            @click="showStats = false"
            class="px-4 sm:px-6 py-2 bg-primary text-primary-foreground rounded-lg font-bold hover:bg-primary/90 transition-colors duration-200 text-sm sm:text-base"
          >
            Close
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, onUnmounted, computed } from "vue";
import {
  Zap,
  Palette,
  Code,
  Heart,
  Instagram,
  Github,
  Globe,
} from "lucide-vue-next";
import { wordList } from "./data/words";

// Constants
const MAX_GUESSES = 6;
const WORD_LENGTH = 5;
const FLIP_DELAY = 100;
const SHAKE_DURATION = 600;
const ANIMATION_DURATION = 300;

// Types
interface Cell {
  letter: string;
  status: "empty" | "pending" | "correct" | "present" | "absent";
}

interface GameStats {
  gamesPlayed: number;
  gamesWon: number;
  currentStreak: number;
  maxStreak: number;
  guessDistribution: number[];
}

type GameStatus = "playing" | "won" | "lost";

// Game state
const loading = ref(false);
const error = ref("");
const words = ref<string[]>(wordList);
const currentWord = ref("");
const currentRow = ref(0);
const currentCol = ref(0);
const gameStatus = ref<GameStatus>("playing");
const currentGuess = ref("");
const shakeRow = ref(-1);
const revealedRows = ref(new Set<number>());

// Settings
const hardMode = ref(false);
const highContrast = ref(false);
const showStats = ref(false);
const shareButtonText = ref("Share");

// Game board (6 rows, 5 columns)
const gameBoard = reactive<Cell[][]>(
  Array(MAX_GUESSES)
    .fill(null)
    .map(() =>
      Array(WORD_LENGTH)
        .fill(null)
        .map(() => ({ letter: "", status: "empty" }))
    )
);

// Keyboard layout
const keyboard = [
  ["Q", "W", "E", "R", "T", "Y", "U", "I", "O", "P"],
  ["A", "S", "D", "F", "G", "H", "J", "K", "L"],
  ["ENTER", "Z", "X", "C", "V", "B", "N", "M", "BACKSPACE"],
];

// Track key statuses
const keyStatuses = reactive<
  Record<string, "correct" | "present" | "absent" | "unused">
>({});

// Statistics
const stats = ref<GameStats>({
  gamesPlayed: 0,
  gamesWon: 0,
  currentStreak: 0,
  maxStreak: 0,
  guessDistribution: [0, 0, 0, 0, 0, 0],
});

// Select random word
const selectRandomWord = () => {
  const randomIndex = Math.floor(Math.random() * words.value.length);
  currentWord.value = words.value[randomIndex].toUpperCase();
  console.log("Current word:", currentWord.value); // For debugging
};

// Computed properties
const isRowComplete = computed(() => currentCol.value === WORD_LENGTH);
const canSubmit = computed(
  () => isRowComplete.value && gameStatus.value === "playing"
);
const winPercentage = computed(() =>
  stats.value.gamesPlayed > 0
    ? Math.round((stats.value.gamesWon / stats.value.gamesPlayed) * 100)
    : 0
);
const maxGuessCount = computed(() =>
  Math.max(...stats.value.guessDistribution, 1)
);

// Show error with shake animation
const showError = (message: string) => {
  error.value = message;
  shakeRow.value = currentRow.value;

  setTimeout(() => {
    error.value = "";
    shakeRow.value = -1;
  }, SHAKE_DURATION);
};

// Get cell class based on status
const getCellClass = (rowIndex: number, colIndex: number, cell: Cell) => {
  const baseClass = "";

  if (cell.status === "correct") {
    return `${baseClass} bg-green-600 border-green-600 text-white animate-flip`;
  } else if (cell.status === "present") {
    return `${baseClass} bg-yellow-600 border-yellow-600 text-white animate-flip`;
  } else if (cell.status === "absent") {
    return `${baseClass} bg-muted border-muted text-white animate-flip`;
  } else if (cell.status === "pending") {
    return `${baseClass} border-primary bg-primary/10 animate-pulse`;
  } else if (cell.letter) {
    return `${baseClass} border-primary bg-primary/5`;
  }

  return baseClass;
};

// Get key class based on status
const getKeyClass = (key: string) => {
  const baseClass = "border border-border";

  if (key === "ENTER" || key === "BACKSPACE") {
    return `${baseClass} bg-muted text-muted-foreground hover:bg-muted/80 px-4`;
  }

  const status = keyStatuses[key];
  if (status === "correct") {
    return `${baseClass} bg-green-600 text-white`;
  } else if (status === "present") {
    return `${baseClass} bg-yellow-600 text-white`;
  } else if (status === "absent") {
    return `${baseClass} bg-muted text-white`;
  }

  return `${baseClass} bg-secondary text-secondary-foreground hover:bg-secondary/80`;
};

// Handle key press
const handleKeyPress = (key: string) => {
  if (gameStatus.value !== "playing") return;

  if (key === "ENTER") {
    handleSubmit();
  } else if (key === "BACKSPACE") {
    handleBackspace();
  } else if (key.length === 1 && currentCol.value < WORD_LENGTH) {
    handleLetterInput(key);
  }
};

// Handle letter input
const handleLetterInput = (letter: string) => {
  if (currentCol.value < WORD_LENGTH) {
    gameBoard[currentRow.value][currentCol.value].letter = letter.toUpperCase();
    gameBoard[currentRow.value][currentCol.value].status = "pending";
    currentCol.value++;
    currentGuess.value += letter.toUpperCase();
  }
};

// Handle backspace
const handleBackspace = () => {
  if (currentCol.value > 0) {
    currentCol.value--;
    gameBoard[currentRow.value][currentCol.value].letter = "";
    gameBoard[currentRow.value][currentCol.value].status = "empty";
    currentGuess.value = currentGuess.value.slice(0, -1);
  }
};

// Handle submit
const handleSubmit = () => {
  if (!canSubmit.value) {
    showError("Not enough letters");
    return;
  }

  if (!words.value.includes(currentGuess.value)) {
    showError("Not in word list");
    return;
  }

  error.value = "";
  checkGuess();
};

// Check guess against current word
const checkGuess = () => {
  const guess = currentGuess.value;
  const target = currentWord.value;
  const result: ("correct" | "present" | "absent")[] = [];

  // Count letters in target word
  const targetLetterCount: Record<string, number> = {};
  for (const letter of target) {
    targetLetterCount[letter] = (targetLetterCount[letter] || 0) + 1;
  }

  // First pass: mark correct letters
  for (let i = 0; i < WORD_LENGTH; i++) {
    if (guess[i] === target[i]) {
      result[i] = "correct";
      targetLetterCount[guess[i]]--;
    }
  }

  // Second pass: mark present/absent letters
  for (let i = 0; i < WORD_LENGTH; i++) {
    if (result[i] === "correct") continue;

    // Hard mode: no yellow blocks (present letters)
    if (hardMode.value) {
      result[i] = "absent";
    } else {
      if (targetLetterCount[guess[i]] > 0) {
        result[i] = "present";
        targetLetterCount[guess[i]]--;
      } else {
        result[i] = "absent";
      }
    }
  }

  // Mark row as revealed for animation
  revealedRows.value.add(currentRow.value);

  // Update game board with animation delay
  for (let i = 0; i < WORD_LENGTH; i++) {
    setTimeout(() => {
      gameBoard[currentRow.value][i].status = result[i];

      // Update key status
      const letter = guess[i];
      const currentKeyStatus = keyStatuses[letter];

      if (
        result[i] === "correct" ||
        (result[i] === "present" && currentKeyStatus !== "correct") ||
        (result[i] === "absent" && !currentKeyStatus)
      ) {
        keyStatuses[letter] = result[i];
      }
    }, i * FLIP_DELAY);
  }

  // Check win condition
  setTimeout(() => {
    if (guess === target) {
      gameStatus.value = "won";
      updateStats(true, currentRow.value + 1);
    } else if (currentRow.value === MAX_GUESSES - 1) {
      gameStatus.value = "lost";
      updateStats(false, MAX_GUESSES);
    } else {
      // Move to next row
      currentRow.value++;
      currentCol.value = 0;
      currentGuess.value = "";
    }
  }, ANIMATION_DURATION + WORD_LENGTH * FLIP_DELAY);
};

// Update statistics
const updateStats = (won: boolean, guesses: number) => {
  stats.value.gamesPlayed++;

  if (won) {
    stats.value.gamesWon++;
    stats.value.currentStreak++;
    stats.value.maxStreak = Math.max(
      stats.value.maxStreak,
      stats.value.currentStreak
    );
    stats.value.guessDistribution[guesses - 1]++;
  } else {
    stats.value.currentStreak = 0;
  }
};

// Share results
const shareResults = async () => {
  const attempts = gameStatus.value === "won" ? currentRow.value + 1 : "X";
  let grid = `Wordle ${attempts}/${MAX_GUESSES}\n\n`;

  for (
    let i = 0;
    i <= (gameStatus.value === "won" ? currentRow.value : MAX_GUESSES - 1);
    i++
  ) {
    for (let j = 0; j < WORD_LENGTH; j++) {
      const status = gameBoard[i][j].status;
      if (status === "correct") {
        grid += "🟩";
      } else if (status === "present") {
        grid += "🟨";
      } else {
        grid += "⬜";
      }
    }
    grid += "\n";
  }

  try {
    if (navigator.share) {
      await navigator.share({ text: grid });
    } else {
      await navigator.clipboard.writeText(grid);
    }
    shareButtonText.value = "Copied!";
    setTimeout(() => {
      shareButtonText.value = "Share";
    }, 2000);
  } catch (error) {
    console.error("Error sharing:", error);
    shareButtonText.value = "Error";
    setTimeout(() => {
      shareButtonText.value = "Share";
    }, 2000);
  }
};

// Start new game
const startNewGame = () => {
  currentRow.value = 0;
  currentCol.value = 0;
  currentGuess.value = "";
  gameStatus.value = "playing";
  error.value = "";
  shakeRow.value = -1;
  revealedRows.value.clear();
  shareButtonText.value = "Share";

  // Reset game board
  for (let i = 0; i < MAX_GUESSES; i++) {
    for (let j = 0; j < WORD_LENGTH; j++) {
      gameBoard[i][j].letter = "";
      gameBoard[i][j].status = "empty";
    }
  }

  // Reset key statuses
  Object.keys(keyStatuses).forEach((key) => {
    delete keyStatuses[key];
  });

  // Select new word
  selectRandomWord();
};

// Keyboard event listener
const handleKeyboardEvent = (event: KeyboardEvent) => {
  const key = event.key.toUpperCase();

  if (key === "ENTER") {
    event.preventDefault();
    handleKeyPress("ENTER");
  } else if (key === "BACKSPACE") {
    event.preventDefault();
    handleKeyPress("BACKSPACE");
  } else if (key.length === 1 && key.match(/[A-Z]/)) {
    event.preventDefault();
    handleKeyPress(key);
  }
};

// Initialize game
onMounted(() => {
  selectRandomWord();
  window.addEventListener("keydown", handleKeyboardEvent);
});

// Cleanup
onUnmounted(() => {
  window.removeEventListener("keydown", handleKeyboardEvent);
});
</script>

<style scoped>
@keyframes flip {
  0% {
    transform: rotateY(0deg);
  }
  50% {
    transform: rotateY(90deg);
  }
  100% {
    transform: rotateY(0deg);
  }
}

@keyframes shake {
  0%,
  100% {
    transform: translateX(0);
  }
  10%,
  30%,
  50%,
  70%,
  90% {
    transform: translateX(-5px);
  }
  20%,
  40%,
  60%,
  80% {
    transform: translateX(5px);
  }
}

@keyframes bounce-in {
  0% {
    transform: scale(0.3) rotate(-5deg);
    opacity: 0;
  }
  50% {
    transform: scale(1.05) rotate(2deg);
  }
  70% {
    transform: scale(0.9) rotate(-1deg);
  }
  100% {
    transform: scale(1) rotate(0deg);
    opacity: 1;
  }
}

@keyframes fade-in {
  0% {
    opacity: 0;
    transform: translateY(20px);
  }
  100% {
    opacity: 1;
    transform: translateY(0);
  }
}

.animate-flip {
  animation: flip 0.6s ease-in-out;
}

.animate-shake {
  animation: shake 0.6s ease-in-out;
}

.animate-bounce-in {
  animation: bounce-in 0.8s ease-out;
}

.animate-fade-in {
  animation: fade-in 0.5s ease-out;
}
</style>
