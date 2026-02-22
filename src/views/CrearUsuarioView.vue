<template>
  <div class="panel-content-centered">
    <div class="card form-card">
      <div class="card-title">Crear Nuevo Usuario</div>

      <p v-if="mensaje" :class="['alert', mensajeTipo === 'ok' ? 'success' : 'error']">
        {{ mensaje }}
      </p>

      <form @submit.prevent="crearUsuario">
        <div class="form-group">
          <label class="form-label">Nombre *</label>
          <input v-model="form.nombre" type="text" class="input-control" required placeholder="Nombre del usuario" />
        </div>
        <div class="form-group">
          <label class="form-label">Email *</label>
          <input v-model="form.email" type="email" class="input-control" required placeholder="correo@mdf.com" />
        </div>
        <div class="form-group">
          <label class="form-label">Contraseña *</label>
          <input v-model="form.password" type="password" class="input-control" required placeholder="Contraseña" minlength="6" />
        </div>
        <div class="form-group">
          <label class="form-label">Rol *</label>
          <select v-model="form.rol" class="input-control" required>
            <option value="usuario">Usuario (puede registrar asistencia)</option>
            <option value="lector">Lector (solo lectura)</option>
            <option value="administrador">Administrador</option>
          </select>
        </div>
        <div style="display:flex; gap:10px; margin-top:8px;">
          <button type="submit" class="btn-primary btn-user" style="flex:1;">
            <i class="fas fa-user-plus"></i> Crear Usuario
          </button>
          <button type="button" @click="$router.push({ name: 'DashboardHome' })" class="btn-primary btn-dark" style="padding:10px 20px;">
            Cancelar
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import api from "../config/axios.Config.js";
import { useAuthStore } from "../stores/authStore";

export default {
  name: "CrearUsuarioView",
  setup() {
    return { authStore: useAuthStore() };
  },
  data() {
    return {
      form: { nombre: "", email: "", password: "", rol: "usuario" },
      mensaje: "",
      mensajeTipo: "",
    };
  },
  methods: {
    async crearUsuario() {
      this.mensaje = "";
      try {
        await api.post("/usuarios", this.form);
        this.mensaje = "Usuario creado correctamente.";
        this.mensajeTipo = "ok";
        this.form.nombre = "";
        this.form.email = "";
        this.form.password = "";
        this.form.rol = "usuario";
      } catch (error) {
        this.mensaje = error.response?.data?.message || "Error al crear el usuario.";
        this.mensajeTipo = "error";
      }
    },
  },
  mounted() {
    if (!this.authStore.isSuperAdmin) {
      this.$router.replace({ name: "DashboardHome" });
    }
  },
};
</script>

<style scoped>
.btn-user { background: #6D28D9; color: #fff; }
</style>
