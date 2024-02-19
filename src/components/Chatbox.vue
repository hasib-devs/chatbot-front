<script setup lang="ts">
import ChatBubbleLeftRight from './icons/ChatBubbleLeftRight.vue';
import { ref } from 'vue';
import PaperAirPlane from './icons/PaperAirPlane.vue';
import { Message } from '../types';

const API_URL = 'http://localhost:5500/chat';

const isOpen = ref(true);
const messages = ref<Message[]>([]);

async function chat(messages: Message[]): Promise<Message> {
  console.log("chat");
  const body = {
    messages: messages
  };

  const response = await fetch(API_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify(body)
  });

  console.log({ response });

  const reader = response.body?.getReader();
  if (!reader) {
    throw new Error("Failed to read response body");
  }
  let content = "";
  while (true) {
    const { done, value } = await reader.read();
    if (done) {
      break;
    }
    const rawjson = new TextDecoder().decode(value);
    const json = JSON.parse(rawjson);
    console.log({ json });
    if (json.done === false) {
      content += json.message.content;
      console.log({ content });
    }

  }
  return { role: "assistant", content: content };
}

async function send() {
  messages.value.push(await chat(messages.value));
}
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
          🙂
        </div>
        <p class="text-center text-sm text-teal-900 mt-1">Chatbot</p>
      </div>

      <div class="absolute bottom-0 w-full border-t px-3 py-3">
        <form @submit.prevent="send" class="flex items-end">
          <input autofocus autocomplete="off" autocorrect="off" auto type="text"
            class="w-full px-3 py-1.5 focus:ring-0 focus:outline-0 placeholder:text-sm flex-1 resize-none " rows="1"
            placeholder="Compose your message" />
          <button type="submit" class="bg-teal-600 text-white px-3 py-1.5 rounded-md">
            <PaperAirPlane />
          </button>
        </form>
      </div>
    </div>
  </div>
</template>


<style scoped></style>