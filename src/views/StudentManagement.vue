<template>
  <main class="page">
    <section class="card">
      <div class="header">
        <div>
          <h1>学生管理システム</h1>
          <p>学生情報の検索、登録、更新、削除は Spring Boot のバックエンドに接続しています。</p>
        </div>

        <button class="primary-button" type="button" @click="openCreateForm">
          新規登録
        </button>
      </div>

      <div class="search-area">
        <label for="student-name">学生名</label>
        <input
          id="student-name"
          v-model.trim="searchName"
          type="text"
          placeholder="例：山田"
          @keyup.enter="loadStudents"
        />
        <button type="button" @click="loadStudents">検索</button>
        <button type="button" @click="clearSearch">クリア</button>
      </div>

      <p v-if="apiError" class="api-error">{{ apiError }}</p>
      <p v-else-if="isLoading">学生データを読み込み中...</p>
      <p v-else>表示件数：{{ students.length }} 件</p>

      <table class="student-table">
        <thead>
          <tr>
            <th>学生ID</th>
            <th>学生名</th>
            <th>誕生日</th>
            <th>年齢</th>
            <th>成績</th>
            <th>クラス</th>
            <th>先生名</th>
            <th>操作</th>
          </tr>
        </thead>

        <tbody>
          <tr v-for="student in students" :key="student.studentId">
            <td>{{ student.studentId }}</td>
            <td>{{ student.studentName }}</td>
            <td>{{ student.birthday }}</td>
            <td>{{ student.age }}</td>
            <td>{{ student.score }}</td>
            <td>{{ student.studentClass }}</td>
            <td>{{ student.teacherName || '-' }}</td>
            <td>
              <button type="button" @click="openEditForm(student)">更新</button>
              <button
                class="danger-button"
                type="button"
                @click="openDeleteForm(student.studentId)"
              >
                削除
              </button>
            </td>
          </tr>

          <tr v-if="!isLoading && students.length === 0">
            <td colspan="8">該当する学生情報がありません。</td>
          </tr>
        </tbody>
      </table>
    </section>

    <div v-if="isFormVisible" class="modal-overlay" @click.self="closeForm">
      <section class="modal-card">
        <div class="modal-header">
          <h2>{{ isEditMode ? '学生更新' : '学生登録' }}</h2>
          <button class="close-button" type="button" @click="closeForm">×</button>
        </div>

        <div class="form-area">
          <label for="form-name">学生名</label>
          <input id="form-name" v-model.trim="form.studentName" type="text" maxlength="50" />

          <label for="form-birthday">誕生日</label>
          <input id="form-birthday" v-model="form.birthday" type="date" />

          <label for="form-age">年齢</label>
          <input id="form-age" v-model.number="form.age" type="number" min="0" max="150" />

          <label for="form-score">成績</label>
          <input id="form-score" v-model.number="form.score" type="number" min="0" max="100" />

          <label for="form-class">クラス</label>
          <select id="form-class" v-model.number="form.studentClass">
            <option :value="0">選択してください</option>
            <option v-for="teacher in teachers" :key="teacher.studentClass" :value="teacher.studentClass">
              {{ teacher.studentClass }}組
            </option>
          </select>
        </div>

        <ul v-if="errorMessages.length > 0" class="error-list">
          <li v-for="message in errorMessages" :key="message">
            {{ message }}
          </li>
        </ul>

        <div class="button-area">
          <button class="primary-button" type="button" :disabled="isSaving" @click="saveStudent">
            {{ isSaving ? '保存中...' : '保存' }}
          </button>
          <button type="button" @click="closeForm">キャンセル</button>
        </div>
      </section>
    </div>

    <div v-if="isDeleteVisible" class="modal-overlay" @click.self="closeDeleteForm">
      <section class="modal-cancelcard">
        <h2>学生削除</h2>
        <p>削除してもよろしいですか？</p>

        <div class="button-area">
          <button class="danger-button" type="button" :disabled="isSaving" @click="confirmDeleteStudent">
            確認
          </button>
          <button type="button" @click="closeDeleteForm">キャンセル</button>
        </div>
      </section>
    </div>
  </main>
</template>

<script setup lang="ts">
import { onMounted, reactive, ref } from 'vue'

type Student = {
  studentId: number
  studentName: string
  birthday: string
  age: number | null | ''
  score: number | null | ''
  studentClass: number
  teacherName?: string
}

type Teacher = {
  studentClass: number
  teacherName: string
}

type StudentPayload = {
  studentName: string
  birthday: string
  age: number
  score: number
  studentClass: number
}

const apiBaseUrl = '/api/students'

const searchName = ref('')
const students = ref<Student[]>([])
const isLoading = ref(false)
const isSaving = ref(false)
const isFormVisible = ref(false)
const isDeleteVisible = ref(false)
const isEditMode = ref(false)
const apiError = ref('')
const errorMessages = ref<string[]>([])
const deleteStudentId = ref<number | null>(null)

const teachers = ref<Teacher[]>([
  { studentClass: 1, teacherName: '田中先生' },
  { studentClass: 2, teacherName: '高橋先生' },
  { studentClass: 3, teacherName: '伊藤先生' }
])

const form = reactive<Student>({
  studentId: 0,
  studentName: '',
  birthday: '',
  age: null,
  score: null,
  studentClass: 0,
  teacherName: ''
})

onMounted(() => {
  loadStudents()
})

