<template>
  <div class="min-h-screen bg-gray-100">
    <header class="bg-white shadow">
      <div class="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
        <h1 class="text-3xl font-bold text-gray-900">Virtual Assistant</h1>
      </div>
    </header>
    <main>
      <div class="max-w-7xl mx-auto py-6 sm:px-6 lg:px-8">
        <div class="px-4 py-6 sm:px-0">
          <div class="bg-white rounded-lg shadow p-6">
            <!-- Chat container -->
            <div class="flex flex-col h-[600px]">
              <!-- Chat messages -->
              <div class="flex-1 overflow-y-auto mb-4 p-4" ref="chatContainer">
                <div v-for="(message, index) in messages" :key="index" class="mb-4">
                  <div :class="[
                    'max-w-[80%] rounded-lg p-3',
                    message.isUser ? 'ml-auto bg-blue-500 text-white' : 'bg-gray-100 text-gray-800'
                  ]">
                    <p>{{ message.message }}</p>
                    <!-- Additional data display -->
                    <div v-if="message.additionalData" class="mt-2 text-sm">
                      <div v-if="message.additionalData.hours">
                        <div v-for="(hours, day) in message.additionalData.hours" :key="day">
                          <strong>{{ day }}:</strong> {{ hours }}
                        </div>
                      </div>
                      <div v-if="message.additionalData.location">
                        <p><strong>Address:</strong> {{ message.additionalData.location.address }}</p>
                        <p><strong>Directions:</strong> {{ message.additionalData.location.directions }}</p>
                      </div>
                      <div v-if="message.additionalData.menu">
                        <div v-for="(item, type) in message.additionalData.menu" :key="type">
                          <p><strong>{{ type }}:</strong> {{ item.name }} - ${{ item.price }}</p>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              <!-- Input area -->
              <div class="border-t pt-4">
                <form @submit.prevent="sendMessage" class="flex gap-4">
                  <input
                    v-model="newMessage"
                    type="text"
                    placeholder="Type your message here..."
                    class="flex-1 rounded-lg border border-gray-300 px-4 py-2 focus:outline-none focus:border-blue-500"
                    :disabled="loading"
                  />
                  <button
                    type="submit"
                    class="bg-blue-500 text-white px-6 py-2 rounded-lg hover:bg-blue-600 focus:outline-none disabled:opacity-50"
                    :disabled="loading || !newMessage.trim()"
                  >
                    {{ loading ? 'Sending...' : 'Send' }}
                  </button>
                </form>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'App',
  data() {
    return {
      messages: [
        {
          message: this.getWelcomeMessage(),
          isUser: false
        }
      ],
      newMessage: '',
      loading: false
    }
  },
  methods: {
    getWelcomeMessage() {
      // Detect browser language
      const browserLang = navigator.language || navigator.userLanguage;
      return  "¡Hola! Soy el asistente virtual. ¿En qué puedo ayudarte?"
    },
    async sendMessage() {
      if (!this.newMessage.trim()) return;
      
      // Add user message
      this.messages.push({
        message: this.newMessage,
        isUser: true
      });
      
      try {
        const response = await fetch('http://localhost:4000/chat', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({ message: this.newMessage })
        });
        
        if (!response.ok) throw new Error('Network response was not ok');
        
        const data = await response.json();
        
        // Add bot response using the response from the server
        this.messages.push({
          message: data.response,
        });
        
      } catch (error) {
        // Get browser language
        const userLang = navigator.language || navigator.userLanguage;
        const errorMessage = userLang.startsWith('es') 
          ? "Lo siento, ha ocurrido un error. Por favor, intenta de nuevo más tarde."
          : "I'm sorry, an error has occurred. Please try again later.";
        
        this.messages.push({
          message: errorMessage,
          isUser: false,
          error: true
        });
      }
      
      this.newMessage = '';
      this.$nextTick(() => {
        this.scrollToBottom();
      });
    },
    scrollToBottom() {
      const container = this.$refs.chatContainer;
      container.scrollTop = container.scrollHeight;
    }
  },
  mounted() {
    this.scrollToBottom();
  }
}
</script>

<style>
@import 'tailwindcss/base';
@import 'tailwindcss/components';
@import 'tailwindcss/utilities';
</style> 