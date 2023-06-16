<template>
  <div id="app">
    <header class="header">
      <img src="./assets/techno.svg" alt="Techno" class="header__logo">
      <div class="header__shopping-cart" @click="activeShoppingCart = true">
        {{ totalShopping | priceFormatting }} | {{ shoppingCart.length }}
      </div>
    </header>
    <section class="products">
      <div class="products__product" @click="openModal(product.id)" v-for="product in products" :key="product.id">
        <img class="products__img" :src="product.img" :alt="product.nome">
        <div class="products__info">
          <span class="products__price">{{ product.price | priceFormatting }}</span>
          <h2 class="products__title">{{ product.name }}</h2>
        </div>
      </div>
    </section>
    <section class="modal" @click="closeModal" v-if="product">
      <div class="modal__container">
        <div class="modal__img">
          <img :src="product.img" :alt="product.name">
        </div>
        <div class="modal__info">
          <button class="btn-close" @click="product = false">X</button>
          <span class="modal__price">{{ product.price | priceFormatting}}</span>
          <h2 class="modal__title">{{ product.name }}</h2>
          <p class="modal__description">{{ product.description }}</p>
          <button class="btn" v-if="product.stock > 0" @click="addItem">Adicionar Item</button>
          <button class="btn sold-off" v-else disabled>Produto Esgotado</button>
        </div>
        <div class="modal__reviews">
          <h2 class="modal__reviews__title">Avaliações</h2>
          <ul class="modal__reviews__list">
            <li class="modal__reviews__review" v-for="review in product.reviews">
              <p class="modal__reviews__description">{{ review.description }}</p>
              <p class="modal__reviews__name">{{ review.name }} | {{ review.star }} estrelas</p>
            </li>
          </ul>
        </div>
      </div>
    </section>
    <section class="shopping-modal" :class="{active: activeShoppingCart}" @click="closeShoppingCart">
      <div class="shopping-modal__container">
        <button class="btn-close" @click="activeShoppingCart = false">X</button>
        <h2 class="shopping-modal__title">Carrinho</h2>
        <div class="shopping-modal__content" v-if="shoppingCart.length > 0">
          <ul class="shopping-modal__list">
            <li class="shopping-modal__item" v-for="(item, index) in shoppingCart">
              <p class="shopping-modal__name">{{ item.name }}</p>
              <p class="shopping-modal__price">{{ item.price | priceFormatting}}</p>
              <button class="shopping-modal__btn" @click="removeItem(index)">X</button>
            </li>
          </ul>
          <p class="shopping-modal__total">{{ totalShopping | priceFormatting }}</p>
          <button class="btn finalize">Finalizar Compra</button>
        </div>
        <p class="shopping-modal__message" v-else>O carrinho está vazio.</p>
      </div>
    </section>
    <div class="alert" :class="{active: activeAlert}">
      <p class="alert__message">
        {{ alertMessage }}
      </p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      products: [],
      product: false,
      shoppingCart: [],
      alertMessage: "Item adicionado",
      activeShoppingCart: false,
      activeAlert: false,
    };
  },
  filters: {
    priceFormatting(value) {
      return value.toLocaleString("pt-BR", {style: "currency", currency: "BRL"});
    }
  },
  computed: {
    totalShopping() {
      let total = 0;
      if (this.shoppingCart.length) {
        this.shoppingCart.forEach( item => {
          total += item.price;
        })
      }
      return total;
    }
  },
  methods: {
    fetchProducts() {
      fetch("/api/products.json")
        .then(response => response.json())
        .then(response => {
          this.products = response;
        });
    },
    fetchProduct(id){
      fetch(`./api/products/${id}/data.json`)
      .then(response => response.json())
      .then(response => {
        this.product = response;
      })
    },
    openModal(id) {
      this.fetchProduct(id);
      window.scrollTo({
        top: 0,
        behavior: "smooth"
      })
    },
    closeModal({target, currentTarget}) {
      if (target === currentTarget) {
        this.product = false;
      }
    },
    addItem() {
      this.product.stock--;
      const {id, name, price} = this.product;
      this.shoppingCart.push({id, name, price});
      this.alert(`${name} adicionado ao carrinho.`);
    },
    removeItem(index) {
      this.shoppingCart.splice(index, 1);
    },
    checkLocalStorage() {
      if (window.localStorage.shoppingCart) {
        this.shoppingCart = JSON.parse(window.localStorage.shoppingCart);
      }
    },
    closeShoppingCart({target, currentTarget}) {
      if (target === currentTarget) {
        this.activeShoppingCart = false;
      }
    },
    compareStock() {
      const items = this.shoppingCart.filter(({id}) => id === this.product.id);
      this.product.stock -= items.length;
    },
    alert(message) {
      this.alertMessage = message;
      this.activeAlert = true;
      setTimeout(() => {
        this.activeAlert = false;
      }, 1500);
    },
    router() {
      const hash = document.location.hash;
      if(hash) {
        this.fetchProduct(hash.replace("#", ""));
      }
    }
  },
  watch: {
    shoppingCart() {
      window.localStorage.shoppingCart = JSON.stringify(this.shoppingCart);
    },
    product() {
      document.title = this.product.name || "Techno";
      const hash = this.product.id || "";
      history.pushState(null, null, `#${hash}`);
      if(this.product) {
        this.compareStock();
      }
    }
  },
  created() {
    this.fetchProducts();
    this.checkLocalStorage();
    this.router();
  },
}
</script>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Noto+Serif:wght@400;700&display=swap');

