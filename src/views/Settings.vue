<script setup>
import Input from '@/components/Input.vue';
</script>

<template>
  <main class="flex flex-col w-screen h-screen bg-zinc-900">
    <div class="flex items-center flex-col w-full h-full p-4 justify-center items-center">
      <div class="w-64">
        <h1 class="text-center p-2 m-2 text-3xl text-blue-100">Settings</h1>
        <Input label="Token" type="password" />
        <Input label="Github username" placeholder="User1337" />
        <Input label="Repo name" placeholder="vue-site-blog" />
        <Input label="Path" placeholder="path/to/your/posts" />
        <Input label="Branch" placeholder="main" />
        <RouterLink class="text-center w-full text-2xl bg-green-500 hover:bg-green-400 text-white rounded hover:rounded-md p-2 w-full block" to="/"> Go back ↩</RouterLink>
      </div>
    </div>
  </main>
</template>

<style scoped></style> 