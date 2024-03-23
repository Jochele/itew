<template>
    <div class="Shopping">
      <h1></h1><br>
  
      <form @submit.prevent="addNewProduct">
        <label for="productName">Product Name: </label>
        <input type="text" id="productName" v-model="newProduct.name" required>
        
        <label for="productPrice"> Price: </label>
        <input type="number" id="productPrice" v-model.number="newProduct.price" min="0" step="0.01" required>
        
        <button type="submit">Add Product</button>
      </form>
      
      <div class="container"></div>
      <h2>Product List:</h2>
      <div class="product-list">
        <div v-for="product in products" :key="product.id" class="product">
          <div v-if="product.id !== editingProductId">
            <p>{{ product.name }} [Price: ₱ {{ product.price }}]</p>
            Qnty: <input type="number" v-model="product.quantityToAdd" min="1" step="1">
            <button @click="addToCart(product, product.quantityToAdd)">Add to Cart</button>
          </div>
  
          <div v-else class="edit-form">
            Product Name: <input type="text-edit" v-model="editedProduct.name"><br>
            Price: <input type="number" v-model.number="editedProduct.price" min="1" step="0.01">
            <button @click="saveEdit">Save</button>
            <button @click="cancelEdit">Cancel</button>
          </div>
        </div>
      </div>
  
      <h2>Cart:</h2>
      <div class="cart">
        <div v-for="(item, index) in cart" :key="index" class="cart-item">
          <p>{{ item.name }} [₱{{ item.price }}] - Quantity: {{ item.quantity }}</p>
          <input type="number" v-model="item.quantity" min="1" @change="updateQuantity(index, item.quantity)">
          <button style="background-color: orange;" @click="removeFromCart(index)">Remove</button>
        </div>
        <p class="total">Total: ₱{{ totalPrice }}</p>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
  return {
    products: [
      { id: 1, name: 'Kuromi Keychain', price: 90, quantityToAdd: 1 },
      { id: 2, name: 'Kuromi ID Lace', price: 50, quantityToAdd: 1 },
      { id: 3, name: 'Kuromi Plushie', price: 200, quantityToAdd: 1 }
    ],
    newProduct: {
      name: '',
      price: null
    },
    editingProductId: null,
    editedProduct: {
      id: null,
      name: '',
      price: null
    }
  };
  },
    watch: {
    products: {
      deep: true,
      handler(newProducts) {
        this.cart.forEach(cartItem => {
          const correspondingProduct = newProducts.find(product => product.id === cartItem.id);
          if (correspondingProduct) {
            cartItem.price = correspondingProduct.price;
          }
        });
      }
    }
  },
  
  props: ['cart', 'router'], 
    computed: {
      totalPrice() {
        return this.cart.reduce((total, item) => total + (item.price * item.quantity), 0);
      }
    },
    methods: {
      addToCart(product, quantity) {
    const isAuthenticated = localStorage.getItem('token'); 
    if (isAuthenticated) {
      let existingItem = this.cart.find(item => item.id === product.id);
      if (existingItem) {
        existingItem.quantity += quantity;
      } else {
        this.$emit('add-to-cart', { id: product.id, name: product.name, price: product.price, quantity });
      }
    } else {
        
      this.$emit('navigate-to-login');
    }
  },
      removeProduct(id) {
        const index = this.products.findIndex(product => product.id === id);
        if (index !== -1) {
          this.products.splice(index, 1);
        }
      },
      removeFromCart(index) {
        this.$emit('remove-from-cart', index);
      },
      updateQuantity(index, quantity) {
        if (quantity < 1) {
          this.$emit('remove-from-cart', index);
        }
      },
      addNewProduct() {
        if (this.newProduct.name && this.newProduct.price) {
          this.products.push({
            id: this.products.length + 1,
            name: this.newProduct.name,
            price: parseFloat(this.newProduct.price),
            quantityToAdd: 1
          });
          this.newProduct.name = '';
          this.newProduct.price = null;
        }
      }
    }
  }
  </script>

  <style>

</style>