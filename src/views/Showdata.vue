<template>
  <div class="container py-5">

    <!-- Header -->
    <div class="text-center mb-4">
      <h2 class="fw-bold text-primary">📋 ระบบแสดงรายชื่อผู้ลงทะเบียน</h2>
      <p class="text-muted">ข้อมูลจาก n8n Webhook API</p>
    </div>

    <!-- Card -->
    <div class="card shadow-lg border-0 rounded-4">
      <div class="card-body">

        <!-- ปุ่ม -->
        <div class="d-flex justify-content-between align-items-center mb-3">
          <h5 class="mb-0">รายการข้อมูล</h5>
          <button class="btn btn-primary" @click="fetchData">
            🔄 โหลดข้อมูล
          </button>
        </div>

        <!-- Loading -->
        <div v-if="loading" class="text-center my-4">
          <div class="spinner-border text-primary"></div>
          <p class="mt-2">กำลังโหลดข้อมูล...</p>
        </div>

        <!-- Table -->
        <div class="table-responsive" v-if="users.length">
          <table class="table table-hover align-middle text-center">
            <thead class="table-primary">
              <tr>
                <th>#</th>
                <th>รหัสนักศึกษา</th>
                <th>ชื่อ-นามสกุล</th>
                <th>แผนก</th>
                <th>เวลา</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(item, index) in users" :key="index">
                <td>{{ index + 1 }}</td>
                <td>{{ item.id }}</td>
                <td>{{ item.fullname }}</td>
                <td>{{ item.department }}</td>
                <td>{{ item.tdate }}</td>
              </tr>
            </tbody>
          </table>
        </div>

        <!-- No data -->
        <div v-else-if="!loading" class="text-center text-muted py-4">
          ❌ ไม่มีข้อมูล
        </div>

      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const users = ref([])
const loading = ref(false)

const fetchData = async () => {
  loading.value = true
  try {
    const response = await fetch('http://localhost:5678/webhook/data')
    const data = await response.json()
    users.value = data
  } catch (error) {
    console.error('Error:', error)
  }
  loading.value = false
}

onMounted(() => {
  fetchData()
})
</script>

<style>
body {
  background-color: #f8f9fa;
}
</style>