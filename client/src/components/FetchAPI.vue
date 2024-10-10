<template>
  <div>
    <h1>List Product</h1>
    <form @submit.prevent="handleSubmit">
      <div>
        <label for="">Name</label>
        <input v-model="productInfo.name" type="text" />
      </div>
      <div>
        <label for="">Price</label>
        <input v-model="productInfo.price" type="number" />
      </div>
      <div>
        <label for="">Quantity</label>
        <input v-model="productInfo.quantity" type="number" />
      </div>
      <button type="submit">Add</button>
    </form>
    <table border="1">
      <thead>
        <tr>
          <th>Id</th>
          <th>Name</th>
          <th>Price</th>
          <th>Quantity</th>
          <th>Option</th>
        </tr>
      </thead>
      <tbody v-for="product in products" :key="product.id">
        <tr>
          <td>{{ product.id }}</td>
          <td>{{ product.name }}</td>
          <td>{{ product.price }}</td>
          <td>{{ product.quantity }}</td>
          <td>
            <button @click="handleEdit(product.id)">Edit</button>
            <button @click="handleDelete(product.id)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>

    <div v-if="isShowLoading">Loading....</div>
  </div>
</template>

<script setup>
import { onMounted, reactive, ref } from "vue";

const products = ref([]);

const isShowLoading = ref(false);
const typeForm = ref("add");
const baseId = ref(null);

const loadData = async () => {
  // fetch("http://localhost:8080/products")
  //   .then((respons) => respons.json())
  //   .then((data) => console.log(data))
  //   .catch((error) => console.log(error));

  try {
    // Mở loading
    isShowLoading.value = true;

    const response = await fetch("http://localhost:8080/products");

    const data = await response.json();

    products.value = data;
  } catch (error) {
    return error;
  } finally {
    // Tắt loading
    isShowLoading.value = false;
  }
};

onMounted(async () => {
  loadData();
});

const productInfo = reactive({
  name: "",
  price: 0,
  quantity: 0,
});

const handleSubmit = async () => {
  // Cập nhật dữ liệu
  if (baseId) {
    await fetch(`http://localhost:8080/products/${baseId.value}`, {
      method: "PUT",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(productInfo),
    });

    loadData();
  } else {
    // Thêm mới dữ liệu
    await fetch("http://localhost:8080/products", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify(productInfo),
    });

    loadData();
  }

  // B1: Validate dữ liệu
  // B2: Gọi API thêm mới dữ liệu

  // B3: Reset Form
  productInfo.name = "";
  productInfo.price = "";
  productInfo.quantity = "";
};

const handleDelete = async (id) => {
  const response = await fetch(`http://localhost:8080/products/${id}`, {
    method: "DELETE",
  });

  if (response.status === 200) {
    // Render lại dữ liệu
    loadData();
  }
};

const handleEdit = async (id) => {
  baseId.value = id;

  // Tìm kiếm sản phẩm theo id
  const response = await fetch(`http://localhost:8080/products/${id}`, {
    method: "GET",
  });

  const data = await response.json();

  if (data) {
    productInfo.name = data.name;
    productInfo.price = data.price;
    productInfo.quantity = data.quantity;
  }
};
</script>

<style></style>
