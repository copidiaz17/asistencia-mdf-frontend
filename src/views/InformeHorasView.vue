<template>
  <div>
    <!-- Encabezado -->
    <div class="card" style="margin-bottom:14px;">
      <div style="display:flex; align-items:center; gap:12px; flex-wrap:wrap;">
        <button @click="$router.back()" class="btn-primary btn-dark" style="padding:7px 14px; font-size:0.85rem;">
          <i class="fas fa-arrow-left"></i> Volver
        </button>
        <h2 style="margin:0; color:var(--color-accent); font-size:1.15rem;">
          📊 Informe de Horas — {{ obra.nombre || '...' }}
        </h2>
      </div>
    </div>

    <!-- Selector de período -->
    <div class="card" style="margin-bottom:14px;">
      <div class="card-title">Período</div>
      <div style="display:flex; gap:12px; flex-wrap:wrap; align-items:flex-end;">
        <div class="form-group" style="margin:0; min-width:80px;">
          <label class="form-label">Año</label>
          <select v-model="anio" class="input-control">
            <option v-for="a in anios" :key="a" :value="a">{{ a }}</option>
          </select>
        </div>
        <div class="form-group" style="margin:0; min-width:130px;">
          <label class="form-label">Mes</label>
          <select v-model="mes" class="input-control">
            <option v-for="(m, i) in meses" :key="i" :value="i + 1">{{ m }}</option>
          </select>
        </div>
        <div class="form-group" style="margin:0; min-width:170px;">
          <label class="form-label">Período</label>
          <select v-model="periodo" class="input-control">
            <option value="Q1">1ª Quincena (1-15)</option>
            <option value="Q2">2ª Quincena (16-fin)</option>
            <option value="MES">Mes completo</option>
          </select>
        </div>
        <button class="btn-primary btn-info" style="height:42px;" @click="cargarReporte" :disabled="cargando">
          <i class="fas fa-search"></i> {{ cargando ? 'Cargando...' : 'Generar' }}
        </button>
      </div>
      <p v-if="desde && hasta" style="margin:10px 0 0; color:var(--color-text-muted); font-size:0.88rem;">
        Período: <strong>{{ formatDate(desde) }}</strong> al <strong>{{ formatDate(hasta) }}</strong>
      </p>
    </div>

    <!-- Sin resultados -->
    <div v-if="buscado && !cargando && datosConRegistros.length === 0" class="card">
      <p style="color:var(--color-text-muted);">No se encontraron registros para el período seleccionado.</p>
    </div>

    <!-- Tabla de resultados -->
    <div v-if="datosConRegistros.length > 0" class="card">
      <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:12px; flex-wrap:wrap; gap:8px;">
        <div class="card-title" style="margin:0;">Resultado</div>
        <button @click="exportarPDF" class="btn-primary btn-danger" style="padding:7px 14px; font-size:0.85rem;">
          <i class="fas fa-file-pdf"></i> Exportar PDF
        </button>
      </div>
      <div class="table-wrapper">
        <table class="data-table">
          <thead>
            <tr>
              <th>Apellido</th><th>Nombre</th><th>DNI</th><th>Categoría</th>
              <th style="text-align:center;">Días Pres.</th><th style="text-align:center;">Días Aus.</th><th style="text-align:center;">Horas Trab.</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="emp in datosConRegistros" :key="emp.id">
              <td>{{ emp.apellido }}</td>
              <td>{{ emp.nombre }}</td>
              <td>{{ emp.dni }}</td>
              <td><span :class="'badge-cat badge-' + emp.categoria.replace(/ /g, '-')">{{ emp.categoria }}</span></td>
              <td style="text-align:center;"><span class="badge-presente">{{ emp.diasPresente }}</span></td>
              <td style="text-align:center;"><span v-if="emp.diasAusente > 0" class="badge-ausente">{{ emp.diasAusente }}</span><span v-else style="color:var(--color-text-muted);">0</span></td>
              <td style="text-align:center; font-weight:800; color:var(--color-accent);">{{ emp.horasTrabajadas }}</td>
            </tr>
          </tbody>
          <tfoot>
            <tr style="background:var(--color-surface-2);">
              <td colspan="4" style="font-weight:800; color:var(--color-accent);">TOTAL</td>
              <td style="text-align:center; font-weight:800; color:var(--color-accent);">{{ totalPresentes }}</td>
              <td style="text-align:center; font-weight:800; color:var(--color-accent);">{{ totalAusentes }}</td>
              <td style="text-align:center; font-weight:800; color:var(--color-accent);">{{ totalHoras }}</td>
            </tr>
          </tfoot>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import api from '../config/axios.Config.js';
import { useRoute } from 'vue-router';
import jsPDF from 'jspdf';
import autoTable from 'jspdf-autotable';

