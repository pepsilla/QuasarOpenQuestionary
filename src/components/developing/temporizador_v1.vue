<template>
  <div class="timerBox">
    {{getMinutes}}:{{getSeconds}}
  </div>
</template>
<script>
import {
  QKnob,
  QChip
} from 'quasar'
export default {
  components: {
    QKnob,
    QChip
  },
  props: {
    tiempo: {
      type: Number,
      required: true
    }
  },
  data () {
    return {
      minutos: 60,
      segundos: 0,
      temporizador: null
    }
  },
  computed: {
    getMinutes () {
      var minutes = this.minutos.toString()
      if (this.minutos < 10) minutes = '0' + minutes
      return minutes
    },
    getSeconds () {
      var seconds = this.segundos.toString()
      if (this.segundos < 10) seconds = '0' + seconds
      return seconds
    }
  },
  methods: {
    cadaSegundo () {
      // console.info('€€€€ LIVE ONE SECOND MOORE €€€')
      if (this.minutos === 0 && this.segundos === 0) {
        this.timerStop()
        // console.info('@@@@ NO MOORE LIVE @@@@')
        this.$emit('tempusfugit')
        return
      }
      if (this.minutos > 0) {
        if (this.segundos === 0) {
          this.minutos -= 1
          this.segundos = 60
        }
      }
      this.segundos -= 1
    },
    timerStart () {
      // console.info('**** GENERANDO UN PULSO CADA SEGUNDO *****')
      this.temporizador = setInterval(this.cadaSegundo, 1000)
    },
    timerStop () {
      clearInterval(this.temporizador)
    }

  },
  mounted () {
    this.minutos = this.tiempo
    this.timerStart()
  }
}
</script>
<style scoped>
.timerBlock {
  color: bisque;
}
</style>
