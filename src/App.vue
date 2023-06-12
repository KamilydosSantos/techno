<template>
  <div id="app">
    {{ produtos }}
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      produtos: []
    };
  },
  methods: {
    fetchProdutos() {
      fetch("/api/produtos.json")
        .then(response => response.json())
        .then(response => {
          this.produtos = response;
        });
    },
  },
  created() {
    this.fetchProdutos();
  },
}
</script>

<style lang="scss">
#app {

}
</style>
