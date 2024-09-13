<template>
  <div class="flex flex-col w-full max-w-xl bg-gray-100 shadow-lg rounded-lg overflow-hidden">
    <div class="flex flex-col flex-grow p-4 overflow-auto space-y-4">
      <!-- Mostrar mensajes del chat -->
      <div
        class="flex space-x-3"
        v-for="(message, index) in messages"
        :key="index"
        :class="{ 'ml-auto justify-end': message.sentByUser }"
      >
        <!-- Imagen de perfil -->
        <div v-if="!message.sentByUser" class="flex-shrink-0 h-12 w-12 rounded-full bg-gray-400">
          <img :src="botProfilePic" alt="bot profile" class="h-12 w-12 rounded-full" />
        </div>
        <div v-if="message.sentByUser" class="flex-shrink-0 h-12 w-12 rounded-full bg-gray-400">
          <img :src="userProfilePic" alt="user profile" class="h-12 w-12 rounded-full" />
        </div>

        <div class="flex flex-col">
          <!-- Texto del mensaje -->
          <div
            :class="{
              'bg-blue-500 text-white p-4 rounded-lg': message.sentByUser,
              'bg-white text-gray-800 p-4 rounded-lg border border-gray-300': !message.sentByUser
            }"
          >
            <p class="text-sm">{{ message.text }}</p>
            <!-- Mostrar imagen si la respuesta de la API tiene una -->
            <img
              v-if="message.image"
              :src="message.image"
              alt="response image"
              class="mt-2 rounded-lg shadow-md"
            />
          </div>
          <!-- Hora del mensaje -->
          <span class="text-xs text-gray-600 mt-1">{{ getRelativeTime(message.time) }}</span>
        </div>
      </div>
    </div>

    <div class="bg-white p-4 border-t border-gray-300">
      <!-- Input para enviar mensaje -->
      <input
        v-model="newMessage"
        @keydown.enter="sendMessage"
        class="w-full h-12 rounded-lg px-4 border border-gray-300 focus:outline-none focus:ring-2 focus:ring-blue-500"
        type="text"
        placeholder="Escribe tu mensaje…"
      />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      newMessage: '',
      userProfilePic: 'https://cdn-icons-png.flaticon.com/512/3237/3237472.png',
      botProfilePic:
        'https://cdn.prod.website-files.com/6411daab15c8848a5e4e0153/6476e947d3fd3c906c9d4da6_4712109.png',
      messages: [{ text: 'Hola, ¿como puedo ayudarte?', time: new Date(), sentByUser: false }]
    }
  },
  methods: {
    getCurrentTime() {
      return new Date()
    },
    getRelativeTime(time) {
      const now = new Date()
      const diffMs = now - new Date(time)
      const diffMins = Math.floor(diffMs / (1000 * 60))

      if (diffMins < 1) {
        return 'Hace un momento'
      } else if (diffMins === 1) {
        return 'Hace 1 minuto'
      } else if (diffMins < 60) {
        return `Hace ${diffMins} minutos`
      } else {
        const hours = Math.floor(diffMins / 60)
        return hours === 1 ? 'Hace 1 hora' : `Hace ${hours} horas`
      }
    },
    async sendMessage() {
      if (this.newMessage.trim() !== '') {
        this.messages.push({
          text: this.newMessage,
          time: this.getCurrentTime(),
          sentByUser: true
        })

        const userMessage = this.newMessage
        this.newMessage = ''

        try {
          const response = await fetch('https://yesno.wtf/api')
          const data = await response.json()

          this.messages.push({
            text: data.answer,
            image: data.image,
            time: this.getCurrentTime(),
            sentByUser: false
          })
        } catch (error) {
          console.error('Error fetching from API:', error)
          this.messages.push({
            text: 'Lo siento, hubo un error al procesar tu solicitud.',
            time: this.getCurrentTime(),
            sentByUser: false
          })
        }
      }
    }
  }
}
</script>

<style scoped></style>
