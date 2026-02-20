<template>
  <main class="min-h-screen bg-white text-black p-6 space-y-6">

    <!-- ── Read-only Banner (Entity / Superadmin) ── -->
    <div v-if="!canWrite"
      class="flex items-center gap-3 px-4 py-3 bg-blue-50 border border-blue-200 rounded-xl text-sm text-blue-700">
      <svg class="w-5 h-5 flex-shrink-0 text-blue-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
          d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
      </svg>
      <span>
        <strong>Akses Baca Sahaja.</strong>
        Anda log masuk sebagai <strong>{{ userRole }}</strong>. Hubungi Auditor atau Admin untuk membuat perubahan pada
        penemuan.
      </span>
    </div>

    <!-- ── Stats Cards ── -->
    <div class="grid grid-cols-2 md:grid-cols-4 gap-4">
      <div v-for="stat in stats" :key="stat.label"
        class="bg-white border border-gray-200 rounded-2xl p-5 flex items-center gap-4 shadow-sm">
        <div :class="['w-11 h-11 rounded-xl flex items-center justify-center flex-shrink-0', stat.iconBg]">
          <svg class="w-5 h-5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" :d="stat.icon" />
          </svg>
        </div>
        <div>
          <p class="text-xs text-gray-500 font-medium">{{ stat.label }}</p>
          <p :class="['text-2xl font-bold mt-0.5', stat.valueColor]">{{ stat.value }}</p>
        </div>
      </div>
    </div>

    <!-- ── Filter Bar ── -->
    <div class="bg-white border border-gray-200 rounded-2xl p-4 flex flex-col sm:flex-row gap-3 shadow-sm">
      <div class="relative flex-1">
        <svg class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-gray-500" fill="none" stroke="currentColor"
          viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
        </svg>
        <input v-model="searchQuery" type="text" placeholder="Cari penemuan..."
          class="w-full bg-gray-50 border border-gray-300 rounded-xl pl-9 pr-4 py-2.5 text-sm text-gray-900 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-orange-500 focus:border-transparent transition-all" />
      </div>
      <select v-model="filterStatus"
        class="bg-gray-50 border border-gray-300 rounded-xl px-4 py-2.5 text-sm text-gray-900 focus:outline-none focus:ring-2 focus:ring-orange-500 transition-all">
        <option value="">Semua Status</option>
        <option value="Kritikal">Kritikal</option>
        <option value="Lazim">Lazim</option>
        <option value="Berulang">Berulang</option>
      </select>
      <button @click="loadFindings" :disabled="isLoading"
        class="flex items-center gap-2 px-4 py-2.5 bg-gray-50 border border-gray-300 rounded-xl text-sm text-gray-600 hover:border-orange-500 hover:text-orange-500 transition-all disabled:opacity-50">
        <svg class="w-4 h-4" :class="{ 'animate-spin': isLoading }" fill="none" stroke="currentColor"
          viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
        </svg>
        Muat Semula
      </button>
      <!-- Add Finding Button — Auditor & Admin only -->
      <button v-if="canWrite" @click="openAddModal"
        class="flex items-center gap-2 px-5 py-2.5 bg-gradient-to-r from-orange-500 to-red-600 rounded-xl text-sm text-white font-semibold hover:shadow-lg hover:shadow-orange-500/30 transition-all">
        <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
        </svg>
        Tambah Penemuan
      </button>
    </div>

    <!-- ── Findings Table ── -->
    <div class="bg-white border border-gray-200 rounded-2xl overflow-hidden shadow-sm">
      <!-- Loading -->
      <div v-if="isLoading" class="flex flex-col items-center justify-center py-20 text-gray-500">
        <svg class="animate-spin h-8 w-8 mb-4 text-orange-500" xmlns="http://www.w3.org/2000/svg" fill="none"
          viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor"
            d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z">
          </path>
        </svg>
        <p class="text-sm">Memuatkan penemuan...</p>
      </div>

      <!-- Empty -->
      <div v-else-if="filteredFindings.length === 0"
        class="flex flex-col items-center justify-center py-20 text-gray-500">
        <svg class="w-12 h-12 mb-4 text-gray-300" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
            d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
        </svg>
        <p class="text-sm font-medium">Tiada penemuan dijumpai</p>
        <p class="text-xs mt-1">Tambah penemuan baharu atau ubah penapis carian</p>
      </div>

      <!-- Table -->
      <table v-else class="w-full text-sm">
        <thead>
          <tr class="border-b border-gray-200 bg-gray-50">
            <th class="px-5 py-3.5 text-left text-xs font-semibold text-gray-500 uppercase tracking-wider">#</th>
            <th class="px-5 py-3.5 text-left text-xs font-semibold text-gray-500 uppercase tracking-wider">Penemuan /
              Pemerhatian</th>
            <th class="px-5 py-3.5 text-left text-xs font-semibold text-gray-500 uppercase tracking-wider">Kategori
            </th>
            <th class="px-5 py-3.5 text-left text-xs font-semibold text-gray-500 uppercase tracking-wider">Status</th>
            <th class="px-5 py-3.5 text-left text-xs font-semibold text-gray-500 uppercase tracking-wider">CAPA</th>
            <th class="px-5 py-3.5 text-left text-xs font-semibold text-gray-500 uppercase tracking-wider">Bukti</th>
            <th class="px-5 py-3.5 text-center text-xs font-semibold text-gray-500 uppercase tracking-wider">Tindakan
            </th>
          </tr>
        </thead>
        <tbody class="divide-y divide-gray-100">
          <tr v-for="(finding, idx) in filteredFindings" :key="finding.id"
            class="hover:bg-orange-50/50 transition-colors group">
            <td class="px-5 py-4 text-gray-400 font-mono text-xs">{{ idx + 1 }}</td>
            <!-- Observation -->
            <td class="px-5 py-4 max-w-xs">
              <p class="text-gray-800 font-medium line-clamp-2 leading-relaxed">{{ finding.observation || '—' }}</p>
              <p class="text-xs text-gray-400 mt-0.5">{{ formatDate(finding.created_at) }}</p>
            </td>
            <!-- Category (own column) -->
            <td class="px-5 py-4">
              <span v-if="finding.categories"
                class="inline-flex items-center px-2.5 py-1 rounded-full text-xs font-medium bg-orange-50 text-orange-600 border border-orange-200">
                {{ finding.categories.category_name }}
              </span>
              <span v-else class="text-gray-400 text-xs">—</span>
            </td>
            <!-- Status -->
            <td class="px-5 py-4">
              <span :class="getStatusBadge(finding.finding_status)">
                {{ finding.finding_status }}
              </span>
            </td>
            <!-- CAPA count -->
            <td class="px-5 py-4">
              <div class="flex items-center gap-1.5">
                <div :class="capaCountBubble(finding)">
                  {{ (finding.capa_actions?.length ?? 0) }}
                </div>
                <span class="text-xs text-gray-500">CAPA</span>
              </div>
            </td>
            <!-- Evidence count -->
            <td class="px-5 py-4">
              <div class="flex items-center gap-1.5">
                <div
                  class="w-5 h-5 rounded-full bg-gray-100 border border-gray-300 flex items-center justify-center text-xs font-bold text-gray-500">
                  {{ (finding.evidence?.length ?? 0) }}
                </div>
                <span class="text-xs text-gray-500">fail</span>
              </div>
            </td>
            <!-- Actions -->
            <td class="px-5 py-4">
              <div class="flex items-center justify-center gap-2">
                <button @click="openDetailModal(finding)"
                  class="px-3 py-1.5 bg-blue-500/10 hover:bg-blue-500/20 border border-blue-500/30 text-blue-600 rounded-lg text-xs font-semibold transition-all">
                  Lihat
                </button>
                <button v-if="canWrite" @click="openCapaModal(finding)"
                  class="px-3 py-1.5 bg-orange-500/10 hover:bg-orange-500/20 border border-orange-500/30 text-orange-600 rounded-lg text-xs font-semibold transition-all">
                  CAPA
                </button>
              </div>
            </td>
          </tr>
        </tbody>
      </table>

      <!-- Footer -->
      <div v-if="!isLoading && filteredFindings.length > 0"
        class="px-5 py-3 border-t border-gray-200 text-xs text-gray-500 bg-gray-50">
        Memaparkan {{ filteredFindings.length }} daripada {{ findings.length }} penemuan
      </div>
    </div>

    <!-- ══════════════════════════════════════════════════════════
         ADD FINDING MODAL  (Auditor / Admin only)
    ══════════════════════════════════════════════════════════ -->
    <Transition name="modal">
      <div v-if="showAddModal && canWrite" @click.self="showAddModal = false"
        class="fixed inset-0 bg-black/70 backdrop-blur-sm flex items-center justify-center z-50 p-4">
        <div class="bg-white border border-gray-200 rounded-2xl shadow-2xl w-full max-w-lg flex flex-col max-h-[90vh]">
          <!-- Header -->
          <div class="px-6 py-5 border-b border-gray-200 flex items-center justify-between flex-shrink-0">
            <div>
              <h3 class="text-lg font-bold text-gray-900">Tambah Penemuan Baharu</h3>
              <p class="text-xs text-gray-500 mt-0.5">Isi maklumat penemuan audit</p>
            </div>
            <button @click="showAddModal = false"
              class="p-2 hover:bg-gray-100 rounded-xl transition-colors text-gray-400 hover:text-gray-700">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>

          <!-- Body -->
          <div class="p-6 overflow-y-auto flex-1 space-y-5">
            <!-- Role badge -->
            <div
              class="flex items-center gap-2 text-xs text-emerald-700 bg-emerald-50 border border-emerald-200 rounded-lg px-3 py-2">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
              </svg>
              Dibenarkan sebagai <strong class="ml-0.5">{{ userRole }}</strong>
            </div>

            <!-- Observation -->
            <div>
              <label class="block text-xs font-semibold text-gray-600 mb-1.5 uppercase tracking-wider">Penemuan /
                Pemerhatian <span class="text-red-500">*</span></label>
              <textarea v-model="newFinding.observation" rows="4"
                placeholder="Nyatakan penemuan audit secara terperinci..."
                class="w-full bg-gray-50 border border-gray-300 rounded-xl px-4 py-3 text-sm text-gray-900 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-orange-500 focus:border-transparent resize-none transition-all"></textarea>
              <p v-if="newFinding.observation.length > 0 && newFinding.observation.trim().length === 0"
                class="text-xs text-red-500 mt-1">Penemuan tidak boleh kosong atau hanya ruang.</p>
            </div>

            <!-- Status -->
            <div>
              <label class="block text-xs font-semibold text-gray-600 mb-1.5 uppercase tracking-wider">Status Penemuan
                <span class="text-red-500">*</span></label>
              <div class="grid grid-cols-3 gap-3">
                <label v-for="s in (['Kritikal', 'Lazim', 'Berulang'] as const)" :key="s" :class="[
                  'flex flex-col items-center gap-1.5 p-3 rounded-xl border cursor-pointer transition-all text-center',
                  newFinding.finding_status === s
                    ? statusSelectActive[s]
                    : 'border-gray-200 bg-gray-50 text-gray-500 hover:border-gray-400'
                ]">
                  <input type="radio" :value="s" v-model="newFinding.finding_status" class="sr-only" />
                  <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" :d="statusIcons[s]" />
                  </svg>
                  <span class="text-xs font-semibold leading-tight">{{ s }}</span>
                </label>
              </div>
            </div>
          </div>

          <!-- Footer -->
          <div class="px-6 py-4 border-t border-gray-200 flex justify-end gap-3 flex-shrink-0">
            <button @click="showAddModal = false"
              class="px-5 py-2 bg-white hover:bg-gray-50 border border-gray-300 rounded-xl text-sm text-gray-600 font-semibold transition-all">
              Batal
            </button>
            <button @click="submitAddFinding" :disabled="!newFinding.observation.trim() || isSaving"
              class="px-5 py-2 bg-gradient-to-r from-orange-500 to-red-600 rounded-xl text-sm text-white font-semibold transition-all hover:shadow-lg hover:shadow-orange-500/30 disabled:opacity-50 disabled:cursor-not-allowed flex items-center gap-2">
              <svg v-if="isSaving" class="w-4 h-4 animate-spin" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor"
                  d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z">
                </path>
              </svg>
              {{ isSaving ? 'Menyimpan...' : 'Simpan Penemuan' }}
            </button>
          </div>
        </div>
      </div>
    </Transition>

    <!-- ══════════════════════════════════════════════════════════
         DETAIL MODAL
    ══════════════════════════════════════════════════════════ -->
    <Transition name="modal">
      <div v-if="showDetailModal && selectedFinding" @click.self="showDetailModal = false"
        class="fixed inset-0 bg-black/70 backdrop-blur-sm flex items-center justify-center z-50 p-4">
        <div class="bg-white border border-gray-200 rounded-2xl shadow-2xl w-full max-w-2xl flex flex-col max-h-[90vh]">
          <!-- Header -->
          <div class="px-6 py-5 border-b border-gray-200 flex items-center justify-between flex-shrink-0">
            <div>
              <h3 class="text-lg font-bold text-gray-900">Butiran Penemuan #{{ selectedFinding.id }}</h3>
              <p class="text-xs text-gray-500 mt-0.5">{{ formatDate(selectedFinding.created_at) }}</p>
            </div>
            <button @click="showDetailModal = false"
              class="p-2 hover:bg-gray-100 rounded-xl transition-colors text-gray-400 hover:text-gray-700">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>

          <div class="p-6 overflow-y-auto flex-1 space-y-6">
            <!-- Status Badge + Category -->
            <div class="flex items-center gap-3 flex-wrap">
              <span :class="getStatusBadge(selectedFinding.finding_status)" class="text-sm px-4 py-1.5">
                {{ selectedFinding.finding_status }}
              </span>
              <span v-if="selectedFinding.categories"
                class="text-xs text-orange-600 bg-orange-50 border border-orange-200 px-3 py-1.5 rounded-full font-medium">
                {{ selectedFinding.categories.category_name }}
              </span>
            </div>

            <!-- Observation -->
            <div>
              <h4 class="text-xs font-semibold text-gray-500 uppercase tracking-wider mb-2">Pemerhatian</h4>
              <div class="bg-gray-50 border border-gray-200 rounded-xl p-4 text-sm text-gray-700 leading-relaxed">
                {{ selectedFinding.observation || '—' }}
              </div>
            </div>

            <!-- Update Status — Auditor / Admin only -->
            <div>
              <h4 class="text-xs font-semibold text-gray-500 uppercase tracking-wider mb-2">Kemaskini Status</h4>
              <div v-if="canWrite" class="flex gap-2 flex-wrap">
                <button v-for="s in (['Kritikal', 'Lazim', 'Berulang'] as const)" :key="s"
                  @click="changeStatus(selectedFinding, s)"
                  :disabled="selectedFinding.finding_status === s || isUpdatingStatus" :class="[
                    'px-4 py-2 rounded-xl text-xs font-semibold border transition-all disabled:opacity-40 disabled:cursor-not-allowed',
                    selectedFinding.finding_status === s ? statusSelectActive[s] : 'border-gray-200 bg-gray-50 text-gray-500 hover:border-gray-400 hover:text-gray-700'
                  ]">
                  {{ s }}
                </button>
              </div>
              <p v-else class="text-xs text-gray-400 italic">Hanya Auditor atau Admin boleh mengemas kini status.</p>
            </div>

            <!-- Evidence Section -->
            <div>
              <h4 class="text-xs font-semibold text-gray-500 uppercase tracking-wider mb-3">Bukti / Dokumen</h4>
              <!-- Upload — Auditor / Admin only -->
              <div v-if="canWrite" class="mb-3">
                <label for="evidenceInput"
                  class="flex items-center gap-3 px-4 py-3 border-2 border-dashed border-gray-300 rounded-xl hover:border-orange-400 hover:bg-orange-50 cursor-pointer transition-all">
                  <svg class="w-5 h-5 text-gray-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12" />
                  </svg>
                  <div>
                    <p class="text-sm text-gray-700 font-medium">
                      {{ pendingFile ? pendingFile.name : 'Muat naik bukti' }}
                    </p>
                    <p class="text-xs text-gray-400">PDF, Word, Excel, Imej (maks 10MB)</p>
                  </div>
                </label>
                <input type="file" id="evidenceInput" @change="handleFileSelect"
                  accept=".pdf,.doc,.docx,.xls,.xlsx,.jpg,.jpeg,.png" class="hidden" />
              </div>
              <button v-if="canWrite && pendingFile" @click="uploadPendingFile" :disabled="isUploading"
                class="w-full py-2 bg-gradient-to-r from-orange-500 to-red-600 rounded-xl text-xs font-semibold text-white transition-all hover:shadow-lg hover:shadow-orange-500/20 disabled:opacity-50 flex items-center justify-center gap-2">
                <svg v-if="isUploading" class="w-3.5 h-3.5 animate-spin" fill="none" viewBox="0 0 24 24">
                  <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                  <path class="opacity-75" fill="currentColor"
                    d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z">
                  </path>
                </svg>
                {{ isUploading ? 'Memuat naik...' : `Muat Naik "${pendingFile.name}"` }}
              </button>
              <!-- Evidence List -->
              <div v-if="(selectedFinding.evidence?.length ?? 0) > 0" class="mt-3 space-y-2">
                <div v-for="ev in selectedFinding.evidence" :key="ev.id"
                  class="flex items-center gap-3 bg-gray-50 border border-gray-200 px-4 py-2.5 rounded-xl text-xs">
                  <svg class="w-4 h-4 text-orange-400 flex-shrink-0" fill="none" stroke="currentColor"
                    viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z" />
                  </svg>
                  <span class="flex-1 text-gray-700 truncate">{{ ev.file_path?.split('/').pop() ?? `Evidence #${ev.id}`
                  }}</span>
                  <a v-if="ev.file_path" :href="getFileUrl(ev.file_path)" target="_blank" rel="noopener"
                    class="text-blue-500 hover:text-blue-700 font-semibold transition-colors">Lihat</a>
                </div>
              </div>
              <p v-else class="text-xs text-gray-400 mt-2">Tiada bukti dimuat naik lagi.</p>
            </div>
          </div>

          <div class="px-6 py-4 border-t border-gray-200 flex justify-end flex-shrink-0">
            <button @click="showDetailModal = false"
              class="px-5 py-2 bg-white hover:bg-gray-50 border border-gray-300 rounded-xl text-sm text-gray-600 font-semibold transition-all">
              Tutup
            </button>
          </div>
        </div>
      </div>
    </Transition>

    <!-- ══════════════════════════════════════════════════════════
         CAPA MODAL  (Auditor / Admin only)
    ══════════════════════════════════════════════════════════ -->
    <Transition name="modal">
      <div v-if="showCapaModal && selectedFinding && canWrite" @click.self="showCapaModal = false"
        class="fixed inset-0 bg-black/70 backdrop-blur-sm flex items-center justify-center z-50 p-4">
        <div class="bg-white border border-gray-200 rounded-2xl shadow-2xl w-full max-w-2xl flex flex-col max-h-[90vh]">
          <!-- Header -->
          <div class="px-6 py-5 border-b border-gray-200 flex items-center justify-between flex-shrink-0">
            <div>
              <h3 class="text-lg font-bold text-gray-900">Tindakan CAPA</h3>
              <p class="text-xs text-gray-500 mt-0.5">Penemuan #{{ selectedFinding.id }}</p>
            </div>
            <button @click="showCapaModal = false"
              class="p-2 hover:bg-gray-100 rounded-xl transition-colors text-gray-400 hover:text-gray-700">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
              </svg>
            </button>
          </div>

          <div class="p-6 overflow-y-auto flex-1 space-y-6">
            <!-- Existing CAPAs -->
            <div v-if="(selectedFinding.capa_actions?.length ?? 0) > 0">
              <h4 class="text-xs font-semibold text-gray-500 uppercase tracking-wider mb-3">Senarai CAPA</h4>
              <div class="space-y-3">
                <div v-for="capa in selectedFinding.capa_actions" :key="capa.id"
                  class="bg-gray-50 rounded-xl p-4 border border-gray-200">
                  <div class="flex items-start justify-between gap-3">
                    <div class="flex-1 min-w-0">
                      <p class="text-xs text-gray-500 font-semibold uppercase tracking-wider mb-1">Punca Akar</p>
                      <p class="text-sm text-gray-700">{{ capa.root_cause || '—' }}</p>
                      <p class="text-xs text-gray-500 font-semibold uppercase tracking-wider mt-3 mb-1">Pelan Tindakan
                      </p>
                      <p class="text-sm text-gray-700">{{ capa.action_plan || '—' }}</p>
                      <div class="flex items-center gap-3 mt-3 text-xs text-gray-500">
                        <span v-if="capa.due_date">📅 {{ formatDate(capa.due_date) }}</span>
                      </div>
                    </div>
                    <div class="flex-shrink-0 flex flex-col items-end gap-2">
                      <span :class="getCapaStatusBadge(capa.status)">{{ capa.status }}</span>
                      <!-- Status change buttons — Auditor / Admin only -->
                      <div class="flex gap-1">
                        <button v-for="st in (['Open', 'In-Progress', 'Closed'] as const)" :key="st"
                          v-show="capa.status !== st" @click="changeCapaStatus(capa, st)" :disabled="isUpdatingStatus"
                          class="px-2 py-0.5 text-xs rounded-lg border border-gray-300 text-gray-500 hover:border-orange-500 hover:text-orange-500 transition-all disabled:opacity-40">{{
                            st }}</button>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <div v-else>
              <p class="text-xs text-gray-500 text-center py-4">Tiada CAPA bagi penemuan ini. Cipta yang baharu di
                bawah.</p>
            </div>

            <!-- New CAPA Form — Auditor / Admin only -->
            <div class="border-t border-gray-200 pt-5">
              <h4 class="text-xs font-semibold text-gray-500 uppercase tracking-wider mb-4">Cipta CAPA Baharu</h4>
              <div class="space-y-4">
                <div>
                  <label class="block text-xs font-semibold text-gray-600 mb-1.5">Punca Akar Masalah</label>
                  <textarea v-model="newCapa.root_cause" rows="2" placeholder="Nyatakan punca akar masalah..."
                    class="w-full bg-gray-50 border border-gray-300 rounded-xl px-4 py-2.5 text-sm text-gray-900 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-orange-500 resize-none transition-all"></textarea>
                </div>
                <div>
                  <label class="block text-xs font-semibold text-gray-600 mb-1.5">Pelan Tindakan <span
                      class="text-red-500">*</span></label>
                  <textarea v-model="newCapa.action_plan" rows="3" placeholder="Nyatakan tindakan yang akan diambil..."
                    class="w-full bg-gray-50 border border-gray-300 rounded-xl px-4 py-2.5 text-sm text-gray-900 placeholder-gray-400 focus:outline-none focus:ring-2 focus:ring-orange-500 resize-none transition-all"></textarea>
                  <p v-if="newCapa.action_plan.length > 0 && newCapa.action_plan.trim().length === 0"
                    class="text-xs text-red-500 mt-1">Pelan tindakan tidak boleh kosong.</p>
                </div>
                <div>
                  <label class="block text-xs font-semibold text-gray-600 mb-1.5">Tarikh Akhir</label>
                  <input v-model="newCapa.due_date" type="date"
                    class="w-full bg-gray-50 border border-gray-300 rounded-xl px-4 py-2.5 text-sm text-gray-900 focus:outline-none focus:ring-2 focus:ring-orange-500 transition-all" />
                </div>
              </div>
            </div>
          </div>

          <!-- Footer -->
          <div class="px-6 py-4 border-t border-gray-200 flex justify-end gap-3 flex-shrink-0">
            <button @click="showCapaModal = false"
              class="px-5 py-2 bg-white hover:bg-gray-50 border border-gray-300 rounded-xl text-sm text-gray-600 font-semibold transition-all">
              Tutup
            </button>
            <button @click="submitCapa" :disabled="!newCapa.action_plan.trim() || isSaving"
              class="px-5 py-2 bg-gradient-to-r from-orange-500 to-red-600 rounded-xl text-sm text-white font-semibold transition-all hover:shadow-lg hover:shadow-orange-500/20 disabled:opacity-50 disabled:cursor-not-allowed flex items-center gap-2">
              <svg v-if="isSaving" class="w-4 h-4 animate-spin" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor"
                  d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z">
                </path>
              </svg>
              {{ isSaving ? 'Menyimpan...' : 'Simpan CAPA' }}
            </button>
          </div>
        </div>
      </div>
    </Transition>

    <!-- ── Toast ── -->
    <Transition name="toast">
      <div v-if="toastVisible" :class="[
        'fixed bottom-6 right-6 flex items-center gap-3 px-5 py-3.5 rounded-2xl shadow-2xl z-[100] text-sm font-semibold',
        toastType === 'success'
          ? 'bg-emerald-500 text-white shadow-emerald-500/30'
          : 'bg-red-500 text-white shadow-red-500/30'
      ]">
        <svg v-if="toastType === 'success'" class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
        </svg>
        <svg v-else class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
        </svg>
        {{ toastMessage }}
      </div>
    </Transition>

  </main>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import { useSync } from '../composables/useSync'
