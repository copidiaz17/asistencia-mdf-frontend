<template>
  <div class="panel-content-centered">
    <div class="card form-card">
      <div class="card-title">{{ editMode ? "Editar Empleado" : "Nuevo Empleado" }}</div>
      <form @submit.prevent="guardar">
        <div style="display:flex; gap:12px;">
          <div class="form-group" style="flex:1;">
            <label class="form-label">Apellido *</label>
            <input type="text" v-model="form.apellido" class="input-control" required placeholder="Apellido" />
          </div>
          <div class="form-group" style="flex:1;">
            <label class="form-label">Nombre *</label>
            <input type="text" v-model="form.nombre" class="input-control" required placeholder="Nombre" />
          </div>
        </div>
        <div style="display:flex; gap:12px;">
          <div class="form-group" style="flex:1;">
            <label class="form-label">DNI *</label>
            <input type="text" v-model="form.dni" class="input-control" required placeholder="Número de DNI" />
          </div>
          <div class="form-group" style="flex:1;">
            <label class="form-label">Fecha de Nacimiento</label>
            <input type="date" v-model="form.fechaNacimiento" class="input-control" />
          </div>
        </div>
        <div class="form-group">
          <label class="form-label">Categoría *</label>
          <select v-model="form.categoria" class="input-control" required>
            <option value="" disabled>Seleccione una categoría</option>
            <option value="ayudante">Ayudante</option>
            <option value="medio oficial">Medio Oficial</option>
            <option value="oficial">Oficial</option>
            <option value="oficial especializado">Oficial Especializado</option>
            <option value="capataz">Capataz</option>
          </select>
        </div>
        <div class="form-group">
          <label class="form-label">Fecha de Ingreso</label>
          <input type="date" v-model="form.fechaIngreso" class="input-control" />
        </div>
        <div style="display:flex; gap:10px; margin-top:8px;">
          <button type="submit" class="btn-primary btn-success" style="flex:1;">
            <i class="fas fa-save"></i> {{ editMode ? "Actualizar" : "Guardar" }}
          </button>
          <button type="button" @click="$router.back()" class="btn-primary btn-dark" style="padding:10px 20px;">
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
  name: "AddEmpleadoView",
  props: ["obraId", "empleadoId"],
  setup() { return { toast: useToast() }; },
  data() {
    return {
      editMode: false,
      form: { nombre: "", apellido: "", dni: "", fechaNacimiento: "", categoria: "", fechaIngreso: "" },
      error: "",
    };
  },
  methods: {
    async cargarEmpleado() {
      try {
        const res = await api.get(`/obras/${this.obraId}/empleados`);
        const emp = (res.data || []).find((e) => e.id === Number(this.empleadoId));
        if (emp) {
          this.form.nombre = emp.nombre;
          this.form.apellido = emp.apellido;
          this.form.dni = emp.dni;
          this.form.categoria = emp.categoria;
          this.form.fechaNacimiento = emp.fechaNacimiento ? emp.fechaNacimiento.split("T")[0] : "";
          this.form.fechaIngreso = emp.fechaIngreso ? emp.fechaIngreso.split("T")[0] : "";
        }
      } catch (e) {
        console.error("Error cargando empleado:", e);
      }
    },
    async guardar() {
      this.error = "";
      try {
        if (this.editMode) {
          await api.put(`/obras/${this.obraId}/empleados/${this.empleadoId}`, this.form);
          this.toast.success("Empleado actualizado correctamente.");
        } else {
          await api.post(`/obras/${this.obraId}/empleados`, this.form);
          this.toast.success("Empleado agregado correctamente.");
        }
        this.$router.back();
      } catch (err) {
        this.error = err.response?.data?.message || "Error al guardar el empleado.";
      }
    },
  },
  mounted() {
    if (this.empleadoId) {
      this.editMode = true;
      this.cargarEmpleado();
    }
  },
};
</script>
