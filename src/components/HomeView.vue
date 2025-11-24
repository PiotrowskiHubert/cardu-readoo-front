<script setup lang="ts">
import { computed, ref, onMounted, onBeforeUnmount } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthStore } from '@/stores/auth'
import ExpansionListView from '@/components/ExpansionListView.vue'
import CardSearchView from '@/components/CardSearchView.vue'
import OfferSearchView from '@/components/OfferSearchView.vue'

const auth = useAuthStore()
const router = useRouter()

const activeSection = ref<'Home' | 'ExpansionView' | 'CardSearch' | 'OfferSearch'>('Home')

const isAdmin = computed(() => auth.user?.roles.includes('ROLE_ADMIN'))

const isLoggedIn = computed(() => auth.isAuthenticated)

const isLoginModalOpen = ref(false)
const loginUsername = ref('')
const loginPassword = ref('')
const loginErrorMessage = ref('')
const isLoginSubmitting = ref(false)

const isUserMenuOpen = ref(false)

const isChangePasswordModalOpen = ref(false)
const oldPassword = ref('')
const newPassword = ref('')
const confirmNewPassword = ref('')
const changePasswordError = ref('')
const isChangePasswordSubmitting = ref(false)

const canSubmitChangePassword = computed(() => {
  return (
    oldPassword.value.trim().length > 0 &&
    newPassword.value.trim().length > 0 &&
    confirmNewPassword.value.trim().length > 0 &&
    newPassword.value === confirmNewPassword.value
  )
})

function openHome() {
  activeSection.value = 'Home'
}

function openExpansionView() {
  activeSection.value = 'ExpansionView'
}

function openCardSearch() {
  activeSection.value = 'CardSearch'
}

function openOfferSearch() {
  activeSection.value = 'OfferSearch'
}

async function handleLogout() {
  auth.logout()
  activeSection.value = 'Home'
  await router.push({ name: 'home' })
}

function openLoginModal() {
  loginErrorMessage.value = ''
  isLoginModalOpen.value = true
}

function closeLoginModal() {
  if (isLoginSubmitting.value) return
  isLoginModalOpen.value = false
  loginUsername.value = ''
  loginPassword.value = ''
  loginErrorMessage.value = ''
}

async function handleLoginSubmit() {
  loginErrorMessage.value = ''
  isLoginSubmitting.value = true
  try {
    await auth.login(loginUsername.value, loginPassword.value)
    isLoginModalOpen.value = false
    loginUsername.value = ''
    loginPassword.value = ''
    isUserMenuOpen.value = false
    await router.push({ name: 'home' })
  } catch {
    loginErrorMessage.value = 'Invalid username or password'
  } finally {
    isLoginSubmitting.value = false
  }
}

function toggleUserMenu() {
  isUserMenuOpen.value = !isUserMenuOpen.value
}

function openChangePasswordModal() {
  isUserMenuOpen.value = false
  changePasswordError.value = ''
  oldPassword.value = ''
  newPassword.value = ''
  confirmNewPassword.value = ''
  isChangePasswordModalOpen.value = true
}

function closeChangePasswordModal() {
  if (isChangePasswordSubmitting.value) return
  isChangePasswordModalOpen.value = false
  oldPassword.value = ''
  newPassword.value = ''
  confirmNewPassword.value = ''
  changePasswordError.value = ''
}

async function handleChangePasswordSubmit() {
  changePasswordError.value = ''

  if (!canSubmitChangePassword.value) {
    changePasswordError.value = 'Fill in both new password fields with identical values.'
    return
  }

  isChangePasswordSubmitting.value = true
  try {
    await auth.resetPassword(oldPassword.value, newPassword.value)

    isChangePasswordModalOpen.value = false
    oldPassword.value = ''
    newPassword.value = ''
    confirmNewPassword.value = ''
    changePasswordError.value = ''
  } catch {
    changePasswordError.value = 'Failed to change password. Please try again.'
  } finally {
    isChangePasswordSubmitting.value = false
  }
}

const isMobileMenuOpen = ref(false)
const isMobile = ref(false)

function handleResize() {
  isMobile.value = window.innerWidth < 768
  if (!isMobile.value) {
    isMobileMenuOpen.value = false
  }
}

onMounted(() => {
  handleResize()
  window.addEventListener('resize', handleResize)
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', handleResize)
})

function toggleMobileMenu() {
  isMobileMenuOpen.value = !isMobileMenuOpen.value
}

function navigateAndCloseMobile(section: 'Home' | 'ExpansionView' | 'CardSearch' | 'OfferSearch') {
  activeSection.value = section
  isMobileMenuOpen.value = false
}
</script>