import type { Finding, CapaAction } from '../services/api'
import { getEvidenceFileUrl } from '../services/api'

const sync = useSync()

// ── Role / Permission ─────────────────────────────────────────────
/**
 * Read role from localStorage (set by Login.vue after successful auth).
 * Backend allows only 'Auditor' | 'Admin' to mutate findings.
 */
const userRole = ref<string>('')
const canWrite = computed(() =>
  userRole.value === 'Auditor' || userRole.value === 'Admin'
)

// ── State ───────────────────────────────────────────────────────
const findings = ref<Finding[]>([])
const isLoading = ref(false)
const isSaving = ref(false)
const isUpdatingStatus = ref(false)
const isUploading = ref(false)

// Modals
const showAddModal = ref(false)
const showDetailModal = ref(false)
const showCapaModal = ref(false)
const selectedFinding = ref<Finding | null>(null)

// Filters
const searchQuery = ref('')
const filterStatus = ref('')

// Toast
const toastVisible = ref(false)
const toastMessage = ref('')
const toastType = ref<'success' | 'error'>('success')

// New Finding Form
const newFinding = ref<{
  observation: string
  finding_status: Finding['finding_status']
}>({
  observation: '',
  finding_status: 'Lazim',
})

// New CAPA Form
const newCapa = ref({
  root_cause: '',
  action_plan: '',
  due_date: '',
})

