<template>
  <div class="panel-content-centered">
    <div class="card" style="max-width:1000px; width:100%;">
      <div class="card-title">✅ Registrar Asistencia</div>

      <div class="form-group" style="max-width:220px;">
        <label class="form-label">Fecha *</label>
        <input type="date" v-model="fecha" class="input-control" />
      </div>

      <div v-if="cargando" style="color:var(--color-text-muted);">Cargando empleados...</div>
      <div v-else-if="empleados.length === 0" style="color:var(--color-text-muted);">
        No hay empleados activos en esta obra.
      </div>

      <template v-else>
        <div class="table-wrapper">
          <table class="data-table" style="margin-top:12px;">
            <thead>
              <tr>
                <th>Apellido</th><th>Nombre</th><th>DNI</th><th>Categoría</th>
                <th style="text-align:center;">Presente</th>
                <th style="text-align:center;">H. Ingreso</th>
                <th style="text-align:center;">H. Salida</th>
                <th>Observación</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="emp in empleados" :key="emp.id">
                <td>{{ emp.apellido }}</td>
                <td>{{ emp.nombre }}</td>
                <td>{{ emp.dni }}</td>
                <td><span :class="'badge-cat badge-' + emp.categoria.replace(/ /g, '-')">{{ emp.categoria }}</span></td>
                <td style="text-align:center;">
                  <input type="checkbox"
                    :checked="registros[emp.id] && registros[emp.id].presente"
                    @change="togglePresente(emp.id, $event.target.checked)"
                    class="checkbox-asistencia"
                    :disabled="!!(registros[emp.id] && registros[emp.id].presente && !esSuperAdmin)"
                  />
                </td>
                <td style="text-align:center;">
                  <input type="time"
                    :value="registros[emp.id] ? registros[emp.id].horarioIngreso : ''"
                    @change="setField(emp.id, 'horarioIngreso', $event.target.value)"
                    :disabled="!registros[emp.id] || !registros[emp.id].presente || (!esSuperAdmin && !!registros[emp.id].horarioIngreso)"
                    class="input-time"
                  />
                </td>
                <td style="text-align:center;">
                  <input type="time"
                    :value="registros[emp.id] ? registros[emp.id].horarioSalida : ''"
                    @change="setField(emp.id, 'horarioSalida', $event.target.value)"
                    :disabled="!registros[emp.id] || !registros[emp.id].presente || (!esSuperAdmin && !!registros[emp.id].horarioSalida)"
                    class="input-time"
                  />
                </td>
                <td>
                  <input type="text"
                    :value="registros[emp.id] ? registros[emp.id].observacion : ''"
                    @input="setField(emp.id, 'observacion', $event.target.value)"
                    class="input-obs"
                    placeholder="Opcional..."
                  />
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <p style="margin:12px 0 0; color:var(--color-accent); font-weight:700; font-size:0.95rem;">
          Presentes: {{ cantidadPresentes }} / {{ empleados.length }}
        </p>

        <div style="display:flex; gap:10px; margin-top:14px;">
          <button class="btn-primary btn-success" style="flex:1;" @click="guardar" :disabled="guardando || !fecha">
            <i class="fas fa-save"></i> {{ guardando ? "Guardando..." : "Guardar Asistencia" }}
          </button>
          <button type="button" class="btn-primary btn-dark" style="padding:10px 20px;" @click="$router.back()">
            Cancelar
          </button>
        </div>
      </template>

      <p v-if="error" class="alert error" style="margin-top:10px;">{{ error }}</p>
    </div>
  </div>
</template>

<script>
import api from "../config/axios.Config.js";
import { useToast } from "vue-toastification";
import { useRoute } from "vue-router";
import { useAuthStore } from "../stores/authStore.js";