<template>
  <div class="min-h-screen flex flex-col bg-slate-100 relative">
    <!-- Top navigation bar -->
    <header class="w-full bg-transparent print:hidden">
      <div class="max-w-6xl mx-auto px-3 sm:px-4 pt-3">
        <div class="bg-slate-900 text-white shadow-md rounded-xl sm:rounded-2xl px-3 sm:px-4 py-2.5 sm:py-3 flex items-center justify-between gap-3">
          <!-- Logo / application name -->
          <div class="flex items-center gap-2">
            <span class="text-sm sm:text-base font-semibold tracking-tight">Cardu Readoo</span>
          </div>

          <!-- Desktop menu -->
          <nav class="hidden md:flex items-center gap-4">
            <button
              type="button"
              class="relative text-sm font-medium hover:text-blue-300"
              :class="{ 'text-blue-300': activeSection === 'Home' }"
              @click="openHome"
            >
              Home
            </button>

            <button
              type="button"
              class="relative text-sm font-medium hover:text-blue-300"
              :class="{ 'text-blue-300': activeSection === 'ExpansionView' }"
              @click="openExpansionView"
            >
              Expansion
            </button>

            <button
              type="button"
              class="relative text-sm font-medium hover:text-blue-300"
              :class="{ 'text-blue-300': activeSection === 'CardSearch' }"
              @click="openCardSearch"
            >
              Card
            </button>

            <button
              type="button"
              class="relative text-sm font-medium hover:text-blue-300"
              :class="{ 'text-blue-300': activeSection === 'OfferSearch' }"
              @click="openOfferSearch"
            >
              Offer
            </button>
          </nav>

          <!-- Right side: user / login -->
          <div class="relative flex items-center gap-2 text-xs sm:text-sm">
            <template v-if="isLoggedIn">
              <button
                type="button"
                class="inline-flex items-center gap-1 rounded-lg border border-slate-500 px-2 sm:px-3 py-1 font-medium hover:bg-slate-800"
                @click="toggleUserMenu"
              >
                <span>Hello, {{ auth.user?.username }}</span>
                <span class="text-[10px]">
                  ▼
                </span>
              </button>
              <button
                type="button"
                class="hidden sm:inline-flex rounded-lg border border-slate-500 px-3 py-1 font-medium hover:bg-slate-800"
                @click="handleLogout"
              >
                Log out
              </button>

              <div
                v-if="isUserMenuOpen"
                class="absolute right-0 top-full mt-1 w-44 rounded-lg border border-slate-700 bg-slate-900 text-xs shadow-lg z-20"
              >
                <button
                  type="button"
                  class="block w-full text-left px-3 py-2 hover:bg-slate-800"
                  @click="openChangePasswordModal"
                >
                  Change password
                </button>
                <button
                  type="button"
                  class="block w-full text-left px-3 py-2 hover:bg-slate-800 md:hidden"
                  @click="handleLogout"
                >
                  Log out
                </button>
              </div>
            </template>
            <template v-else>
              <button
                type="button"
                class="rounded-lg border border-slate-500 px-2 sm:px-3 py-1 font-medium hover:bg-slate-800"
                @click="openLoginModal"
              >
                Log in
              </button>
            </template>

            <!-- Hamburger button only on mobile -->
            <button
              type="button"
              class="md:hidden inline-flex items-center justify-center rounded border border-slate-600 p-1 ml-1 bg-slate-800/80"
              @click="toggleMobileMenu"
            >
              <span class="sr-only">Toggle navigation</span>
              <span v-if="!isMobileMenuOpen" class="text-lg leading-none">☰</span>
              <span v-else class="text-lg leading-none">✕</span>
            </button>
          </div>
        </div>

        <!-- Mobile dropdown menu, visually attached to rounded bar -->
        <nav
          v-if="isMobileMenuOpen"
          class="md:hidden mt-1 rounded-xl sm:rounded-2xl bg-slate-900 text-white shadow-md overflow-hidden"
        >
          <div class="px-4 py-2 flex flex-col gap-1">
            <button
              type="button"
              class="w-full text-left py-1.5 text-sm hover:text-blue-300"
              :class="{ 'text-blue-300': activeSection === 'Home' }"
              @click="navigateAndCloseMobile('Home')"
            >
              Home
            </button>
            <button
              type="button"
              class="w-full text-left py-1.5 text-sm hover:text-blue-300"
              :class="{ 'text-blue-300': activeSection === 'ExpansionView' }"
              @click="navigateAndCloseMobile('ExpansionView')"
            >
              Expansion
            </button>
            <button
              type="button"
              class="w-full text-left py-1.5 text-sm hover:text-blue-300"
              :class="{ 'text-blue-300': activeSection === 'CardSearch' }"
              @click="navigateAndCloseMobile('CardSearch')"
            >
              Card
            </button>
            <button
              type="button"
              class="w-full text-left py-1.5 text-sm hover:text-blue-300"
              :class="{ 'text-blue-300': activeSection === 'OfferSearch' }"
              @click="navigateAndCloseMobile('OfferSearch')"
            >
              Offer
            </button>
          </div>
        </nav>
      </div>
    </header>

    <main class="flex-1 flex px-3 sm:px-4 py-4">
      <div class="w-full max-w-6xl mx-auto">
        <div
          v-if="activeSection === 'Home'"
          class="w-full bg-white rounded-xl shadow-md p-6"
        >
          <p class="text-slate-700 text-base sm:text-lg">
            Welcome to the application. Select one of the options in the top menu to continue.
          </p>
        </div>
        <div v-else>
          <ExpansionListView v-if="activeSection === 'ExpansionView'" :is-admin="isAdmin" />
          <CardSearchView v-else-if="activeSection === 'CardSearch'" :is-admin="isAdmin" />
          <OfferSearchView v-else-if="activeSection === 'OfferSearch'" />
        </div>
      </div>
    </main>

    <!-- Login modal -->
    <div
      v-if="isLoginModalOpen"
      class="fixed inset-0 z-50 flex items-center justify-center bg-black/60"
    >
      <div
        class="w-full max-w-sm bg-white rounded-xl shadow-lg p-8 relative"
        role="dialog"
        aria-modal="true"
      >
        <button
          type="button"
          class="absolute right-3 top-3 text-slate-400 hover:text-slate-600 text-sm"
          @click="closeLoginModal"
          :disabled="isLoginSubmitting"
        >
          ✕
        </button>

        <h1 class="text-2xl font-semibold mb-6 text-center text-slate-900">Log in</h1>

        <form @submit.prevent="handleLoginSubmit" class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-slate-700 mb-1" for="login-username">
              Username
            </label>
            <input
              id="login-username"
              v-model="loginUsername"
              type="text"
              autocomplete="username"
              class="w-full rounded-lg border border-slate-300 px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-slate-700 mb-1" for="login-password">
              Password
            </label>
            <input
              id="login-password"
              v-model="loginPassword"
              type="password"
              autocomplete="current-password"
              class="w-full rounded-lg border border-slate-300 px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
            />
          </div>

          <p v-if="loginErrorMessage" class="text-sm text-red-600">
            {{ loginErrorMessage }}
          </p>

          <button
            type="submit"
            :disabled="isLoginSubmitting"
            class="w-full inline-flex justify-center items-center rounded-lg bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
          >
            {{ isLoginSubmitting ? 'Logging in...' : 'Log in' }}
          </button>
        </form>
      </div>
    </div>

    <!-- Change password modal -->
    <div
      v-if="isChangePasswordModalOpen"
      class="fixed inset-0 z-50 flex items-center justify-center bg-black/60"
    >
      <div
        class="w-full max-w-sm bg-white rounded-xl shadow-lg p-8 relative"
        role="dialog"
        aria-modal="true"
      >
        <button
          type="button"
          class="absolute right-3 top-3 text-slate-400 hover:text-slate-600 text-sm"
          @click="closeChangePasswordModal"
          :disabled="isChangePasswordSubmitting"
        >
          ✕
        </button>

        <h1 class="text-2xl font-semibold mb-6 text-center text-slate-900">Change password</h1>

        <form @submit.prevent="handleChangePasswordSubmit" class="space-y-4">
          <div>
            <label class="block text-sm font-medium text-slate-700 mb-1" for="old-password">
              Old password
            </label>
            <input
              id="old-password"
              v-model="oldPassword"
              type="password"
              autocomplete="current-password"
              class="w-full rounded-lg border border-slate-300 px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-slate-700 mb-1" for="new-password">
              New password
            </label>
            <input
              id="new-password"
              v-model="newPassword"
              type="password"
              autocomplete="new-password"
              class="w-full rounded-lg border border-slate-300 px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
            />
          </div>

          <div>
            <label class="block text-sm font-medium text-slate-700 mb-1" for="confirm-new-password">
              Confirm new password
            </label>
            <input
              id="confirm-new-password"
              v-model="confirmNewPassword"
              type="password"
              autocomplete="new-password"
              class="w-full rounded-lg border border-slate-300 px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-blue-500"
            />
          </div>

          <p v-if="newPassword && confirmNewPassword && newPassword !== confirmNewPassword" class="text-xs text-red-600">
            New passwords do not match.
          </p>

          <p v-if="changePasswordError" class="text-sm text-red-600">
            {{ changePasswordError }}
          </p>

          <button
            type="submit"
            :disabled="!canSubmitChangePassword || isChangePasswordSubmitting"
            class="w-full inline-flex justify-center items-center rounded-lg bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
          >
            {{ isChangePasswordSubmitting ? 'Changing...' : 'Change password' }}
          </button>
        </form>
      </div>
    </div>
  </div>
</template>