// Evidence upload
const pendingFile = ref<File | null>(null)

// ── Computed ─────────────────────────────────────────────────────
const filteredFindings = computed(() => {
  let result = findings.value

  if (searchQuery.value) {
    const q = searchQuery.value.toLowerCase()
    result = result.filter(f =>
      (f.observation ?? '').toLowerCase().includes(q) ||
      (f.categories?.category_name ?? '').toLowerCase().includes(q)
    )
  }

  if (filterStatus.value) {
    result = result.filter(f => f.finding_status === filterStatus.value)
  }

  return result
})

const stats = computed(() => [
  {
    label: 'Jumlah Penemuan',
    value: findings.value.length,
    valueColor: 'text-gray-900',
    iconBg: 'bg-gradient-to-br from-gray-600 to-gray-700',
    icon: 'M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z',
  },
  {
    label: 'Kritikal',
    value: findings.value.filter(f => f.finding_status === 'Kritikal').length,
    valueColor: 'text-red-600',
    iconBg: 'bg-gradient-to-br from-red-600 to-red-700',
    icon: 'M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z',
  },
  {
    label: 'Lazim',
    value: findings.value.filter(f => f.finding_status === 'Lazim').length,
    valueColor: 'text-amber-600',
    iconBg: 'bg-gradient-to-br from-amber-500 to-amber-600',
    icon: 'M15 12a3 3 0 11-6 0 3 3 0 016 0z M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z',
  },
  {
    label: 'Berulang',
    value: findings.value.filter(f => f.finding_status === 'Berulang').length,
    valueColor: 'text-orange-600',
    iconBg: 'bg-gradient-to-br from-orange-500 to-orange-600',
    icon: 'M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15',
  },
])

