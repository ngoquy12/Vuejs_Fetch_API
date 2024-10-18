<template>
  <div>
    <h1>Danh sách sản phẩm</h1>
    <div style="margin: 10px 0">
      <button @click="handleShowForm">Thêm mới sản phẩm</button>
      <input type="text" v-model="inputSearch" />

      <select v-model="filterByStatus">
        <option value="">Tất cả</option>
        <option value="1">Đang bán</option>
        <option value="0">Ngừng bán</option>
      </select>

      <select v-model="sortByPrice">
        <option value="">Sắp xếp sản phẩm theo giá</option>
        <option value="asc">Giá tăng dần</option>
        <option value="desc">Giá giảm dần</option>
      </select>

      <button @click="handleRefreshFilter">X Hủy bộ lọc</button>
      <button @click="fetchData()">
        <i class="fa-solid fa-rotate-right"></i>
      </button>
    </div>
    <table border="1" style="width: 100%">
      <thead>
        <tr>
          <th>
            <input type="checkbox" />
          </th>
          <th>Name</th>
          <th>Hình ảnh</th>
          <th>Price</th>
          <th>Quantity</th>
          <th>Status</th>
          <th>Description</th>
          <th>Option</th>
        </tr>
      </thead>
      <tbody v-for="product in products" :key="product.id">
        <tr>
          <td><input type="checkbox" /></td>
          <td>{{ product.name }}</td>
          <td>
            <div style="display: flex; justify-content: center; margin: 8px">
              <img height="50" :src="product.image" alt="" />
            </div>
          </td>
          <td>{{ product.price }}</td>
          <td>{{ product.quantity }}</td>
          <td>{{ product.status ? "Đang bán" : "Ngừng bán" }}</td>

          <td>{{ product.description }}</td>
          <td>
            <button>Edit</button>
            <button @click="handleDelete(product.id)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>

    <div
      style="display: flex; justify-content: space-between; margin-top: 20px"
    >
      <div>
        Hiển thị <b>{{ products.length }}</b> trên
        <b>{{ totalRecord }} bản ghi</b>
      </div>
      <div style="display: flex; gap: 16px">
        <button :disabled="currentPage === 1" @click="handlePrevPage">
          Prev
        </button>
        <div>
          <button
            v-for="(page, index) in pages"
            :key="index"
            :class="page === currentPage ? 'btn-active' : ''"
            @click="handleChangePage(page)"
          >
            {{ page }}
          </button>
        </div>
        <button :disabled="currentPage === pages" @click="handleNextPage">
          Next
        </button>
        <select v-model="pageSize">
          <option value="10">10 bản ghi</option>
          <option value="20">20 bản ghi</option>
          <option value="30">30 bản ghi</option>
          <option value="40">40 bản ghi</option>
          <option value="50">50 bản ghi</option>
        </select>
      </div>
    </div>

    <!-- Form Thêm và sửa thông tin sản phẩm -->
    <FormProduct :isShowFom="isShowForm" @onClose="handleCloseForm" />
  </div>
</template>

<script setup>
import axios from "axios";
import FormProduct from "./FormProduct.vue";
import { onMounted, ref, watch } from "vue";

// <--------------- Các biến liên quan đến danh sách sản phẩm ---------------->
const products = ref([]); // Danh sách sản phẩm
const pages = ref([]); // Mảng chứa tổng số trang
const totalRecord = ref(0); // Tổng số tất cả bản ghi
const pageSize = ref(10); // Số bản ghi / trang
const currentPage = ref(1); // Trang hiện tại
const inputSearch = ref(""); // Từ khóa tìm kiếm
const filterByStatus = ref(""); // Lọc bản ghi theo trạng thái
const sortByPrice = ref(""); // Sắp xếp sản phẩm theo giá

const isShowForm = ref(false);

// Delay thời gian tìm kiếm
const debounce = (cb, delay) => {
  let timeout;

  return function (...args) {
    clearInterval(timeout);

    timeout = setTimeout(() => cb.apply(this, args), delay);
  };
};

// Gọi API lấy danh sách sản phẩm
const fetchData = debounce(async () => {
  try {
    let response;

    if (filterByStatus.value) {
      response = await axios.get(
        `http://localhost:8080/products?_page=${
          currentPage.value
        }&_limit=${+pageSize.value}&name_like=${inputSearch.value.trim()}&_sort=price&_order=${
          sortByPrice.value
        }&status=${filterByStatus.value}`
      );
    } else {
      response = await axios.get(
        `http://localhost:8080/products?_page=${
          currentPage.value
        }&_limit=${+pageSize.value}&name_like=${inputSearch.value.trim()}&_sort=price&_order=${
          sortByPrice.value
        }`
      );
    }

    if (response.headers["x-total-count"]) {
      const totalPages = parseInt(response.headers["x-total-count"]);

      totalRecord.value = totalPages;

      pages.value = Math.ceil(totalPages / pageSize.value);
    }

    products.value = response.data;
  } catch (error) {
    // Xử lý các lỗi
    console.log(error);
  }
}, 300);

// Cách viết ngắn gọn của Gọi API lấy danh sách sản phẩm
// const fetchData = debounce(async () => {
//   try {
//     const baseUrl = `http://localhost:8080/products?_page=${currentPage.value}&_limit=${+pageSize.value}&name_like=${inputSearch.value.trim()}&_sort=price&_order=${sortByPrice.value}`;

//     // Thêm tham số status chỉ khi nó có giá trị
//     const url = filterByStatus.value
//       ? `${baseUrl}&status=${filterByStatus.value}`
//       : baseUrl;

//     const response = await axios.get(url);

//     if (response.headers["x-total-count"]) {
//       const totalPages = parseInt(response.headers["x-total-count"]);
//       pages.value = Math.ceil(totalPages / pageSize.value);
//     }

//     products.value = response.data;
//   } catch (error) {
//     // Xử lý các lỗi
//     console.log(error);
//   }
// }, 300);

onMounted(() => {
  fetchData();
});

// Hàm lấy thông tin chi tiết một sản phẩm theo id
const getProductById = async (id) => {
  try {
    const response = await axios.get(`http://localhost:8080/products/${id}`);
    console.log("Data: ", response.data);
  } catch (error) {
    // Xử lý lỗi
  }
};

// Khi biến currentPage thay đổi sẽ gọi API
watch([currentPage, pageSize, inputSearch, filterByStatus, sortByPrice], () => {
  fetchData();
});

// Chuyển tới trang tiếp theo
const handleNextPage = () => {
  if (currentPage.value < pages.value) {
    currentPage.value += 1;
  }
};

// Quay lại trang trước đó
const handlePrevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value -= 1;
  }
};

// Chọn vào trang chỉ định
const handleChangePage = (page) => {
  currentPage.value = page;
};

// Hàm xóa thông tin một sản phẩm theo id
const handleDelete = async (id) => {
  const isConfirm = confirm("Bạn có muốn xóa sản phẩm này không? ");

  if (isConfirm) {
    const response = await axios.delete(`http://localhost:8080/products/${id}`);

    if (response.status === 200) {
      fetchData();
    }
  }
};

// Hàm mở form quản lý sản phẩm
const handleShowForm = () => {
  isShowForm.value = true;
};

// Hàm đóng form quản lý sản phẩm
const handleCloseForm = () => {
  isShowForm.value = false;
};

// Hủy tất cả bộ lọc
const handleRefreshFilter = () => {
  inputSearch.value = "";
  filterByStatus.value = "";
  sortByPrice.value = "";
};
</script>

<style>
.btn-active {
  background-color: red;
  color: #fff;
}
</style>
