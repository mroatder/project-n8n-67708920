<template>
    <div class="container py-5">
      <div class="toast-container position-fixed bottom-0 end-0 p-3" style="z-index: 2000">
        <div v-if="toast.show" :class="['toast show align-items-center text-white border-0 shadow-lg', toast.type === 'success' ? 'bg-success' : 'bg-danger']" role="alert">
          <div class="d-flex">
            <div class="toast-body">
              <span v-if="toast.type === 'success'">✅</span>
              <span v-else>❌</span>
              {{ toast.message }}
            </div>
            <button type="button" class="btn-close btn-close-white me-2 m-auto" @click="toast.show = false"></button>
          </div>
        </div>
      </div>
  
      <div class="text-center mb-5">
        <h2 class="fw-bold text-primary">📦 ระบบสต็อกสินค้า (Bootstrap 5)</h2>
        <p class="text-muted">จัดการข้อมูล Google Sheets ผ่าน n8n Webhook</p>
      </div>
  
      <div class="card shadow border-0 rounded-4">
        <div class="card-body p-4">
          <div class="d-flex justify-content-between align-items-center mb-4">
            <h5 class="mb-0 fw-bold text-secondary">รายการสินค้าคงคลัง</h5>
            <div class="d-flex gap-2">
              <button class="btn btn-success px-4 rounded-pill shadow-sm" @click="openModal">
                ➕ เพิ่มสินค้าใหม่
              </button>
              <button class="btn btn-outline-primary px-4 rounded-pill" @click="fetchData" :disabled="loading">
                <span v-if="loading" class="spinner-border spinner-border-sm me-1"></span>
                🔄 อัปเดตตาราง
              </button>
            </div>
          </div>
  
          <div class="table-responsive" v-if="!loading && products.length > 0">
            <table class="table table-hover align-middle">
              <thead class="table-light text-secondary">
                <tr>
                  <th class="py-3">รหัสสินค้า</th>
                  <th class="py-3">ชื่อสินค้า</th>
                  <th class="py-3 text-center">จำนวน</th>
                  <th class="py-3 text-end">ราคา/หน่วย</th>
                  <th class="py-3 text-end">รวมเป็นเงิน</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(item, index) in products" :key="index">
                  <td><span class="badge bg-light text-dark border fw-bold">{{ item['รหัสสินค้า'] }}</span></td>
                  <td class="fw-bold">{{ item['ชื่อสินค้า'] }}</td>
                  <td class="text-center">
                    <span :class="['badge rounded-pill px-3', Number(item['จำนวน']) < 15 ? 'bg-warning text-dark' : 'bg-success']">
                      {{ item['จำนวน'] }}
                    </span>
                  </td>
                  <td class="text-end">{{ Number(item['ราคา']).toLocaleString() }} ฿</td>
                  <td class="text-end fw-bold text-primary">
                    {{ (Number(item['จำนวน']) * Number(item['ราคา'])).toLocaleString() }} ฿
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
  
          <div v-if="loading" class="text-center py-5">
            <div class="spinner-border text-primary"></div>
            <p class="mt-2 text-muted">กำลังดึงข้อมูลล่าสุด...</p>
          </div>
          <div v-else-if="products.length === 0" class="text-center py-5">
            <p class="text-muted">❌ ไม่พบข้อมูลในระบบ หรือ API ขัดข้อง</p>
          </div>
        </div>
      </div>
  
      <div v-if="isModalOpen" class="modal fade show" style="display: block; background: rgba(0,0,0,0.5);" tabindex="-1">
        <div class="modal-dialog modal-dialog-centered">
          <div class="modal-content border-0 shadow-lg rounded-4 overflow-hidden">
            <div class="modal-header bg-primary text-white border-0">
              <h5 class="modal-title fw-bold">➕ เพิ่มสินค้าใหม่เข้าระบบ</h5>
              <button type="button" class="btn-close btn-close-white" @click="closeModal"></button>
            </div>
            <div class="modal-body p-4">
              <form @submit.prevent="insertData">
                <div class="mb-3">
                  <label class="form-label fw-bold small text-muted">รหัสสินค้า</label>
                  <input v-model="form.id" type="text" class="form-control rounded-3" placeholder="เช่น P001" required>
                </div>
                <div class="mb-3">
                  <label class="form-label fw-bold small text-muted">ชื่อสินค้า</label>
                  <input v-model="form.name" type="text" class="form-control rounded-3" required>
                </div>
                <div class="row">
                  <div class="col-6 mb-3">
                    <label class="form-label fw-bold small text-muted">จำนวน</label>
                    <input v-model.number="form.amount" type="number" class="form-control rounded-3" required>
                  </div>
                  <div class="col-6 mb-3">
                    <label class="form-label fw-bold small text-muted">ราคา/หน่วย</label>
                    <input v-model.number="form.price" type="number" class="form-control rounded-3" required>
                  </div>
                </div>
                <div class="d-grid gap-2 mt-4">
                  <button type="submit" class="btn btn-primary btn-lg rounded-pill fw-bold" :disabled="inserting">
                    {{ inserting ? 'กำลังบันทึก...' : 'บันทึกข้อมูล' }}
                  </button>
                  <button type="button" class="btn btn-light rounded-pill" @click="closeModal">ยกเลิก</button>
                </div>
              </form>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted, reactive } from 'vue'
  
  // --- Data States ---
  const products = ref([])
  const loading = ref(false)
  const inserting = ref(false)
  const isModalOpen = ref(false)
  
  const form = ref({
    id: '',
    name: '',
    amount: 0,
    price: 0
  })
  
  // --- Popup (Toast) State ---
  const toast = reactive({
    show: false,
    message: '',
    type: 'success'
  })
  
  const showToast = (msg, type = 'success') => {
    toast.message = msg
    toast.type = type
    toast.show = true
    setTimeout(() => { toast.show = false }, 3000) // หายไปเองใน 3 วินาที
  }
  
  // --- Logic ---
  const openModal = () => { isModalOpen.value = true }
  const closeModal = () => { isModalOpen.value = false }
  
  const fetchData = async () => {
    loading.value = true
    try {
      const response = await fetch('http://localhost:5678/webhook/product')
      if (!response.ok) throw new Error('API Error')
      const data = await response.json()
      products.value = Array.isArray(data) ? data : [data]
    } catch (err) {
      showToast('ไม่สามารถดึงข้อมูลได้', 'danger')
    } finally {
      loading.value = false
    }
  }
  
  const insertData = async () => {
    inserting.value = true
    try {
      const response = await fetch('http://localhost:5678/webhook/product', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          "รหัสสินค้า": form.value.id,
          "ชื่อสินค้า": form.value.name,
          "จำนวน": form.value.amount,
          "ราคา": form.value.price
        })
      })
  
      if (response.ok) {
        showToast('บันทึกข้อมูลสำเร็จแล้ว!')
        form.value = { id: '', name: '', amount: 0, price: 0 }
        closeModal()
        fetchData()
      } else {
        throw new Error('บันทึกไม่สำเร็จ')
      }
    } catch (err) {
      showToast('เกิดข้อผิดพลาด: ' + err.message, 'danger')
    } finally {
      inserting.value = false
    }
  }
  
  onMounted(() => fetchData())
  </script>
  
  <style scoped>
  @import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;600;700&display=swap');
  
  body {
    background-color: #f8f9fa;
    font-family: 'Sarabun', sans-serif;
  }
  
  .toast {
    min-width: 250px;
  }
  
  /* Animation */
  .modal.fade.show {
    animation: fadeIn 0.2s ease-out;
  }
  
  @keyframes fadeIn {
    from { opacity: 0; transform: translateY(-10px); }
    to { opacity: 1; transform: translateY(0); }
  }
  </style>