<template>
  <div class="chat-container">
    <!-- 검색창 -->
    <div class="search-container">
      <input
        type="text"
        v-model="inputMessage"
        placeholder="Ask me about movies..."
        @keydown.enter="getChatbot"
        class="search-input"
      />
      <button @click="getChatbot" class="search-btn">
        <i class="fas fa-search"></i>
      </button>
    </div>

    <!-- 결과 메시지 -->
    <div class="results-container">
      <div v-for="(message, index) in messages" :key="index" class="result">
        <ChatComponent
          v-if="message.role === 'assistant'"
          :content="message.content"
          class="assistant-message"
        />
        <span v-if="message.role === 'user'" class="user-message">
          {{ message.content }}
        </span>
      </div>
    </div>
        <!-- 로딩 중 표시 -->
        <div v-if="loading" class="loading-container">
          <img src="/src/assets/m2.gif" alt="loading-image" class="loadingimg">
          <div>
            <span>Loading. . .</span>
          </div>
    </div>

    <!-- 에러 메시지 -->
    <div v-if="error" class="error-message">
      {{ error }}
    </div>
  </div>
</template>



<script setup>
import { ref } from "vue";
import axios from "axios";
import ChatComponent from "@/components/ChatComponent.vue";

const messages = ref([
  { role: "system", content: "영화 추천" },
  { role: "assistant", content: "영화를 추천해드릴게요!" },
]);

const inputMessage = ref("");
const loading = ref(false);
const error = ref(""); // 에러 상태 추가
const AI_API_KEY = import.meta.env.VITE_OPENAI_API_KEY;

const getChatbot = async () => {
  if (inputMessage.value.trim() === "") return;

  messages.value = [];

  loading.value = true;
  error.value = "";

  const userMessage = { role: "user", content: inputMessage.value };
  messages.value.push(userMessage);
  inputMessage.value = "";

  try {
    const response = await axios.post(
      "https://api.openai.com/v1/chat/completions",
      {
        model: "gpt-4",
        messages: [
          {
            role: "system",
            content: `너는 영화 추천 및 관련 정보를 제공하는 전문가야, 답변을 줄 때 영화 제목을 먼저 주고 그뒤에 은/는을 붙인다음에 추천한 이유 한줄로 설명해줘.
            그리고 내용들은 한국에서 상영하는 영화이면서, 인기순으로 나열했을때 가장 인기있는 항목이야.`,
          },
          ...messages.value,
        ],
      },
      {
        headers: {
          Authorization: `Bearer ${AI_API_KEY}`,
          "Content-Type": "application/json",
        },
      }
    );
    const botMessage = response.data.choices[0].message.content;
    messages.value.push({ role: "assistant", content: botMessage });
  } catch (err) {
    error.value = "Failed to fetch data. Please try again later.";
    console.error(err);
  } finally {
    loading.value = false;
  }
};
</script>



<style scoped>
.loadingimg {
  width: 100px;
  height: 150px;
}
.search-container {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: white;
  padding: 10px 20px;
  border-radius: 25px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  margin-top: 20px;
}

.search-input {
  flex: 1;
  border: none;
  outline: none;
  font-size: 16px;
  color: #333;
}

.search-input::placeholder {
  color: #aaa;
}

.search-btn {
  background-color: #000000;
  border: none;
  border-radius: 50%;
  color: white;
  padding: 10px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.search-btn:hover {
  background-color: #e55555;
}

.loading-container {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 20px 0;
}

.spinner {
  width: 30px;
  height: 30px;
  border: 4px solid rgba(0, 0, 0, 0.2);
  border-top: 4px solid #ff6b6b;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.error-message {
  color: red;
  text-align: center;
  margin: 10px 0;
}

.results-container {
  margin-top: 20px;
}

.result {
  margin-bottom: 15px;
}

.user-message {
  display: inline-block;
  background-color: #f1f1f1;
  color: #333;
  padding: 10px;
  border-radius: 10px;
  margin-left: auto;
  max-width: 80%;
  word-wrap: break-word;
}

.assistant-message {
  display: inline-block;
  color: white;
  padding: 10px;
  border-radius: 10px;
  margin-right: auto;
  max-width: 80%;
  word-wrap: break-word;
}

</style>
