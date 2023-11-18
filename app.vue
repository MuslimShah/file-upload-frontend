<template>
  <div>
    <v-container class="">
      <v-row no-gutters justify="center" align="center" class="mt-6">
        <v-col sm="6" md="6" lg="6" class="text-center">
          <h3>Add Products</h3>
          <v-sheet width="300" class="mx-auto">
            <v-form @submit.prevent>
              <v-text-field
                v-model="productName"
                label="Product name"
              ></v-text-field>
              <v-text-field v-model="price" label="Price"></v-text-field>
              <v-file-input
                label="Choose Image"
                @change="onFileSelected"
              ></v-file-input>
              <v-btn
                type="submit"
                block
                class="mt-2 bg-primary"
                :loading="loading"
                @click="submitForm"
                >Submit</v-btn
              >
            </v-form>
          </v-sheet>
        </v-col>
      </v-row>
      <v-row justify="center" align="center" class="mt-6">
        <h3>Products</h3>
      </v-row>
      <v-row justify="center" align="center" class="mt-6">
        
        <v-col v-for="(data, index) in products" :key="index" 
        sm="3" md="4" lg="4"
        >
          <v-card
            class="mx-auto"
            max-width="200"
            height="200"
            :title="data.name"
            theme="dark"
          >
            <v-img :src="data.image"></v-img>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script setup>
import { onBeforeMount } from "vue";
import axios from "axios";
const products = ref([]);
const productName = ref();
const price = ref();
const image = ref();
let loading = ref(false);
let baseUrl = ref("http://localhost:3000");

onBeforeMount(async () => {
  const data = await axios.get(`${baseUrl.value}/api/v1/products`);
  products.value = data.data.products;
});
function onFileSelected(e) {
  const file = e.target.files[0];
  console.log(file);
  image.value = file;
}

async function submitForm() {
  try {
    loading.value = true;
    const formData = new FormData();
    formData.append("image", image.value);
    const response = await axios.post(
      `${baseUrl.value}/api/v1/products/uploads`,
      formData,
      {
        headers: {
          "Content-Type": "multipart/form-data",
        },
      }
    );
    const imageSrc = response.data.src;
    const productData = new FormData();
    productData.append("name", productName.value);
    productData.append("image", imageSrc);
    productData.append("price", price.value);
    const data = await axios.post(
      "http://localhost:3000/api/v1/products",
      productData
    );
    const getImages = await axios.get("http://localhost:3000/api/v1/products");
    products.value = getImages.data.products;
    loading.value = false;
  } catch (error) {
    console.log(error);
    loading.value = false;
  }
}
</script>