// ── Style Helpers ────────────────────────────────────────────────
const statusIcons: Record<Finding['finding_status'], string> = {
  'Kritikal': 'M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z',
  'Lazim': 'M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z',
  'Berulang': 'M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15',
}

const statusSelectActive: Record<Finding['finding_status'], string> = {
  'Kritikal': 'border-red-500 bg-red-50 text-red-600',
  'Lazim': 'border-amber-500 bg-amber-50 text-amber-600',
  'Berulang': 'border-orange-500 bg-orange-50 text-orange-600',
}

const getStatusBadge = (status: Finding['finding_status']) => {
  const base = 'inline-flex items-center px-3 py-1 rounded-full text-xs font-semibold'
  switch (status) {
    case 'Kritikal': return `${base} bg-red-50 text-red-600 border border-red-200`
    case 'Lazim': return `${base} bg-amber-50 text-amber-600 border border-amber-200`
    case 'Berulang': return `${base} bg-orange-50 text-orange-600 border border-orange-200`
    default: return `${base} bg-gray-100 text-gray-500`
  }
}

const getCapaStatusBadge = (status: CapaAction['status']) => {
  const base = 'inline-flex items-center px-2.5 py-0.5 rounded-full text-xs font-semibold'
  switch (status) {
    case 'Open': return `${base} bg-blue-50 text-blue-600 border border-blue-200`
    case 'In-Progress': return `${base} bg-amber-50 text-amber-600 border border-amber-200`
    case 'Closed': return `${base} bg-emerald-50 text-emerald-600 border border-emerald-200`
    default: return `${base} bg-gray-100 text-gray-500`
  }
}

