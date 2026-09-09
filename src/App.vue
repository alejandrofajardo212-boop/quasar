<template>
  <q-layout view="lHh Lpr lFf">
    <!-- ENCABEZADO PRINCIPAL -->
    <q-header elevated class="bg-primary text-white">
      <q-toolbar class="q-py-xs">
        <q-avatar icon="build" color="white" text-color="primary" size="42px" />
        <q-toolbar-title class="text-h5 text-weight-bold q-ml-sm">Taller Don Efraín</q-toolbar-title>
        <q-btn color="secondary" icon="add" label="Nuevo Servicio" size="md" @click="abrirCrear" unelevated class="text-weight-bold" />
      </q-toolbar>
    </q-header>

    <!-- NOTIFICACIÓN EN PANTALLA -->
    <div v-if="notificacionVisible" class="q-pa-md fixed-top-right z-top" style="max-width: 380px; margin-top: 60px;">
      <q-banner inline-actions class="bg-positive text-white rounded-borders shadow-5 text-body1">
        <template v-slot:avatar>
          <q-icon name="check_circle" color="white" size="28px" />
        </template>
        {{ notificacionTexto }}
      </q-banner>
    </div>

    <q-page-container class="bg-grey-2">
      <q-page class="q-pa-md">

        <!-- TARJETAS DE MÉTRICAS -->
        <div class="row q-col-gutter-md q-mb-lg">
          <div class="col-12 col-sm-6 col-md-3">
            <q-card flat bordered class="bg-green-1 text-green-10 text-center q-pa-md shadow-1">
              <div class="row items-center justify-center q-gutter-x-xs">
                <q-icon name="attach_money" size="24px" />
                <span class="text-subtitle2 text-weight-bold">Total Recibido (Caja)</span>
              </div>
              <div class="text-h4 text-weight-bolder q-mt-xs">${{ calcTotalRecibido().toLocaleString() }}</div>
            </q-card>
          </div>

          <div class="col-12 col-sm-6 col-md-3">
            <q-card flat bordered class="bg-red-1 text-red-10 text-center q-pa-md shadow-1">
              <div class="row items-center justify-center q-gutter-x-xs">
                <q-icon name="money_off" size="24px" />
                <span class="text-subtitle2 text-weight-bold">Saldo Pendiente</span>
              </div>
              <div class="text-h4 text-weight-bolder q-mt-xs">${{ calcTotalPendiente().toLocaleString() }}</div>
            </q-card>
          </div>

          <div class="col-12 col-sm-6 col-md-3">
            <q-card flat bordered class="bg-blue-1 text-blue-10 text-center q-pa-md shadow-1">
              <div class="row items-center justify-center q-gutter-x-xs">
                <q-icon name="build" size="24px" />
                <span class="text-subtitle2 text-weight-bold">En Reparación</span>
              </div>
              <div class="text-h4 text-weight-bolder q-mt-xs">{{ countReparacion() }}</div>
            </q-card>
          </div>

          <div class="col-12 col-sm-6 col-md-3">
            <q-card flat bordered class="bg-purple-1 text-purple-10 text-center q-pa-md shadow-1">
              <div class="row items-center justify-center q-gutter-x-xs">
                <q-icon name="local_shipping" size="24px" />
                <span class="text-subtitle2 text-weight-bold">Despachados / Entregados</span>
              </div>
              <div class="text-h4 text-weight-bolder q-mt-xs">{{ countDespachados() }}</div>
            </q-card>
          </div>
        </div>

        <!-- MENSAJE SIN REGISTROS -->
        <div v-if="servicios.length === 0" class="text-center q-pa-xl text-grey-7">
          <q-icon name="devices_other" size="100px" color="grey-5" />
          <div class="text-h5 q-mt-md text-weight-medium">No hay servicios registrados en el taller.</div>
        </div>

        <!-- LISTADO DE TARJETAS -->
        <div v-else class="row q-col-gutter-md">
          <div v-for="s in servicios" :key="s.id" class="col-12 col-sm-6 col-md-4">
            <q-card flat bordered class="shadow-2" :class="{
              'bg-red-1 border-red': s.estadoPago === 'Pendiente',
              'bg-amber-1 border-amber': s.estadoPago === 'Abono',
              'bg-white border-green': s.estadoPago === 'Pagado'
            }">
              <q-card-section class="q-pb-sm">
                <div class="row items-center no-wrap">
                  <div class="col">
                    <div class="text-h6 text-weight-bold text-grey-9">{{ s.cliente }}</div>
                    <div class="text-subtitle1 text-primary text-weight-bold">
                      <q-icon name="smartphone" size="20px" /> {{ s.marca }} {{ s.modelo }}
                    </div>
                  </div>
                  <div class="col-auto">
                    <q-chip clickable @click="abrirEstado(s)" :color="getColorEquipo(s.estadoEquipo)" text-color="white" size="md" :icon="getIcono(s.estadoEquipo)" class="text-weight-bold">
                      {{ s.estadoEquipo }}
                      <q-tooltip class="text-body2">Clic para cambiar estado o cobro</q-tooltip>
                    </q-chip>
                  </div>
                </div>
              </q-card-section>

              <q-separator />

              <q-card-section class="q-py-md text-body1">
                <div class="q-mb-xs"><strong>Reparación:</strong> {{ s.tipoReparacion }}</div>
                <div class="q-mb-xs"><strong>Técnico:</strong> {{ s.tecnico }}</div>
                <div class="q-mb-xs text-grey-8"><strong>Recepción:</strong> {{ s.fechaHora.replace('T', ' ') }}</div>
                
                <div v-if="s.direccion" class="text-body2 text-weight-bold text-primary q-mt-xs bg-blue-1 q-pa-xs rounded-borders">
                  <q-icon name="local_shipping" size="18px" /> Domicilio: {{ s.direccion }}
                </div>

                <div class="q-mt-sm text-subtitle1"><strong>Precio:</strong> <span class="text-weight-bold">${{ (s.precio || 0).toLocaleString() }}</span></div>
                
                <div v-if="s.estadoPago !== 'Pagado'" class="text-body2 text-weight-bold text-negative bg-red-1 q-pa-xs rounded-borders q-mt-xs">
                  Abonado: ${{ getAbonado(s).toLocaleString() }} | Debe: ${{ getPendiente(s).toLocaleString() }}
                </div>

                <div class="q-mt-sm row items-center justify-between">
                  <div>
                    <strong>Pago: </strong>
                    <q-badge :color="getColorPago(s.estadoPago)" class="text-subtitle2 q-px-sm">{{ s.estadoPago }}</q-badge>
                  </div>
                  <q-btn v-if="getPendiente(s) > 0" size="sm" color="primary" icon="attach_money" label="Cobrar" @click="abrirEstado(s)" unelevated class="text-weight-bold" />
                </div>

                <!-- ALERTA SI NO HA SIDO ENTREGADO AÚN -->
                <div v-if="s.estadoEquipo !== 'Entregado'" class="q-mt-sm text-caption text-weight-bold text-orange-9 bg-orange-1 q-pa-xs rounded-borders text-center">
                  <q-icon name="schedule" /> {{ s.estadoEquipo === 'Despachado' ? 'En camino (Sin recibir por cliente)' : 'Equipo en taller (Sin entregar)' }}
                </div>

                <!-- CALIFICACIÓN SOLO Y ÚNICAMENTE SI ESTÁ ENTREGADO -->
                <div v-if="s.estadoEquipo === 'Entregado'" class="q-mt-md bg-amber-2 q-pa-sm rounded-borders">
                  <div class="row items-center">
                    <strong class="q-mr-xs text-body2">Calificación del Cliente:</strong>
                    <q-rating v-model="s.calificacion" max="5" size="1.4em" color="orange" readonly />
                  </div>
                  <div v-if="s.observaciones" class="text-body2 text-italic text-grey-9 q-mt-xs">
                    <strong>Reseña:</strong> "{{ s.observaciones }}"
                  </div>
                </div>
              </q-card-section>

              <q-separator />

              <q-card-actions align="between" class="q-px-md">
                <span v-if="s.estadoEquipo === 'Entregado'" class="text-caption text-weight-bold text-negative">
                  <q-icon name="lock" /> Registro Finalizado
                </span>
                <span v-else></span>

                <div>
                  <q-btn flat round color="primary" icon="edit" size="md" @click="abrirEditar(s)" :disable="s.estadoEquipo === 'Entregado'">
                    <q-tooltip class="text-body2">{{ s.estadoEquipo === 'Entregado' ? 'Servicio entregado (No editable)' : 'Editar Servicio' }}</q-tooltip>
                  </q-btn>
                  <q-btn flat round color="negative" icon="delete" size="md" @click="delId = s.id; modalDel = true" :disable="s.estadoEquipo === 'Entregado'">
                    <q-tooltip class="text-body2">{{ s.estadoEquipo === 'Entregado' ? 'Servicio entregado (No eliminable)' : 'Eliminar Servicio' }}</q-tooltip>
                  </q-btn>
                </div>
              </q-card-actions>
            </q-card>
          </div>
        </div>

        <!-- MODAL DE CAMBIO DE ESTADO Y COBRO -->
        <q-dialog v-model="modalEstado" persistent>
          <q-card style="width: 460px; max-width: 90vw;">
            <q-card-section class="bg-primary text-white row items-center">
              <div class="text-h6">Estado, Cobro y Entrega</div>
              <q-space />
              <q-btn icon="close" flat round dense @click="modalEstado = false" />
            </q-card-section>

            <q-card-section class="q-pa-md q-gutter-y-md" v-if="curServicio">
              <div class="text-subtitle1 text-weight-bold text-center text-primary">
                {{ curServicio.cliente }} - {{ curServicio.marca }} {{ curServicio.modelo }}
              </div>
              
              <q-select v-model="curServicio.estadoEquipo" :options="ESTADOS" label="Estado del equipo *" outlined dense class="text-body1" />

              <q-input v-model="curServicio.direccion" label="Dirección de envío a domicilio (Opcional)" outlined dense icon="place" />

              <div class="bg-grey-3 q-pa-md rounded-borders text-body1">
                <div class="row justify-between"><span>Precio total:</span><strong>${{ (curServicio.precio || 0).toLocaleString() }}</strong></div>
                <div class="row justify-between text-positive"><span>Abonado hasta hoy:</span><strong>${{ (curServicio.montoPagado || 0).toLocaleString() }}</strong></div>
                <q-separator class="q-my-xs" />
                <div class="row justify-between text-subtitle1 text-weight-bolder" :class="getPendiente(curServicio) > 0 ? 'text-negative' : 'text-positive'">
                  <span>Falta por pagar:</span><span>${{ getPendiente(curServicio).toLocaleString() }}</span>
                </div>
              </div>

              <div v-if="getPendiente(curServicio) > 0" class="q-gutter-y-xs">
                <div class="row q-col-gutter-xs">
                  <div class="col-7"><q-input v-model.number="inputAbono" type="number" label="Monto a abonar ($)" outlined dense /></div>
                  <div class="col-5"><q-btn color="secondary" label="Pagar Todo" size="sm" class="full-width fit text-weight-bold" @click="curServicio.montoPagado = curServicio.precio" unelevated /></div>
                </div>
                <q-btn v-if="inputAbono > 0" color="positive" icon="add" label="Sumar Abono" size="sm" class="full-width text-weight-bold" @click="sumarAbonoModal" unelevated />
              </div>

              <!-- MENSAJE DE BLOQUEO DE COBRO -->
              <div v-if="isFinal(curServicio.estadoEquipo) && getPendiente(curServicio) > 0" class="q-pa-sm bg-red-1 text-negative border-red rounded-borders row items-center">
                <q-icon name="error" size="24px" class="q-mr-xs" />
                <div class="text-body2 text-weight-bold">
                  ¡Atención! Debe cancelar el total (${{ getPendiente(curServicio).toLocaleString() }}) antes de despachar o entregar.
                </div>
              </div>

              <!-- HABILITACIÓN DE CALIFICACIÓN: SOLO SI 'ENTREGADO' Y SALDO $0 -->
              <div v-if="curServicio.estadoEquipo === 'Entregado' && getPendiente(curServicio) === 0" class="bg-amber-1 q-pa-md rounded-borders text-center q-gutter-y-sm">
                <div class="text-subtitle2 text-weight-bold text-grey-9">
                  <q-icon name="stars" color="orange" size="20px" /> Calificación del Cliente (Equipo recibido)
                </div>
                <q-rating v-model="curServicio.calificacion" max="5" size="2em" color="orange" />
                <q-input v-model="curServicio.observaciones" label="Comentario / Reseña del cliente" type="textarea" outlined dense rows="2" bg-color="white" />
              </div>
            </q-card-section>

            <q-card-actions align="right" class="q-pa-md">
              <q-btn label="Cancelar" color="grey" flat size="md" @click="modalEstado = false" />
              <q-btn label="Guardar Cambios" color="primary" @click="saveEstado" :disable="curServicio && isFinal(curServicio.estadoEquipo) && getPendiente(curServicio) > 0" size="md" unelevated class="text-weight-bold" />
            </q-card-actions>
          </q-card>
        </q-dialog>

        <!-- MODAL FORMULARIO (NUEVO Y EDITAR) -->
        <q-dialog v-model="modalForm" persistent>
          <q-card style="width: 620px; max-width: 90vw;">
            <q-card-section class="bg-primary text-white row items-center">
              <div class="text-h6">{{ editId !== null ? 'Editar Servicio' : 'Nuevo Servicio' }}</div>
              <q-space />
              <q-btn icon="close" flat round dense @click="modalForm = false" />
            </q-card-section>

            <q-form @submit="saveServicio" class="q-gutter-md q-pa-md">
              <q-input v-model="form.cliente" label="Nombre del Cliente *" outlined dense :rules="[val => !!val && val.trim() !== '' || 'El nombre es obligatorio']" />
              
              <div class="row q-col-gutter-sm">
                <div class="col-6">
                  <q-select v-model="form.marca" :options="MARCAS" label="Marca *" outlined dense :rules="[val => !!val || 'Seleccione marca']" />
                </div>
                <div class="col-6">
                  <q-input v-model="form.modelo" label="Modelo *" outlined dense :rules="[val => !!val && val.trim() !== '' || 'El modelo es obligatorio']" />
                </div>
              </div>

              <q-input v-model="form.direccion" label="Dirección de envío a domicilio (Opcional)" outlined dense icon="place" />

              <div class="row q-col-gutter-sm">
                <div class="col-6">
                  <q-select v-model="form.tipoReparacion" :options="REPARACIONES" label="Tipo de Reparación *" outlined dense :rules="[val => !!val || 'Seleccione reparación']" />
                </div>
                <div class="col-6">
                  <q-select v-model="form.tecnico" :options="TECNICOS" label="Técnico *" outlined dense :rules="[val => !!val || 'Seleccione técnico']" />
                </div>
              </div>

              <div class="row q-col-gutter-sm">
                <div class="col-4">
                  <q-input v-model="form.fechaHora" label="Fecha Recepción" type="datetime-local" outlined dense readonly stack-label />
                </div>
                <div class="col-4">
                  <q-input v-model.number="form.precio" label="Precio Total ($) *" type="number" outlined dense :rules="[val => val !== null && val !== '' && val >= 0 || 'Ingrese precio válido']" />
                </div>
                <div class="col-4">
                  <q-input v-model.number="form.montoPagado" label="Abono ($) *" type="number" outlined dense :rules="[val => val !== null && val >= 0 && val <= (form.precio || 0) || 'Inválido']" />
                </div>
              </div>

              <div class="row q-col-gutter-sm">
                <div class="col-6">
                  <q-select v-model="form.metodoPago" :options="['Efectivo','Transferencia','Tarjeta']" label="Método de Pago *" outlined dense :rules="[val => !!val || 'Obligatorio']" />
                </div>
                <div class="col-6">
                  <q-select v-model="form.estadoEquipo" :options="ESTADOS" label="Estado del Equipo *" outlined dense :disable="editId === null" :rules="[val => !!val || 'Obligatorio']" />
                </div>
              </div>

              <!-- HABILITACIÓN DE CALIFICACIÓN SOLO SI ES 'ENTREGADO' -->
              <div v-if="form.estadoEquipo === 'Entregado' && getPendiente(form) === 0" class="q-pa-sm bg-amber-1 rounded-borders text-center q-gutter-y-xs">
                <div class="text-subtitle2">Calificación del cliente</div>
                <q-rating v-model="form.calificacion" max="5" size="1.8em" color="orange" />
                <q-input v-model="form.observaciones" label="Comentario / Reseña del cliente" type="textarea" outlined dense rows="2" bg-color="white" />
              </div>

              <div v-if="isFinal(form.estadoEquipo) && getPendiente(form) > 0" class="q-pa-sm bg-red-1 text-negative border-red rounded-borders text-body2 text-weight-bold">
                ¡Atención! Debe cancelar el precio total antes de despachar o entregar.
              </div>

              <div class="row justify-end q-gutter-sm q-mt-md">
                <q-btn label="Cancelar" color="grey" flat size="md" @click="modalForm = false" />
                <q-btn label="Guardar Servicio" color="primary" type="submit" :disable="isFinal(form.estadoEquipo) && getPendiente(form) > 0" size="md" unelevated class="text-weight-bold" />
              </div>
            </q-form>
          </q-card>
        </q-dialog>

        <!-- MODAL CONFIRMACIÓN DE ELIMINACIÓN -->
        <q-dialog v-model="modalDel" persistent>
          <q-card>
            <q-card-section class="row items-center q-pa-md">
              <q-avatar icon="warning" color="negative" text-color="white" />
              <span class="q-ml-md text-body1 text-weight-bold">¿Desea eliminar este registro del taller?</span>
            </q-card-section>
            <q-card-actions align="right" class="q-pa-md">
              <q-btn flat label="Cancelar" color="grey" size="md" @click="modalDel = false" />
              <q-btn label="Eliminar" color="negative" @click="confirmarEliminar" size="md" unelevated />
            </q-card-actions>
          </q-card>
        </q-dialog>

      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core'

