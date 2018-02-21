<template>
  <div class="Evaluador">
    <!-- Contenedor real de la página. Espacio aprovechable disponible para presentación de unidad didáctica -->
    <div class="ContainerInfo">
      <div class="ContainerResult">
        <div class="digits" v-if="!started">
        <!--Contenedor info pregunta actual de preguntas totales y botones navegación (top-left) -->
            <button class="ActionButton" @click="empezar">EMPEZAR</button>
        </div>
        <div  class="infoRev">
        <!--Contenedor info estado cuestionario y acciones (top-center)-->
          <div v-if="corregir">
            <!-- Si todas las preguntas tienen seleccionada al menos una respuesta permitimos intento de corrección -->
            <div v-if="getMaxTries">
              <!-- Si nos quedan intentos -->
              <div v-if="!review">
                <!-- Estamos en modo revisión. No se permiten cambios en selección de respuestas, solo consulta -->
                Activa el botón corregir para revisar las respuestas a este ejercicio.<br>
                Te {{getTries}} de revisión.<br> 
              </div>
              <div v-else>
                
                <!-- Informar del estado de las respuestas por preguntas -->
                <span v-if="isAllOk">
                    Enhorabuena {{nombreFormando}} has resuelto el formulario de manera correcta.<br>
                    Activa el botón cerrar para continuar con el curso.
                </span>
                <!-- Activamos un nuevo intento para permitir la modificación de respuestas -->
                <span v-else>Activa el botón nuevo intento para modificar tus respuestas.<br></span>
              </div>
            </div>
            <div v-else>
              <!-- Ya hemos agotado el número de intentos, solo nos queda salir y volver a entrar -->
              Ya no te quedan más intentos o se terminó el tiempo para resolver el ejercicio<br>
              <strong> Cierra esta ventana y vulve a la autoevaluación desde el índice del curso.</strong><br>
              <button class="ActionButton" @click="closeUnit">CERRAR</button>
            </div>
          </div>
          <div v-else>
            <!-- Información inicial del estado de la evaluación -->
            <div>
            ¡Hola {{userName}}!
            <p>
              <i><b>
              El ejercicio de autoevaluación consta de {{evaluacion.numPreguntas}} preguntas.<br>
              Dispones de {{evaluacion.tiempo}} minutos para resolver el ejercicio.<br>
              Dispones de {{evaluacion.maxIntentos}} intentos para revisar los resultados.<br>
              Tienes que contestar todas las preguntas para poder evaluar los resultados. Una vez resuelvas correctamente todas las preguntas podrás continuar con el curso.
              </b></i>
            </p>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- Render modelo pregunta + respuestas -->
    <div v-if="started" class="ContainerResult">
        <div class="ContainerInfo">
            <div class="digits">
                <span class="labelText">Pregunta:</span>{{pin}} de {{evaluacion.numPreguntas}}<br>
                <div>
                    <button class="ActionButton" :disabled="isAnterior" @click="goAnterior">Anterior</button>
                    <button class="ActionButton" :disabled="isSiguiente" @click="goSiguiente">Siguiente</button>
                </div>
                <br>
                <span class="labelText">Tiempo disponible:</span>
                <tempus :tiempo="evaluacion.tiempo" @tempusfugit="tempusFugit"/>
            </div>
            <div v-if="corregir">
                <div v-if="getMaxTries">
                    <div v-if="!review">
                        <button class="ActionButton" @click="toggleReview" >CORREGIR</button>
                    </div>
                    <div v-else>
                        <span v-if="isAllOk"><button class="ActionButton"  @click="completeUnit">CERRAR</button></span>
                        <span v-else><button class="ActionButton" @click="newIntent">NUEVO INTENTO</button></span>
                    </div>
                </div>
            </div>
            <!--div class="tempus">
                
            </div--> 
        </div>
        <div class="navPregunta"> 
            <q-list v-model="page">
                form-pregunta:adPregunta->./contentModules/testForm/Pregunta.vue
                <div class="Pregunta">
                    <q-chip pointing="down">
                    {{page}}
                    </q-chip>
                    <span class="textoPregunta">{{getPregunta.tittle}}</span>
                </div>
                <form-pregunta :pregunta="getPregunta" :review="review" :noFeedback="noFeedback" @changeQuestion="changeAnswer"/>
            </q-list>
        </div>
        
    </div>
    <!-- Nota a pié de página -->
    <span class="AdvertText">Si agotas el tiempo o los intentos de revisión sin resolver el ejercicio, deberás cerrar esta ventana y volver a entrar desde el índice del curso.</span> 
  </div>
