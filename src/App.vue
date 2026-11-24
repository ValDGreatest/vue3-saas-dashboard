<script setup>
import { ref, computed, onMounted } from 'vue'
import { defineStore } from 'pinia'
import { 
  Shield, 
  Users, 
  DollarSign, 
  TrendingUp, 
  AlertCircle, 
  Trash2, 
  Search, 
  Key, 
  Sparkles, 
  CheckCircle2, 
  LayoutDashboard 
} from 'lucide-vue-next'

// --- 1. PINIA STORE (State Management & RBAC) ---
const useAuthStore = defineStore('auth', {
  state: () => ({
    role: 'Admin', // Roles: 'Admin' | 'Editor' | 'Viewer'
    user: {
      name: 'Valentine O.',
      email: 'val@saasflow.io',
      plan: 'Enterprise SaaS'
    }
  }),
  actions: {
    setRole(newRole) {
      this.role = newRole
    }
  }
})

const auth = useAuthStore()

// Role capabilities
const isAdmin = computed(() => auth.role === 'Admin')
const canDelete = computed(() => auth.role === 'Admin')
const isViewer = computed(() => auth.role === 'Viewer')

// --- 2. LIVE REST API & SEARCH FILTER ---
const products = ref([])
const loading = ref(true)
const searchQuery = ref('')
const selectedCategory = ref('All')

onMounted(async () => {
  try {
    const res = await fetch('https://dummyjson.com/products?limit=8')
    const data = await res.json()
    products.value = data.products
  } catch (err) {
    console.error('Failed to load products:', err)
  } finally {
    loading.value = false
  }
})

const categories = computed(() => {
  const cats = ['All', ...new Set(products.value.map(p => p.category))]
  return cats
})

const filteredProducts = computed(() => {
  return products.value.filter(item => {
    const matchesSearch = item.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
                          item.category.toLowerCase().includes(searchQuery.value.toLowerCase())
    const matchesCategory = selectedCategory.value === 'All' || item.category === selectedCategory.value
    return matchesSearch && matchesCategory
  })
})

function deleteItem(id) {
  if (!isAdmin.value) {
    alert('Permission Denied: Only Admin users can delete SaaS inventory records.')
    return
  }
  products.value = products.value.filter(p => p.id !== id)
}
</script>

