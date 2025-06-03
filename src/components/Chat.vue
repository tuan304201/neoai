<template>
  <div class="max-w-md mx-auto p-4 flex flex-col h-screen">
    <h1 class="text-2xl font-bold text-white mb-4 text-center">AIOPS TECH</h1>
    <!-- Chat area -->
    <div class="flex-1 overflow-y-auto mb-4 space-y-4 no-scrollbar pr-4">
      <div
        v-for="message in messages"
        :key="message.id"
        :class="['flex', message.sender === 'user' ? 'justify-end' : 'justify-start']"
      >
        <div
          :class="['max-w-xs p-3 rounded-lg break-words', message.sender === 'user' ? 'bg-blue-600' : 'bg-gray-800']"
        >
          <p class="text-sm">{{ message.text }}</p>
          <span class="text-xs text-gray-400">{{ message.time }}</span>
        </div>
      </div>
      <!-- Loading spinner -->
      <div v-if="isLoading" class="flex justify-start">
        <div class="max-w-xs p-3 rounded-lg bg-gray-800 flex items-center">
          <div class="animate-spin rounded-full h-5 w-5 border-t-2 border-b-2 border-blue-500"></div>
          <span class="ml-2 text-sm text-gray-400">Đang xử lý...</span>
        </div>
      </div>
    </div>

    <!-- Input area -->
    <div class="flex gap-2 pr-3">
      <div class="relative flex-1">
        <input
          v-model="newMessage"
          @keyup.enter="sendMessage"
          placeholder="Nhập tin nhắn..."
          class="flex-1 bg-[#242628] border border-gray-700 rounded p-2 pr-10 text-white focus:outline-none focus:border-gray-600 text-sm w-full"
        />
        <button
          @click="sendMessage"
          class="absolute right-2 top-1/2 transform -translate-y-1/2 bg-blue-600 hover:bg-blue-700 p-1 rounded transition"
        >
          <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24">
            <path
              fill="currentColor"
              d="M20.33 3.67a1.45 1.45 0 0 0-1.47-.35L4.23 8.2A1.44 1.44 0 0 0 4 10.85l6.07 3l3 6.09a1.44 1.44 0 0 0 1.29.79h.1a1.43 1.43 0 0 0 1.26-1l4.95-14.59a1.41 1.41 0 0 0-.34-1.47M4.85 9.58l12.77-4.26l-7.09 7.09Zm9.58 9.57l-2.84-5.68l7.09-7.09Z"
            />
          </svg>
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, watch, nextTick } from "vue";

const messages = ref([
  { id: 1, sender: "bot", text: "Chào bạn! Tôi là AIOPS TECH, sẵn sàng hỗ trợ bạn.", time: "14:30" },
  { id: 2, sender: "user", text: "Chào AIOPS TECH! Bạn có thể làm gì?", time: "14:32" },
  {
    id: 3,
    sender: "bot",
    text: "Tôi có thể trả lời câu hỏi, kể chuyện, và thậm chí đưa ra vài lời khuyên hài hước!",
    time: "14:33",
  },
  {
    id: 4,
    sender: "user",
    text: "Đây là một tin nhắn rất dài để kiểm tra xem text có bị tràn ra ngoài ô không, hãy thử một chuỗi siêu dài không có khoảng cách như supercalifragilisticexpialidocioussupercalifragilisticexpialidocioussupercalifragilisticexpialidocious",
    time: "14:34",
  },
]);
const newMessage = ref("");
const isLoading = ref(false);

const sendMessage = () => {
  if (newMessage.value.trim()) {
    const time = new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
    messages.value.push({
      id: messages.value.length + 1,
      sender: "user",
      text: newMessage.value,
      time,
    });

    // Show loading spinner
    isLoading.value = true;

    // Simulate bot response
    setTimeout(() => {
      messages.value.push({
        id: messages.value.length + 1,
        sender: "bot",
        text: "Đang xử lý... Đây là câu trả lời giả từ AI!",
        time: new Date().toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" }),
      });
      // Hide loading spinner
      isLoading.value = false;
    }, 1000);

    newMessage.value = "";
  }
};

// Auto-scroll to bottom when new messages are added or loading state changes
watch(
  [messages, isLoading],
  async () => {
    await nextTick(); // Wait for DOM update
    const chatContainer = document.querySelector(".no-scrollbar");
    chatContainer.scrollTo({
      top: chatContainer.scrollHeight,
      behavior: "smooth",
    });
  },
  { deep: true },
);

onMounted(async () => {
  await nextTick(); // Wait for DOM to render
  const chatContainer = document.querySelector(".no-scrollbar");
  chatContainer.scrollTo({
    top: chatContainer.scrollHeight,
    behavior: "smooth",
  });
});
</script>

<style scoped>
/* Ẩn thanh cuộn nhưng vẫn cho phép cuộn */
.no-scrollbar {
  -ms-overflow-style: none; /* IE and Edge */
  scrollbar-width: none; /* Firefox */
  scroll-behavior: smooth; /* Smooth scrolling */
}
.no-scrollbar::-webkit-scrollbar {
  display: none; /* Chrome, Safari, Opera */
}

/* Đảm bảo text không tràn ra ngoài */
.break-words {
  overflow-wrap: break-word;
  word-wrap: break-word; /* For older browsers */
  word-break: break-word; /* Ensure long words break */
}
</style>
