<template>
  <div id="app">
    <section class="products">
      <div v-for="product in products" :key="product.id" class="products__product">
        <img :src="product.img" :alt="product.nome" class="products__product__img">
        <div class="products__product__info">
          <span class="products__product__info__price">{{ product.price }}</span>
          <h2 class="products__product__info__title">{{ product.name }}</h2>
        </div>
      </div>
    </section>
  </div>
</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      products: []
    };
  },
  methods: {
    fetchProducts() {
      fetch("/api/products.json")
        .then(response => response.json())
        .then(response => {
          this.products = response;
        });
    },
  },
  created() {
    this.fetchProducts();
  },
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Noto+Serif:wght@400;700&display=swap');

* {
  box-sizing: border-box;
}

body{
  margin: 0;
  padding: 0;
  background: linear-gradient(to right, #1a1a1a 30%, #ffffff 30%);
}

#app {
  font-family: Noto serif;
  padding: 0 80px;
  h1,
  h2, 
  ul,
  li {
    padding: 0;
    margin: 0;
  }
  ul {
    list-style: none;
  }
  .products {
    max-width: 900px;
    margin: 100px auto 0 auto;
    .products__product {
      display: flex;
      align-items: center;
      margin-bottom: 40px;
      background: #ffffff;
      box-shadow: 0 0 2rem rgba(0, 0, 0, .1);
      .products__product__img {
        max-width: 300px;
        margin-right: 40px;
      }
      .products__product__info {
        .products__product__info__title {
          font-size: 3rem;
          line-height: 1;
        }
        .products__product__info__price {
          color: rgba(0, 0, 0, .5);
        }
      }
    }
  }
}
</style>
