<template>
  <div class="chat-container">
      <header class="chat-header">
          <div class="logo-container"></div>
          <span class="logo-text">{{ appName }}</span>
      </header>
      <div class="messages">
          <div v-for="message in messages" :key="message.id"
               :class="['message', { 'mine': message.isMine, 'assistant': message.role === 'assistant' }]">
              <template v-if="message.role === 'assistant'">
                  <img src="../images/amd_logo.png" class="assistant-logo" />
                  <div class="message-content-container">
                      <div v-if="message.isLoading">
                          <Vue3Lottie :animationData="AnimationJSON" :height="16" :width="16" />
                      </div>
                      <div v-else class="message-content">
                          {{ message.text }}
                      </div>
                  </div>
              </template>
              <template v-else>
                  <div class="message-content">{{ message.text }}</div>
              </template>
          </div>
      </div>
      <div class="suggested-questions" v-if="!messageSent">
          <div v-for="question in suggestedQuestions" :key="question" @click="selectQuestion(question)" class="suggested-question">
              {{ question }}
          </div>
      </div>

      <div class="message-input">
          
          <button class="menu-toggle-button"
                  :class="{'menu-open': isMenuOpen}"
                  @click="toggleMenu">
              <i v-if="isMenuOpen" class="fas fa-times"></i> 
              <i v-else class="fas fa-plus"></i> 
          </button>

          <textarea ref="textarea" class="message-input-field" v-model="newMessage" rows="1" @keyup.enter="sendMessage" placeholder="Schreibe eine Nachricht..." @input="adjustTextareaHeight"></textarea>

          <button class="send-button" @click="sendMessage" :disabled="isSending || !newMessage.trim()">
              <i class="fas fa-arrow-up"></i>
          </button>
      </div>
      <div v-if="isMenuOpen" class="overlay">
          <div class="menu-container">
              <MenuOverlay v-if="isMenuOpen" @closeMenu="toggleMenu" />
          </div>
      </div>
      
  </div>
  
</template>

<script>
  import '../css/styles.css';
  import axios from 'axios';
  
  import { Vue3Lottie } from 'vue3-lottie'
  import AnimationJSON from './assets/Animation.json';   
  import MenuOverlay from './components/MenuOverlay.vue';

  axios.defaults.headers.common['X-Api-Key'] = import.meta.env.VITE_API_KEY;

  export default {
      components: {
          Vue3Lottie,
          MenuOverlay
      },
      data() {
          return {
              newMessage: '',
              messages: [],
              threadId: null,
              lastMessageId: null,
              isSending: false,
              isProcessing: false,
              messageSent: false,                
              suggestedQuestions: import.meta.env.VITE_SUGGESTED_QUESTIONS.split(';'),
              AnimationJSON,
              isMenuOpen: false,
              appName: import.meta.env.VITE_NAME
          };
      },
      mounted() {
          this.createThread();
          this.$nextTick(() => {
              const textarea = this.$refs.textarea; 
              textarea.style.height = "0px";
              textarea.style.height = textarea.scrollHeight + "px";
          });            
      },
      updated() {
          this.scrollToBottom();
      },
      methods: {
          adjustTextareaHeight(event) {
              const textarea = event.target;
              textarea.style.height = '0'; 
              textarea.style.height = '${textarea.scrollHeight}px'; 
              
          },
          scrollToBottom() {
              this.$nextTick(() => {
                  const container = this.$el.querySelector(".messages");
                  container.scrollTop = container.scrollHeight;
              });
          },
          async createThread() {
              try {
                  const response = await axios.post(import.meta.env.VITE_THREAD_URL, {}, {                        
                  });
                  this.threadId = response.data.id; 
                  //console.log("Erhaltene Thread-ID:", response.data.id);
                  
              } catch (error) {
                  console.error("Fehler beim Erstellen des Threads:", error);
                  // Error handling
              }
          },
          async sendMessage() {
              if (this.newMessage.trim() === '' || !this.threadId) {
                  return;
              }

              // Add user message to the chat
              this.messages.push({ text: this.newMessage, isMine: true, role: 'user' });
              // Add a temporary assistant message to indicate that the assistant is typing
              const tempMessageIndex = this.messages.push({
                  isLoading: true,
                  isMine: false,
                  role: 'assistant'
              }) - 1;
              this.newMessage = '';

              try {
                  this.isProcessing = true;
                  this.isSending = true;

                  const response = await axios.post(import.meta.env.VITE_MESSAGE_URL, {
                      ThreadId: this.threadId,
                      Message: this.newMessage,
                      AssistantId: import.meta.env.VITE_ASSISTANT_ID
                  });

                  // Remove the temporary assistant message
                  this.messages.splice(tempMessageIndex, 1);

                  // Process the assistant's response
                  const firstAssistantMessage = response.data.data.find(msg => msg.role === "assistant");
                  if (firstAssistantMessage) {
                      firstAssistantMessage.content.forEach(content => {
                          if (content.type === "text") {
                              this.messages.push({ text: content.text.value, isMine: false, role: 'assistant', createdAt: firstAssistantMessage.created_at });
                          }
                      });
                  }

                  this.messageSent = true;
                  this.scrollToBottom();
              } catch (error) {
                  console.error("Fehler beim Senden der Nachricht:", error);
                  // Remove the temporary assistant message
                  this.messages.splice(tempMessageIndex, 1);
              } finally {
                  this.isSending = false;
                  this.isProcessing = false;
              }
          },
          selectQuestion(question) {
              this.newMessage = question; // Set the selected question as the new message
              this.sendMessage(); // Send selected question
              this.messageSent = true; // After sending the question, hide the suggested questions
          },
          toggleMenu() {
              this.isMenuOpen = !this.isMenuOpen;
          }
      }
  };
</script>