<template>
  <div class="dashboard-wrapper">
    <!-- Top Navigation Header -->
    <header class="top-nav">
      <div class="brand">
        <div class="brand-icon">
          <LayoutDashboard :size="20" color="#ffffff" />
        </div>
        <div class="brand-text">
          <h1>SaaSFlow Metrics</h1>
          <span class="badge-version">Vue 3 + Pinia</span>
        </div>
      </div>

      <!-- Role Switcher -->
      <div class="role-selector">
        <span class="role-label">Active Role:</span>
        <div class="role-pills">
          <button 
            v-for="r in ['Admin', 'Editor', 'Viewer']" 
            :key="r"
            @click="auth.setRole(r)"
            :class="['role-btn', { active: auth.role === r }]"
          >
            {{ r }}
          </button>
        </div>
      </div>
    </header>

    <!-- Main Container -->
    <main class="dashboard-content">
      <!-- Role Notification Banner -->
      <div v-if="!isAdmin" class="role-alert">
        <AlertCircle :size="18" />
        <span>
          Viewing dashboard in <strong>{{ auth.role }} mode</strong>. Destructive actions (deletions) and secret API keys are locked.
        </span>
      </div>

      <!-- SaaS KPI Metric Cards -->
      <section class="kpi-grid">
        <div class="kpi-card">
          <div class="kpi-header">
            <span>Monthly Recurring Revenue</span>
            <DollarSign :size="18" class="icon-indigo" />
          </div>
          <div class="kpi-value">$48,500</div>
          <div class="kpi-trend positive">
            <TrendingUp :size="14" /> +14.2% vs last month
          </div>
        </div>

        <div class="kpi-card">
          <div class="kpi-header">
            <span>Active Subscriptions</span>
            <Users :size="18" class="icon-indigo" />
          </div>
          <div class="kpi-value">1,420</div>
          <div class="kpi-trend positive">
            <TrendingUp :size="14" /> +8.5% new signups
          </div>
        </div>

        <div class="kpi-card">
          <div class="kpi-header">
            <span>Net Revenue Churn</span>
            <CheckCircle2 :size="18" class="icon-emerald" />
          </div>
          <div class="kpi-value">0.82%</div>
          <div class="kpi-trend neutral">
            Target benchmark: &lt; 1.5%
          </div>
        </div>
      </section>

      <!-- Revenue Chart Simulation -->
      <section class="chart-section">
        <div class="section-title">
          <h3>MRR Revenue Velocity</h3>
          <span class="subtext">Real-time reactive telemetry</span>
        </div>
        <div class="bars-container">
          <div class="bar-group" v-for="(val, idx) in [45, 62, 58, 75, 88, 100]" :key="idx">
            <div class="bar-fill" :style="{ height: val + '%' }"></div>
            <span class="bar-label">{{ ['Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov'][idx] }}</span>
          </div>
        </div>
      </section>

      <!-- Admin-Only Secret API Config -->
      <section v-if="isAdmin" class="admin-panel">
        <div class="admin-header">
          <Key :size="18" class="icon-indigo" />
          <h3>Admin Credentials & Webhook Tokens</h3>
        </div>
        <div class="api-box">
          <code>sk_live_948201830194810293810239102</code>
          <button class="btn-rotate">Rotate Secret</button>
        </div>
      </section>

      <!-- REST API Live Catalog & Filter Table -->
      <section class="table-section">
        <div class="table-controls">
          <div>
            <h3>Production Inventory & Billing Catalog</h3>
            <p class="subtext">Live asynchronous REST data with reactive filtering</p>
          </div>

          <div class="filter-controls">
            <!-- Search Input -->
            <div class="search-box">
              <Search :size="16" class="search-icon" />
              <input 
                v-model="searchQuery" 
                type="text" 
                placeholder="Filter titles or categories..." 
              />
            </div>

            <!-- Category Filter -->
            <select v-model="selectedCategory" class="category-select">
              <option v-for="c in categories" :key="c" :value="c">{{ c }}</option>
            </select>
          </div>
        </div>

        <!-- Table -->
        <div class="table-card">
          <div v-if="loading" class="loading-state">
            <Sparkles :size="20" class="spin" />
            <span>Fetching live API records...</span>
          </div>

          <table v-else class="data-table">
            <thead>
              <tr>
                <th>Item Title</th>
                <th>Category</th>
                <th>Price</th>
                <th>Stock</th>
                <th class="text-right">Actions</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in filteredProducts" :key="item.id">
                <td class="item-title">{{ item.title }}</td>
                <td><span class="badge-cat">{{ item.category }}</span></td>
                <td class="item-price">${{ item.price }}</td>
                <td>
                  <span :class="['stock-badge', item.stock < 10 ? 'low' : 'ok']">
                    {{ item.stock }} in stock
                  </span>
                </td>
                <td class="text-right">
                  <button 
                    @click="deleteItem(item.id)" 
                    :disabled="!canDelete"
                    :class="['btn-delete', { disabled: !canDelete }]"
                    :title="canDelete ? 'Delete record' : 'Admin access required'"
                  >
                    <Trash2 :size="15" />
                  </button>
                </td>
              </tr>
              <tr v-if="filteredProducts.length === 0">
                <td colspan="5" class="empty-state">No matching records found.</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
/* Modern Sleek Dark Mode SaaS Aesthetic */
.dashboard-wrapper {
  min-height: 100vh;
  background-color: #090d16;
  color: #f1f5f9;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  padding-bottom: 40px;
}

.top-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 16px 32px;
  background-color: #0f172a;
  border-bottom: 1px solid #1e293b;
}

.brand {
  display: flex;
  align-items: center;
  gap: 12px;
}

.brand-icon {
  width: 38px;
  height: 38px;
  background: linear-gradient(135deg, #6366f1, #4f46e5);
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.3);
}

.brand-text h1 {
  font-size: 1.15rem;
  font-weight: 700;
  margin: 0;
  color: #ffffff;
}

.badge-version {
  font-size: 0.72rem;
  color: #818cf8;
  background-color: rgba(99, 102, 241, 0.15);
  padding: 2px 8px;
  border-radius: 9999px;
  font-weight: 600;
}

.role-selector {
  display: flex;
  align-items: center;
  gap: 10px;
}

.role-label {
  font-size: 0.8rem;
  color: #94a3b8;
}

.role-pills {
  display: flex;
  background-color: #1e293b;
  border-radius: 8px;
  padding: 3px;
  border: 1px solid #334155;
}

.role-btn {
  background: transparent;
  border: none;
  color: #94a3b8;
  padding: 6px 14px;
  font-size: 0.78rem;
  font-weight: 600;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
}

.role-btn.active {
  background-color: #6366f1;
  color: #ffffff;
  box-shadow: 0 2px 8px rgba(99, 102, 241, 0.4);
}

.dashboard-content {
  max-width: 1100px;
  margin: 28px auto 0 auto;
  padding: 0 24px;
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.role-alert {
  display: flex;
  align-items: center;
  gap: 12px;
  background-color: rgba(245, 158, 11, 0.1);
  border: 1px solid rgba(245, 158, 11, 0.3);
  color: #fbbf24;
  padding: 12px 18px;
  border-radius: 10px;
  font-size: 0.85rem;
}

.kpi-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(260px, 1fr));
  gap: 18px;
}

.kpi-card {
  background-color: #111827;
  border: 1px solid #1f2937;
  padding: 20px;
  border-radius: 12px;
}