const TECNICOS = ['Don Efraín', 'Técnico 1', 'Técnico 2']
const MARCAS = ['Samsung', 'Apple', 'Xiaomi', 'Motorola', 'Huawei', 'Honor', 'Oppo', 'Realme', 'ZTE', 'Otra']
const REPARACIONES = ['Cambio de pantalla', 'Cambio de batería', 'Cambio de pin de carga', 'Liberación', 'Mantenimiento de software', 'Cambio de flex', 'Otros']
const ESTADOS = ['Recibido', 'En reparación', 'Listo para entregar', 'Despachado', 'Entregado']

const servicios = useLocalStorage('taller_efrain_servicios_v1', [])

const modalForm = ref(false)
const modalDel = ref(false)
const modalEstado = ref(false)
const editId = ref(null)
const delId = ref(null)
const curServicio = ref(null)
const inputAbono = ref(0)
const form = ref({})

const notificacionVisible = ref(false)
const notificacionTexto = ref('')

function lanzarNotificacion(msg) {
  notificacionTexto.value = msg
  notificacionVisible.value = true
  setTimeout(() => {
    notificacionVisible.value = false
  }, 3500)
}

const getNowIso = () => {
  const now = new Date()
  now.setMinutes(now.getMinutes() - now.getTimezoneOffset())
  return now.toISOString().slice(0, 16)
}

