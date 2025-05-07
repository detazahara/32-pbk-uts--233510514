<template>
  <div class="container">
    <header class="header-box">
      <h1>Study Tracker 🎓</h1>
      <div class="progress-counter">
        <div class="progress-bar" :style="{ width: progressPercent + '%' }"></div>
        <span>Weekly Progress: {{ totalHours }}/40 hours</span>
      </div>
    </header>

    <div class="controls-box">
      <div class="filter-buttons">
        <button
          v-for="filter in filters"
          :key="filter"
          :class="{ active: currentFilter === filter }"
          @click="currentFilter = filter"
        >
          {{ filter }}
        </button>
      </div>
      
      <form @submit.prevent="addSession" class="form-box">
        <input
          type="text"
          v-model.trim="newSession.subject"
          placeholder="Mata Pelajaran"
          required
          class="form-input"
        />
        <textarea
          v-model.trim="newSession.description"
          placeholder="Deskripsi Sesi (opsional)"
          class="form-textarea"
        ></textarea>
        <div class="input-group">
          <input
            type="number"
            v-model.number="newSession.duration"
            placeholder="Durasi (jam)"
            min="1"
            required
            class="duration-input"
          />
          <input
            type="text"
            v-model.trim="newSession.topic"
            placeholder="Topik"
            class="duration-input"
          />
          <button type="submit" class="submit-btn">Tambah Sesi! 📖</button>
        </div>
      </form>
    </div>

    <transition-group name="list" tag="div" class="session-list">
      <div
        v-for="session in filteredSessions"
        :key="session.id"
        class="session-card"
      >
        <label class="checkbox-box">
          <input type="checkbox" v-model="session.completed" @change="saveSessions" />
          <span class="checkmark"></span>
        </label>
        <div class="session-content" :class="{ completed: session.completed }">
          <h3>{{ session.subject }} <small>({{ session.topic || 'Umum' }})</small></h3>
          <p v-if="session.description" class="session-description">{{ session.description }}</p>
          <div class="session-meta">
            <span class="meta-item">⏳ {{ session.duration }} jam</span>
            <span class="meta-item">🗓 {{ formatDate(session.date) }}</span>
          </div>
        </div>
        <button @click="deleteSession(session.id)" class="delete-btn" title="Hapus Sesi">
          <i class="fas fa-trash-alt"></i>
        </button>
      </div>
    </transition-group>
  </div>
</template>

<script>
import { ref, computed } from 'vue';

export default {
  name: 'StudyTracker',
  setup() {
    const sessions = ref(JSON.parse(localStorage.getItem('sessions') || '[]'));
    const filters = ['All', 'Completed', 'Pending'];
    const currentFilter = ref('All');

    const newSession = ref({ subject: '', description: '', duration: null, topic: '', date: new Date().toISOString(), completed: false });

    const addSession = () => {
      sessions.value.push({
        ...newSession.value,
        id: Date.now(),
      });
      saveSessions();
      newSession.value = { subject: '', description: '', duration: null, topic: '', date: new Date().toISOString(), completed: false };
    };

    const deleteSession = id => {
      sessions.value = sessions.value.filter(s => s.id !== id);
      saveSessions();
    };

    const saveSessions = () => {
      localStorage.setItem('sessions', JSON.stringify(sessions.value));
    };

    const totalHours = computed(() => sessions.value.reduce((sum, s) => sum + (s.completed ? s.duration : 0), 0));
    const progressPercent = computed(() => Math.min((totalHours.value / 40) * 100, 100));

    const filteredSessions = computed(() => {
      if (currentFilter.value === 'All') return sessions.value;
      if (currentFilter.value === 'Completed') return sessions.value.filter(s => s.completed);
      if (currentFilter.value === 'Pending') return sessions.value.filter(s => !s.completed);
      return sessions.value;
    });

    const formatDate = iso => new Date(iso).toLocaleDateString('id-ID', { weekday: 'short', month: 'short', day: 'numeric' });

    return { filters, currentFilter, newSession, addSession, sessions, deleteSession, saveSessions, filteredSessions, totalHours, progressPercent, formatDate };
  }
};
</script>

<style scoped>
.container {
  max-width: 700px;
  margin: 2rem auto;
  padding: 1rem;
  background: #1e1e2f;
  border-radius: 12px;
  box-shadow: 0 8px 16px rgba(0,0,0,0.3);
  color: #ececff;
  display: grid;
  gap: 1.5rem;
}

.header-box h1 {
  font-size: 2rem;
  margin-bottom: 0.5rem;
}

.progress-counter {
  position: relative;
  background: #2e2e42;
  border-radius: 8px;
  overflow: hidden;
  padding: 0.5rem;
  text-align: center;
}

.progress-bar {
  background: linear-gradient(90deg, #4ade80, #22d3ee);
  height: 100%;
  transition: width 0.4s ease;
}

.controls-box {
  background: #2e2e42;
  padding: 1rem;
  border-radius: 12px;
  box-shadow: 0 4px 8px rgba(0,0,0,0.2);
}

.filter-buttons button {
  background: transparent;
  border: 1px solid #4ade80;
  color: #4ade80;
  cursor: pointer;
  transition: background 0.3s;
}

.filter-buttons button.active,
.filter-buttons button:hover {
  background: #4ade80;
  color: #1e1e2f;
}

.form-box input,
.form-box textarea {
  background: #1e1e2f;
  border: 1px solid #3a3a5a;
}

.submit-btn {
  background: #22d3ee;
  border: none;
  color: #1e1e2f;
  cursor: pointer;
}

.session-list {
  display: grid;
  gap: 1rem;
}

.session-card {
  display: grid;
  grid-template-columns: auto 1fr auto;
  gap: 1rem;
  align-items: center;
  background: #2e2e42;
  padding: 1rem;
  border-radius: 12px;
  position: relative;
  overflow: hidden;
}

.session-content.completed h3,
.session-content.completed .meta-item,
.session-content.completed p {
  text-decoration: line-through;
  opacity: 0.6;
}

.checkbox-box {
  position: relative;
  display: inline-block;
  width: 24px;
  height: 24px;
}

.checkbox-box input {
  opacity: 0;
  width: 0;
  height: 0;
}

.checkmark {
  position: absolute;
  top: 0;
  left: 0;
  height: 24px;
  width: 24px;
  background-color: #1e1e2f;
  border: 2px solid #4ade80;
  border-radius: 4px;
}

.checkbox-box input:checked ~ .checkmark {
  background-color: #4ade80;
}

.checkmark:after {
  content: "";
  position: absolute;
  display: none;
}

.checkbox-box input:checked ~ .checkmark:after {
  display: block;
}

.checkbox-box .checkmark:after {
  left: 6px;
  top: 2px;
  width: 6px;
  height: 12px;
  border: solid white;
  border-width: 0 2px 2px 0;
  transform: rotate(45deg);
}

.delete-btn {
  background: transparent;
  border: none;
  cursor: pointer;
  font-size: 1.2rem;
  color: #f87171;
}

.list-enter-active, .list-leave-active {
  transition: all 0.5s;
}
.list-enter-from, .list-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

@media (max-width: 600px) {
  .session-card { grid-template-columns: 1fr; }
  .input-group { grid-template-columns: 1fr; }
}    
</style>