</template>

<script>
  import adPregunta from './contentModules/testForm/Pregunta.vue'
  import adTempo from './developing/temporizador_v1.vue'
  import {
    QPagination,
    QList,
    QChip,
    QKnob
  } from 'quasar'
  export default {
    props: {
      evaluacion: {
        type: Object,
        required: true,
        validator: function (value) {
          // console.info('from Evaluacion.vue: ', value)
          // console.info(this.data)
          return value
        }
      },
      userName: {
        type: String,
        required: true
      },
      point: Number,
      questions: Object
    },
    components: {
      QPagination,
      QList,
      QChip,
      QKnob,
      'formPregunta': adPregunta,
      'tempus': adTempo
    },
    computed: {
      isAnterior () {
        if (this.pin > 1) return false
        return true
      },
      isSiguiente () {
        if (this.pin < this.evaluacion.numPreguntas) return false
        return true
      },
      page () {
        // console.log(this.point, this.pin, this.local)
        if (this.local === this.point) return this.pin
        else {
          this.pin = this.point
          this.local = this.pin
        }
        return this.pin
      },
      getPregunta () {
        if (this.evaluacion.randQuestions === true) {
          console.info('Gen. Aleatoria:')
          console.log(this.questions.PREGUNTAS[this.pin - 1][0])
          console.info('Natural:')
          console.log(this.evaluacion.PREGUNTAS[this.pin - 1])
          return this.questions.PREGUNTAS[this.pin - 1][0]
        }
        console.info('Natural:')
        console.log(this.evaluacion.PREGUNTAS[this.pin - 1])
        return this.evaluacion.PREGUNTAS[this.pin - 1]
      },
      getMaxTries () {
        if (this.tries < this.evaluacion.maxIntentos) return true
        return false
      },
      getTries () {
        var retorno = ''
        var totIntentos = this.evaluacion.maxIntentos - this.tries
        if (totIntentos === 1) retorno += 'queda '
        else retorno += 'quedan '
        retorno += totIntentos
        if (totIntentos === 1) retorno += ' intento'
        else retorno += ' intentos'
        return (retorno)
      },
      allSelected () {
        return false
      }
    },
    methods: {
      checkNoFeedback () {
        // console.info('CHECKnOfEEDBACK')
        var respuesta, dataQuestion
        var selected = 0, trued = 0
        if (this.evaluacion.randQuestions === true) {
          dataQuestion = this.questions.PREGUNTAS[this.pin - 1][0]
          // console.log(dataQuestion)
          for (respuesta in dataQuestion.RESPUESTAS) {
            // console.log(dataQuestion.RESPUESTAS[respuesta])
            // console.info(dataQuestion.RESPUESTAS[respuesta].isSelected, dataQuestion.RESPUESTAS[respuesta].isTrue)
            if (dataQuestion.RESPUESTAS[respuesta].isTrue)trued += 1
            if (dataQuestion.RESPUESTAS[respuesta].isSelected)selected += 1
          }
          // console.log(selected, trued)
          if (selected !== trued) this.noFeedback = true
          else this.noFeedback = false
        }
      },
      empezar () {
        console.info('********** EMPEZANDO ******************')
        this.started = true
      },
      tempusFugit () {
        console.info('El tiempo se acabó')
        this.tries = this.evaluacion.maxIntentos
        this.review = true
        this.corregir = true
      },
      closeUnit () {
        console.info('Petición de cierre de la unidad')
        this.$emit('closeUnit')
      },
      completeUnit () {
        console.info('Evaluación completada con éxito')
        this.$emit('unitCompleted')
      },
      changeAnswer (indice) {
        console.info('from Evaluacion Vue Question index: ', this.page, indice)
        if (this.evaluacion.randQuestions === true) {
          console.log(this.questions.PREGUNTAS)
          for (var pregunta = 0; pregunta < this.evaluacion.numPreguntas; pregunta++) {
            var dataQuestion = this.questions.PREGUNTAS[pregunta][0]
            var questionSelected = false
            console.info('dataQuestion: ', dataQuestion)
            for (var respuesta in dataQuestion.RESPUESTAS) {
              if (dataQuestion.RESPUESTAS[respuesta].isSelected) {
                questionSelected = true
                break
              }
            }
            if (questionSelected === false) {
              this.corregir = false
              return
            }
          }
          this.corregir = true
          // return
        }
        else {
          console.log(this.evaluacion.PREGUNTAS)
          this.corregir = false
        }
      },
      chekAllQuestions () {
        // Comnprobar si el questionario ha sido completado adecuadamente.
        console.info('Evaluacion.chekAllQuestions')
        var pregunta, respuesta
        var question
        var isOk = false
        var allIsOk = 0
        if (this.evaluacion.randQuestions === true) {
          // q
          for (pregunta = 0; pregunta < this.evaluacion.numPreguntas; pregunta++) {
            question = this.questions.PREGUNTAS[pregunta][0]
            for (respuesta in question.RESPUESTAS) {
              if (question.RESPUESTAS[respuesta].isSelected === question.RESPUESTAS[respuesta].isTrue) isOk = true
              else {
                isOk = false
                break
              }
            }
            if (isOk) {
              this.questions.PREGUNTAS[pregunta][0].isOk = true
              allIsOk += 1
            }
          }
          if (allIsOk === this.evaluacion.numPreguntas) {
            // 20180602 Asegurar completed en la plataforma para evitar cierre desde ventana sin validad botón cerrar del formulario.
            this.isAllOk = true
            // console.info('Emit event completed')
            this.$emit('isCompleted')
          }
          else this.isAllOk = false
        }
        else {
          // q
          return false
        }
      },
      goAnterior () {
        if (this.pin > 1) {
          this.pin -= 1
          this.checkNoFeedback()
        }
      },
      goSiguiente () {
        if (this.pin < this.evaluacion.numPreguntas) {
          this.pin += 1
          this.checkNoFeedback()
        }
      },
      topPage () {
      },
      toggleReview () {
        console.info('Review is: ' + this.review)
        this.chekAllQuestions()
        this.checkNoFeedback()
        this.review = true
        console.info('Now is:' + this.review)
      },
      newIntent () {
        this.review = false
        this.tries += 1
      },
      mounted () {
        this.nombreFormando = this.lmsData.userName
      }
    },
    watch: {
      userName: function (value) {
        console.info('User name cambó a ' + value)
      }
    },
    data () {
      return {
        pin: 1,
        local: 1,
        tries: 0,
        started: false,
        corregir: false,
        isAllOk: false,
        review: false,
        noFeedback: false
      }
    }
  }
