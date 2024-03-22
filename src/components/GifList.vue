<template>
  <div class="gif-list p-5 bg-cyan-950 max-w-screen-xl mx-auto">
    <!-- Título -->
    <h1
      class="title-container bg-cyan-900 py-4 font-sans font-bold text-white text-center left-10 text-4xl mb-5 rounded-md border border-cyan-800">
      GALERIA DE GIFS</h1>

    <!-- Lista dos gifs -->
    <div class="gif-items-container grid grid-cols-5 gap-4 p-5 max-h-screen overflow-y-auto justify-center"
      ref="gifContainer" @scroll="handleScroll">
      <div v-for="gif in gifs" :key="gif.id" :class="{ 'selected': selectedGifId === gif.id }"
        class="gif-item mb-5 relative shadow rounded-sm overflow-hidden flex justify-center items-center border border-gray-300"
        @click="selectGif(gif.id)">
        <img :src="gif.images.fixed_height.url" alt="gif" class="w-full h-auto" />
      </div>
    </div>

    <!-- Exibe o gif selecionado -->
    <div v-if="selectedGifId"
      class="overlay fixed top-0 left-0 w-full h-full bg-black bg-opacity-70 z-50 cursor-pointer flex justify-center items-center transition-opacity"
      :class="{ 'opacity-0': !selectedGifUrl, 'opacity-100': selectedGifUrl }">
      <button @click="selectGif(null)"
        class="absolute top-4 right-4 text-white bg-gray-900 rounded-md px-4 py-2 border border-white font-sans font-bold text-xl">
        Fechar
      </button>
      <img :src="selectedGifUrl" alt="gif" class="max-w-4xl max-h-4xl" @click="selectGif(null)">
    </div>

    <!-- Pesquisa -->
    <div
      class="search-bar max-w-screen-xl fixed bottom-0 right-12 left-12 bg-white p-3 font-sans font-bold text-xl rounded-sm">
      <div class="search-bar flex max-w-screen-xl mx-auto">
        <input v-model="searchTerm" type="text" placeholder="Pesquise aqui..."
          class="flex-1 border border-cyan-900 rounded px-5 py-2 w-full" />
        
        <button @click="searchGifs" class="ml-4 px-6 py-2 bg-cyan-900 text-white rounded">Pesquisar</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Gifs',
  data() {
    return {
      gifs: [], 
      errorMessage: '', 
      searchTerm: '', 
      selectedGifId: null, 
      selectedGifUrl: '', 
      offset: 0, 
    };
  },
  created() {
    
    this.fetchGifs(); 
  },
  methods: {
    async fetchGifs() {
      // Buscar GIFs da API
      try {
        let url = `https://api.giphy.com/v1/gifs/trending?api_key=tfPCpnK8YjPy15IA1X3wc4bvOdRb9M3U&limit=${this.limit}&offset=${this.offset}`;
        if (this.searchTerm) {
          url = `https://api.giphy.com/v1/gifs/search?api_key=tfPCpnK8YjPy15IA1X3wc4bvOdRb9M3U&q=${this.searchTerm}&limit=${this.limit}&offset=${this.offset}`;
        }
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error('Erro ao carregar GIFs');
        }
        const data = await response.json();
        this.gifs = this.gifs.concat(data.data);
        this.offset += this.limit;
      } catch (error) {
        this.errorMessage = error.message;
      }
    },
    searchGifs() {
      // Pesquisar GIFs 
      this.offset = 0;
      this.gifs = [];
      this.fetchGifs();
    },
    selectGif(id) {
      // Selecionar um GIF
      const gif = this.gifs.find(gif => gif.id === id);
      if (gif) {
        this.selectedGifId = gif.id;
        this.selectedGifUrl = gif.images.original.url; 
      } else {
        this.selectedGifId = null;
        this.selectedGifUrl = null;
      }
    },
    async handleScroll() {
      // Rolagem
      const container = this.$refs.gifContainer;
      if (container.scrollTop + container.clientHeight >= container.scrollHeight) {
        await this.fetchGifs();
      }
    }
  }
};
</script>

<style>
/* Estiliza o scroll */
::-webkit-scrollbar {
  width: 12px; 
}

::-webkit-scrollbar-track {
  background: #f1f1f1; 
}

::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 6px; 
}

::-webkit-scrollbar-thumb:hover {
  background: #555; 
}

/* Remove a barra de rolagem do navegador */
body {
  overflow: hidden;
}
</style>
