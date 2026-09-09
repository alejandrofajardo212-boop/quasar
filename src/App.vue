<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-avatar icon="build" color="white" text-color="primary" />
        <q-toolbar-title class="text-weight-bold">Taller Don Efraín</q-toolbar-title>
        <q-btn color="secondary" icon="add" label="Nuevo Servicio" @click="abrirCrear" unelevated />
      </q-toolbar>
    </q-header>

    <div v-if="notificacionVisible" class="q-pa-md fixed-top-right z-top" style="max-width: 350px; margin-top: 60px;">
      <q-banner inline-actions class="bg-positive text-white rounded-borders shadow-5">
        <template v-slot:avatar>
          <q-icon name="check_circle" color="white" />
        </template>
        {{ notificacionTexto }}
      </q-banner>
    </div>

    <q-page-container class="bg-grey-2">
      <q-page class="q-pa-md">

        <div class="row q-col-gutter-sm q-mb-md">
          <!-- Total Dinero Recibido -->
          <div class="col-12 col-sm-6 col-md-3">
            <q-card flat bordered class="bg-green-1 text-green-10 text-center q-pa-sm">
              <div class="row items-center justify-center q-gutter-x-xs">
                <q-icon name="attach_money" size="20px" />
                <span class="text-caption text-weight-bold">Total Recibido (Caja)</span>
              </div>
              <div class="text-h5 text-weight-bolder q-mt-xs">${{ calcTotalRecibido().toLocaleString() }}</div>
            </q-card>
          </div>

          <div class="col-12 col-sm-6 col-md-3">
            <q-card flat bordered class="bg-red-1 text-red-10 text-center q-pa-sm">
              <div class="row items-center justify-center q-gutter-x-xs">
                <q-icon name="money_off" size="20px" />
                <span class="text-caption text-weight-bold">Saldo Pendiente (Por Cobrar)</span>
              </div>
              <div class="text-h5 text-weight-bolder q-mt-xs">${{ calcTotalPendiente().toLocaleString() }}</div>
            </q-card>
          </div>

          <div class="col-12 col-sm-6 col-md-3">
            <q-card flat bordered class="bg-blue-1 text-blue-10 text-center q-pa-sm">
              <div class="row items-center justify-center q-gutter-x-xs">
                <q-icon name="build" size="20px" />
                <span class="text-caption text-weight-bold">En Reparación</span>
              </div>
              <div class="text-h5 text-weight-bolder q-mt-xs">{{ countReparacion() }}</div>
            </q-card>
          </div>

          <div class="col-12 col-sm-6 col-md-3">
            <q-card flat bordered class="bg-purple-1 text-purple-10 text-center q-pa-sm">
              <div class="row items-center justify-center q-gutter-x-xs">
                <q-icon name="local_shipping" size="20px" />
                <span class="text-caption text-weight-bold">Despachados / Entregados</span>
              </div>
              <div class="text-h5 text-weight-bolder q-mt-xs">{{ countDespachados() }}</div>
            </q-card>
          </div>
        </div>

        <div v-if="servicios.length === 0" class="text-center q-pa-xl text-grey-7">
          <q-icon name="devices_other" size="80px" color="grey-5" />
          <div class="text-h6 q-mt-md">No hay servicios registrados en el taller.</div>
        </div>

        <div v-else class="row q-col-gutter-md">
          <div v-for="s in servicios" :key="s.id" class="col-12 col-sm-6 col-md-4">
            <q-card flat bordered :class="{
              'bg-red-1 border-red': s.estadoPago === 'Pendiente',
              'bg-amber-1 border-amber': s.estadoPago === 'Abono',
              'bg-white': s.estadoPago === 'Pagado'
            }">
              <q-card-section class="q-pb-xs">
                <div class="row items-center no-wrap">
                  <div class="col">
                    <div class="text-h6 text-weight-bold">{{ s.cliente }}</div>
                    <div class="text-subtitle2 text-grey-8"><q-icon name="smartphone" /> {{ s.equipo }}</div>
                  </div>
                  <div class="col-auto">
                    <q-chip clickable @click="abrirEstado(s)" :color="getColorEquipo(s.estadoEquipo)" text-color="white" size="md" :icon="getIcono(s.estadoEquipo)">
                      {{ s.estadoEquipo }}
                      <q-tooltip>Clic para estado / cobro / entrega</q-tooltip>
                    </q-chip>
                  </div>
                </div>
              </q-card-section>

              <q-separator />

              <q-card-section class="q-py-sm text-body2">
                <div><strong>Reparación:</strong> {{ s.tipoReparacion }} | <strong>Técnico:</strong> {{ s.tecnico }}</div>
                <div><strong>Recepción:</strong> {{ s.fechaHora }}</div>
                
                <div v-if="s.direccion" class="text-caption text-weight-bold text-primary q-mt-xs">
                  <q-icon name="local_shipping" /> Domicilio: {{ s.direccion }}
                </div>

                <div class="q-mt-xs"><strong>Precio:</strong> ${{ (s.precio || 0).toLocaleString() }}</div>
                
                <div v-if="s.estadoPago !== 'Pagado'" class="text-caption text-weight-bold text-negative">
                  Abonado: ${{ getAbonado(s).toLocaleString() }} | Debe: ${{ getPendiente(s).toLocaleString() }}
                </div>

                <div class="q-mt-xs row items-center justify-between">
                  <div>
                    <strong>Pago: </strong>
                    <q-badge :color="getColorPago(s.estadoPago)">{{ s.estadoPago }}</q-badge>
                  </div>
                  <q-btn v-if="getPendiente(s) > 0" size="xs" color="primary" icon="attach_money" label="Cobrar" @click="abrirEstado(s)" unelevated />
                </div>

                <div v-if="isFinal(s.estadoEquipo)" class="q-mt-xs bg-amber-1 q-pa-sm rounded-borders">
                  <div class="row items-center">
                    <strong class="q-mr-xs">Calificación:</strong>
                    <q-rating v-model="s.calificacion" max="5" size="1.2em" color="orange" />
                  </div>
                  <div v-if="s.observaciones" class="text-caption text-italic text-grey-9 q-mt-xs">
                    <strong>Reseña:</strong> "{{ s.observaciones }}"
                  </div>
                </div>
              </q-card-section>

              <q-separator />

              <q-card-actions align="right">
                <q-btn flat round color="primary" icon="edit" @click="abrirEditar(s)"><q-tooltip>Editar</q-tooltip></q-btn>
                <q-btn flat round color="negative" icon="delete" @click="delId = s.id; modalDel = true"><q-tooltip>Eliminar</q-tooltip></q-btn>
              </q-card-actions>
            </q-card>
          </div>
        </div>

        <q-dialog v-model="modalEstado" persistent>
          <q-card style="width: 440px; max-width: 90vw;">
            <q-card-section class="bg-primary text-white row items-center">
              <div class="text-h6">Estado, Cobro y Domicilio</div>
              <q-space /><q-btn icon="close" flat round dense v-close-popup />
            </q-card-section>

            <q-card-section class="q-pa-md q-gutter-y-md" v-if="curServicio">
              <div class="text-subtitle1 text-weight-bold text-center">{{ curServicio.cliente }} - {{ curServicio.equipo }}</div>
              <q-select v-model="curServicio.estadoEquipo" :options="ESTADOS" label="Estado del equipo *" outlined dense />

              <q-input v-model="curServicio.direccion" label="Dirección de envío a domicilio (Opcional)" outlined dense icon="place" />

              <div class="bg-grey-3 q-pa-sm rounded-borders text-body2">
                <div class="row justify-between"><span>Precio total:</span><strong>${{ (curServicio.precio || 0).toLocaleString() }}</strong></div>
                <div class="row justify-between text-positive"><span>Abonado hasta hoy:</span><strong>${{ (curServicio.montoPagado || 0).toLocaleString() }}</strong></div>
                <q-separator class="q-my-xs" />
                <div class="row justify-between text-subtitle2 text-weight-bolder" :class="getPendiente(curServicio) > 0 ? 'text-negative' : 'text-positive'">
                  <span>Falta por pagar:</span><span>${{ getPendiente(curServicio).toLocaleString() }}</span>
                </div>
              </div>

              <div v-if="getPendiente(curServicio) > 0" class="q-gutter-y-xs">
                <div class="row q-col-gutter-xs">
                  <div class="col-7"><q-input v-model.number="inputAbono" type="number" label="Monto a abonar" outlined dense /></div>
                  <div class="col-5"><q-btn color="secondary" label="Pagar Todo" size="sm" class="full-width fit" @click="curServicio.montoPagado = curServicio.precio" unelevated /></div>
                </div>
                <q-btn v-if="inputAbono > 0" color="positive" icon="add" label="Sumar Abono" size="sm" class="full-width" @click="sumarAbonoModal" unelevated />
              </div>

              <div v-if="isFinal(curServicio.estadoEquipo) && getPendiente(curServicio) > 0" class="q-pa-sm bg-red-1 text-negative border-red rounded-borders row items-center">
                <q-icon name="error" size="24px" class="q-mr-xs" />
                <div class="text-caption text-weight-bold">
                  ¡Atención! Debe completar el pago (${{ getPendiente(curServicio).toLocaleString() }}) antes de realizar el envío o entrega.
                </div>
              </div>

              <div v-if="isFinal(curServicio.estadoEquipo) && getPendiente(curServicio) === 0" class="bg-amber-1 q-pa-sm rounded-borders text-center q-gutter-y-sm">
                <div class="text-caption text-weight-bold">Calificación del cliente</div>
                <q-rating v-model="curServicio.calificacion" max="5" size="2em" color="orange" />
                
                <q-input
                  v-model="curServicio.observaciones"
                  label="Comentario / Reseña del cliente (Opcional)"
                  type="textarea"
                  outlined
                  dense
                  rows="2"
                  bg-color="white"
                />

                <div v-if="curServicio.calificacion > 0" class="text-subtitle2 text-positive text-weight-bold q-mt-xs">
                  <q-icon name="check_circle" /> ¡Gracias por su reseña!
                </div>
              </div>
            </q-card-section>

            <q-card-actions align="right" class="q-pa-md">
              <q-btn label="Cancelar" color="grey" flat v-close-popup />
              <q-btn label="Guardar" color="primary" @click="saveEstado" :disable="curServicio && isFinal(curServicio.estadoEquipo) && getPendiente(curServicio) > 0" />
            </q-card-actions>
          </q-card>
        </q-dialog>

        <q-dialog v-model="modalForm" persistent>
          <q-card style="width: 600px; max-width: 90vw;">
            <q-card-section class="bg-primary text-white row items-center">
              <div class="text-h6">{{ editId !== null ? 'Editar Servicio' : 'Nuevo Servicio' }}</div>
              <q-space /><q-btn icon="close" flat round dense v-close-popup />
            </q-card-section>

            <q-form @submit="saveServicio" class="q-gutter-md q-pa-md">
              <q-input v-model="form.cliente" label="Cliente *" outlined dense :rules="[val => !!val || 'Obligatorio']" />
              <q-input v-model="form.equipo" label="Equipo (Ej: iPhone 12) *" outlined dense :rules="[val => !!val || 'Obligatorio']" />
              <q-input v-model="form.direccion" label="Dirección de envío a domicilio (Opcional)" outlined dense icon="place" />

              <div class="row q-col-gutter-sm">
                <div class="col-6"><q-select v-model="form.tipoReparacion" :options="REPARACIONES" label="Reparación *" outlined dense /></div>
                <div class="col-6"><q-select v-model="form.tecnico" :options="TECNICOS" label="Técnico *" outlined dense /></div>
              </div>

              <div class="row q-col-gutter-sm">
                <div class="col-4"><q-input v-model="form.fechaHora" label="Fecha *" type="datetime-local" outlined dense stack-label /></div>
                <div class="col-4"><q-input v-model.number="form.precio" label="Precio ($) *" type="number" outlined dense :rules="[val => val >= 0 || 'Inválido']" /></div>
                <div class="col-4"><q-input v-model.number="form.montoPagado" label="Abonado ($) *" type="number" outlined dense :rules="[val => val <= (form.precio || 0) || 'Excede precio']" /></div>
              </div>

              <div class="row q-col-gutter-sm">
                <div class="col-6"><q-select v-model="form.metodoPago" :options="['Efectivo','Transferencia','Tarjeta']" label="Método pago *" outlined dense /></div>
                <div class="col-6"><q-select v-model="form.estadoEquipo" :options="ESTADOS" label="Estado equipo *" outlined dense /></div>
              </div>

              <div v-if="isFinal(form.estadoEquipo) && getPendiente(form) === 0" class="q-pa-sm bg-amber-1 rounded-borders text-center q-gutter-y-xs">
                <div class="text-caption text-grey-8">Calificación del cliente</div>
                <q-rating v-model="form.calificacion" max="5" size="1.8em" color="orange" />
                <q-input v-model="form.observaciones" label="Comentario / Reseña del cliente" type="textarea" outlined dense rows="2" bg-color="white" />
              </div>

              <div v-if="isFinal(form.estadoEquipo) && getPendiente(form) > 0" class="q-pa-sm bg-red-1 text-negative border-red rounded-borders text-caption text-weight-bold">
                ¡Atención! Debe abonar el total (${{ getPendiente(form).toLocaleString() }}) antes de despachar o entregar.
              </div>

              <div class="row justify-end q-gutter-sm">
                <q-btn label="Cancelar" color="grey" flat v-close-popup />
                <q-btn label="Guardar" color="primary" type="submit" :disable="isFinal(form.estadoEquipo) && getPendiente(form) > 0" />
              </div>
            </q-form>
          </q-card>
        </q-dialog>

        <q-dialog v-model="modalDel" persistent>
          <q-card>
            <q-card-section class="row items-center">
              <q-avatar icon="warning" color="negative" text-color="white" />
              <span class="q-ml-sm text-body1">¿Desea eliminar este registro?</span>
            </q-card-section>
            <q-card-actions align="right">
              <q-btn flat label="Cancelar" color="grey" v-close-popup />
              <q-btn flat label="Eliminar" color="negative" @click="confirmarEliminar" />
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