export default {
  name: 'InformeHorasView',
  setup() { return { route: useRoute() }; },
  data() {
    const hoy = new Date();
    return {
      obra: {}, datos: [], cargando: false, buscado: false,
      anio: hoy.getFullYear(), mes: hoy.getMonth() + 1, periodo: 'MES',
      desde: null, hasta: null,
      meses: ['Enero','Febrero','Marzo','Abril','Mayo','Junio','Julio','Agosto','Septiembre','Octubre','Noviembre','Diciembre'],
    };
  },
  computed: {
    anios() { const a = new Date().getFullYear(); return [a - 1, a, a + 1]; },
    totalPresentes() { return this.datosConRegistros.reduce((s, e) => s + e.diasPresente, 0); },
    totalAusentes()  { return this.datosConRegistros.reduce((s, e) => s + e.diasAusente, 0); },
    totalHoras() {
      const min = this.datosConRegistros.reduce((s, e) => s + e.totalMinutos, 0);
      if (min <= 0) return '—';
      const h = Math.floor(min / 60), m = min % 60;
      return m > 0 ? h + 'h ' + m + 'm' : h + 'h';
    },
    periodoTexto() { return (!this.desde || !this.hasta) ? '' : this.formatDate(this.desde) + ' al ' + this.formatDate(this.hasta); },
    datosConRegistros() { return this.datos.filter((e) => e.totalRegistros > 0); },
  },
  methods: {
    calcularRango() {
      const anio = this.anio;
      const mes = String(this.mes).padStart(2, '0');
      if (this.periodo === 'Q1') return { desde: anio + '-' + mes + '-01', hasta: anio + '-' + mes + '-15' };
      const ultimo = new Date(anio, this.mes, 0).getDate();
      if (this.periodo === 'Q2') return { desde: anio + '-' + mes + '-16', hasta: anio + '-' + mes + '-' + String(ultimo).padStart(2,'0') };
      return { desde: anio + '-' + mes + '-01', hasta: anio + '-' + mes + '-' + String(ultimo).padStart(2,'0') };
    },
    async cargarObra() {
      try { const res = await api.get('/obras/' + this.route.params.obraId); this.obra = res.data; } catch {}
    },
    async cargarReporte() {
      const { desde, hasta } = this.calcularRango();
      this.desde = desde; this.hasta = hasta;
      this.cargando = true; this.buscado = false;
      try {
        const res = await api.get('/obras/' + this.route.params.obraId + '/asistencia/reporte?desde=' + desde + '&hasta=' + hasta);
        this.datos = res.data || [];
      } catch { this.datos = []; }
      finally { this.cargando = false; this.buscado = true; }
    },
    formatDate(dateStr) { if (!dateStr) return ''; const [y, m, d] = dateStr.split('-'); return d + '/' + m + '/' + y; },
    periodoLabel() { if (this.periodo === 'Q1') return '1a Quincena'; if (this.periodo === 'Q2') return '2a Quincena'; return 'Mes Completo'; },
    exportarPDF() {
      const doc = new jsPDF({ orientation: 'landscape', unit: 'mm', format: 'a4' });
      const mesNombre = this.meses[this.mes - 1];
      doc.setFontSize(16); doc.setTextColor(45, 45, 126);
      doc.text('Informe de Horas - ' + (this.obra.nombre || 'Obra'), 14, 18);
      doc.setFontSize(10); doc.setTextColor(90, 96, 128);
      doc.text(this.periodoLabel() + ' | ' + mesNombre + ' ' + this.anio + ' | ' + this.periodoTexto, 14, 25);
      let curY = 31;
      if (this.obra.ubicacion) { doc.text('Ubicacion: ' + this.obra.ubicacion, 14, curY); curY += 6; }
      const body = this.datosConRegistros.map(emp => [emp.apellido, emp.nombre, emp.dni, emp.categoria, emp.diasPresente, emp.diasAusente, emp.horasTrabajadas]);
      body.push([
        { content: 'TOTAL', colSpan: 4, styles: { fontStyle: 'bold', halign: 'right' } },
        { content: String(this.totalPresentes), styles: { fontStyle: 'bold', halign: 'center' } },
        { content: String(this.totalAusentes),  styles: { fontStyle: 'bold', halign: 'center' } },
        { content: this.totalHoras,             styles: { fontStyle: 'bold', halign: 'center' } },
      ]);
      autoTable(doc, {
        startY: curY,
        head: [['Apellido','Nombre','DNI','Categoria','Dias Pres.','Dias Aus.','Horas Trab.']],
        body,
        headStyles: { fillColor: [45, 45, 126], textColor: [255,255,255], fontStyle: 'bold', halign: 'center' },
        columnStyles: {
          0:{cellWidth:40},1:{cellWidth:40},2:{cellWidth:28},3:{cellWidth:50},
          4:{cellWidth:25,halign:'center'},5:{cellWidth:25,halign:'center'},6:{cellWidth:32,halign:'center',fontStyle:'bold'}
        },
        alternateRowStyles: { fillColor: [232, 235, 248] },
        styles: { fontSize: 9, cellPadding: 3 },
        margin: { left: 14, right: 14 },
      });
      const pageCount = doc.internal.getNumberOfPages();
      for (let i = 1; i <= pageCount; i++) {
        doc.setPage(i); doc.setFontSize(8); doc.setTextColor(150);
        doc.text('Generado el ' + new Date().toLocaleDateString('es-AR') + ' - Pag. ' + i + ' de ' + pageCount, 14, doc.internal.pageSize.height - 8);
      }
      doc.save('informe-horas_' + (this.obra.nombre||'obra').replace(/ /g,'-') + '_' + this.periodoLabel().replace(/ /g,'-') + '_' + mesNombre + '-' + this.anio + '.pdf');
    },
  },
  async created() { await this.cargarObra(); await this.cargarReporte(); },
};
</script>

<style scoped>
.badge-cat { display:inline-block; padding:2px 8px; border-radius:999px; font-size:0.78rem; font-weight:700; text-transform:capitalize; }
.badge-ayudante              { background:#e5e7eb; color:#374151; }
.badge-medio-oficial         { background:#dbeafe; color:#1e40af; }
.badge-oficial               { background:#dcfce7; color:#15803d; }
.badge-oficial-especializado { background:#fef3c7; color:#b45309; }
.badge-capataz               { background:#fee2e2; color:#dc2626; }
.badge-presente { background:#dcfce7; color:#15803d; border-radius:4px; padding:2px 8px; font-size:0.80rem; font-weight:700; }
.badge-ausente  { background:#fee2e2; color:#dc2626; border-radius:4px; padding:2px 8px; font-size:0.80rem; font-weight:700; }
</style>
