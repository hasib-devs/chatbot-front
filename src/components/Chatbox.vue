<script setup lang="ts">
import ChatBubbleLeftRight from './icons/ChatBubbleLeftRight.vue';
import { nextTick, ref } from 'vue';
import PaperAirPlane from './icons/PaperAirPlane.vue';
import { Message } from '../types';
import ThinkingLoader from './ThinkingLoader.vue';
const API_URL = 'http://localhost:5500/chat';

const scrollElement = ref<HTMLElement | null>(null);

const isOpen = ref(true);
const isThinking = ref(false);
const isTyping = ref(false);
const messages = ref<Message[]>([
  { role: "assistant", content: "Hello, I'm a chatbot. How can I help you?" }
]);

let controller = new AbortController();
let signal = controller.signal;

async function stopChat() {
  isThinking.value = false;
  isTyping.value = false;

  controller.abort();
  controller = new AbortController();
  signal = controller.signal;
}

function scrollToBottom() {
  nextTick(() => {
    if (scrollElement.value) {
      scrollElement.value.scrollTop = scrollElement.value.scrollHeight;
    }
  });
}

async function streamToString(body: any) {
  const index = messages.value.length - 1;
  const reader = body?.pipeThrough(new TextDecoderStream()).getReader();
  isThinking.value = false;
  isTyping.value = true;
  while (reader) {
    let stream = await reader.read();
    if (stream.done) break;
    const chunks = stream.value
      .replaceAll(/^data: /gm, "")
      .split("\n")
      .filter((c: string) => Boolean(c.length) && c !== "[DONE]")
      .map((c: string) => JSON.parse(c));
    if (chunks) {
      for (let chunk of chunks) {
        const content = chunk.choices[0].delta.content;
        if (!content) continue;
        messages.value[index].content += content;
      }
    }
    scrollToBottom();
  }
  isTyping.value = false;
}

async function chat(messages: Message[]) {
  const body = {
    messages: messages
  };

  try {
    const response = await fetch(API_URL, {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify(body),
      signal
    });
    await streamToString(response.body);
  } catch (error) {
    // console.error(error);
  }



}

async function send(event: Event) {
  stopChat();
  isThinking.value = true;

  const target = event.target as HTMLFormElement;
  const formData = new FormData(target);
  const message = formData.get("message");
  target.reset();

  messages.value.push({ role: "user", content: message as string });
  messages.value.push({ role: "assistant", content: '' });
  scrollToBottom();

  await chat(messages.value);
}
</script>

<template>
  <div class="fixed bottom-20 right-20">
    <div @click="isOpen = !isOpen"
      class="cursor-pointer w-20 h-20 rounded-full flex justify-center items-center shadow-lg  bg-teal-600 text-white">
      <ChatBubbleLeftRight class="w-10 h-10" />
    </div>

    <!-- Box -->
    <div v-if="isOpen"
      class="absolute w-[350px] h-[550px] overflow-hidden bg-white bottom-[120%] right-[50%] rounded-lg shadow-lg">
      <!-- Box Header -->
      <div class="h-[85px] bg-red-50 p-3">
        <div class="w-10 h-10 bg-teal-50 rounded-full flex justify-center items-center mx-auto">
          🙂
        </div>
        <p class="text-center text-sm text-teal-900 mt-1">Chatbot</p>
      </div>

      <!-- Box messages -->
      <div class="h-[390px] overflow-y-auto" ref="scrollElement">
        <div v-for="msg in messages" class="text-sm">
          <!-- User text -->
          <div v-if="msg.role === 'user'" class="flex justify-end">
            <div class="bg-slate-200 text-gray-800 px-3 py-2 rounded-md m-2">
              {{ msg.content }}
            </div>
          </div>

          <!-- Bot text -->
          <div v-else class="flex justify-start px-3 py-2">
            <p class="text-2xl" v-if="!msg.content.length && isThinking">🤔</p>
            <p class="text-2xl" v-else>😎</p>
            <div class="bg-teal-600 text-white w-full px-3 py-2 rounded-md m-2">
              <p v-if="msg.content.length">
                {{ msg.content }}
              </p>
              <div v-if="!msg.content.length && isThinking" class="px-4 py-1.5">
                <ThinkingLoader />
              </div>
            </div>
          </div>
        </div>

        <div class="mt-2 flex justify-center">
          <button @click="stopChat" v-if="isTyping"
            class="bg-gray-600 text-xs text-white px-3 py-1.5 rounded-md">Stop</button>
        </div>
      </div>

      <!-- Box footer -->
      <div class="absolute bottom-0 w-full border-t px-3 py-3">
        <form @submit.prevent="send" class="flex items-end">
          <input autofocus autocomplete="off" autocorrect="off" name="message" auto type="text"
            class="w-full px-3 py-1.5 focus:ring-0 focus:outline-0 placeholder:text-sm flex-1 resize-none " rows="1"
            placeholder="Compose your message..." />
          <button type="submit" class="bg-teal-600 text-white px-3 py-1.5 rounded-md">
            <PaperAirPlane />
          </button>
        </form>
      </div>
    </div>
  </div>
</template>


<style scoped></style>