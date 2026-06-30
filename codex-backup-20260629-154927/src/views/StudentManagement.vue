<template>
  <main class="page">
    <section class="card">
      <div class="header">
        <div>
          <h1>学生管理システム</h1>
          <p>学生情報の検索、登録、更新、削除を行います。</p>
        </div>

        <button class="primary-button" @click="openCreateForm">
          新規登録
        </button>
      </div>

      <div class="search-area">
        <label>学生名</label>
        <input v-model.trim="searchName" type="text" placeholder="例：山田" />
        <button @click="clearSearch">クリア</button>
      </div>

      <p>表示件数：{{ filteredStudents.length }} 件</p>

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
          <tr v-for="student in filteredStudents" :key="student.studentId">
            <td>{{ student.studentId }}</td>
            <td>{{ student.studentName }}</td>
            <td>{{ student.birthday }}</td>
            <td>{{ student.age }}</td>
            <td>{{ student.score }}</td>
            <td>{{ student.studentClass }}</td>
            <td>{{ getTeacherName(student.studentClass) }}</td>
            <td>
              <button @click="openEditForm(student)">更新</button>
              <button class="danger-button" @click="openDeleteForm(student.studentId)">
                削除
              </button>
            </td>
          </tr>

          <tr v-if="filteredStudents.length === 0">
            <td colspan="8">該当する学生情報がありません。</td>
          </tr>
        </tbody>
      </table>
    </section>

    <div v-if="isFormVisible" class="modal-overlay" @click.self="closeForm">
      <section class="modal-card">
        <div class="modal-header">
          <h2>{{ isEditMode ? '学生更新' : '学生登録' }}</h2>
          <button class="close-button" @click="closeForm">×</button>
        </div>

        <div class="form-area">
          <label>学生名</label>
          <input v-model.trim="form.studentName" type="text" maxlength="50" />

          <label>誕生日</label>
          <input v-model="form.birthday" type="date" />

          <label>年齢</label>
          <input v-model.number="form.age" type="number" min="1" max="150" />

          <label>成績</label>
          <input v-model.number="form.score" type="number" min="0" max="100" />

          <label>クラス</label>
         <select v-model.number="form.studentClass"> 
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
          <button class="primary-button" @click="saveStudent">保存</button>
          <button @click="closeForm">キャンセル</button>
        </div>
      </section>
    </div>


    <div v-if="isVisible" class="modal-overlay" @click.self="closeForm">
      <section class="modal-cancelcard">
        <div >
          <h2>学生削除</h2>
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
  studentId: number
  studentName: string
  birthday: string
  age: number | null | ''
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

const storageKey = 'student-management-students'

const defaultStudents: Student[] = [
  {
    studentId: 1,
    studentName: '山田太郎',
    birthday: '2001-04-10',
    age: 23,
    score: 82,
    studentClass: 1
  },
  {
    studentId: 2,
    studentName: '佐藤花子',
    birthday: '2002-08-21',
    age: 22,
    score: 91,
    studentClass: 2
  },
  {
    studentId: 3,
    studentName: '鈴木一郎',
    birthday: '2000-12-05',
    age: 24,
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
  studentId: 0,
  studentName: '',
  birthday: '',
  age: null,
  score: null,
  studentClass: 0
})

const filteredStudents = computed(() => {
  if (searchName.value === '') {
    return students.value
  }

  return students.value.filter((student) =>
    student.studentName.includes(searchName.value)
  )
})

function openCreateForm() {
  isEditMode.value = false
  isFormVisible.value = true
  resetForm()
}
function openDeleteForm(studentId: number) {
  deleteStudentId.value = studentId
  isVisible.value = true
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
    studentId: getNextStudentId(),
    studentName: form.studentName,
    birthday: form.birthday,
    age: form.age,
    score: form.score,
    studentClass: form.studentClass
  })
}

function updateStudent() {
  const targetIndex = students.value.findIndex(
    (student) => student.studentId === form.studentId
  )

  if (targetIndex === -1) {
    alert('更新対象の学生情報が見つかりません。')
    return
  }

  students.value[targetIndex] = {
    studentId: form.studentId,
    studentName: form.studentName,
    birthday: form.birthday,
    age: form.age,
    score: form.score,
    studentClass: form.studentClass
  }
}

function deleteStudent(studentId: number) {
  // if (!confirm('削除してもよろしいですか？')) {
  //   return
  // }

  students.value = students.value.filter(
    (student) => student.studentId !== studentId
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
  form.studentId = 0
  form.studentName = ''
  form.birthday = ''
  form.age = null
  form.score = null
  form.studentClass = 0
}

function getNextStudentId() {
  if (students.value.length === 0) {
    return 1
  }

  return Math.max(...students.value.map((student) => student.studentId)) + 1
}

function getTeacherName(studentClass: number) {
  const teacher = teachers.value.find(
    (item) => item.studentClass === studentClass
  )

  return teacher ? teacher.teacherName : ''
}

function validateForm() {
  const messages: string[] = []

  if (form.studentName === '') {
    messages.push('学生名を入力してください。')
  }
  if (form.studentName.length > 25) {
    messages.push('25文字以下を入力してください。')
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