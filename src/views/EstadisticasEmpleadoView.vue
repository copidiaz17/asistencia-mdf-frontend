<template>
  <div class="panel-content-centered">
    <div class="card" style="max-width:1100px; width:100%;">
      <div class="card-title">📊 Estadísticas de Empleado</div>

      <!-- Filtros -->
      <div class="filtros-container">
        <div class="filtro-grupo">
          <label class="form-label">Obra</label>
          <select v-model="obraSeleccionada" @change="cargarEmpleados" class="input-control">
            <option value="">Seleccionar obra...</option>
            <option v-for="obra in obras" :key="obra.id" :value="obra.id">{{ obra.nombre }}</option>
          </select>
        </div>
        <div class="filtro-grupo" v-if="obraSeleccionada">
          <label class="form-label">Empleado</label>
          <select v-model="empleadoSeleccionado" class="input-control">
            <option value="">Seleccionar empleado...</option>
            <option v-for="emp in empleados" :key="emp.id" :value="emp.id">{{ emp.apellido }}, {{ emp.nombre }} — {{ emp.dni }}</option>
          </select>
        </div>
        <div class="filtro-grupo">
          <label class="form-label">Período</label>
          <select v-model="periodo" class="input-control">
            <option value="mes">Mes completo</option>
            <option value="quincena">Quincena</option>
          </select>
        </div>
        <div class="filtro-grupo">
          <label class="form-label">Año</label>
          <input v-model.number="year" type="number" class="input-control" min="2020" :max="currentYear" style="max-width:100px;" />
        </div>
        <div class="filtro-grupo">
          <label class="form-label">Mes</label>
          <select v-model.number="month" class="input-control">
            <option v-for="m in 12" :key="m" :value="m">{{ nombreMes(m) }}</option>
          </select>
        </div>
        <div class="filtro-grupo" v-if="periodo === 'quincena'">
          <label class="form-label">Quincena</label>
          <select v-model.number="quincena" class="input-control">
            <option :value="1">1ra (1-15)</option>
            <option :value="2">2da (16-fin)</option>
          </select>
        </div>
        <div class="filtro-grupo" style="justify-content:flex-end;">
          <button @click="consultarEstadisticas" class="btn-primary btn-info" style="margin-top:22px;" :disabled="!empleadoSeleccionado || cargando">
            <i class="fas fa-search"></i> {{ cargando ? 'Consultando...' : 'Consultar' }}
          </button>
        </div>
      </div>

      <p v-if="error" class="alert error">{{ error }}</p>

      <!-- Resultados -->
      <template v-if="estadisticas">
        <div class="stats-header">
          <div>
            <h2 style="margin:0 0 4px; color:var(--color-accent); font-size:1.3rem;">{{ estadisticas.empleado.nombre }}</h2>
            <p style="margin:0; color:var(--color-text-muted); font-size:0.88rem;">
              DNI: {{ estadisticas.empleado.dni }} ·
              <span :class="'badge-cat badge-' + estadisticas.empleado.categoria.replace(/ /g, '-')">{{ estadisticas.empleado.categoria }}</span> ·
              Obra: {{ estadisticas.empleado.obra }}
            </p>
          </div>
          <div style="text-align:right; color:var(--color-text-muted); font-size:0.88rem;">
            <strong style="color:var(--color-accent);">{{ periodo === 'mes' ? 'Mes completo' : `${quincena}ra Quincena` }}</strong><br>
            {{ formatDate(estadisticas.periodo.fechaInicio) }} — {{ formatDate(estadisticas.periodo.fechaFin) }}<br>
            {{ estadisticas.periodo.diasPeriodo }} días en el período
          </div>
        </div>

        <!-- Cards de estadísticas -->
        <div class="stats-grid">
          <div class="stat-card stat-presente">
            <div class="stat-numero">{{ estadisticas.estadisticas.diasAsistidos }}</div>
            <div class="stat-label">Días Asistidos</div>
          </div>
          <div class="stat-card stat-ausente">
            <div class="stat-numero">{{ estadisticas.estadisticas.diasInasistidos }}</div>
            <div class="stat-label">Días Inasistidos</div>
          </div>
          <div class="stat-card stat-sin-registro">
            <div class="stat-numero">{{ estadisticas.estadisticas.diasSinRegistro }}</div>
            <div class="stat-label">Sin Registro</div>
          </div>
          <div class="stat-card stat-horas">
            <div class="stat-numero">{{ estadisticas.estadisticas.totalHorasTrabajadas }}h</div>
            <div class="stat-label">Horas Trabajadas</div>
          </div>
          <div class="stat-card stat-promedio">
            <div class="stat-numero">{{ estadisticas.estadisticas.promedioHorasDiarias }}h</div>
            <div class="stat-label">Prom. Horas/Día</div>
          </div>
          <div class="stat-card stat-porcentaje">
            <div class="stat-numero">{{ calcularPorcentajeAsistencia() }}%</div>
            <div class="stat-label">% Asistencia</div>
          </div>
        </div>

        <!-- Detalle diario -->
        <div style="margin-top:20px;">
          <div class="card-title">Detalle Diario</div>
          <div class="table-wrapper">
            <table class="data-table">
              <thead>
                <tr>
                  <th>Fecha</th><th style="text-align:center;">Estado</th>
                  <th style="text-align:center;">H. Ingreso</th><th style="text-align:center;">H. Salida</th>
                  <th style="text-align:center;">Horas</th><th>Observación</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="det in estadisticas.detalle" :key="det.fecha">
                  <td>{{ formatDate(det.fecha) }}</td>
                  <td style="text-align:center;"><span :class="det.presente ? 'badge-presente' : 'badge-ausente'">{{ det.presente ? 'Presente' : 'Ausente' }}</span></td>
                  <td style="text-align:center;">{{ det.horarioIngreso ? formatHora(det.horarioIngreso) : '—' }}</td>
                  <td style="text-align:center;">{{ det.horarioSalida ? formatHora(det.horarioSalida) : '—' }}</td>
                  <td style="text-align:center; font-weight:700; color:var(--color-accent);">{{ det.horasTrabajadas > 0 ? det.horasTrabajadas + 'h' : '—' }}</td>
                  <td>{{ det.observacion || '—' }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </template>
    </div>
  </div>
</template>

<script>
import api from "../config/axios.Config.js";

export default {
  name: "EstadisticasEmpleadoView",
  data() {
    const now = new Date();
    return {
      obras: [], empleados: [], obraSeleccionada: "", empleadoSeleccionado: "",
      periodo: "mes", year: now.getFullYear(), month: now.getMonth() + 1, quincena: 1,
      currentYear: now.getFullYear(), estadisticas: null, cargando: false, error: "",
    };
  },
  methods: {
    async cargarObras() {
      try { const res = await api.get("/obras"); this.obras = res.data || []; } catch {}
    },
    async cargarEmpleados() {
      if (!this.obraSeleccionada) { this.empleados = []; this.empleadoSeleccionado = ""; return; }
      try { const res = await api.get(`/obras/${this.obraSeleccionada}/empleados`); this.empleados = res.data || []; this.empleadoSeleccionado = ""; } catch {}
    },
    async consultarEstadisticas() {
      if (!this.empleadoSeleccionado) { this.error = "Debe seleccionar un empleado."; return; }
      this.cargando = true; this.error = ""; this.estadisticas = null;
      try {
        const params = { periodo: this.periodo, year: this.year, month: this.month };
        if (this.periodo === "quincena") params.quincena = this.quincena;
        const res = await api.get(`/estadisticas/empleado/${this.empleadoSeleccionado}`, { params });
        if (res.data.ok) { this.estadisticas = res.data; }
        else { this.error = res.data.error || "Error al obtener estadísticas."; }
      } catch (err) {
        this.error = err.response?.data?.error || "Error al consultar las estadísticas.";
      } finally { this.cargando = false; }
    },
    nombreMes(m) { return ["Enero","Febrero","Marzo","Abril","Mayo","Junio","Julio","Agosto","Septiembre","Octubre","Noviembre","Diciembre"][m - 1]; },
    formatDate(dateStr) { if (!dateStr) return ""; const p = dateStr.split("T")[0].split("-"); return `${p[2]}/${p[1]}/${p[0]}`; },
    formatHora(hora) { return hora ? hora.substring(0, 5) : "—"; },
    calcularPorcentajeAsistencia() {
      if (!this.estadisticas) return 0;
      const total = this.estadisticas.estadisticas.diasAsistidos + this.estadisticas.estadisticas.diasInasistidos;
      return total === 0 ? 0 : Math.round((this.estadisticas.estadisticas.diasAsistidos / total) * 100);
    },
  },
  mounted() { this.cargarObras(); },
};
</script>

<style scoped>
.filtros-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 12px;
  margin: 16px 0;
  padding: 16px;
  background: var(--color-surface-2);
  border-radius: var(--radius-md);
}
.filtro-grupo { display: flex; flex-direction: column; gap: 2px; }

