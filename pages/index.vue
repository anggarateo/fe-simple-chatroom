<template>
  <section>
    <div class="space-y-2 top-results h-[calc(100vh-190px)] sm:h-[calc(100vh-150px)]">
      <InfiniteLoading
        class="flex items-center justify-center"
        target=".top-results"
        top
        @infinite="getMessages"
      >
        <template #spinner />
        <template #complete>
          <div />
        </template>
      </InfiniteLoading>
      
      <div
        v-for="message in messages"
        :key="message.id + message.user"
        class="result flex"
        :class="{ 'justify-end': message.user === userAgent }"
      >
        <div class="max-w-[calc(100vw-100px)] sm:max-w-[calc(100vw-200px)]">
          <h1
            class="font-medium"
            :class="{ 'text-end': message.user === userAgent }"
          >
            {{ uaParse(message.user) }}
          </h1>

          <p
            class="max-h-[calc(100vh-200px)] bg-transparent rounded-lg p-2 backdrop-contrast-75 whitespace-pre-wrap overflow-y-auto break-words"
            :class="message.user === userAgent ? 'rounded-tr-none rounded-bl-none' : 'rounded-tl-none rounded-br-none'"
          >
            {{ message.content }}
          </p>
    
          <h1 class="text-end">
            {{ dateFormat(message.created_at) }}
          </h1>
        </div>
      </div>
    </div>

    <form
      class="fixed bottom-0 left-0 theme w-full"
      @submit.prevent="onChat"
    >
      <div class="relative flex items-end justify-between gap-2 p-2 sm:p-4 backdrop-brightness-75">
        <NuxtEmojiPicker
          v-if="showEmotPicker"
          :theme="$colorMode.value === 'light' ? 'light' : 'dark'"
          display-recent
          class="absolute bottom-20"
          @select="onSelectEmoji"
        />

        <button
          type="button"
          @click="showEmotPicker = !showEmotPicker"
        >
          <IconEmoji class="w-8 h-8" />
        </button>

        <div class="w-full rounded-md theme contrast-75 p-2">
          <textarea
            ref="input"
            v-model="message"
            class="w-full focus:outline-none bg-transparent resize-none max-h-32"
            rows="1"
            @keyup="autoHeight, pointCursor()"
            @keydown="enterToSend"
            @keydown.esc="showEmotPicker = false"
            @mouseup="pointCursor()"
          />
        </div>
    
        <button
          type="submit"
          :disabled="loading || !message"
        >
          <IconSend class="w-8 h-8" />
        </button>
      </div>
    </form>
  </section>
</template>

<script lang="ts" setup>
import ActionCable from 'actioncable'
import { UAParser } from 'ua-parser-js'
import InfiniteLoading from "v3-infinite-loading";
import "v3-infinite-loading/lib/style.css";

interface Message {
  id: number,
  user: string,
  content: string,
  created_at: string,
  updated_at: string
}

const { public: { apiUrl, wsUrl } } = useRuntimeConfig()
const colorMode = useColorMode()

const messages = ref<Message[]>([])
const message = ref('')
const showEmotPicker = ref(false)
const input = ref()
const pagination = ref({
  page: 1,
  limit: 10
})
const userAgent = navigator.userAgent
const loading = ref(false)
const cursor = ref(0)

const cable = ActionCable.createConsumer(wsUrl)

const channel = cable.subscriptions.create(
  'MessagesChannel',
  {
    received(response: Message) {
      console.log('received', response)
      if (response.user !== userAgent) {
        messages.value.push(response)
        scrollToBottom()
      }
    },
  }
)

watch(() => colorMode.value, (val, oldVal) => {
  if (showEmotPicker.value && (val === 'light' || oldVal === 'light')) {
    showEmotPicker.value = false
  
    setTimeout(() => {
      showEmotPicker.value = true
    }, 10)
  }
})

onUnmounted(() => {
  channel.unsubscribe()
})

const getMessages = async ($state?: any) => {
  const {
    data: { value },
    error
  } = await useFetch(apiUrl, {
    params: pagination.value
  })
  
  if (value) {
    const res: any = value
    
    if (res.messages?.length < pagination.value.limit) $state?.complete()

    if (res.messages?.length) {
      messages.value.unshift(...res.messages?.reverse())
      $state?.loaded()
    }

    pagination.value.page++
  }

  if (error.value) {
    console.error('e', error.value)
    $state?.error()
  }
}

const onChat = async () => {
  loading.value = true

  const {
    data: { value },
    status,
    error
  } = await useFetch(apiUrl + 'messages', {
    method: 'POST',
    body: {
      content: message.value,
      user: userAgent
    }
  })

  if (value) {
    message.value = ''
    input.value.style.height = '1px'
    showEmotPicker.value = false
    scrollToBottom()

    const res: any = value
    messages.value.push(res.message)
  }

  if (status.value === 'pending') loading.value = true

  if (error.value) {
    console.error('e', error.value)
  }

  loading.value = false
}

const enterToSend = (e: KeyboardEvent) => {
  if (e.key === 'Enter' && !e.shiftKey) {
    onChat()
  }
}

const onSelectEmoji = (emoji: any) => {
  message.value = message.value.slice(0, cursor.value) + emoji.i + message.value.slice(cursor.value)
}

const autoHeight = () => {
  input.value.style.height = '1px'
  input.value.style.height = input.value.scrollHeight + 'px'
  console.log(input.value.style)
}

const dateFormat = (date?: string) => {
  if (!date) {
    return
  }

  const d = new Date(date)

  return d.toLocaleString('id-ID')
}

const uaParse = (ua?: string) => {
  if (!ua) return
  const parse = UAParser(ua)
  return `${parse.browser?.name || ''}${parse.browser?.major || ''}${parse.os?.name || ''}${parse.os?.version || ''}${parse.device?.type || ''}`
}

const pointCursor = () => (cursor.value = input.value.selectionStart)

const scrollToBottom = () => (
  setTimeout(() => {
    window.scrollTo({
      behavior: 'smooth',
      top: document.body.scrollHeight
    })
  }, 100)
)
</script>

<style>
.top-results {
  /* height: 74vh; */
  overflow-y: auto;
  padding: 16px;
}
</style>
