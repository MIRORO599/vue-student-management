<template>
  <main class="page">
    <section class="card">
      <div class="header">
        <div>
          <h1>商品管理システム</h1>
          <p>商品情報の検索、登録、更新、削除を行います。</p>
        </div>

        <button class="primary-button" @click="openCreateForm">
          新規登録
        </button>
      </div>

      <div class="search-area">
        <label>商品名</label>
        <input v-model.trim="searchName" type="text" placeholder="例：iphone" />
        <button @click="clearSearch">クリア</button>
      </div>

      <p>表示件数：{{ filteredProducts.length }} 件</p>

      <table class="product-table">
        <thead>
          <tr>
            <th>商品ID</th>
            <th>商品名</th>
            <th>カテゴリ</th>
             <th>単価</th>
             <th>在庫数</th>
            <th>登録日</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="product in filteredProducts" :key="product.productId">
            <td>{{ product.productId }}</td>
            <td>{{ product.productName }}</td>
            <td>{{ product.category }}</td>
            <td>{{ product.price}}</td>
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
            <td colspan="8">該当する商品情報がありません。</td>
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
          <label>商品ID</label>
          <input v-model="form.productId" type="number" />

          <label>商品名</label>
          <input v-model="form.productName" type="text" />

          <label>カテゴリ</label>
          <input v-model="form.category" type="text />

          <label>単価</label>
          <input v-model.number="form.price" type="number" min="0"  />

          <label>在庫数</label>
          <input v-model.number="form.stockQuantity" type="number" min="0"  />

          <label>登録日</label>
          <input v-model="form.createdAt" type="date" />
        </div>

        <ul v-if="errorMessages.length > 0" class="error-list">
          <li v-for="message in errorMessages" :key="message">
            {{ message }}
          </li>
        </ul>

        <div class="button-area">
          <button class="primary-button" @click="saveStudent">保存</button>
          <button @click="closeForm">キャンセル</button>
        </div>
      </section>
    </div>


    <div v-if="isVisible" class="modal-overlay" @click.self="closeForm">
      <section class="modal-cancelcard">
        <div>
          <h2>商品削除</h2>
          <!-- <button class="close-button" @click="closeForm">×</button> -->
        </div>

        <div>
          <label>削除してもよろしいですか？</label>
        </div>

        <div class="button-area">
          <button class="primary-button" @click="confirmDeleteStudent">確認</button>
          <button @click="closeDeleteForm">キャンセル</button>
        </div>
      </section>
    </div>
  </main>
</template>

<script setup lang="ts">
import { computed, reactive, ref, watch } from 'vue'

type Student = {
  productId: number
  productName: string
  category: string
  price: number | null | ''
  score: number | null | ''
  studentClass: number
}

type Teacher = {
  studentClass: number
  teacherName: string
}

const searchName = ref('')
const isFormVisible = ref(false)
const isVisible = ref(false)
const isEditMode = ref(false)
const errorMessages = ref<string[]>([])
const deleteStudentId = ref<number | null>(null)

const storageKey = 'product-management-students'

const defaultStudents: Student[] = [
  {
    productId: 1,
    productName: '山田太郎',
    category: '2001-04-10',
    price: 23,
    score: 82,
    studentClass: 1
  },
  {
    productId: 2,
    productName: '佐藤花子',
    category: '2002-08-21',
    price: 22,
    score: 91,
    studentClass: 2
  },
  {
    productId: 3,
    productName: '鈴木一郎',
    category: '2000-12-05',
    price: 24,
    score: 76,
    studentClass: 1
  }
]

const savedStudents = localStorage.getItem(storageKey)

const students = ref<Student[]>(
  savedStudents
    ? JSON.parse(savedStudents)
    : defaultStudents
)

const teachers = ref<Teacher[]>([
  {
    studentClass: 1,
    teacherName: '田中先生'
  },
  {
    studentClass: 2,
    teacherName: '高橋先生'
  },
  {
    studentClass: 3,
    teacherName: '伊藤先生'
  }
])