async function loadStudents() {
  isLoading.value = true
  apiError.value = ''

  try {
    const searchParams = new URLSearchParams()

    if (searchName.value !== '') {
      searchParams.set('name', searchName.value)
    }

    const requestUrl = searchParams.toString()
      ? `${apiBaseUrl}?${searchParams.toString()}`
      : apiBaseUrl

    const response = await fetch(requestUrl)

    if (!response.ok) {
      throw new Error(`読み込み失敗：${response.status}`)
    }

    students.value = await response.json()
  } catch (error) {
    apiError.value = getErrorMessage(error)
  } finally {
    isLoading.value = false
  }
}

function openCreateForm() {
  isEditMode.value = false
  isFormVisible.value = true
  errorMessages.value = []
  resetForm()
}

function openEditForm(student: Student) {
  isEditMode.value = true
  isFormVisible.value = true
  errorMessages.value = []

  form.studentId = student.studentId
  form.studentName = student.studentName
  form.birthday = student.birthday
  form.age = student.age
  form.score = student.score
  form.studentClass = student.studentClass
  form.teacherName = student.teacherName || ''
}

function openDeleteForm(studentId: number) {
  deleteStudentId.value = studentId
  isDeleteVisible.value = true
}

async function saveStudent() {
  errorMessages.value = validateForm()

  if (errorMessages.value.length > 0) {
    return
  }

  isSaving.value = true
  apiError.value = ''

  try {
    const method = isEditMode.value ? 'PUT' : 'POST'
    const url = isEditMode.value ? `${apiBaseUrl}/${form.studentId}` : apiBaseUrl

    const response = await fetch(url, {
      method,
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(toPayload())
    })

    if (!response.ok) {
      throw new Error(`保存失敗：${response.status}`)
    }

    closeForm()
    await loadStudents()
  } catch (error) {
    errorMessages.value = [getErrorMessage(error)]
  } finally {
    isSaving.value = false
  }
}

async function confirmDeleteStudent() {
  if (deleteStudentId.value === null) {
    return
  }

  isSaving.value = true
  apiError.value = ''

  try {
    const response = await fetch(`${apiBaseUrl}/${deleteStudentId.value}`, {
      method: 'DELETE'
    })

    if (!response.ok) {
      throw new Error(`削除失敗：${response.status}`)
    }

    closeDeleteForm()
    await loadStudents()
  } catch (error) {
    apiError.value = getErrorMessage(error)
  } finally {
    isSaving.value = false
  }
}

function clearSearch() {
  searchName.value = ''
  loadStudents()
}

function closeForm() {
  isFormVisible.value = false
  errorMessages.value = []
  resetForm()
}

function closeDeleteForm() {
  isDeleteVisible.value = false
  deleteStudentId.value = null
}

function resetForm() {
  form.studentId = 0
  form.studentName = ''
  form.birthday = ''
  form.age = null
  form.score = null
  form.studentClass = 0
  form.teacherName = ''
}

function toPayload(): StudentPayload {
  return {
    studentName: form.studentName,
    birthday: form.birthday,
    age: Number(form.age),
    score: Number(form.score),
    studentClass: form.studentClass
  }
}

function validateForm() {
  const messages: string[] = []

  if (form.studentName === '') {
    messages.push('学生名を入力してください。')
  }

  if (form.studentName.length > 25) {
    messages.push('学生名は25文字以下で入力してください。')
  }

  if (form.birthday === '') {
    messages.push('誕生日を入力してください。')
  }

  if (form.age === null || form.age === '' || Number.isNaN(form.age)) {
    messages.push('年齢を入力してください。')
  } else if (form.age < 0 || form.age > 150) {
    messages.push('年齢は0以上150以下で入力してください。')
  }

  if (form.score === null || form.score === '' || Number.isNaN(form.score)) {
    messages.push('成績を入力してください。')
  } else if (form.score < 0 || form.score > 100) {
    messages.push('成績は0以上100以下で入力してください。')
  }

  if (form.studentClass === 0) {
    messages.push('クラスを選択してください。')
  }

  return messages
}

function getErrorMessage(error: unknown) {
  if (error instanceof Error) {
    return error.message
  }

  return 'バックエンドへのリクエストでエラーが発生しました。Spring Boot が起動しているか確認してください。'
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
  border-radius: 8px;
  background: #fff;
  box-shadow: 0 8px 24px rgb(0 0 0 / 8%);
}

.header {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
}

.header h1 {
  margin: 0 0 8px;
  font-size: 28px;
}

.header p {
  margin: 0;
  color: #5f6675;
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

.student-table {
  width: 100%;
  border-collapse: collapse;
}

.student-table th,
.student-table td {
  padding: 12px 10px;
  border-bottom: 1px solid #e4e7ee;
  text-align: center;
}

.student-table th {
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

.api-error,
.error-list {
  margin-top: 18px;
  padding: 12px 16px;
  border-radius: 8px;
  background: #fff2f2;
  color: #d33;
}

.error-list {
  padding-left: 32px;
}

button {
  min-height: 36px;
  padding: 6px 14px;
  border: 1px solid #cfd3dc;
  border-radius: 8px;
  background: #fff;
  cursor: pointer;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.65;
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

.modal-card,
.modal-cancelcard {
  width: 100%;
  max-width: 560px;
  padding: 24px;
  border-radius: 8px;
  background: #fff;
  box-shadow: 0 12px 36px rgb(0 0 0 / 20%);
}

.modal-cancelcard {
  max-width: 420px;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
}

.modal-header h2,
.modal-cancelcard h2 {
  margin: 0;
}

.close-button {
  width: 36px;
  padding: 0;
  font-size: 22px;
  line-height: 1;
}

@media (max-width: 760px) {
  .page {
    padding: 16px;
  }

  .header,
  .search-area {
    align-items: stretch;
    flex-direction: column;
  }

  .student-table {
    font-size: 14px;
  }

  .form-area {
    grid-template-columns: 1fr;
  }
}
</style>
