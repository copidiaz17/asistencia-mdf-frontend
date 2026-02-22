<template>
  <div>
    <!-- Encabezado de la obra -->
    <div class="card" style="margin-bottom:14px;">
      <div style="display:flex; justify-content:space-between; align-items:center; flex-wrap:wrap; gap:10px;">
        <div>
          <h2 style="margin:0; color:var(--color-accent); font-size:1.2rem;">🏗 {{ obra.nombre || 'Cargando...' }}</h2>
          <p v-if="obra.ubicacion" style="margin:4px 0 0; color:var(--color-text-muted); font-size:0.88rem;">📍 {{ obra.ubicacion }}</p>
        </div>
        <div style="display:flex; gap:8px; flex-wrap:wrap;">
          <button v-if="authStore.canModify" @click="goToAddEmpleado" class="btn-action btn-create-obra">
            <i class="fas fa-user-plus"></i> Agregar Empleado
          </button>
          <button @click="goToRegistrarAsistencia" class="btn-action" style="background:#d97706;">
            <i class="fas fa-clipboard-check"></i> Registrar Asistencia
          </button>
          <button @click="goToInforme" class="btn-action btn-info">
            <i class="fas fa-chart-bar"></i> Informe de Horas
          </button>
        </div>
      </div>
    </div>

    <!-- Listado de Empleados -->
    <div class="card" style="margin-bottom:14px;">
      <div class="card-title">👷 Empleados de la Obra</div>
      <div v-if="cargando" style="color:var(--color-text-muted);">Cargando...</div>
      <div v-else-if="empleados.length === 0" style="color:var(--color-text-muted); font-size:0.9rem;">
        No hay empleados registrados en esta obra.
      </div>
      <div v-else class="table-wrapper">
        <table class="data-table">
          <thead>
            <tr>
              <th>Apellido</th><th>Nombre</th><th>DNI</th><th>Categoría</th><th>F. Ingreso</th>
              <th v-if="authStore.canModify">Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="emp in empleados" :key="emp.id">
              <td>{{ emp.apellido }}</td>
              <td>{{ emp.nombre }}</td>
              <td>{{ emp.dni }}</td>
              <td><span :class="'badge-cat badge-' + emp.categoria.replace(/ /g, '-')">{{ emp.categoria }}</span></td>
              <td>{{ formatDate(emp.fechaIngreso) }}</td>
              <td v-if="authStore.canModify">
                <button class="btn-small btn-info-sm" @click="editarEmpleado(emp.id)"><i class="fas fa-edit"></i> Editar</button>
                <button class="btn-small btn-danger-sm" @click="darDeBaja(emp)"><i class="fas fa-user-slash"></i> Baja</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Asistencia del día -->
    <div class="card" style="margin-bottom:14px;">
      <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:12px; flex-wrap:wrap; gap:8px;">
        <div class="card-title" style="margin:0;">✅ Asistencia de Hoy</div>
        <button v-if="authStore.canModify" @click="goToRegistrarAsistencia" class="btn-action" style="background:#d97706; padding:7px 12px; font-size:0.85rem;">
          {{ asistenciaHoy.length > 0 && asistenciaHoy.some(e => e.presente) ? 'Editar Asistencia' : 'Registrar Asistencia' }}
        </button>
      </div>
      <div v-if="cargandoHoy" style="color:var(--color-text-muted);">Cargando...</div>
      <template v-else>
        <p style="color:var(--color-text-muted); font-size:0.88rem; margin:0 0 10px;">
          Presentes: <strong style="color:var(--color-accent);">{{ cantidadPresentesHoy }}</strong> / {{ asistenciaHoy.length }}
        </p>
        <div v-if="asistenciaHoy.length > 0" class="table-wrapper">
          <table class="data-table">
            <thead>
              <tr><th>Apellido</th><th>Nombre</th><th>Categoría</th><th style="text-align:center;">Presente</th><th style="text-align:center;">H. Ingreso</th><th style="text-align:center;">H. Salida</th><th>Observación</th></tr>
            </thead>
            <tbody>
              <tr v-for="emp in asistenciaHoy" :key="emp.id">
                <td>{{ emp.apellido }}</td>
                <td>{{ emp.nombre }}</td>
                <td><span :class="'badge-cat badge-' + emp.categoria.replace(/ /g, '-')">{{ emp.categoria }}</span></td>
                <td style="text-align:center;"><span :class="emp.presente ? 'badge-presente' : 'badge-ausente'">{{ emp.presente ? 'Sí' : 'No' }}</span></td>
                <td style="text-align:center;">{{ emp.presente && emp.horarioIngreso ? emp.horarioIngreso.substring(0,5) : '—' }}</td>
                <td style="text-align:center;">{{ emp.presente && emp.horarioSalida ? emp.horarioSalida.substring(0,5) : '—' }}</td>
                <td>{{ emp.observacion || '—' }}</td>
              </tr>
            </tbody>
          </table>
        </div>
        <p v-else style="color:var(--color-text-muted); font-size:0.9rem;">Sin empleados o sin registros para hoy.</p>
      </template>
    </div>

    <!-- Historial de asistencia -->
    <div v-if="historialPasado.length > 0" class="card">
      <div class="card-title">📋 Historial de Asistencia</div>
      <div class="table-wrapper">
        <table class="data-table">
          <thead>
            <tr><th>Fecha</th><th>Presentes</th><th>Total</th><th>% Asistencia</th></tr>
          </thead>
          <tbody>
            <tr v-for="h in historialPasado" :key="h.fecha" class="fila-historial" @click="verDetalle(h.fecha)" title="Ver detalle del día">
              <td>{{ formatDate(h.fecha) }}</td>
              <td>{{ h.presentes }}</td>
              <td>{{ h.total }}</td>
              <td>{{ h.total > 0 ? ((h.presentes / h.total) * 100).toFixed(0) : 0 }}%</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import api from "../config/axios.Config.js";