const capaCountBubble = (finding: Finding) => {
  const count = finding.capa_actions?.length ?? 0
  const base = 'w-5 h-5 rounded-full flex items-center justify-center text-xs font-bold border'
  if (count === 0) return `${base} bg-gray-100 border-gray-300 text-gray-400`
  const hasOpen = finding.capa_actions?.some(c => c.status !== 'Closed')
  return `${base} ${hasOpen ? 'bg-amber-50 border-amber-300 text-amber-600' : 'bg-emerald-50 border-emerald-300 text-emerald-600'}`
}

// ── Utility ──────────────────────────────────────────────────────
const formatDate = (dateStr: string | null) => {
  if (!dateStr) return '—'
  return new Date(dateStr).toLocaleDateString('ms-MY', {
    year: 'numeric', month: 'short', day: 'numeric',
  })
}

const getFileUrl = (filePath: string) => getEvidenceFileUrl(filePath)

// ── Toast ────────────────────────────────────────────────────────
const showToast = (message: string, type: 'success' | 'error' = 'success') => {
  toastMessage.value = message
  toastType.value = type
  toastVisible.value = true
  setTimeout(() => { toastVisible.value = false }, 3500)
}

// ── API Actions ──────────────────────────────────────────────────

const loadFindings = async () => {
  isLoading.value = true
  try {
    findings.value = await sync.getFindings()
  } catch (err: any) {
    console.error('loadFindings:', err)
    const msg = err?.response?.status === 403
      ? 'Akses ditolak. Anda tidak mempunyai kebenaran untuk melihat penemuan.'
      : 'Gagal memuatkan penemuan'
    showToast(msg, 'error')
  } finally {
    isLoading.value = false
  }
}

