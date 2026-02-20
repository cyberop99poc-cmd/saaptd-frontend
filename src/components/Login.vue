<template>
  <div class="w-full min-h-screen bg-gradient-to-br from-red-50 to-white flex items-center justify-center p-4">
    <!-- Animated background elements -->
    <div class="fixed inset-0 overflow-hidden pointer-events-none">
      <div
        class="absolute top-0 right-0 w-96 h-96 bg-gradient-to-br from-red-200/20 to-red-300/20 rounded-full blur-3xl animate-pulse">
      </div>
      <div
        class="absolute bottom-0 left-0 w-96 h-96 bg-gradient-to-tr from-red-200/20 to-red-300/20 rounded-full blur-3xl animate-pulse"
        style="animation-delay: 2s"></div>
    </div>

    <div class="relative z-10 w-full max-w-md">
      <div class="bg-white rounded-2xl shadow-2xl border border-red-100/50 overflow-hidden backdrop-blur-xl">
        <!-- Header -->
        <div class="bg-gradient-to-r from-red-600 to-red-700 px-8 py-8 text-center">
          <div
            class="w-16 h-16 rounded-full bg-white/20 backdrop-blur flex items-center justify-center mx-auto mb-4 shadow-lg">
            <svg class="w-8 h-8 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
          </div>
          <h1 class="text-3xl font-bold text-white">SAAPTD</h1>
          <p class="text-red-100 text-sm mt-2">Sistem Automasi Audit Pengurusan Tentera Darat</p>
          <p class="text-red-200 text-xs mt-1 font-medium">
            {{ isLogin ? 'Log Masuk Sistem' : 'Daftar Akaun Baharu' }}
          </p>
        </div>

        <!-- Form -->
        <div class="px-8 py-8 space-y-5">

          <!-- Username -->
          <div>
            <label class="block text-sm font-semibold text-gray-700 mb-2">
              Username <span class="text-red-500">*</span>
            </label>
            <input v-model="form.username" type="text" placeholder="Masukkan username" autocomplete="username"
              class="w-full px-4 py-3 border border-red-200/50 rounded-xl focus:ring-2 focus:ring-red-500 focus:border-transparent outline-none transition-all bg-white/50 backdrop-blur"
              :class="{ 'border-red-400 bg-red-50': fieldErrors.username }">
            <p v-if="fieldErrors.username" class="mt-1 text-xs text-red-600">{{ fieldErrors.username }}</p>
          </div>

          <!-- Password -->
          <div>
            <label class="block text-sm font-semibold text-gray-700 mb-2">
              Kata Laluan <span class="text-red-500">*</span>
            </label>
            <div class="relative">
              <input v-model="form.password" :type="showPassword ? 'text' : 'password'"
                placeholder="Masukkan kata laluan" autocomplete="current-password"
                @keyup.enter="!isLogin ? undefined : handleSubmit()"
                class="w-full px-4 py-3 pr-11 border border-red-200/50 rounded-xl focus:ring-2 focus:ring-red-500 focus:border-transparent outline-none transition-all bg-white/50 backdrop-blur"
                :class="{ 'border-red-400 bg-red-50': fieldErrors.password }">
              <button type="button" @click="showPassword = !showPassword"
                class="absolute right-3 top-1/2 -translate-y-1/2 text-gray-400 hover:text-gray-600 transition-colors"
                tabindex="-1">
                <svg v-if="!showPassword" class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M15 12a3 3 0 11-6 0 3 3 0 016 0zM2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z" />
                </svg>
                <svg v-else class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.543-7a9.97 9.97 0 011.563-3.029m5.858.908a3 3 0 114.243 4.243M9.878 9.878l4.242 4.242M9.88 9.88l-3.29-3.29m7.532 7.532l3.29 3.29M3 3l3.59 3.59m0 0A9.953 9.953 0 0112 5c4.478 0 8.268 2.943 9.543 7a10.025 10.025 0 01-4.132 5.411m0 0L21 21" />
                </svg>
              </button>
            </div>
            <p v-if="fieldErrors.password" class="mt-1 text-xs text-red-600">{{ fieldErrors.password }}</p>
          </div>

          <!-- ── Registration-only fields ── -->
          <template v-if="!isLogin">

            <!-- Full Name -->
            <div>
              <label class="block text-sm font-semibold text-gray-700 mb-2">Nama Penuh</label>
              <input v-model="form.full_name" type="text" placeholder="Masukkan nama penuh (pilihan)"
                class="w-full px-4 py-3 border border-red-200/50 rounded-xl focus:ring-2 focus:ring-red-500 focus:border-transparent outline-none transition-all bg-white/50 backdrop-blur">
            </div>

            <!-- Role -->
            <div>
              <label class="block text-sm font-semibold text-gray-700 mb-2">
                Peranan (Role) <span class="text-red-500">*</span>
              </label>
              <select v-model="form.role"
                class="w-full px-4 py-3 border border-red-200/50 rounded-xl focus:ring-2 focus:ring-red-500 focus:border-transparent outline-none transition-all bg-white/50 backdrop-blur appearance-none cursor-pointer"
                :class="{ 'border-red-400 bg-red-50': fieldErrors.role }">
                <option value="" disabled>-- Pilih Peranan --</option>
                <option value="Superadmin">Superadmin</option>
                <option value="Admin">Admin</option>
                <option value="Auditor">Auditor</option>
                <option value="Entity">Entity</option>
              </select>
              <p v-if="fieldErrors.role" class="mt-1 text-xs text-red-600">{{ fieldErrors.role }}</p>
              <!-- Role description hint -->
              <p v-if="form.role" class="mt-1 text-xs text-gray-500">{{ roleHint }}</p>
            </div>

            <!-- Entity Name (only for Entity role) -->
            <div v-if="form.role === 'Entity'">
              <label class="block text-sm font-semibold text-gray-700 mb-2">
                Nama Entiti <span class="text-red-500">*</span>
              </label>
              <input v-model="form.entity_name" type="text" placeholder="Masukkan nama entiti / unit"
                @keyup.enter="handleSubmit"
                class="w-full px-4 py-3 border border-red-200/50 rounded-xl focus:ring-2 focus:ring-red-500 focus:border-transparent outline-none transition-all bg-white/50 backdrop-blur"
                :class="{ 'border-red-400 bg-red-50': fieldErrors.entity_name }">
              <p v-if="fieldErrors.entity_name" class="mt-1 text-xs text-red-600">{{ fieldErrors.entity_name }}</p>
            </div>

          </template>
          <!-- ── End registration fields ── -->

          <!-- Error message -->
          <div v-if="errorMessage"
            class="px-4 py-3 bg-red-50 border border-red-200 rounded-xl text-sm text-red-700 flex items-start gap-2">
            <svg class="w-4 h-4 mt-0.5 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
            </svg>
            <span>{{ errorMessage }}</span>
          </div>

          <!-- Remember me (login only) -->
          <div v-if="isLogin" class="flex items-center gap-2">
            <input v-model="rememberMe" type="checkbox" id="remember"
              class="w-4 h-4 cursor-pointer text-red-600 focus:ring-red-500 rounded">
            <label for="remember" class="text-sm text-gray-600 cursor-pointer">Ingat saya</label>
          </div>

          <!-- Submit button -->
          <button @click="handleSubmit" :disabled="isLoading"
            class="w-full px-6 py-3 bg-gradient-to-r from-red-600 to-red-700 text-white font-semibold rounded-xl hover:shadow-lg transition-all shadow-md disabled:opacity-50 disabled:cursor-not-allowed flex items-center justify-center gap-2">
            <svg v-if="!isLoading" class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M11 16l-4-4m0 0l4-4m-4 4h14m-5 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h7a3 3 0 013 3v1" />
            </svg>
            <svg v-if="isLoading" class="w-5 h-5 animate-spin" fill="none" viewBox="0 0 24 24">
              <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" />
              <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v8H4z" />
            </svg>
            <span>{{ isLoading ? (isLogin ? 'Sedang log masuk...' : 'Sedang mendaftar...') : (isLogin ? 'Log Masuk' :
              'Daftar')
              }}</span>
          </button>

          <!-- Toggle login / register -->
          <div class="relative py-2">
            <div class="absolute inset-0 flex items-center">
              <div class="w-full border-t border-gray-200"></div>
            </div>
            <div class="relative flex justify-center text-sm">
              <span class="px-2 bg-white text-gray-500">atau</span>
            </div>
          </div>

          <button @click="toggleMode" :disabled="isLoading"
            class="w-full px-6 py-3 border-2 border-red-600 text-red-600 font-semibold rounded-xl hover:bg-red-50 transition-all flex items-center justify-center gap-2 disabled:opacity-50 disabled:cursor-not-allowed">
            <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M18 9v3m0 0v3m0-3h3m-3 0h-3m-2-5a4 4 0 11-8 0 4 4 0 018 0zM3 20a6 6 0 0112 0v1H3v-1z" />
            </svg>
            <span>{{ isLogin ? 'Buat Akaun Baharu' : 'Kembali ke Log Masuk' }}</span>
          </button>

        </div>
      </div>

      <p class="text-center text-xs text-gray-500 mt-6">© 2026 SAAPTD. All rights reserved.</p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { loginUser, registerUser } from '../services/api'