import { useAuthStore } from "../stores/authStore";
import { useRoute } from "vue-router";
import { useToast } from "vue-toastification";

export default {
  name: "ObraDetalleView",
  setup() {
    return { authStore: useAuthStore(), route: useRoute(), toast: useToast() };
  },
  data() {
    return { obra: {}, empleados: [], historialAsistencia: [], asistenciaHoy: [], cargando: true, cargandoHoy: false };
  },
  computed: {
    hoy() { return new Date().toISOString().split("T")[0]; },
    cantidadPresentesHoy() { return this.asistenciaHoy.filter((e) => e.presente).length; },
    historialPasado() { return this.historialAsistencia.filter((h) => h.fecha < this.hoy); },
  },
  watch: {
    "route.params.obraId": {
      immediate: true,
      async handler(obraId) {
        if (!obraId) return;
        this.obra = {}; this.empleados = []; this.historialAsistencia = []; this.asistenciaHoy = [];
        this.cargando = true;
        await this.fetchData();
        this.cargando = false;
      },
    },
  },
  methods: {
    async fetchData() {
      const obraId = this.route.params.obraId;
      try {
        const [obraRes, empRes, histRes] = await Promise.all([
          api.get(`/obras/${obraId}`),
          api.get(`/obras/${obraId}/empleados`),
          api.get(`/obras/${obraId}/asistencia/historial`),
        ]);
        this.obra = obraRes.data;
        this.empleados = empRes.data || [];
        this.historialAsistencia = histRes.data || [];
      } catch (e) { console.error("Error cargando obra:", e); }
      await this.fetchAsistenciaHoy();
    },
    async fetchAsistenciaHoy() {
      const obraId = this.route.params.obraId;
      this.cargandoHoy = true;
      try {
        const res = await api.get(`/obras/${obraId}/asistencia?fecha=${this.hoy}`);
        this.asistenciaHoy = res.data || [];
      } catch (e) { console.error(e); }
      finally { this.cargandoHoy = false; }
    },
    formatDate(dateStr) {
      if (!dateStr) return "—";
      const parts = dateStr.split("T")[0].split("-");
      return `${parts[2]}/${parts[1]}/${parts[0]}`;
    },
    goToAddEmpleado() { this.$router.push({ name: "AddEmpleado", params: { obraId: this.route.params.obraId } }); },
    goToRegistrarAsistencia() { this.$router.push({ name: "RegistrarAsistencia", params: { obraId: this.route.params.obraId } }); },
    goToInforme() { this.$router.push({ name: "InformeHoras", params: { obraId: this.route.params.obraId } }); },
    verDetalle(fecha) { this.$router.push({ name: "AsistenciaDetalle", params: { obraId: this.route.params.obraId, fecha } }); },
    editarEmpleado(empleadoId) { this.$router.push({ name: "EditarEmpleado", params: { obraId: this.route.params.obraId, empleadoId } }); },
    async darDeBaja(emp) {
      if (!confirm(`¿Confirmar baja de ${emp.nombre} ${emp.apellido}?`)) return;
      try {
        await api.delete(`/obras/${this.route.params.obraId}/empleados/${emp.id}`);
        this.toast.success("Empleado dado de baja correctamente.");
        await this.fetchData();
      } catch (e) { this.toast.error("Error al dar de baja el empleado."); }
    },
  },
};
</script>

<style scoped>
.badge-cat {
  display: inline-block; padding: 2px 8px; border-radius: 999px;
  font-size: 0.78rem; font-weight: 700; text-transform: capitalize;
}
.badge-ayudante              { background: #e5e7eb; color: #374151; }
.badge-medio-oficial         { background: #dbeafe; color: #1e40af; }
.badge-oficial               { background: #dcfce7; color: #15803d; }
.badge-oficial-especializado { background: #fef3c7; color: #b45309; }
.badge-capataz               { background: #fee2e2; color: #dc2626; }
.badge-presente { background: #dcfce7; color: #15803d; border-radius: 4px; padding: 2px 10px; font-size: 0.82rem; font-weight: 700; }
.badge-ausente  { background: #fee2e2; color: #dc2626; border-radius: 4px; padding: 2px 10px; font-size: 0.82rem; font-weight: 700; }
.btn-info-sm  { background: rgba(59,95,204,0.12); color: #3B5FCC; }
.btn-danger-sm { background: rgba(220,38,38,0.10); color: #dc2626; }
.fila-historial { cursor: pointer; }
.fila-historial:hover { background: var(--color-surface-2) !important; }
</style>