const getNowIso = () => new Date().toISOString().slice(0, 16)
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
  form.value = { cliente: '', equipo: '', direccion: '', tipoReparacion: 'Cambio de pantalla', tecnico: 'Don Efraín', fechaHora: getNowIso(), precio: 200, montoPagado: 0, metodoPago: 'Efectivo', estadoPago: 'Pendiente', estadoEquipo: 'Recibido', calificacion: 0, observaciones: '' }
  modalForm.value = true
}

function abrirEditar(item) {
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
  if (!isFinal(form.value.estadoEquipo)) form.value.calificacion = 0

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
  if (!isFinal(s.estadoEquipo)) s.calificacion = 0

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

const getColorEquipo = e => ({ Recibido: 'orange-8', 'En reparación': 'blue-8', 'Listo para entregar': 'purple-8', Despachado: 'teal-8', Entregado: 'positive' })[e] || 'grey'
const getColorPago = p => ({ Pagado: 'positive', Abono: 'warning', Pendiente: 'negative' })[p] || 'grey'
const getIcono = e => ({ Recibido: 'inbox', 'En reparación': 'build', 'Listo para entregar': 'notifications', Despachado: 'local_shipping', Entregado: 'check_circle' })[e] || 'devices'
</script>

<style scoped>
.border-red { border: 2px solid #f44336 !important; }
.border-amber { border: 2px solid #ffc107 !important; }
</style>