<template>
  <div class="panel-content-centered">
    <div class="card" style="max-width:1000px; width:100%;">
      <div style="display:flex; justify-content:space-between; align-items:flex-start; flex-wrap:wrap; gap:12px; margin-bottom:14px;">
        <div>
          <div class="card-title" style="margin:0 0 4px;">Detalle de Asistencia</div>
          <p style="margin:0; color:var(--color-text-muted); font-size:0.88rem;">
            Fecha: <strong style="color:var(--color-accent);">{{ formatDate(fecha) }}</strong>
          </p>
        </div>
        <button v-if="authStore.canModify && esHoy" class="btn-action" style="background:#d97706;" @click="editarAsistencia">
          <i class="fas fa-edit"></i> Editar Asistencia
        </button>
      </div>

      <div v-if="cargando" style="color:var(--color-text-muted);">Cargando...</div>
      <div v-else-if="empleados.length === 0" style="color:var(--color-text-muted);">
        No hay registros de asistencia para esta fecha.
      </div>

      <template v-else>
        <p style="margin:0 0 12px; color:var(--color-text-muted); font-size:0.88rem;">
          Presentes: <strong style="color:var(--color-accent);">{{ cantidadPresentes }}</strong> / {{ empleados.length }}
          &nbsp;·&nbsp; {{ empleados.length > 0 ? ((cantidadPresentes / empleados.length) * 100).toFixed(0) : 0 }}% asistencia
        </p>
        <div class="table-wrapper">
          <table class="data-table">
            <thead>
              <tr><th>Apellido</th><th>Nombre</th><th>DNI</th><th>Categoría</th><th style="text-align:center;">Presente</th><th style="text-align:center;">H. Ingreso</th><th style="text-align:center;">H. Salida</th><th>Observación</th></tr>
            </thead>
            <tbody>
              <tr v-for="emp in empleados" :key="emp.id">
                <td>{{ emp.apellido }}</td>
                <td>{{ emp.nombre }}</td>
                <td>{{ emp.dni }}</td>
                <td><span :class="'badge-cat badge-' + emp.categoria.replace(/ /g, '-')">{{ emp.categoria }}</span></td>
                <td style="text-align:center;"><span :class="emp.presente ? 'badge-presente' : 'badge-ausente'">{{ emp.presente ? "Sí" : "No" }}</span></td>
                <td style="text-align:center;">{{ emp.presente && emp.horarioIngreso ? formatHora(emp.horarioIngreso) : "—" }}</td>
                <td style="text-align:center;">{{ emp.presente && emp.horarioSalida ? formatHora(emp.horarioSalida) : "—" }}</td>
                <td>{{ emp.observacion || "—" }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </template>

      <button type="button" class="btn-primary btn-dark" style="width:100%; margin-top:14px;" @click="$router.back()">
        <i class="fas fa-arrow-left"></i> Volver
      </button>
    </div>
  </div>
</template>

<script>
import api from "../config/axios.Config.js";
import { useAuthStore } from "../stores/authStore";

export default {
  name: "AsistenciaDetalleView",
  props: ["obraId", "fecha"],
  setup() { return { authStore: useAuthStore() }; },
  data() { return { empleados: [], cargando: false, error: "" }; },
  computed: {
    cantidadPresentes() { return this.empleados.filter((e) => e.presente).length; },
    esHoy() { return this.fecha === new Date().toISOString().split("T")[0]; },
  },
  methods: {
    formatDate(dateStr) {
      if (!dateStr) return "";
      const parts = dateStr.split("T")[0].split("-");
      return `${parts[2]}/${parts[1]}/${parts[0]}`;
    },
    formatHora(hora) { return hora ? hora.substring(0, 5) : "—"; },
    editarAsistencia() {
      this.$router.push({ name: "RegistrarAsistencia", params: { obraId: this.obraId }, query: { fecha: this.fecha } });
    },
    async cargarDetalle() {
      this.cargando = true;
      try {
        const res = await api.get(`/obras/${this.obraId}/asistencia?fecha=${this.fecha}`);
        this.empleados = res.data;
      } catch (err) { this.error = "Error al cargar el detalle."; }
      finally { this.cargando = false; }
    },
  },
  mounted() { this.cargarDetalle(); },
};
</script>

<style scoped>
.badge-cat { display:inline-block; padding:2px 8px; border-radius:999px; font-size:0.78rem; font-weight:700; text-transform:capitalize; }
.badge-ayudante              { background:#e5e7eb; color:#374151; }
.badge-medio-oficial         { background:#dbeafe; color:#1e40af; }
.badge-oficial               { background:#dcfce7; color:#15803d; }
.badge-oficial-especializado { background:#fef3c7; color:#b45309; }
.badge-capataz               { background:#fee2e2; color:#dc2626; }
.badge-presente { background:#dcfce7; color:#15803d; border-radius:4px; padding:2px 10px; font-size:0.82rem; font-weight:700; }
.badge-ausente  { background:#fee2e2; color:#dc2626; border-radius:4px; padding:2px 10px; font-size:0.82rem; font-weight:700; }
</style>
