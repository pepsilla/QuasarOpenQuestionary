<template>
  <div>
    <div v-for="(dataResponse, indice) in pregunta.RESPUESTAS">
      form-respuesta:aResponse->./Respuesta.vue
      <form-respuesta :respuesta="dataResponse" :indice="indice" :review="review" :noFeedback="noFeedback" @userSel="userSelection"/>
    </div>
  </div>
</template>

<script>
  import aResponse from './Respuesta.vue'
  
  export default {
    props: {
      pregunta: {
        type: Object,
        required: true,
        validator: function (value) {
          console.info('from Pregunta.vue: ', value)
          if (value.PREGUNTAS && value.tittle) {
            if (typeof (value.RESPUESTAS) !== 'object') return null
            if (typeof (value.tittle) !== 'string') return null
          }
          return value
        }
      },
      review: {
        type: Boolean,
        required: true,
        default: false
      },
      noFeedback: {
        type: Boolean,
        required: true
      }
    },
    data () {
      return {
      }
    },
    components: {
      'formRespuesta': aResponse
    },
    methods: {
      userSelection (indice) {
        console.info('userSelection from Pregunta.vue:', indice)
        this.$emit('changeQuestion', indice)
      }
    }
}
</script>

<style>
.Pregunta {
  display: flex;
  padding: 20px;
  vertical-align: middle;
}
</style>