const getAbonado = s => !s ? 0 : (s.montoPagado !== undefined ? s.montoPagado : (s.estadoPago === 'Pagado' ? (s.precio || 0) : 0))
const getPendiente = s => !s ? 0 : Math.max(0, (s.precio || 0) - getAbonado(s))
const calcEstadoPago = (precio, abonado) => (abonado >= precio && precio > 0) ? 'Pagado' : (abonado > 0 ? 'Abono' : 'Pendiente')
const isFinal = e => e === 'Entregado' || e === 'Despachado'

function calcTotalRecibido() {
  let total = 0
  for (let i = 0; i < servicios.value.length; i++) {
    total += getAbonado(servicios.value[i])
  }
  return total
}

function calcTotalPendiente() {
  let total = 0
  for (let i = 0; i < servicios.value.length; i++) {
    total += getPendiente(servicios.value[i])
  }
  return total
}

const countReparacion = () => servicios.value.filter(s => s.estadoEquipo === 'En reparación').length
const countDespachados = () => servicios.value.filter(s => isFinal(s.estadoEquipo)).length

function abrirCrear() {
  editId.value = null
  form.value = { 
    cliente: '', 
    marca: 'Samsung',
    modelo: '',
    direccion: '', 
    tipoReparacion: 'Cambio de pantalla', 
    tecnico: 'Don Efraín', 
    fechaHora: getNowIso(), 
    precio: null, 
    montoPagado: 0, 
    metodoPago: 'Efectivo', 
    estadoPago: 'Pendiente', 
    estadoEquipo: 'Recibido', 
    calificacion: 0, 
    observaciones: '' 
  }
  modalForm.value = true
}

