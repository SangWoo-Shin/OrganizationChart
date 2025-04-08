<template>
    <header class="bg-white shadow-md w-full px-6 py-4 flex items-center justify-between">
      <div class="flex items-center space-x-4">
        <img src="@/assets/logo.png" alt="Logo" class="w-10 h-10 rounded-full" />
        <h1 class="text-xl font-bold text-blue-700">Agentnoon</h1>
      </div>
      
      <nav class="flex items-center space-x-6">
        <a href="https://agentnoon.com" class="text-slate-600 hover:text-slate-900 font-medium">Home</a>
      </nav>
    </header>
  </template>
  
  <script>
  export default {
    name: 'AppHeader',
  }
  </script>