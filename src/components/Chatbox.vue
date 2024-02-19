<script setup lang="ts">
import ChatBubbleLeftRight from './icons/ChatBubbleLeftRight.vue';
import { ref } from 'vue';
const isOpen = ref(true);
</script>

<template>
  <div class="fixed bottom-20 right-20">
    <div @click="isOpen = !isOpen"
      class="cursor-pointer w-20 h-20 rounded-full flex justify-center items-center shadow-lg  bg-teal-600 text-white">
      <ChatBubbleLeftRight class="w-10 h-10" />
    </div>

    <!-- Box -->
    <div v-if="isOpen" class="absolute w-[350px] h-[550px] bg-white bottom-[120%] right-[50%] rounded-lg shadow-lg">
      <div class="h-24 bg-red-50 p-3">
        <div class="w-10 h-10 bg-teal-50 rounded-full flex justify-center items-center mx-auto">
          <!-- <ChatBubbleLeftRight class="h-4 w-4 text-teal-700" /> -->
          🙂
        </div>
        <p class="text-center text-sm text-teal-900 mt-1">Chatbot</p>
      </div>

      <div class="absolute bottom-0 w-full border-t px-3 py-3">
        <input autofocus autocomplete="off" autocorrect="off" auto type="text"
          class="w-full px-3 py-1.5 focus:ring-0 focus:outline-0 placeholder:text-sm" placeholder="Compose your message">
      </div>
    </div>
  </div>
</template>


<style scoped></style>