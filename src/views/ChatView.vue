<script setup>
import { onMounted, onUnmounted, ref } from 'vue'

const message = ref('')
const pendingMessage = ref('')
const messages = ref([])
const socket = ref(null)
const chatDisabled = ref(false)

async function sendMessage() {
  if (!socket.value) {
    return
  }

  // send message to the server
  console.log('send message:', message.value)
  socket.value.send(JSON.stringify({ type: 'human', message: message.value }))

  messages.value.push({
    type: 'sent',
    text: message.value,
  })
  message.value = ''
}

function handleKeyDown(event) {
  if (event.key === 'Enter' && !event.shiftKey) {
    event.preventDefault()
    sendMessage()
  }
}

onMounted(() => {
  socket.value = new WebSocket('ws://localhost:8000/ws/v1/chat/?token=test_token&user_id=1')
  socket.value.onmessage = (event) => {
    try {
      console.log('socket onmessage')
      if (!event.data) {
        console.log('no data')
        return
      }

      const data = JSON.parse(event.data)
      if (data.pending) {
        pendingMessage.value = data.message
      } else {
        chatDisabled.value = false
        messages.value.push({
          type: 'received',
          text: pendingMessage.value,
        })
        pendingMessage.value = ''
        return
      }
    } catch (e) {
      console.error(e)
      chatDisabled.value = false
    }
  }
})

onUnmounted(() => {
  if (socket.value) {
    // close the connection when the component is unmounted
    socket.value.close()
  }
})
</script>

<template>
  <main>
    <div>
      socket test demo
      <div>
        <ul>
          <li v-for="msg in messages" :key="msg.text">
            <span v-if="msg.type === 'sent'">You:</span>
            <span v-else>Bot:</span>
            {{ msg.text }}
          </li>
          <li v-if="pendingMessage">{{ pendingMessage }}</li>
        </ul>
      </div>
      <form @submit.prevent="sendMessage">
        <textarea
          id="chat-input"
          v-model="message"
          rows="3"
          cols="100"
          @keydown="handleKeyDown"
          :disabled="chatDisabled"
        ></textarea>
        <button type="submit" :disabled="chatDisabled">Send</button>
      </form>
    </div>
  </main>
</template>