const openAddModal = () => {
  if (!canWrite.value) return
  newFinding.value = { observation: '', finding_status: 'Lazim' }
  showAddModal.value = true
}

const submitAddFinding = async () => {
  if (!canWrite.value) {
    showToast('Anda tidak mempunyai kebenaran untuk menambah penemuan', 'error')
    return
  }
  if (!newFinding.value.observation.trim()) {
    showToast('Sila isi penemuan / pemerhatian', 'error')
    return
  }
  isSaving.value = true
  try {
    await sync.addFinding({
      observation: newFinding.value.observation.trim(),
      finding_status: newFinding.value.finding_status,
    })
    showToast('Penemuan berjaya disimpan!')
    showAddModal.value = false
    await loadFindings()
  } catch (err: any) {
    console.error('addFinding:', err)
    const msg = err?.response?.status === 403
      ? 'Akses ditolak. Hanya Auditor atau Admin boleh menambah penemuan.'
      : err?.response?.data?.error || 'Gagal menyimpan penemuan'
    showToast(msg, 'error')
  } finally {
    isSaving.value = false
  }
}

const openDetailModal = (finding: Finding) => {
  selectedFinding.value = { ...finding, capa_actions: finding.capa_actions, evidence: finding.evidence }
  pendingFile.value = null
  showDetailModal.value = true
}