</script>

<style scoped>
.Evaluador {
  display: flex;
  flex-direction: column;
  padding: 10px;
  width: 100vw;
}
.ContainerInfo {
  display: flex;
  flex-flow: column;
}
.InfoTime {
  color: blue;
}
.ContainerResult {
  display: flex;
  flex-direction: row;
  font-size: 16px;
  text-align: justify;
  padding: 5px;
  margin: 5px;
}
.Pregunta {
  display: flex;
  align-items: center;
}
.textoPregunta {
  font-size: 22px;
  margin-left: 20px;
}
.labelText {
  font-size: 13px;
}
.AdvertText {
  text-align: right;
  font-size: 10px;
  font-style: oblique;
  margin-right: 20px;
  margin-top: 5px;
}
.ActionButton {
  font-size: 12px;
  background-color: #bf360c;
  border: 1px solid beige;
  border-radius: 10px;
  color: white;
  margin: 2px;
  padding: 5px;
}
.digits {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  border: 2px solid rgb(238, 151, 151);
  border-radius: 10px;
  font-size: 26px;
  background-color: #bf360c;
  color: white;
  padding: 10px;
  margin-left: 5px;
  margin-right: 5px;
}
.tempus {
  display: flex;
  flex-flow: column;
  align-items: center;
  text-align: center;
  border: 2px solid grey;
  border-radius: 10px;
  font-size: 26px;
  background-color: rgb(109, 109, 109);
  color: white;
  margin-left: 5px;
  margin-right: 5px;
  padding: 5px;
}
.infoRev {
  display: flex;
  flex-flow: column;
  flex-grow: 1;
  align-items: center;
  border: 1px solid grey;
  border-radius: 10px;
  padding: 10px;
}
.navPregunta {
    flex: 1;
}
</style>
