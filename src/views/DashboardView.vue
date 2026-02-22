<template>
  <div class="dashboard">
    <header class="header">
      <img src="/logo-mdf.jpg" alt="MDF Obras Civiles" class="logo" />

      <div class="header-main-area">
        <h1 class="main-dashboard-title">CONTROL DE ASISTENCIA — MDF Obras Civiles</h1>
        <div class="action-buttons-container">
          <button v-if="authStore.canModify" @click="navigate('CrearObra')" class="btn-action btn-create-obra">
            <i class="fas fa-hard-hat"></i> Agregar Obra
          </button>
          <button @click="navigate('EstadisticasEmpleado')" class="btn-action btn-stats">
            <i class="fas fa-chart-bar"></i> Estadísticas
          </button>
          <button v-if="authStore.isSuperAdmin" @click="navigate('CrearUsuario')" class="btn-action btn-user">
            <i class="fas fa-user-plus"></i> Crear Usuario
          </button>
        </div>
      </div>

      <div class="user-profile-widget">
        <p class="welcome-text">¡Bienvenido!</p>
        <p class="user-name">{{ authStore.user?.nombre || authStore.user?.email }}</p>
        <button @click="logout" class="btn-logout-visible" type="button">
          <span class="logout-icon">⎋</span>
          <span class="logout-text">Salir</span>
        </button>
      </div>

      <button class="hamburger" @click="toggleSidebar" type="button">☰</button>
    </header>

    <div v-if="sidebarOpen" class="sidebar-overlay" @click="sidebarOpen = false"></div>

    <div class="main-content">
      <aside :class="['sidebar', { open: sidebarOpen }]">
        <h2 class="sidebar-title">Obras</h2>
        <ul class="obra-list">
          <li
            v-for="obra in obras"
            :key="obra.id"
            :class="{ selected: obra.id == obraActiva }"
            @click="seleccionarObra(obra.id)"
          >
            <span class="icon">🏗</span>
            <span class="obra-nombre">{{ obra.nombre }}</span>
            <div v-if="obra.id == obraActiva" class="active-indicator"></div>
          </li>
        </ul>
        <p v-if="obras.length === 0" style="color: var(--color-text-muted); font-size: 0.85rem; text-align:center; margin-top:20px;">
          Sin obras registradas
        </p>
      </aside>

      <main class="panel-central">
        <router-view />
      </main>
    </div>
  </div>
</template>

<script>
import api from "../config/axios.Config.js";
import { useAuthStore } from "../stores/authStore";

export default {
  name: "DashboardView",
  setup() {
    const authStore = useAuthStore();
    return { authStore };
  },
  data() {
    return { obras: [], sidebarOpen: false, obraActiva: null };
  },
  watch: {
    "$route.params.obraId": {
      immediate: true,
      handler(newId) { this.obraActiva = newId ? Number(newId) : null; },
    },
  },
  methods: {
    toggleSidebar() { this.sidebarOpen = !this.sidebarOpen; },
    async cargarObras() {
      try {
        const res = await api.get("/obras");
        this.obras = res.data || [];
      } catch (e) {
        this.obras = [];
      }
    },
    logout() { this.authStore.logout(); },
    navigate(name) { this.sidebarOpen = false; this.$router.push({ name }); },
    seleccionarObra(id) {
      this.obraActiva = Number(id);
      this.sidebarOpen = false;
      this.$router.push({ name: "ObraDetalle", params: { obraId: id } });
    },
  },
  mounted() { this.cargarObras(); },
};
</script>