@keyframes fadeIn {
  from {
    transform: translate3d(50px, 0, 0);
  }
  to {
    transform: translate3d(0px, 0, 0);
  }
}

@keyframes fadeInDown {
  from {
    transform: translate3d(0, -30px, 0);
    opacity: 0;
  }
  to {
    transform: translate3d(0, 0px, 0);
    opacity: 1;
  }
}

$breakpoint: 900px;

@mixin responsive {
  @media (max-width: $breakpoint) {
    @content;
  }
}

* {
  box-sizing: border-box;
}

body{
  margin: 0;
  padding: 0;
  background: linear-gradient(to right, #1a1a1a 30%, #ffffff 30%);
}

#app {
  @include responsive {
    padding: 0 10px;
  }
  font-family: "Noto Serif";
  padding: 0 80px;
  h1,
  h2, 
  p,
  ul,
  li {
    padding: 0;
    margin: 0;
  }
  ul {
    list-style: none;
  }
  .btn {
    margin-top: 80px;
    background: #000000;
    cursor: pointer;
    color: #ffffff;
    border: none;
    font-size: 1rem;
    padding: 10px 25px;
    font-family: "Noto Serif";
    &:active {
      background: #808080;
    }
    &.sold-off {
      background: #808080;
    }
    &.finalize {
      @include responsive {
        margin-top: 20px;
      }
      display: block;
      margin-left: auto;
      margin-top: 0;
    }
  }
  .btn-close {
    border-radius: 50%;
    border: 2px solid #000000;
    width: 40px;
    height: 40px;
    position: absolute;
    top: -10px;
    right: -10px;
    font-size: 1rem;
    background: #ffffff;
    box-shadow: 0 3px 4px rgba(0, 0, 0, .1), 0 4px 10px rgba(0, 0, 0, .2);
    cursor: pointer;
  }
  .header {
    display: flex;
    justify-content: space-between;
    max-width: 900px;
    padding: 20px 0;
    margin: 0 auto;
    .header__logo {
      width: 80px;
    }
    .header__shopping-cart {
      display: flex;
      align-items: center;
      cursor: pointer;
      &::after {
        content: "";
        display: inline-block;
        background: url("./assets/shopping.svg");
        width: 25px;
        height: 25px;
        margin-left: 10px;
      }
    }
  }
  .products {
    @include responsive {
      margin-top: 40px;
    }
    max-width: 900px;
    margin: 100px auto 0 auto;
    .products__product {
      @include responsive {
        flex-direction: column;
        align-items: flex-start;
        max-width: 300px;
        margin: 30px auto;
      }
      display: flex;
      align-items: center;
      margin-bottom: 40px;
      background: #ffffff;
      box-shadow: 0 0 2rem rgba(0, 0, 0, .1);
      cursor: pointer;
      .products__img {
        @include responsive {
          max-width: 100%;
        }
        max-width: 300px;
        margin-right: 40px;
      }
      .products__info {
        @include responsive {
          padding: 20px;
        }
        .products__title {
          @include responsive {
            font-size: 1.5rem;
          }
          font-size: 3rem;
          line-height: 1;
        }
        .products__price {
          color: rgba(0, 0, 0, .5);
        }
      }
    }
  }
  .modal {
    @include responsive {
      padding: 10px;
    }
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    padding: 80px;
    display: flex;
    flex-direction: column;
    align-items: center;
    &::before {
      content: "";
      position: fixed;
      top: 0;
      left: 0;
      width: 100%;
      height: 100vh;
      background: rgba(0, 0, 0, .5);
    }
    .modal__container {
      @include responsive {
        grid-gap: 20px;
        background: #ffffff;
        padding: 10px 0;
      }
      position: relative;
      background: linear-gradient(to right, transparent 250px, #ffffff 250px);
      z-index: 1;
      display: grid;
      align-items: end;
      grid-gap: 50px;
      padding: 50px 50px 50px 0;
      animation: fadeIn .3s forwards;
      .modal__img {
        @include responsive {
          grid-row: 2;
        }
        grid-column: 1;
        box-shadow: 0 3px 4px rgba(0, 0, 0, .1), 0 4px 10px rgba(0, 0, 0, .2);
        & img {
          @include responsive {
            width: 100%;
            max-width: initial;
          }
          max-width: 300px;
          display: block;
        }
      }
      .modal__info {
        @include responsive {
          grid-column: 1;
          padding: 10px;
        }
        max-width: 600px;
        grid-column: 2;
        .modal__title {
          font-size: 3rem;
        }
      }
      .modal__reviews {
        @include responsive {
          grid-column: 1;
          padding: 10px;
        }
        grid-column: 2;
        .modal__reviews__title {
          font-size: 1.75rem;
        }
        .modal__reviews__review {
          border-bottom: 1px solid rgba(0, 0, 0, .1);
          padding-bottom: 20px;
          .modal__reviews__description {
            color: rgba(0, 0, 0, .7);
            margin: 20px 0 5px 0;
          }
          .modal__reviews__name {
            font-weight: bold;
          }
        }
      }
    }
  }
  .shopping-modal {
    @include responsive {
      padding: 10px;
    }
    position: absolute;
    display: flex;
    flex-direction: column;
    top: 0px;
    left: 0px;
    width: 100%;
    padding: 20px;
    display: none;
    &::before {
      content: "";
      position: fixed;
      top: 0px;
      left: 0px;
      width: 100%;
      height: 100vh;
      background: rgba(0, 0, 0, .5);
    }
    &.active {
      display: block;
    }
    .shopping-modal__container {
      position: relative;
      margin: 80px auto;
      background: #ffffff;
      padding: 40px;
      max-width: 800px;
      animation: fadeInDown .3s forwards;
      .shopping-modal__title {
        margin-bottom: 10px;
        border-bottom: 2px solid #000000;
        padding-bottom: 20px;
      }
      .shopping-modal__item {
        display: grid;
        grid-template-columns: 1fr 1fr 50px;
        border-bottom: 1px solid rgba(0, 0, 0, .1);
        padding: 10px 0;
        .shopping-modal__price {
          text-align: right;
        }
        .shopping-modal__btn {
          border: none;
          font-size: 1rem;
          cursor: pointer;
          background: #ffffff;
        }
      }
      .shopping-modal__total {
        text-align: right;
        padding: 10px 50px 10px 0;
        margin-bottom: 20px;
        border-bottom: 2px solid #000000;
      }
    }
  }
  .alert {
    position: absolute;
    top: 20px;
    left: 0px;
    z-index: 10;
    width: 100%;
    text-align: center;
    display: none;
    &.active {
      display: block;
      animation: fadeInDown .3s forwards;
    }
    .alert__message {
      background: #ffffff;
      display: inline-block;
      padding: 20px 40px;
      border: 2px solid #000000;
      box-shadow: 0 3px 4px rgba(0, 0, 0, .1), 0 4px 10px rgba(0, 0, 0, .2);
    }
  }
}
</style>