function abrirEditar(item) {
  if (item.estadoEquipo === 'Entregado') return
  editId.value = item.id
  form.value = { ...item, montoPagado: getAbonado(item) }
  modalForm.value = true
}

function abrirEstado(item) {
  curServicio.value = JSON.parse(JSON.stringify(item))
  curServicio.value.montoPagado = getAbonado(item)
  inputAbono.value = 0
  modalEstado.value = true
}

function sumarAbonoModal() {
  if (inputAbono.value > 0 && curServicio.value) {
    const total = curServicio.value.precio || 0
    curServicio.value.montoPagado = Math.min(total, (curServicio.value.montoPagado || 0) + inputAbono.value)
    inputAbono.value = 0
  }
}

function saveServicio() {
  if (isFinal(form.value.estadoEquipo) && getPendiente(form.value) > 0) return
  form.value.estadoPago = calcEstadoPago(form.value.precio || 0, form.value.montoPagado || 0)
  
  // Si no está Entregado, limpia la calificación por seguridad
  if (form.value.estadoEquipo !== 'Entregado') form.value.calificacion = 0

  if (editId.value !== null) {
    const idx = servicios.value.findIndex(s => s.id === editId.value)
    if (idx !== -1) servicios.value[idx] = { ...form.value, id: editId.value }
    lanzarNotificacion(`¡Servicio de ${form.value.cliente} actualizado!`)
  } else {
    servicios.value.push({ ...form.value, id: Date.now() })
    lanzarNotificacion(`¡Nuevo servicio de ${form.value.cliente} registrado!`)
  }
  modalForm.value = false
}

