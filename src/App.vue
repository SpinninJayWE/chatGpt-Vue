<template>
  <div>
    <div class="message-list">
      <div v-for="message in messages" :key="message.id" :class="message.role" v-html="markdown(message.content)">
      </div>
    </div>
    <div class="input-wrapper">
      <input v-model="inputText" @keydown.enter="sendMessage" placeholder="Type a message...">
      <button @click="sendMessage">Send</button>
    </div>
  </div>
</template>
<script setup lang="ts">
import { ref } from '@vue/reactivity';
const { marked, Prism } = (window as any)
const messages = ref<{ role: string, content: string }[]>([]);
const inputText = ref('');

async function sendMessage() {
  if (inputText.value !== '') {
    const userMessage = {
      role: 'user',
      content: inputText.value,
    };

    messages.value.push(userMessage);

    const response = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer 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',
      },
      body: JSON.stringify({
        model: 'gpt-3.5-turbo',
        messages: messages.value,
      }),
    });

    const data = await response.json();

    const aiMessage = data.choices[0].message;
    aiMessage.role = 'assistant';

    messages.value.push(aiMessage);

    inputText.value = '';
  }
}

function markdown(text: string) {
  return marked.marked(text, {
    highlight: function (code: any, lang: any) {
      if (lang && Prism.languages[lang]) {
        return Prism.highlight(code, Prism.languages[lang], lang);
      }
      return code;
    }
  });
}
</script>
<style scoped>
.message-list {
  width: 100%;
  height: 300px;
  overflow-y: scroll;
  border: 1px solid #ccc;
  padding: 10px;
}

.user {
  text-align: right;
  margin-bottom: 5px;
}

.assistant {
  text-align: left;
  margin-bottom: 5px;
}

.input-wrapper {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 10px;
  padding: 10px;
  background-color: #f8f8f8;
  border-top: 1px solid #ccc;
}

input {
  flex: 1;
  margin-right: 10px;
  padding: 5px;
  font-size: 16px;
}

button {
  padding: 5px 10px;
  font-size: 16px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
</style>