.kpi-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #9ca3af;
  font-size: 0.85rem;
  font-weight: 500;
}

.kpi-value {
  font-size: 1.85rem;
  font-weight: 800;
  color: #ffffff;
  margin: 10px 0 6px 0;
}

.kpi-trend {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 0.78rem;
  font-weight: 600;
}

.kpi-trend.positive { color: #34d399; }
.kpi-trend.neutral { color: #94a3b8; }
.icon-indigo { color: #818cf8; }
.icon-emerald { color: #34d399; }

.chart-section {
  background-color: #111827;
  border: 1px solid #1f2937;
  border-radius: 12px;
  padding: 22px;
}

.section-title h3 {
  font-size: 1rem;
  font-weight: 700;
  margin: 0;
  color: #ffffff;
}

.subtext {
  font-size: 0.78rem;
  color: #64748b;
  margin: 4px 0 0 0;
}

.bars-container {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  height: 140px;
  padding-top: 24px;
}

.bar-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  width: 12%;
  height: 100%;
  justify-content: flex-end;
}

.bar-fill {
  width: 100%;
  background: linear-gradient(180deg, #6366f1, #312e81);
  border-radius: 6px 6px 0 0;
  transition: height 0.4s ease;
}

.bar-label {
  font-size: 0.72rem;
  color: #64748b;
}

.admin-panel {
  background-color: #0f172a;
  border: 1px dashed #4f46e5;
  border-radius: 12px;
  padding: 18px 22px;
}

.admin-header {
  display: flex;
  align-items: center;
  gap: 8px;
  color: #818cf8;
  font-size: 0.9rem;
  font-weight: 600;
  margin-bottom: 12px;
}

.admin-header h3 {
  margin: 0;
}

.api-box {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background-color: #090d16;
  border: 1px solid #1e293b;
  padding: 10px 14px;
  border-radius: 8px;
}

.api-box code {
  color: #a5b4fc;
  font-family: monospace;
  font-size: 0.85rem;
}

.btn-rotate {
  background-color: #4f46e5;
  color: #ffffff;
  border: none;
  padding: 6px 12px;
  border-radius: 6px;
  font-size: 0.78rem;
  font-weight: 600;
  cursor: pointer;
}

.table-section {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.table-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 12px;
}

.table-controls h3 {
  margin: 0;
  font-size: 1rem;
  font-weight: 700;
}

.filter-controls {
  display: flex;
  gap: 10px;
}

.search-box {
  position: relative;
  display: flex;
  align-items: center;
}

.search-icon {
  position: absolute;
  left: 10px;
  color: #64748b;
}

.search-box input {
  background-color: #111827;
  border: 1px solid #1f2937;
  border-radius: 8px;
  padding: 8px 12px 8px 32px;
  font-size: 0.82rem;
  color: #ffffff;
  outline: none;
  width: 220px;
}

.search-box input:focus {
  border-color: #6366f1;
}

.category-select {
  background-color: #111827;
  border: 1px solid #1f2937;
  border-radius: 8px;
  padding: 8px 12px;
  color: #f1f5f9;
  font-size: 0.82rem;
  outline: none;
}

.table-card {
  background-color: #111827;
  border: 1px solid #1f2937;
  border-radius: 12px;
  overflow: hidden;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  text-align: left;
  font-size: 0.82rem;
}

.data-table thead {
  background-color: #0c121e;
  border-bottom: 1px solid #1f2937;
  color: #94a3b8;
  font-size: 0.75rem;
  text-transform: uppercase;
}

.data-table th, .data-table td {
  padding: 14px 18px;
}

.data-table tbody tr {
  border-bottom: 1px solid #172033;
}

.data-table tbody tr:hover {
  background-color: rgba(99, 102, 241, 0.04);
}

.item-title {
  font-weight: 600;
  color: #ffffff;
}

.badge-cat {
  background-color: #1e293b;
  color: #cbd5e1;
  padding: 3px 8px;
  border-radius: 6px;
  font-size: 0.72rem;
}

.item-price {
  font-family: monospace;
  color: #34d399;
  font-weight: 600;
}

.stock-badge {
  font-size: 0.72rem;
  font-weight: 600;
}

.stock-badge.ok { color: #94a3b8; }
.stock-badge.low { color: #f59e0b; }

.btn-delete {
  background: transparent;
  border: 1px solid #ef4444;
  color: #ef4444;
  padding: 6px 8px;
  border-radius: 6px;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-delete:hover:not(.disabled) {
  background-color: rgba(239, 68, 68, 0.15);
}

.btn-delete.disabled {
  opacity: 0.3;
  cursor: not-allowed;
  border-color: #64748b;
  color: #64748b;
}

.text-right { text-align: right; }

.loading-state, .empty-state {
  text-align: center;
  padding: 36px;
  color: #64748b;
}
</style>