import type { RegisterPayload } from '../services/api'

// ── Valid roles as defined by the backend authController ──────────
const VALID_ROLES = ['Superadmin', 'Admin', 'Auditor', 'Entity'] as const
type UserRole = typeof VALID_ROLES[number]

const router = useRouter()
const isLogin = ref(true)
const rememberMe = ref(false)
const isLoading = ref(false)
const errorMessage = ref('')
const showPassword = ref(false)

const form = ref({
  username: '',
  password: '',
  full_name: '',
  role: '' as UserRole | '',
  entity_name: '',
})

// Per-field validation errors
const fieldErrors = ref({
  username: '',
  password: '',
  role: '',
  entity_name: '',
})

// Role description hints matching backend roles
const roleHint = computed(() => {
  const hints: Record<UserRole, string> = {
    Superadmin: 'Akses penuh sistem — pentadbir utama.',
    Admin: 'Urus pengguna dan konfigurasi sistem.',
    Auditor: 'Jalankan audit dan isi borang penilaian.',
    Entity: 'Entiti yang diaudit — lihat laporan berkaitan entiti sahaja.',
  }
  return form.value.role ? hints[form.value.role as UserRole] : ''
})

// ── Helpers ───────────────────────────────────────────────────────

function clearErrors() {
  errorMessage.value = ''
  fieldErrors.value = { username: '', password: '', role: '', entity_name: '' }
}

