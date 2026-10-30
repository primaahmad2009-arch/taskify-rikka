<script setup>
import { computed, ref, watch } from 'vue'
const starter = ['Tugas Matematika', 'Tugas PBB', 'Tugas B.indonesia', 'Tugas B.english']
const tasks = ref(JSON.parse(localStorage.getItem('taskify-rikka-tasks') || 'null') || starter.map((title, id) => ({ id, title, done: false })))
const inputTask = ref(''), filter = ref('all'), dark = ref(false), editingId = ref(null), editText = ref('')
const deleteTarget = ref(null), puzzle = ref(null), selected = ref([]), puzzleError = ref('')
const puzzles = [
    { prompt: 'Pilih semua potongan yang menampilkan bintang ⭐', tiles: ['⭐', '🌙', '⭐', '💜', '⭐', '🦇'], answer: [0, 2, 4] },
    { prompt: 'Pilih semua ikon bulan 🌙', tiles: ['💜', '🌙', '⭐', '🌙', '🦇', '🌙'], answer: [1, 3, 5] },
    { prompt: 'Pilih semua hati ungu 💜', tiles: ['💜', '⭐', '🦇', '💜', '🌙', '💜'], answer: [0, 3, 5] },
    { prompt: 'Pilih semua kelelawar 🦇', tiles: ['🌙', '🦇', '💜', '⭐', '🦇', '🦇'], answer: [1, 4, 5] },
    { prompt: 'Pilih semua bintang ⭐', tiles: ['🌙', '💜', '⭐', '🦇', '⭐', '⭐'], answer: [2, 4, 5] }
]
let puzzleIndex = 0
const visibleTasks = computed(() => tasks.value.filter(t => filter.value === 'all' || (filter.value === 'done' ? t.done : !t.done)))
const completed = computed(() => tasks.value.filter(t => t.done).length)
const pending = computed(() => tasks.value.length - completed.value)
const completionRate = computed(() => tasks.value.length ? Math.round(completed.value / tasks.value.length * 100) : 0)
watch(tasks, v => localStorage.setItem('taskify-rikka-tasks', JSON.stringify(v)), { deep: true })
function addTask() { const title = inputTask.value.trim(); if (!title) return; tasks.value.unshift({ id: Date.now(), title, done: false }); inputTask.value = '' }
function beginEdit(t) { editingId.value = t.id; editText.value = t.title }
function saveEdit(t) { const title = editText.value.trim(); if (title) t.title = title; editingId.value = null }
function startDelete(t) { deleteTarget.value = t; puzzleError.value = ''; selected.value = []; const base = puzzles[puzzleIndex++ % puzzles.length]; puzzle.value = { ...base, answer: [...base.answer] } }
function toggleTile(i) { selected.value = selected.value.includes(i) ? selected.value.filter(x => x !== i) : [...selected.value, i] }
function verifyDelete() { const a = [...selected.value].sort((x, y) => x - y), b = [...puzzle.value.answer].sort((x, y) => x - y); if (a.length === b.length && a.every((v, i) => v === b[i])) { tasks.value = tasks.value.filter(t => t.id !== deleteTarget.value.id); deleteTarget.value = null; puzzle.value = null } else { puzzleError.value = 'Belum tepat! Coba puzzle baru.'; selected.value = []; const base = puzzles[puzzleIndex++ % puzzles.length]; puzzle.value = { ...base, answer: [...base.answer] } } }
</script>
<template>
    <div class="app" :class="{ dark }">
        <header>
            <div class="brand"><span class="logo">✓</span> Taskify <b>Vue</b></div><button class="theme"
                @click="dark = !dark">{{ dark ? '☀️' : '🌙' }}</button>
        </header>
        <main class="panel">
            <div class="eyebrow">✦ WICKED EYE OF THE VIOLET ✦</div>
            <h1>Taskify Vue</h1>
            <p class="subtitle">Kelola tugasmu dengan mudah dan lebih produktif</p>
            <p class="magic">✦ Darkness, focus, and a little magic. ✦</p>
            <form class="add" @submit.prevent="addTask"><input v-model="inputTask" placeholder="Masukkan tugas baru..."
                    maxlength="100"><button>＋ Tambah</button></form>
            <section class="stats">
                <div class="stat-card"><span class="stat-icon">📚</span><small>Total
                        Tugas</small><strong>{{ tasks.length }}</strong></div>
                <div class="stat-card"><span class="stat-icon">⏳</span><small>Belum
                        Selesai</small><strong>{{ pending }}</strong></div>
                <div class="stat-card"><span class="stat-icon">✨</span><small>Selesai</small><strong>{{ completed
                        }}</strong></div>
                <div class="stat-card"><span class="stat-icon">🎯</span><small>Progress</small><strong>{{ completionRate
                        }}%</strong></div>
            </section>
            <div class="progress"><span>{{ completed }} dari {{ tasks.length }} selesai</span>
                <nav><button
                        v-for="f in [{ id: 'all', label: 'Semua' }, { id: 'todo', label: 'Belum' }, { id: 'done', label: 'Selesai' }]"
                        :key="f.id" :class="{ active: filter === f.id }" @click="filter = f.id">{{ f.label }}</button>
                </nav>
            </div>
            <div class="tasks">
                <article v-for="t in visibleTasks" :key="t.id" class="task"><input type="checkbox"
                        v-model="t.done"><template v-if="editingId === t.id"><input class="edit" v-model="editText"
                            @keyup.enter="saveEdit(t)"><button @click="saveEdit(t)">Simpan</button></template><span
                        v-else class="taskname" :class="{ done: t.done }">{{ t.title }}</span><span class="status">{{
                            t.done ? 'Selesai' :
                                'Belum' }}</span><button class="icon" title="Edit" @click="beginEdit(t)">✎</button><button
                        class="delete" title="Hapus" @click="startDelete(t)">×</button></article>
                <p v-if="!visibleTasks.length" class="empty">Tidak ada tugas di sini ✨</p>
            </div>
            <footer>✦ Satu tugas selesai, satu langkah menuju kemenangan! ✦</footer>
        </main>
        <div v-if="puzzle && deleteTarget" class="overlay">
            <section class="modal">
                <div class="modalmark">🔮</div>
                <h2>Verifikasi Penghapusan</h2>
                <p class="target">Hapus tugas: <b>{{ deleteTarget.title }}</b></p>
                <p>{{ puzzle.prompt }}</p>
                <div class="tiles"><button v-for="(tile, i) in puzzle.tiles" :key="i"
                        :class="{ chosen: selected.includes(i) }" @click="toggleTile(i)">{{ tile }}</button></div>
                <p class="hint">Pilih semua gambar yang sesuai, lalu tekan Verifikasi.</p>
                <p v-if="puzzleError" class="error">{{ puzzleError }}</p>
                <div class="modalactions"><button class="cancel"
                        @click="puzzle = null; deleteTarget = null">Batal</button><button class="confirm"
                        @click="verifyDelete">Verifikasi</button></div>
            </section>
        </div>
    </div>
</template>