const openCapaModal = (finding: Finding) => {
  if (!canWrite.value) return
  selectedFinding.value = { ...finding, capa_actions: finding.capa_actions, evidence: finding.evidence }
  newCapa.value = { root_cause: '', action_plan: '', due_date: '' }
  showCapaModal.value = true
}

const changeStatus = async (finding: Finding, status: Finding['finding_status']) => {
  if (!canWrite.value) {
    showToast('Anda tidak mempunyai kebenaran untuk mengemas kini status', 'error')
    return
  }
  isUpdatingStatus.value = true
  try {
    await sync.setFindingStatus(finding.id, status)
    showToast('Status penemuan dikemaskini!')
    const f = findings.value.find(x => x.id === finding.id)
    if (f) f.finding_status = status
    if (selectedFinding.value?.id === finding.id) selectedFinding.value!.finding_status = status
  } catch (err: any) {
    console.error('changeStatus:', err)
    const msg = err?.response?.status === 403
      ? 'Akses ditolak. Hanya Auditor atau Admin boleh mengemas kini status.'
      : err?.response?.data?.error || 'Gagal kemaskini status'
    showToast(msg, 'error')
  } finally {
    isUpdatingStatus.value = false
  }
}

const submitCapa = async () => {
  if (!canWrite.value) {
    showToast('Anda tidak mempunyai kebenaran untuk menambah CAPA', 'error')
    return
  }
  if (!newCapa.value.action_plan.trim() || !selectedFinding.value) {
    showToast('Sila isi pelan tindakan', 'error')
    return
  }
  isSaving.value = true
  try {
    await sync.createCapa({
      finding_id: selectedFinding.value.id,
      root_cause: newCapa.value.root_cause || undefined,
      action_plan: newCapa.value.action_plan.trim(),
      due_date: newCapa.value.due_date || undefined,
    })
    showToast('CAPA berjaya disimpan!')
    newCapa.value = { root_cause: '', action_plan: '', due_date: '' }
    await loadFindings()
    const fresh = findings.value.find(f => f.id === selectedFinding.value!.id)
    if (fresh) selectedFinding.value = { ...fresh }
  } catch (err: any) {
    console.error('createCapa:', err)
    const msg = err?.response?.status === 403
      ? 'Akses ditolak. Hanya Auditor atau Admin boleh menambah CAPA.'
      : err?.response?.data?.error || 'Gagal menyimpan CAPA'
    showToast(msg, 'error')
  } finally {
    isSaving.value = false
  }
}