function toggleMode() {
  isLogin.value = !isLogin.value
  clearErrors()
  form.value = { username: '', password: '', full_name: '', role: '', entity_name: '' }
  showPassword.value = false
}

/** Validate role against backend enum — returns error string or '' */
function validateRole(role: string): string {
  if (!role) return 'Sila pilih peranan pengguna.'
  if (!VALID_ROLES.includes(role as UserRole)) {
    return `Peranan tidak sah. Pilih salah satu: ${VALID_ROLES.join(', ')}`
  }
  return ''
}

/** Client-side validation before calling the API */
function validateForm(): boolean {
  clearErrors()
  let valid = true

  if (!form.value.username.trim()) {
    fieldErrors.value.username = 'Username diperlukan.'
    valid = false
  }
  if (!form.value.password) {
    fieldErrors.value.password = 'Kata laluan diperlukan.'
    valid = false
  }

  if (!isLogin.value) {
    const roleError = validateRole(form.value.role)
    if (roleError) {
      fieldErrors.value.role = roleError
      valid = false
    }
    if (form.value.role === 'Entity' && !form.value.entity_name.trim()) {
      fieldErrors.value.entity_name = 'Nama entiti diperlukan untuk peranan Entity.'
      valid = false
    }
  }

  return valid
}

/** Redirect to the appropriate page based on user role */
function redirectByRole(role: string) {
  switch (role as UserRole) {
    case 'Superadmin':
    case 'Admin':
      router.push('/dashboard')
      break
    case 'Auditor':
      router.push('/dashboard')
      break
    case 'Entity':
      router.push('/dashboard')
      break
    default:
      // Unknown role — send to dashboard but log a warning
      console.warn(`[Login] Unknown role received from server: "${role}". Defaulting to /dashboard.`)
      router.push('/dashboard')
  }
}

// ── Main submit handler ───────────────────────────────────────────
const handleSubmit = async () => {
  if (!validateForm()) return

  isLoading.value = true

  try {
    if (isLogin.value) {
      // ── LOGIN ──────────────────────────────────────────────────
      const result = await loginUser(form.value.username.trim(), form.value.password)

      if (result.success && result.data) {
        const { token, user } = result.data

        // Validate the role received from the server
        if (!VALID_ROLES.includes(user.role as UserRole)) {
          errorMessage.value = `Peranan pengguna tidak dikenali: "${user.role}". Sila hubungi pentadbir.`
          return
        }

        // Persist session info
        localStorage.setItem('isAuthenticated', 'true')
        localStorage.setItem('authToken', token)
        localStorage.setItem('currentUser', user.username)
        localStorage.setItem('userRole', user.role)
        localStorage.setItem('userFullName', user.full_name ?? user.username)
        localStorage.setItem('userId', String(user.id))
        if (user.entity_name) {
          localStorage.setItem('entityName', user.entity_name)
        }

        redirectByRole(user.role)
      } else {
        errorMessage.value = result.error || 'Log masuk gagal. Sila semak username dan kata laluan.'
      }
    } else {
      // ── REGISTER ───────────────────────────────────────────────
      const payload: RegisterPayload = {
        username: form.value.username.trim(),
        password: form.value.password,
        role: form.value.role as UserRole,
      }
      if (form.value.full_name.trim()) payload.full_name = form.value.full_name.trim()
      if (form.value.entity_name.trim()) payload.entity_name = form.value.entity_name.trim()

      const result = await registerUser(payload)

      if (result.success) {
        // Auto-login after registration
        const loginResult = await loginUser(form.value.username.trim(), form.value.password)
        if (loginResult.success && loginResult.data) {
          const { token, user } = loginResult.data
          localStorage.setItem('isAuthenticated', 'true')
          localStorage.setItem('authToken', token)
          localStorage.setItem('currentUser', user.username)
          localStorage.setItem('userRole', user.role)
          localStorage.setItem('userFullName', user.full_name ?? user.username)
          localStorage.setItem('userId', String(user.id))
          if (user.entity_name) {
            localStorage.setItem('entityName', user.entity_name)
          }
          redirectByRole(user.role)
        } else {
          // Registration succeeded but auto-login failed — send to login page
          isLogin.value = true
          errorMessage.value = 'Akaun berjaya dibuat. Sila log masuk.'
        }
      } else {
        errorMessage.value = result.error || 'Pendaftaran gagal. Sila cuba lagi.'
      }
    }
  } catch (err: any) {
    errorMessage.value = err.message || 'Ralat berlaku. Sila cuba lagi.'
  } finally {
    isLoading.value = false
  }
}
</script>

<style scoped>
select option {
  background-color: white;
  color: #374151;
}
</style>
