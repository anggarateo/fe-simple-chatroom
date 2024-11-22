<template>
  <header class="sticky top-0 left-0 brightness-75 theme w-full z-50">
    <div class="flex items-center justify-between p-2 sm:p-4">
      <h1 class="text-xl font-bold">
        Simple Chatroom
      </h1>

      <div class="flex items-center gap-2">
        <button
          type="button"
          :class="{ 'backdrop-contrast-50 text-white rounded p-1': $colorMode.value === 'dark' }"
          @click="$colorMode.preference = 'dark'"
        >
          <IconDark class="w-8 h-8" />
        </button>
        <button
          type="button"
          :class="{ 'backdrop-contrast-50 text-white rounded p-1': $colorMode.value === 'light' }"
          @click="$colorMode.preference = 'light'"
        >
          <IconLight class="w-8 h-8" />
        </button>
        <button
          type="button"
          :class="{ 'backdrop-contrast-50 text-white rounded p-1': $colorMode.value === 'sepia' }"
          @click="$colorMode.preference = 'sepia'"
        >
          <IconSepia class="w-8 h-8" />
        </button>
      </div>
    </div>
  </header>
</template>
