<template>
  <div class="panel-content-centered">
    <div class="card form-card">
      <div class="card-title">Nueva Obra</div>
      <form @submit.prevent="crearObra">
        <div class="form-group">
          <label class="form-label">Nombre de la Obra *</label>
          <input type="text" v-model="form.nombre" class="input-control" required placeholder="Nombre de la obra" />
        </div>
        <div class="form-group">
          <label class="form-label">Ubicación</label>
          <input type="text" v-model="form.ubicacion" class="input-control" placeholder="Dirección o descripción del lugar" />
        </div>
        <div style="display:flex; gap:10px; margin-top:8px;">
          <button type="submit" class="btn-primary btn-success" style="flex:1;">
            <i class="fas fa-save"></i> Guardar Obra
          </button>
          <button type="button" @click="$router.push({ name: 'DashboardHome' })" class="btn-primary btn-dark" style="padding:10px 20px;">
            Cancelar
          </button>
        </div>
        <p v-if="error" class="alert error">{{ error }}</p>
      </form>
    </div>
  </div>
</template>

<script>
import api from "../config/axios.Config.js";
import { useToast } from "vue-toastification";

export default {
  name: "CrearObraView",
  setup() { return { toast: useToast() }; },
  data() {
    return { form: { nombre: "", ubicacion: "" }, error: "" };
  },
  methods: {
    async crearObra() {
      this.error = "";
      try {
        await api.post("/obras", this.form);
        this.toast.success("Obra creada correctamente.");
        this.$router.push({ name: "DashboardHome" });
      } catch (err) {
        this.error = err.response?.data?.message || "Error al crear la obra.";
      }
    },
  },
};
</script>
