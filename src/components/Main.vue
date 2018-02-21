<template>
  <q-layout
    ref="layout"
    view="lHh Lpr fff"
    :left-class="{'bg-grey-2': true}"
  >
    <div>
      <!--En la versión anterior tenía el hack en node-modules. Corregido -->
      <q-toolbar slot="header" class="glossy" style="background: #bf360c">
        <img src="statics/logo.png" />
        <q-toolbar-title>
          {{evaluacion.ejerName}}
          <div slot="subtitle">
            {{evaluacion.id}}
          </div>
        </q-toolbar-title>
      </q-toolbar>
    </div>
    evaluacion:Evaluacion->Evaluacion_v1.vue
    <evaluacion
      :evaluacion="evaluacion"
      :point="point"
      :questions="newPreguntas"
      :userName="getFormName"
      v-if="configured"
      @closeUnit="closeNavigatorWindow"
      @unitCompleted="completeThisUnit"
    />
  </q-layout>
</template>

<script>
import Evaluacion from './Evaluacion_v1.vue'
import axios from 'axios'
import {
  dom,
  event,
  Toast,
  QLayout,
  QToolbar,
  QToolbarTitle,
  Events
} from 'quasar'

var saw = require('scorm-api-wrapper')

saw.configure()

const { viewport } = dom,
  { position } = event,
  moveForce = 30,
  rotateForce = 40,
  RAD_TO_DEG = 180 / Math.PI

function getRotationFromAccel (accelX, accelY, accelZ) {
  /* Reference: http://stackoverflow.com/questions/3755059/3d-accelerometer-calculate-the-orientation#answer-30195572 */
  const sign = accelZ > 0 ? 1 : -1
  const miu = 0.001

  return {
    roll:
      Math.atan2(
        accelY,
        sign * Math.sqrt(Math.pow(accelZ, 2) + miu * Math.pow(accelX, 2))
      ) * RAD_TO_DEG,
    pitch:
      -Math.atan2(
      accelX,
      Math.sqrt(Math.pow(accelY, 2) + Math.pow(accelZ, 2))
    ) * RAD_TO_DEG
  }
}