const changeCapaStatus = async (capa: CapaAction, status: CapaAction['status']) => {
  if (!canWrite.value) return
  isUpdatingStatus.value = true
  try {
    await sync.setCapaStatus(capa.id, status)
    showToast('Status CAPA dikemaskini!')
    await loadFindings()
    const fresh = findings.value.find(f => f.id === selectedFinding.value!.id)
    if (fresh) selectedFinding.value = { ...fresh }
  } catch (err: any) {
    console.error('changeCapaStatus:', err)
    showToast(err?.response?.data?.error || 'Gagal kemaskini status CAPA', 'error')
  } finally {
    isUpdatingStatus.value = false
  }
}

const handleFileSelect = (event: Event) => {
  const input = event.target as HTMLInputElement
  const file = input.files?.[0]
  if (!file) return
  if (file.size > 10 * 1024 * 1024) {
    showToast('Saiz fail melebihi had 10MB', 'error')
    return
  }
  pendingFile.value = file
}

const uploadPendingFile = async () => {
  if (!pendingFile.value || !selectedFinding.value) return
  if (!canWrite.value) {
    showToast('Anda tidak mempunyai kebenaran untuk memuat naik bukti', 'error')
    return
  }
  isUploading.value = true
  try {
    await sync.uploadFile(pendingFile.value)
    showToast('Fail berjaya dimuat naik!')
    pendingFile.value = null
    await loadFindings()
    const fresh = findings.value.find(f => f.id === selectedFinding.value!.id)
    if (fresh) selectedFinding.value = { ...fresh }
  } catch (err: any) {
    console.error('uploadFile:', err)
    showToast(err?.response?.data?.error || 'Gagal memuat naik fail', 'error')
  } finally {
    isUploading.value = false
  }
}

// ── Mount ─────────────────────────────────────────────────────────
onMounted(() => {
  // Read role saved by Login.vue: localStorage.setItem('userRole', user.role)
  userRole.value = localStorage.getItem('userRole') || ''
  loadFindings()
})
</script>

<style scoped>
/* Modal enter/leave transitions */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.2s ease;
}

.modal-enter-active .bg-white,
.modal-leave-active .bg-white {
  transition: transform 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .bg-white {
  transform: scale(0.95) translateY(12px);
}

.modal-leave-to .bg-white {
  transform: scale(0.95) translateY(12px);
}

/* Toast transitions */
.toast-enter-active,
.toast-leave-active {
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.toast-enter-from,
.toast-leave-to {
  opacity: 0;
  transform: translateY(16px) scale(0.95);
}

/* Clamp */
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
