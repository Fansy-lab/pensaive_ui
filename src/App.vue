<script setup lang="ts">
import { ref, watch, nextTick } from "vue";

const question = ref("");
const answer = ref("");
const currentView = ref<"question" | "answer">("question");
const isAnimating = ref(false);
const answerWords = ref<string[]>([]);
const displayedWords = ref<string[]>([]);
const answerContainer = ref(null);

const sampleAnswers = [
  "Vue 3 is a progressive JavaScript framework that makes building user interfaces intuitive and efficient. With its composition API and reactive system, developers can create dynamic applications with clean, maintainable code.Vue 3 is a progressive JavaScript framework that makes building user interfaces intuitive and efficient. With its composition API and reactive system, developers can create dynamic applications with clean, maintainable code.",
  "Tailwind CSS is a utility-first CSS framework that provides low-level utility classes to build custom designs. It promotes consistency and rapid development through its comprehensive design system.Tailwind CSS is a utility-first CSS framework that provides low-level utility classes to build custom designs. It promotes consistency and rapid development through its comprehensive design system.",
  "Tailwind CSS is a utility-first CSS framework that provides low-level utility classes to build custom designs. It promotes consistency and rapid development through its comprehensive design system.TypeScript adds static type definitions to JavaScript, enabling better code quality, enhanced IDE support, and early error detection during development.",
  "Animation enhances user experience by providing visual feedback and creating smooth transitions between interface states, making applications feel more responsive and engagingTailwind CSS is a utility-first CSS framework that provides low-level utility classes to build custom designs. It promotes consistency and rapid development through its comprehensive design system..",
];

const askQuestion = async () => {
  if (!question.value.trim() || isAnimating.value) return;

  isAnimating.value = true;

  // Simulate API delay
  await new Promise((resolve) => setTimeout(resolve, 500));

  // Get random answer
  const randomAnswer =
    sampleAnswers[Math.floor(Math.random() * sampleAnswers.length)];
  answer.value = randomAnswer;
  answerWords.value = randomAnswer.split(" ");
  displayedWords.value = [];

  // Transition to answer view
  currentView.value = "answer";

  await nextTick();
  await new Promise((resolve) => setTimeout(resolve, 600)); // Wait for transition

  // Animate words appearing one by one
  for (let i = 0; i < answerWords.value.length; i++) {
    await new Promise((resolve) => setTimeout(resolve, 80));
    displayedWords.value.push(answerWords.value[i]);

    // Scroll immediately after word is added
    await nextTick();
    const el = answerContainer.value;
    if (el) {
      el.scrollTop = el.scrollHeight;
    }
  }

  isAnimating.value = false;
};

const goBackToQuestion = () => {
  currentView.value = "question";
  question.value = "";
  answer.value = "";
  answerWords.value = [];
  displayedWords.value = [];
  isAnimating.value = false;
};

const handleKeyPress = (event: KeyboardEvent) => {
  if (event.key === "Enter" && !event.shiftKey) {
    event.preventDefault();
    askQuestion();
  }
};
</script>

<template>
  <div
    class="min-h-screen bg-gradient-to-br from-slate-900 via-purple-900 to-slate-900 relative overflow-hidden">
    <!-- Question Section -->
    <div
      class="absolute inset-0 flex items-center justify-center p-8 transition-transform duration-700 ease-in-out"
      :class="currentView === 'answer' ? '-translate-x-full' : 'translate-x-0'">
      <div class="w-full max-w-xl">
        <div class="text-center mb-16">
          <img src="/logo.png" alt="Logo" class="w-24 h-24 mx-auto mb-4" />
          <h1
            class="text-6xl font-bold text-white mb-6 bg-gradient-to-r from-blue-400 to-purple-400 bg-clip-text text-transparent">
            Ask Anything
          </h1>
          <p class="text-xl text-gray-300 opacity-80">
            Uncover insights from your messages, notion pages, and emails.
          </p>
        </div>

        <div class="relative">
          <input
            v-model="question"
            @keydown="handleKeyPress"
            placeholder="What would you like to know?"
            class="w-full bg-transparent text-white placeholder-gray-400 text-xl py-4 px-2 border-b border-white/20 focus:border-white/40 outline-none transition-all duration-300 text-center"
            :disabled="isAnimating" />

          <div class="flex justify-center mt-8">
            <button
              @click="askQuestion"
              :disabled="!question.trim() || isAnimating"
              class="px-8 py-3 bg-white/10 hover:bg-white/20 disabled:bg-white/5 disabled:cursor-not-allowed text-white rounded-full transition-all duration-300 font-medium backdrop-blur-sm border border-white/20 hover:border-white/30 transform hover:scale-105 disabled:hover:scale-100">
              <span v-if="isAnimating" class="flex items-center">
                <div
                  class="animate-spin rounded-full h-5 w-5 border-2 border-white/30 border-t-white mr-2"></div>
                Thinking...
              </span>
              <span v-else>Ask Question</span>
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Answer Section -->
    <div
      class="absolute inset-0 flex items-center justify-center p-8 bg-gradient-to-bl from-indigo-900/80 to-purple-900/80 backdrop-blur-sm transition-transform duration-700 ease-in-out"
      :class="currentView === 'answer' ? 'translate-x-0' : 'translate-x-full'">
      <div class="w-full max-w-3xl">
        <!-- Question Display -->
        <div class="text-center mb-8">
          <div
            class="inline-block bg-white/10 backdrop-blur-lg rounded-2xl px-6 py-3 border border-white/20 mb-8">
            <p class="text-gray-300 text-sm uppercase tracking-wider mb-2">
              Your Question
            </p>
            <p class="text-white text-lg font-medium">{{ question }}</p>
          </div>

          <h2 class="text-4xl font-bold text-white mb-4">Answer</h2>
          <div
            class="w-24 h-1 bg-gradient-to-r from-blue-400 to-purple-400 mx-auto rounded-full"></div>
        </div>

        <div
          ref="answerContainer"
          class="bg-white/10 backdrop-blur-lg rounded-2xl p-6 border border-white/20 shadow-2xl mb-8 h-[50dvh] overflow-y-auto scroll-on-hover">
          <div class="text-gray-100 text-xl leading-relaxed">
            <span
              v-for="(word, index) in displayedWords"
              :key="index"
              class="word-animation inline-block mr-1"
              :style="{ animationDelay: `${index * 0.01}s` }">
              {{ word }}
            </span>
          </div>
        </div>

        <!-- Back Button -->
        <div class="text-center">
          <button
            @click="goBackToQuestion"
            class="px-8 py-3 bg-white/10 hover:bg-white/20 text-white rounded-full transition-all duration-300 font-medium backdrop-blur-sm border border-white/20 hover:border-white/30 transform hover:scale-105">
            Back
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Smooth transitions for all elements */
.transition-transform {
  transition-property: transform;
}
.scroll-on-hover {
  overflow-y: auto;
  scrollbar-width: thin; /* Firefox */
  scrollbar-color: rgba(255, 255, 255, 0.3) transparent; /* Firefox */
}

/* Chrome, Edge, Safari */
.scroll-on-hover::-webkit-scrollbar {
  width: 8px;
  opacity: 0;
  transition: opacity 0.3s;
}

.scroll-on-hover::-webkit-scrollbar-track {
  background: transparent;
}

.scroll-on-hover::-webkit-scrollbar-thumb {
  background-color: rgba(255, 255, 255, 0.3);
  border-radius: 4px;
}

.scroll-on-hover:hover::-webkit-scrollbar {
  opacity: 1;
}
</style>