export default {
  name: "RegistrarAsistenciaView",
  props: ["obraId"],
  setup() {
    return { toast: useToast(), route: useRoute(), authStore: useAuthStore() };
  },
  data() {
    return {
      fecha: new Date().toISOString().split("T")[0],
      empleados: [],
      registros: {},
      cargando: false,
      guardando: false,
      error: "",
    };
  },
  created() {
    const fechaQuery = this.route?.query?.fecha;
    if (fechaQuery) this.fecha = fechaQuery;
  },
  computed: {
    cantidadPresentes() { return Object.values(this.registros).filter((r) => r && r.presente).length; },
    esSuperAdmin() { return this.authStore.isSuperAdmin; },
  },
  watch: {
    fecha(nuevaFecha) { if (nuevaFecha) this.cargarAsistencia(); },
  },
  methods: {
    togglePresente(empId, checked) {
      const prev = this.registros[empId] || { presente: false, horarioIngreso: "", horarioSalida: "", observacion: "", horarioIngresoFijo: false };
      let horarioIngreso = prev.horarioIngreso;
      if (checked && !prev.horarioIngresoFijo && !prev.horarioIngreso) {
        const now = new Date();
        horarioIngreso = String(now.getHours()).padStart(2, "0") + ":" + String(now.getMinutes()).padStart(2, "0");
      } else if (!checked) {
        horarioIngreso = "";
      }
      this.registros = {
        ...this.registros,
        [empId]: { ...prev, presente: checked, horarioIngreso, horarioSalida: checked ? prev.horarioSalida : "" },
      };
    },
    setField(empId, campo, valor) {
      const prev = this.registros[empId] || { presente: false, horarioIngreso: "", horarioSalida: "", observacion: "", horarioIngresoFijo: false };
      this.registros = { ...this.registros, [empId]: { ...prev, [campo]: valor } };
    },
    async cargarAsistencia() {
      this.cargando = true;
      this.error = "";
      try {
        const res = await api.get(`/obras/${this.obraId}/asistencia?fecha=${this.fecha}`);
        this.empleados = res.data;
        const mapa = {};
        res.data.forEach((emp) => {
          mapa[emp.id] = {
            presente: emp.presente === true,
            horarioIngreso: emp.horarioIngreso ? emp.horarioIngreso.substring(0, 5) : "",
            horarioSalida: emp.horarioSalida ? emp.horarioSalida.substring(0, 5) : "",
            observacion: emp.observacion || "",
            horarioIngresoFijo: !!emp.horarioIngreso,
          };
        });
        this.registros = mapa;
      } catch (err) {
        this.error = "Error al cargar la asistencia.";
      } finally {
        this.cargando = false;
      }
    },
    async guardar() {
      if (!this.fecha) { this.error = "Debe seleccionar una fecha."; return; }
      for (const emp of this.empleados) {
        const reg = this.registros[emp.id];
        if (reg && reg.presente && !reg.horarioIngreso) {
          this.error = `${emp.apellido} ${emp.nombre} está marcado como presente pero falta el horario de ingreso.`;
          return;
        }
      }
      this.guardando = true;
      this.error = "";
      try {
        const registros = this.empleados.map((emp) => {
          const reg = this.registros[emp.id] || {};
          return { empleadoId: emp.id, presente: !!reg.presente, horarioIngreso: reg.horarioIngreso || null, horarioSalida: reg.horarioSalida || null, observacion: reg.observacion || null };
        });
        await api.post(`/obras/${this.obraId}/asistencia`, { fecha: this.fecha, registros });
        this.toast.success("Asistencia registrada correctamente.");
        this.$router.back();
      } catch (err) {
        this.error = err.response?.data?.message || "Error al guardar la asistencia.";
      } finally {
        this.guardando = false;
      }
    },
  },
  mounted() { this.cargarAsistencia(); },
};
</script>

<style scoped>
.badge-cat { display:inline-block; padding:2px 8px; border-radius:999px; font-size:0.78rem; font-weight:700; text-transform:capitalize; }
.badge-ayudante              { background:#e5e7eb; color:#374151; }
.badge-medio-oficial         { background:#dbeafe; color:#1e40af; }
.badge-oficial               { background:#dcfce7; color:#15803d; }
.badge-oficial-especializado { background:#fef3c7; color:#b45309; }
.badge-capataz               { background:#fee2e2; color:#dc2626; }

.checkbox-asistencia { width:18px; height:18px; cursor:pointer; accent-color:var(--color-accent); }

.input-time {
  background: #fff; border: 1.5px solid var(--color-surface-2); border-radius:6px;
  color: var(--color-text); padding:4px 6px; font-size:0.85rem; width:100px;
}
.input-time:disabled { opacity:0.4; cursor:not-allowed; }
.input-time:focus { outline:none; border-color:var(--color-accent); }

.input-obs {
  background:#fff; border:1.5px solid var(--color-surface-2); border-radius:6px;
  color:var(--color-text); padding:4px 8px; font-size:0.85rem; width:160px;
}
.input-obs:focus { outline:none; border-color:var(--color-accent); }
</style>