export default {
  components: {
    QLayout,
    QToolbar,
    QToolbarTitle,
    Events,
    Evaluacion: Evaluacion
  },
  data () {
    return {
      orienting: window.DeviceOrientationEvent && !this.$q.platform.is.desktop,
      rotating: window.DeviceMotionEvent && !this.$q.platform.is.desktop,
      moveX: 0,
      moveY: 0,
      rotateY: 0,
      rotateX: 0,
      point: 1,
      configured: false,
      evaluacion: {},
      evalConfigLocation: './statics/open_questionary/data_model.json',
      lmsData: {
        tini: null,
        tfin: null,
        tStringInvertido: null,
        tinvertido: null,
        scormApi: false,
        simApi: false,
        autoStart: true,
        status: 'Not Atempted',
        userName: 'No Name'
      },
      newPreguntas: {
        PREGUNTAS: []
      }
    }
  },
  computed: {
    position () {
      const transform = `rotateX(${this.rotateX}deg) rotateY(${this.rotateY}deg)`
      return {
        top: this.moveY + 'px',
        left: this.moveX + 'px',
        '-webkit-transform': transform,
        '-ms-transform': transform,
        transform
      }
    },
    getFormName () {
      // Get user name fro scormAPI
      return this.lmsData.userName
    }
  },
  methods: {
    reorderQuestions () {
      // console.info('ReorderQuestions(IN)')
      // console.log(this.evaluacion.PREGUNTAS)
      var listado = this.evaluacion.PREGUNTAS.slice(0)
      var lon = listado.length
      for (var i = 0; i < lon; i++) {
        var aleatorio = Math.floor(Math.random() * listado.length)
        // var seleccion = listado[aleatorio]
        // console.info(seleccion)
        this.newPreguntas.PREGUNTAS.push(listado.splice(aleatorio, 1))
      }
      // console.info('new Array reordered:')
      // console.log(this.newPreguntas)
    },
    completeThisUnit () {
      // console.info('Unidad completada', this.unidades[this.indexUnit].isCompleted, this.indexUnit)
      // this.unidades[this.indexUnit].isCompleted = true
      // this.state = 4
      // console.info('Unidad completada', this.unidades[this.indexUnit].isCompleted, this.indexUnit)
      // console.info('Esta unidad ha sido completada')
      this.completed()
      this.getIntervalo()
      this.timeSet()
      this.conmit()
      this.finish()
      this.closeNavigatorWindow()
    },
    getIntervalo () {
      if (this.lmsData.tini) {
        var thisEnd = new Date()
        var intervalo = new Date(thisEnd - this.lmsData.tini)
        // console.info(typeof (intervalo))
        // Si ya existe un contaje de tiempo lo añade.
        if (this.lmsData.tinvertido) {
          this.lmsData.tinvertido = new Date(
            this.lmsData.tinvertido.getTime() + intervalo.getTime()
          )
        }
        else this.lmsData.tinvertido = new Date(intervalo)

        intervalo = new Date(this.lmsData.tinvertido.getTime())
        // console.info(typeof (intervalo))
        var horas = String(intervalo.getHours() - 1)
        if (horas.length < 2) horas = '0' + horas
        // console.info(horas)

        var minutos = String(intervalo.getMinutes())
        if (minutos.length < 2) minutos = '0' + minutos
        // console.info(minutos)

        var segundos = String(intervalo.getSeconds())
        if (segundos.length < 2) segundos = '0' + segundos
        // console.info(segundos)

        var tinvertido = horas + ':' + minutos + ':' + segundos
        this.lmsData.tini = thisEnd
        this.lmsData.tStringInvertido = tinvertido
      }
    },
    initialize () {
      saw.lmsInitialize()
    },
    incompleted () {
      saw.setScormValue('cmi.core.lesson_status', 'incomplete')
    },
    completed () {
      saw.setScormValue('cmi.core.lesson_status', 'completed')
    },
    getName () {
      var apeNombre = saw.getScormValue('cmi.core.student_name')
      var arrApeNombre = apeNombre.split(',')
      if (arrApeNombre.length === 2) this.lmsData.userName = arrApeNombre[1]
      else this.lmsData.userName = apeNombre
    },
    getId () {
      this.lmsData.userName = saw.getScormValue('cmi.core.student_id')
    },
    getStatus () {
      this.lmsData.status = saw.getScormValue('cmi.core.lesson_status')
    },
    conmit () {
      saw.lmsCommit()
    },
    timeSet () {
      saw.setScormValue('cmi.core.session_time', this.lmsData.tStringInvertido)
    },
    finish () {
      saw.lmsFinish()
    },
    simApi (value) {
      this.lmsData.simApi = value
    },
    move (evt) {
      const { width, height } = viewport(),
        { top, left } = position(evt),
        halfH = height / 2,
        halfW = width / 2

      this.moveX = (left - halfW) / halfW * -moveForce
      this.moveY = (top - halfH) / halfH * -moveForce
      this.rotateY = left / width * rotateForce * 2 - rotateForce
      this.rotateX = -(top / height * rotateForce * 2 - rotateForce)
    },
    rotate (evt) {
      if (
        evt.rotationRate &&
        evt.rotationRate.beta !== null &&
        evt.rotationRate.gamma !== null
      ) {
        this.rotateX = evt.rotationRate.beta * 0.7
        this.rotateY = evt.rotationRate.gamma * -0.7
      }
      else {
        /* evt.acceleration may be null in some cases, so we'll fall back
           to evt.accelerationIncludingGravity */
        const accelX = evt.acceleration.x || evt.accelerationIncludingGravity.x,
          accelY = evt.acceleration.y || evt.accelerationIncludingGravity.y,
          accelZ = evt.acceleration.z || evt.accelerationIncludingGravity.z - 9.81,
          rotation = getRotationFromAccel(accelX, accelY, accelZ)

        this.rotateX = rotation.roll * 0.7
        this.rotateY = rotation.pitch * -0.7
      }
    },
    orient (evt) {
      if (evt.beta === null || evt.gamma === null) {
        window.removeEventListener('deviceorientation', this.orient, false)
        this.orienting = false
        window.addEventListener('devicemotion', this.rotate, false)
      }
      else {
        this.rotateX = evt.beta * 0.7
        this.rotateY = evt.gamma * -0.7
      }
    },
    notify (eventName) {
      Toast.create(`Event '${eventName}' was triggered`)
    },
    closeNavigatorWindow () {
      window.top.close()
    }
  },
  mounted () {
    this.$nextTick(() => {
      if (this.orienting) {
        window.addEventListener('deviceorientation', this.orient, false)
      }
      else if (this.rotating) {
        window.addEventListener('devicemove', this.rotate, false)
      }
      else {
        document.addEventListener('mousemove', this.move)
      }
      if (saw.API) {
        this.lmsData.scormApi = true
        this.lmsData.simApi = false
        console.info('Scorm yes')
        if (this.lmsData.autoStart) saw.lmsInitialize()
        if (saw.LMSInitialized) {
          saw.setScormValue('cmi.core.lesson_status', 'incomplete')
          saw.lmsCommit()
          this.getName()
          this.lmsData.status = saw.getScormValue('cmi.core.lesson_status')
          console.log(this.lmsData)
        }
        else {
          this.lmsData.scormApi = false
          this.lmsData.simApi = false
          this.lmsData.userName = 'Error Api Scorm'
          this.lmsData.status = 'ERR_NOENT'
        }
      }
      else {
        this.lmsData.scormApi = false
        console.info('Scorm No')
        this.lmsData.userName = 'Local Deploy'
      }
      this.lmsData.tini = new Date()
      console.log(this.lmsData)
    })
  },
  created () {
    // console.info('created')
    // console.info(this.evalConfigLocation)
    axios
      .get(this.evalConfigLocation)
      .then(response => {
        this.evaluacion = response.data
        // this.evaluacion = JSON.parse(response.data)
        // console.info(this.unidades[this.indexUnit])
        // this.source = this.unidades[this.indexUnit]
        // console.info('created')
        // console.info('response:')
        // console.log(response.data)
        if (this.evaluacion.randQuestions === true) this.reorderQuestions()
        this.configured = true
      })
      .catch(e => {
        console.error('Error')
        // this.errors.push(e)
      })
  },
  beforeDestroy () {
    if (this.orienting) {
      window.removeEventListener('deviceorientation', this.orient, false)
    }
    else if (this.rotating) {
      window.removeEventListener('devicemove', this.rotate, false)
    }
    else {
      document.removeEventListener('mousemove', this.move)
    }
    if (saw.LMSInitialized) saw.lmsFinish()
  }
}
</script>

<style>

</style>
