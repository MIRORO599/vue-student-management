<template>
  <main class="page">
    <section class="card">
      <div class="header">
        <div>
          <h1>商品管理システム</h1>
          <p>商品情報の検索、登録、更新、削除を行います。</p>
        </div>
      </div>

      <div class="search-row">
        <div class="search-area">
          <label>商品名</label>
          <input v-model.trim="searchName" type="text" placeholder="例：iPhone" />
          <button @click="clearSearch">クリア</button>
        </div>

        <button class="primary-button" @click="openCreateForm">
          新規登録
        </button>
      </div>

      <p>表示件数：{{ filteredProducts.length }} 件</p>

      <table class="product-table">
        <thead>
          <tr>
            <th>No.</th>
            <th>商品名</th>
            <th>カテゴリ</th>
            <th>単価</th>
            <th>在庫数</th>
            <th>登録日</th>
            <th>操作</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="(product, index) in filteredProducts" :key="product.productId">
            <td>{{ index + 1 }}</td>
            <td>{{ product.productName }}</td>
            <td>{{ product.category }}</td>
            <td>{{ product.price }}</td>
            <td>{{ product.stockQuantity }}</td>
            <td>{{ product.createdAt }}</td>
            <td>
              <button @click="openEditForm(product)">更新</button>
              <button class="danger-button" @click="openDeleteForm(product.productId)">
                削除
              </button>
            </td>
          </tr>

          <tr v-if="filteredProducts.length === 0">
            <td colspan="7">該当する商品情報がありません。</td>
          </tr>
        </tbody>
      </table>
    </section>

    <div v-if="isFormVisible" class="modal-overlay" @click.self="closeForm">
      <section class="modal-card">
        <div class="modal-header">
          <h2>{{ isEditMode ? '商品更新' : '商品登録' }}</h2>
          <button class="close-button" @click="closeForm">×</button>
        </div>

        <div class="form-area">
          <!-- <label>商品ID</label>
          <input v-model="form.productId" type="number" disabled /> -->

          <label>商品名</label>
          <input v-model.trim="form.productName" type="text" />

          <label>カテゴリ</label>
          <input v-model.trim="form.category" type="text" />

          <label>単価</label>
          <input v-model.number="form.price" type="number" min="0" />

          <label>在庫数</label>
          <input v-model.number="form.stockQuantity" type="number" min="0" />

          <label>登録日</label>
          <input v-model="form.createdAt" type="date" />
        </div>

        <ul v-if="errorMessages.length > 0" class="error-list">
          <li v-for="message in errorMessages" :key="message">
            {{ message }}
          </li>
        </ul>

        <div class="button-area">
          <button class="primary-button" @click="saveProduct">保存</button>
          <button @click="closeForm">キャンセル</button>
        </div>
      </section>
    </div>

    <div v-if="isDeleteVisible" class="modal-overlay" @click.self="closeDeleteForm">
      <section class="modal-cancelcard">
        <div>
          <h2>商品削除</h2>
        </div>

        <div>
          <label>削除してもよろしいですか？</label>
        </div>

        <div class="button-area">
          <button class="primary-button" @click="confirmDeleteProduct">確認</button>
          <button @click="closeDeleteForm">キャンセル</button>
        </div>
      </section>
    </div>
  </main>
</template>

<script setup lang="ts">
import { computed, reactive, ref, watch } from 'vue'

type Product = {
  productId: number
  productName: string
  category: string
  price: number | null | ''
  stockQuantity: number | null | ''
  createdAt: string
}

const searchName = ref('')
const isFormVisible = ref(false)
const isDeleteVisible = ref(false)
const isEditMode = ref(false)
const errorMessages = ref<string[]>([])
const deleteProductId = ref<number | null>(null)

const storageKey = 'product-management-products'

const defaultProducts: Product[] = [
  {
    productId: 1,
    productName: 'iPhone',
    category: 'スマートフォン',
    price: 120000,
    stockQuantity: 10,
    createdAt: '2026-06-26'
  },
  {
    productId: 2,
    productName: 'MacBook',
    category: 'パソコン',
    price: 180000,
    stockQuantity: 5,
    createdAt: '2026-06-26'
  },
  {
    productId: 3,
    productName: 'AirPods',
    category: 'イヤホン',
    price: 30000,
    stockQuantity: 20,
    createdAt: '2026-06-26'
  }
]

const savedProducts = localStorage.getItem(storageKey)

const products = ref<Product[]>(
  savedProducts
    ? JSON.parse(savedProducts)
    : defaultProducts
)

watch(
  products,
  (newProducts) => {
    localStorage.setItem(storageKey, JSON.stringify(newProducts))
  },
  { deep: true }
)

const form = reactive<Product>({
  productId: 0,
  productName: '',
  category: '',
  price: null,
  stockQuantity: null,
  createdAt: ''
})

const filteredProducts = computed(() => {
  if (searchName.value === '') {
    return products.value
  }

  return products.value.filter((product) =>
    product.productName.includes(searchName.value)
  )
})

function openCreateForm() {
  isEditMode.value = false
  isFormVisible.value = true
  errorMessages.value = []
  resetForm()
  form.productId = getNextProductId()
}

function openEditForm(product: Product) {
  isEditMode.value = true
  isFormVisible.value = true
  errorMessages.value = []

  form.productId = product.productId
  form.productName = product.productName
  form.category = product.category
  form.price = product.price
  form.stockQuantity = product.stockQuantity
  form.createdAt = product.createdAt
}

