<template>
  <div class="login-page">
    <div class="login-container">
      <img src="/logo-mdf.jpg" alt="MDF Obras Civiles" class="logo" />
      <h1 class="login-title">CONTROL DE ASISTENCIA</h1>
      <p class="login-subtitle">MDF Obras Civiles SRL</p>

      <form @submit.prevent="login" class="login-form">
        <div class="input-group">
          <label>Email</label>
          <input v-model="email" type="email" placeholder="usuario@mdf.com" required autocomplete="email" />
        </div>
        <div class="input-group">
          <label>Contraseña</label>
          <input v-model="password" type="password" placeholder="••••••••" required autocomplete="current-password" />
        </div>
        <button type="submit" class="btn-submit" :disabled="loading">
          <span v-if="loading"><i class="fas fa-spinner fa-spin"></i> Ingresando...</span>
          <span v-else><i class="fas fa-sign-in-alt"></i> Ingresar</span>
        </button>
        <p v-if="error" class="error">{{ error }}</p>
      </form>
    </div>
  </div>
</template>

<script>
import api from "../config/axios.Config.js";
import { useAuthStore } from "../stores/authStore";
import "../assets/css/login.css";

export default {
  name: "LoginView",
  data() {
    return { email: "", password: "", error: null, loading: false };
  },
  methods: {
    async login() {
      this.error = null;
      this.loading = true;
      try {
        const res = await api.post("/auth/login", { email: this.email, password: this.password });
        const token = res.data?.token;
        if (!token) throw new Error("No vino token en la respuesta");
        localStorage.setItem("token", token);
        const authStore = useAuthStore();
        authStore.loadUserFromToken(token);
        this.$router.push({ name: "Dashboard" });
      } catch (err) {
        this.error = err?.response?.data?.message || err?.message || "Error al iniciar sesión";
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>
