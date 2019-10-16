<template>
  <div class="home">
    <h1>New Product</h1>
    <div>
      Name:
      <input type="text" v-model="newProductName" />
      {{ newProductName }}
      <br />
      Image Url:
      <input type="text" v-model="newProductImageUrl" />
      <br />
      Description:
      <input type="text" v-model="newProductDescription" />
      <br />
      Price:
      <input type="decimal" v-model="newProductPrice" />
    </div>
    <button v-on:click="createProduct()">Create</button>
    <div>
      Search by name:
      <input type="text" v-model="titleFilter" list="names" />
      <button v-on:click="setSortAttribute('name')" class="btn btn-primary">
        Sort by Name
        <span v-if="sortAttribute === 'name' && sortAscending === 1">^</span>
        <span v-if="sortAttribute === 'name' && sortAscending === -1">v</span>
      </button>
      <button v-on:click="setSortAttribute('price')" class="btn btn-primary">
        Sort by Price
        <span v-if="sortAttribute === 'price' && sortAscending === 1">^</span>
        <span v-if="sortAttribute === 'price' && sortAscending === -1">v</span>
      </button>
    </div>
    <datalist id="names">
      <option v-for="product in products">{{ product.name }}</option>
    </datalist>

    <h1>All Products</h1>
    {{ currentProduct }}

    <div v-for="product in orderBy(filterBy(products, titleFilter, 'name'), sortAttribute, sortAscending)">
      <h2>Name: {{ product.name }}</h2>
      <img v-bind:src="product.image_url" v-bind:alt="product.name" />
      <div>
        <button v-on:click="showProduct(product)">More Info</button>
      </div>
      <div v-if="product === currentProduct">
        <p>Description: {{ product.description }}</p>
        <p>Price: {{ product.price }}</p>
        <p>Tax: {{ product.tax }}</p>
        <p>Total: {{ product.total }}</p>

        <h4>Edit Product</h4>
        <div>
          Name:
          <input type="text" v-model="product.name" />
          <br />
          Image Url:
          <input type="text" v-model="product.image_url" />
          <br />
          Description:
          <input type="text" v-model="product.description" />
          <br />
          Price:
          <input type="decimal" v-model="product.price" />
          <br />
          <button v-on:click="updateProduct()">Update</button>
          <br />
          <button v-on:click="destroyProduct()">Destroy</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
img {
  width: 250px;
}
</style>

<script>
import axios from "axios";
import Vue2Filters from "vue2-filters";

export default {
  mixins: [Vue2Filters.mixin],
  data: function() {
    return {
      products: [],
      newProductName: "",
      newProductImageUrl: "",
      newProductDescription: "",
      newProductPrice: "",
      currentProduct: {},
      titleFilter: "",
      sortAttribute: "name",
      sortAscending: 1
    };
  },
  created: function() {
    axios.get("/api/products").then(response => {
      this.products = response.data;
      console.log(this.products);
    });
  },
  methods: {
    createProduct: function() {
      var params = {
        name: this.newProductName,
        image_url: this.newProductImageUrl,
        description: this.newProductDescription,
        price: this.newProductPrice
      };
      axios
        .post("/api/products", params)
        .then(response => {
          console.log("Success", response.data);
          this.products.push(response.data);
          this.newProductName = "";
          this.newProductImageUrl = "";
          this.newProductDescription = "";
          this.newProductPrice = "";
        })
        .catch(error => {
          console.log(error.response.data.errors);
        });
    },
    showProduct: function(product) {
      if (this.currentProduct === product) {
        this.currentProduct = {};
      } else {
        this.currentProduct = product;
      }
    },
    updateProduct: function(product) {
      var params = {
        name: product.name,
        image_url: product.image_url,
        description: product.description,
        price: product.price
      };
      axios
        .patch("/api/products/" + product.id, params)
        .then(response => {
          console.log("Success", response.data);
        })
        .catch(error => {
          console.log(error.response.data.errors);
        });
    },
    destroyProduct: function(product) {
      axios.delete("/api/products" + product.id).then(response => {
        console.log("Success", response.data);
        var index = this.products.indexOf(product);
        console.log(index);
        this.products.splice(index, 1);
      });
    },
    setSortAttribute: function(attribute) {
      this.sortAttribute = attribute;
      if (this.sortAttribute === attribute) {
        this.sortAscending = this.sortAscending * -1;
      } else {
        this.sortAscending = 1;
        this.sortAttribute = attribute;
      }
    }
  }
};
</script>