watch(
  students,
  (newStudents) => {
    localStorage.setItem(
      storageKey,
      JSON.stringify(newStudents)
    )
  },
  { deep: true }
)

const form = reactive<Student>({
  productId: 0,
  productName: '',
  category: '',
  price: null,
  score: null,
  studentClass: 0
})

const filteredProducts = computed(() => {
  if (searchName.value === '') {
    return students.value
  }

  return students.value.filter((product) =>
    product.productName.includes(searchName.value)
  )
})

function openCreateForm() {
  isEditMode.value = false
  isFormVisible.value = true
  resetForm()
}
function openDeleteForm(productId: number) {
  deleteStudentId.value = productId
  isVisible.value = true
}
function openEditForm(product: Student) {
  isEditMode.value = true
  isFormVisible.value = true
  errorMessages.value = []

  form.productId = product.productId
  form.productName = product.productName
  form.category = product.category
  form.price = product.price
  form.score = product.score
  form.studentClass = product.studentClass
}

function saveStudent() {
  errorMessages.value = validateForm()

  if (errorMessages.value.length > 0) {
    return
  }

  if (isEditMode.value) {
    updateStudent()
  } else {
    createStudent()
  }

  closeForm()
}

function createStudent() {
  students.value.push({
    productId: getNextStudentId(),
    productName: form.productName,
    category: form.category,
    price: form.price,
    score: form.score,
    studentClass: form.studentClass
  })
}

function updateStudent() {
  const targetIndex = students.value.findIndex(
    (product) => product.productId === form.productId
  )

  if (targetIndex === -1) {
    alert('更新対象の商品情報が見つかりません。')
    return
  }

  students.value[targetIndex] = {
    productId: form.productId,
    productName: form.productName,
    category: form.category,
    price: form.price,
    score: form.score,
    studentClass: form.studentClass
  }
}

function deleteStudent(productId: number) {
  // if (!confirm('削除してもよろしいですか？')) {
  //   return
  // }

  students.value = students.value.filter(
    (product) => product.productId !== productId
  )
}

function confirmDeleteStudent() {
  if (deleteStudentId.value === null) {
    return
  }

  deleteStudent(deleteStudentId.value)
  closeDeleteForm()
}

function closeDeleteForm() {
  isVisible.value = false
  deleteStudentId.value = null
}

function clearSearch() {
  searchName.value = ''
}

function closeForm() {
  isFormVisible.value = false
  errorMessages.value = []
  resetForm()
}

function resetForm() {
  form.productId = 0
  form.productName = ''
  form.category = ''
  form.price = null
  form.score = null
  form.studentClass = 0
}

function getNextStudentId() {
  if (students.value.length === 0) {
    return 1
  }

  return Math.max(...students.value.map((product) => product.productId)) + 1
}

function getTeacherName(studentClass: number) {
  const teacher = teachers.value.find(
    (item) => item.studentClass === studentClass
  )

  return teacher ? teacher.teacherName : ''
}

function validateForm() {
  const messages: string[] = []

  // if (form.productName === '') {
  //   messages.push('商品名を入力してください。')
  // }
  // if (form.productName.length > 25) {
  //   messages.push('25文字以下を入力してください。')
  // }

  // if (form.category === '') {
  //   messages.push('誕生日を入力してください。')
  // }

  // if (form.price === null || form.price === '' || Number.isNaN(form.price)) {
  //   messages.push('年齢を入力してください。')
  // } else if (form.price < 0 || form.price > 150) {
  //   messages.push('年齢は0以上150以下で入力してください。')
  // }

  // if (form.score === null || form.score === '' || Number.isNaN(form.score)) {
  //   messages.push('成績を入力してください。')
  // } else if (form.score < 0 || form.score > 100) {
  //   messages.push('成績は0以上100以下で入力してください。')
  // }

  // if (form.studentClass === 0) {
  //   messages.push('クラスを選択してください。')
  // }

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
  /* display: flex; */
  justify-content: space-between;
  gap: 16px;
}

.search-area {
  display: flex;
  align-items: center;
  gap: 8px;
  margin: 24px 0 12px;
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