function openDeleteForm(productId: number) {
  deleteProductId.value = productId
  isDeleteVisible.value = true
}

function saveProduct() {
  errorMessages.value = validateForm()

  if (errorMessages.value.length > 0) {
    return
  }

  if (isEditMode.value) {
    updateProduct()
  } else {
    createProduct()
  }

  closeForm()
}

function createProduct() {
  products.value.push({
    productId: form.productId,
    productName: form.productName,
    category: form.category,
    price: form.price,
    stockQuantity: form.stockQuantity,
    createdAt: form.createdAt
  })
}

function updateProduct() {
  const targetIndex = products.value.findIndex(
    (product) => product.productId === form.productId
  )

  if (targetIndex === -1) {
    alert('更新対象の商品情報が見つかりません。')
    return
  }

  products.value[targetIndex] = {
    productId: form.productId,
    productName: form.productName,
    category: form.category,
    price: form.price,
    stockQuantity: form.stockQuantity,
    createdAt: form.createdAt
  }
}

function deleteProduct(productId: number) {
  products.value = products.value.filter(
    (product) => product.productId !== productId
  )
}

function confirmDeleteProduct() {
  if (deleteProductId.value === null) {
    return
  }

  deleteProduct(deleteProductId.value)
  closeDeleteForm()
}

function closeForm() {
  isFormVisible.value = false
  errorMessages.value = []
  resetForm()
}

function closeDeleteForm() {
  isDeleteVisible.value = false
  deleteProductId.value = null
}

function clearSearch() {
  searchName.value = ''
}

function resetForm() {
  form.productId = 0
  form.productName = ''
  form.category = ''
  form.price = null
  form.stockQuantity = null
  form.createdAt = ''
}

function getNextProductId() {
  if (products.value.length === 0) {
    return 1
  }

  return Math.max(...products.value.map((product) => product.productId)) + 1
}

function validateForm() {
  const messages: string[] = []

  if (form.productName === '') {
    messages.push('商品名を入力してください。')
  }

  if (form.productName.length > 25) {
    messages.push('商品名は25文字以下で入力してください。')
  }

  if (form.category === '') {
    messages.push('カテゴリを入力してください。')
  }

  if (form.category.length > 25) {
    messages.push('カテゴリは25文字以下で入力してください。')
  }

  if (form.price === null || form.price === '' || Number.isNaN(form.price)) {
    messages.push('単価を入力してください。')
  } else if (form.price < 0) {
    messages.push('単価は0以上で入力してください。')
  }

  if (
    form.stockQuantity === null ||
    form.stockQuantity === '' ||
    Number.isNaN(form.stockQuantity)
  ) {
    messages.push('在庫数を入力してください。')
  } else if (form.stockQuantity < 0) {
    messages.push('在庫数は0以上で入力してください。')
  }

  if (form.createdAt === '') {
    messages.push('登録日を入力してください。')
  }

  return messages
}
</script>

<style scoped>
.page {
  min-height: 100vh;
  padding: 32px;
  background: #f6f7fb;
  color: #222;
}

.card {
  max-width: 1100px;
  margin: 0 auto 24px;
  padding: 24px;
  border-radius: 14px;
  background: #fff;
  box-shadow: 0 8px 24px rgb(0 0 0 / 8%);
}

.header {
  margin-bottom: 48px;
}

.search-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  margin: 24px 0 12px;
}

.search-area {
  display: flex;
  align-items: center;
  gap: 8px;
}

.header .primary-button {
  margin-top: 0;
}

.primary-button {
  min-height: 40px;
  padding: 0 18px;
  border-color: #2563eb;
  background: #2563eb;
  color: #fff;
  white-space: nowrap;
}

input,
select {
  min-height: 36px;
  padding: 6px 10px;
  border: 1px solid #cfd3dc;
  border-radius: 8px;
}

.product-table {
  width: 100%;
  border-collapse: collapse;
}

.product-table th,
.product-table td {
  padding: 12px 10px;
  border-bottom: 1px solid #e4e7ee;
  text-align: center;
}

.product-table th {
  background: #f0f2f8;
}

.form-area {
  display: grid;
  grid-template-columns: 120px 1fr;
  gap: 12px;
  max-width: 500px;
}

.button-area {
  display: flex;
  gap: 8px;
  margin-top: 20px;
}

.error-list {
  margin-top: 18px;
  padding: 12px 16px 12px 32px;
  border-radius: 8px;
  background: #fff2f2;
  color: #d33;
}

button {
  min-height: 36px;
  padding: 6px 14px;
  border: 1px solid #cfd3dc;
  border-radius: 8px;
  background: #fff;
  cursor: pointer;
}

.primary-button {
  min-height: 36px;
  padding: 6px 14px;
  border-color: #2563eb;
  background: #2563eb;
  color: #fff;
}

.danger-button {
  border-color: #dc2626;
  background: #dc2626;
  color: #fff;
}

.modal-overlay {
  position: fixed;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 24px;
  background: rgb(0 0 0 / 45%);
  z-index: 1000;
}

.modal-card {
  width: 100%;
  max-width: 560px;
  padding: 24px;
  border-radius: 14px;
  background: #fff;
  box-shadow: 0 12px 36px rgb(0 0 0 / 20%);
}

.modal-cancelcard {
  max-width: 560px;
  padding: 24px;
  border-radius: 14px;
  background: #fff;
  box-shadow: 0 12px 36px rgb(0 0 0 / 20%);
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
}

.close-button {
  width: 36px;
  padding: 0;
  font-size: 22px;
  line-height: 1;
}
</style>