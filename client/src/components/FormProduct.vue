<template>
  <div v-if="isShowFom" class="overlay">
    <form class="form" @submit.prevent="handleCreateProduct">
      <header>
        <h3>Thêm mới sản phẩm</h3>
        <i class="fa-solid fa-xmark" @click="handleCloseForm"></i>
      </header>

      <main>
        <div class="form-group">
          <label for="name">Tên sản phẩm</label>
          <input type="text" id="name" v-model="product.name" name="name" />
          <p>Tên sản phẩm không được để trống</p>
        </div>
        <div class="form-group">
          <label for="categoryId">Tên danh mục</label>
          <select v-model="product.categoryId" name="categoryId">
            <option
              v-for="category in categories"
              :key="category.id"
              :value="category.id"
            >
              {{ category.name }}
            </option>
          </select>
        </div>

        <div class="form-group">
          <label for="image">Hình ảnh</label>
          <input v-model="product.image" type="text" id="image" />
        </div>

        <div class="form-group">
          <label for="price">Giá</label>
          <input type="number" id="price" />
        </div>

        <div class="form-group">
          <label for="quantity">Số lượng</label>
          <input v-model="product.quantity" type="number" id="quantity" />
        </div>

        <div class="form-group">
          <label for="description">Mô tả</label>
          <textarea v-model="product.description" id="description"></textarea>
        </div>
      </main>

      <footer>
        <button @click="handleCloseForm" type="button" class="cancel-button">
          Hủy
        </button>
        <button type="submit" class="add-button">Thêm</button>
      </footer>
    </form>
  </div>
</template>

<script setup>
import axios from "axios";
import { onMounted, reactive, ref } from "vue";

const { isShowFom } = defineProps(["isShowFom"]);
const emit = defineEmits(["onClose"]);

const categories = ref([]); // Danh sách danh mục sản phẩm

// Lấy danh sách tất cả danh mục
const fetchAllCategory = async () => {
  const response = await axios.get("http://localhost:8080/categories");

  categories.value = response.data;
};

onMounted(() => {
  fetchAllCategory();
});

const product = reactive({
  name: "",
  price: 0,
  quantity: 0,
  description: "",
  categoryId: "",
  image: "",
  status: 1,
  createdAt: new Date(),
});

// Đóng form
const handleCloseForm = () => {
  emit("onClose");
};

// Validate dữ liệu
const validateData = (name) => {};

const handleCreateProduct = () => {};

// Hàm thêm mới sản phẩm
const handleAddProduct = async () => {
  try {
    const response = await axios.post("http://localhost:8080/products", {
      name: "Mèn mén",
      price: 20000,
      quantity: 10,
      description: "Hơi khô",
    });

    fetchData();
  } catch (error) {}
};
</script>

<style scoped>
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.form {
  background-color: white;
  border-radius: 8px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
  width: 400px;
  padding: 20px;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #ccc;
  padding-bottom: 10px;
}

header h3 {
  margin: 0;
  font-size: 20px;
}

header i {
  cursor: pointer;
  font-size: 20px;
  color: #888;
}

main {
  margin-top: 15px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  font-size: 13px;
}

.form-group input,
.form-group select,
.form-group textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  box-sizing: border-box;
}

.form-group textarea {
  resize: none;
  height: 80px;
}

footer {
  display: flex;
  justify-content: flex-end;
  margin-top: 20px;
}

.cancel-button {
  background-color: #f44336;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 10px;
}

.add-button {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
}
</style>
