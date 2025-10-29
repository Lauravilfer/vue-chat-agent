<script setup>
import { ref } from "vue";

const isOpen = ref(false);
const messages = ref([]);
const userMessage = ref("");

// Función para enviar mensajes al sistema n8n
const sendMessage = async () => {
  if (!userMessage.value.trim()) return;

  // Agrega el mensaje del usuario a la lista
  messages.value.push({ sender: "user", text: userMessage.value });

  try {
    // Envía el mensaje al webhook de n8n
    const response = await fetch("http://localhost:5678/webhook/students_platform", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ message: userMessage.value }),
    });

    const data = await response.json();

    // Extrae el mensaje del array y agrega la respuesta del sistema n8n a la lista
    const botReply = data[0]?.output || "No se recibió una respuesta válida.";
    messages.value.push({ sender: "bot", text: botReply });
  } catch (error) {
    console.error("Error al enviar el mensaje:", error);
    messages.value.push({ sender: "bot", text: "Hubo un error. Intenta nuevamente." });
  }

  userMessage.value = "";
};
</script>

<template>
  <div class="chat-container" :class="{ open: isOpen }">
    <div class="chat-header" @click="isOpen = !isOpen">
      <div class="chat-icon">💬</div>
    </div>
    <div class="chat-body" v-if="isOpen">
      <div class="messages">
        <div
          v-for="(message, index) in messages"
          :key="index"
          :class="['message', message.sender]"
        >
          {{ message.text }}
        </div>
      </div>
      <div class="chat-input">
        <input
          type="text"
          v-model="userMessage"
          @keyup.enter="sendMessage"
          placeholder="Escribe un mensaje..."
        />
        <button @click="sendMessage">Enviar</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.chat-container {
  position: fixed;
  bottom: 20px;
  right: 20px;
  width: 300px;
  max-height: 400px;
  display: flex;
  flex-direction: column;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  border-radius: 10px;
  overflow: hidden;
  background-color: white;
  z-index: 1000;
}

.chat-container.open .chat-body {
  display: block;
}

.chat-header {
  background-color: #0078ff;
  color: white;
  padding: 10px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.chat-icon {
  font-size: 24px;
}

.chat-body {
  display: none;
  flex: 1;
  flex-direction: column;
  padding: 10px;
}

.messages {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 10px;
}

.message {
  margin: 5px 0;
  padding: 8px;
  border-radius: 5px;
}

.message.user {
  background-color: #0078ff;
  color: white;
  align-self: flex-end;
}

.message.bot {
  background-color: #f1f1f1;
  color: black;
  align-self: flex-start;
}

.chat-input {
  display: flex;
  gap: 5px;
}

.chat-input input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.chat-input button {
  background-color: #0078ff;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 5px;
  cursor: pointer;
}
</style>