<template>
  <div>
    <Header />
    <main>
      <slot />
    </main>
  </div>
</template>

<script lang="ts" setup>
</script>

<style>
body, .theme {
  background-color: #fff;
  color: rgba(0,0,0,0.8);
}
.dark-mode body, .dark-mode .theme {
  background-color: #091a28;
  color: #ebf4f1;
}
.sepia-mode body, .sepia-mode .theme {
  background-color: #f1e7d0;
  color: #433422;
}

*::-webkit-scrollbar {
  height: 8px;
  width: 8px;               /* width of the entire scrollbar */
  background: whitesmoke;        /* color of the tracking area */
}
*::-webkit-scrollbar-track {
  border-radius: 2px;       /* roundness of the scroll thumb */
  background: whitesmoke;        /* color of the tracking area */
  margin: 2px 0;
}
*::-webkit-scrollbar-thumb {
  background-color: gray;    /* color of the scroll thumb */
  border-radius: 20px;       /* roundness of the scroll thumb */
}
</style>