.stats-header {
  display: flex; justify-content: space-between; align-items: flex-start;
  flex-wrap: wrap; gap: 16px; margin: 20px 0 16px;
  padding-bottom: 16px; border-bottom: 2px solid var(--color-surface-2);
}
.stats-grid {
  display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
  gap: 12px; margin-bottom: 20px;
}
.stat-card { padding: 16px; border-radius: var(--radius-md); text-align: center; border: 2px solid; }
.stat-numero { font-size: 1.8rem; font-weight: 900; margin-bottom: 4px; }
.stat-label  { font-size: 0.72rem; text-transform: uppercase; letter-spacing: 0.05em; font-weight: 700; }

.stat-presente    { background: #f0fdf4; border-color: #86efac; color: #15803d; }
.stat-ausente     { background: #fef2f2; border-color: #fca5a5; color: #dc2626; }
.stat-sin-registro{ background: #eff6ff; border-color: #93c5fd; color: #1e40af; }
.stat-horas       { background: #fffbeb; border-color: #fcd34d; color: #b45309; }
.stat-promedio    { background: var(--color-surface-2); border-color: #c0c8e8; color: var(--color-accent); }
.stat-porcentaje  { background: #f0fdf4; border-color: #6ee7b7; color: #065f46; }

.badge-cat { display:inline-block; padding:2px 8px; border-radius:999px; font-size:0.78rem; font-weight:700; text-transform:capitalize; }
.badge-ayudante              { background:#e5e7eb; color:#374151; }
.badge-medio-oficial         { background:#dbeafe; color:#1e40af; }
.badge-oficial               { background:#dcfce7; color:#15803d; }
.badge-oficial-especializado { background:#fef3c7; color:#b45309; }
.badge-capataz               { background:#fee2e2; color:#dc2626; }
.badge-presente { background:#dcfce7; color:#15803d; border-radius:4px; padding:2px 10px; font-size:0.82rem; font-weight:700; }
.badge-ausente  { background:#fee2e2; color:#dc2626; border-radius:4px; padding:2px 10px; font-size:0.82rem; font-weight:700; }
</style>