function saveEstado() {
  const s = curServicio.value
  if (!s || (isFinal(s.estadoEquipo) && getPendiente(s) > 0)) return
  s.estadoPago = calcEstadoPago(s.precio || 0, s.montoPagado || 0)
  
  // Si no está Entregado, limpia la calificación por seguridad
  if (s.estadoEquipo !== 'Entregado') s.calificacion = 0

  const idx = servicios.value.findIndex(item => item.id === s.id)
  if (idx !== -1) {
    servicios.value[idx] = { ...s }
    lanzarNotificacion(`¡Estado guardado como "${s.estadoEquipo}"!`)
  }
  modalEstado.value = false
}

function confirmarEliminar() {
  servicios.value = servicios.value.filter(s => s.id !== delId.value)
  modalDel.value = false
  lanzarNotificacion('Servicio eliminado correctamente.')
}

const getColorEquipo = e => ({ Recibido: 'orange-9', 'En reparación': 'blue-8', 'Listo para entregar': 'purple-8', Despachado: 'teal-8', Entregado: 'positive' })[e] || 'grey'
const getColorPago = p => ({ Pagado: 'positive', Abono: 'warning', Pendiente: 'negative' })[p] || 'grey'
const getIcono = e => ({ Recibido: 'inbox', 'En reparación': 'build', 'Listo para entregar': 'notifications', Despachado: 'local_shipping', Entregado: 'check_circle' })[e] || 'devices'
</script>

<style scoped>
.border-red { border: 2px solid #f44336 !important; }
.border-amber { border: 2px solid #ffc107 !important; }
.border-green { border: 1px solid #4caf50 !important; }

.text-body1 { font-size: 1.05rem !important; line-height: 1.5; }
.text-body2 { font-size: 0.95rem !important; }
</style>