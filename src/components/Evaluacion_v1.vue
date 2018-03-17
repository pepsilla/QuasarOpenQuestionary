<template>
  <div class="Evaluador container">
    <div class="cont_total pos_rel align-items-center row" style=" ">
      <q-transition mode="out-in" appear :enter="enter" :leave="leave" >
        <div class="  pant_inicio  row" v-if="!started" style=" " key="inicio">
          <div v-if="corregir">
            <div v-if="getMaxTries">
              <div v-if="!review">
                Activa el botón corregir para revisar las respuestas a este ejercicio.<br>
                Te {{getTries}} de revisión.<br> 
              </div>
              <div v-else>
                <span v-if="isAllOk">
                    Enhorabuena {{nombreFormando}} has resuelto el formulario de manera correcta.<br>
                    Activa el botón cerrar para continuar con el curso.
                </span>
                <span v-else>Activa el botón nuevo intento para modificar tus respuestas.<br></span>
              </div>
            </div>
            <div v-else>
              Ya no te quedan más intentos o se terminó el tiempo para resolver el ejercicio<br>
              <strong> Cierra esta ventana y vulve a la autoevaluación desde el índice del curso.</strong><br>
              <button class="actionButton" @click="closeUnit">CERRAR</button>
            </div>
          </div>
          <div v-else class=" transTop  items-center row justify-center" >
            <div class="col-12 col-md-10 col-lg-6 alerta" style=" ">
                <h4>¡Hola {{userName}}!</h4>
                <ul class="listado">
                  <li>El ejercicio de autoevaluación consta de {{evaluacion.numPreguntas}} preguntas.</li>
                  <li>Dispones de {{evaluacion.tiempo}} minutos para resolver el ejercicio.</li>
                  <li>Dispones de {{evaluacion.maxIntentos}} intentos para revisar los resultados.</li>
                  <li>Tienes que contestar todas las preguntas para poder evaluar los resultados. Una vez resuelvas correctamente todas las preguntas podrás continuar con el curso.</li>   
                </ul>
                <div class="justify-center row"  >
                  <button class="  actionButton " @click="empezar">EMPEZAR</button>
                </div>
                <div class="advertencia bg-green-4"> 
                  <q-icon class="icono_l" name="warning"/>
                  Si agotas el tiempo o los intentos de revisión sin resolver el ejercicio, deberás cerrar esta ventana y volver a entrar desde el índice del curso.
                </div>
            </div>
          </div>
        </div>
        <div v-if="started" class="ContainerResult pant_inicio  items-center justify-center row" key="preguntas">
          <div class="transTop   row col-12   col-lg-10">
            <div class="col-12 col-md-3  caja_tiempo">
              <div class="col_preguntas ">
                  <div class="digits row">
                    <div class="col-xs-12 col-sm-6 col-md-12">
                        <span class="labelText">Pregunta:<br></span>{{pin}} de {{evaluacion.numPreguntas}}<br>
                        <div>
                            <button class="actionButton" :disabled="isAnterior" @click="goAnterior">Anterior</button>
                            <button class="actionButton" :disabled="isSiguiente" @click="goSiguiente">Siguiente</button>
                        </div> 

                        <div class="labelText">Tiempo disponible:</div>
                        <tempus ref="temporizador" :tiempo="evaluacion.tiempo" @tempusfugit="tempusFugit"/>
                    </div>
                    <div class="intentos  col-xs-12 col-sm-6 col-md-12 ">
                      <div class="num_intentos">REVISIONES:</div>
                      <q-icon v-for="pointIntent in evaluacion.maxIntentos" class="icono_l ico_try" :name="getNameIconConsumed(pointIntent)"/>
                    </div>
                  </div>
                  <div v-if="corregir">
                    <div v-if="getMaxTries">
                      <div v-if="!review">
                          <q-btn class="actionButton but_corregir  bg-blue-3" @click="toggleReview" icon ="ion-checkmark-circled" color="green-6" >REVISAR</q-btn>
                      </div>
                      <div v-else>
                        <span v-if="isAllOk"><button class="actionButton"  @click="completeUnit">CERRAR</button></span>
                        <span v-else><button class="actionButton" @click="newIntent">NUEVO INTENTO</button></span>
                      </div>
                    </div>
                    <div v-else class="no_more_tries text-negative labelText">No te quedan más intentos</div>
                  </div>
                </div>
              </div>
              <q-list v-model="page" class="navPregunta  alerta col-12 col-md-9 row caja_preg"><!-- q-list PREGUNTAS -->
                <q-chip pointing="right" class="globo_preg">
                  {{page}}
                </q-chip>
                <div class="pregunta">
                  <div class="textoPregunta">{{getPregunta.tittle}}</div>
                  <form-pregunta :pregunta="getPregunta" :review="review" :noFeedback="noFeedback" @changeQuestion="changeAnswer" class="caja_resp" />  
                </div>
              </q-list>
              <div class="row clearboth  row_steps"   v-if="review"><!-- row_steps -->    
                <h6 class=" ">Revisión:</h6> 
                <div class="row_rev row">
                  <div v-for="(preg, index) in   this.questions.PREGUNTAS" class="col-2 col-md-1 bot_step " v-if=" index  < evaluacion.numPreguntas " :title="preg[0].tittle"> 
                    <index-info @goPin="goPin(index)"  :corregir="review"  :pregunta="preg[0]  " :pin="pin" :index="index"/> 
                  </div> 
                </div>
                1.2,2.3,3.2,4.1,5.1,6.1,7.4,8.3,9.2,10.4,11.3,12.1,13.3,14.1,15.3,16.3,17.1,18.2,19.1,20.4 
              </div>
            </div>            
          </div>
      </q-transition>
    </div>
  </div>
</template>

<script>
  import adPregunta from './contentModules/testForm/Pregunta.vue'
  import adTempo from './developing/temporizador_v1.vue'
  import indexInfo from './contentModules/testForm/IndexInfo.vue'
  import {
    QBtn,
    QTransition,
    QIcon,
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
      QBtn,
      QTransition,
      QIcon,
      QPagination,
      QList,
      QChip,
      QKnob,
      'formPregunta': adPregunta,
       'tempus' : adTempo,
      'indexInfo':indexInfo 
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
        if (this.local === this.point) return this.pin
        else {
          this.pin = this.point
          this.local = this.pin
        }
        return this.pin
      },
      getPregunta () {
        if (this.evaluacion.randQuestions === true) {
          return this.questions.PREGUNTAS[this.pin - 1][0]
        }
        return this.evaluacion.PREGUNTAS[this.pin - 1]
      },
      getMaxTries () {
        if (this.tries <   this.evaluacion.maxIntentos) return true
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
      getTriesQuedan(){
        var totIntentos = this.evaluacion.maxIntentos - this.tries
        console.log("intentos " + this.tries +" de " + totIntentos)
        return totIntentos
      },
      allSelected () {
        return false
      }
    },
    methods: {
      checkNoFeedback () {
        var respuesta, dataQuestion
        var selected = 0, trued = 0
        if (this.evaluacion.randQuestions === true) {
          dataQuestion = this.questions.PREGUNTAS[this.pin - 1][0]
          for (respuesta in dataQuestion.RESPUESTAS) {
            if (dataQuestion.RESPUESTAS[respuesta].isTrue)trued += 1
            if (dataQuestion.RESPUESTAS[respuesta].isSelected)selected += 1
          }
          if (selected !== trued) this.noFeedback = true
          else this.noFeedback = false
        }
      },
      getNameIconConsumed(pointerFor){
        if (pointerFor <= this.tries) return "ion-android-checkbox"
        return "ion-android-checkbox-blank"
      },
      empezar () {
        this.started = true
      },
      tempusFugit () {
        this.tries = this.evaluacion.maxIntentos
        this.review = true
        this.corregir = true
      },
      closeUnit () {
        this.$emit('closeUnit')
      },
      completeUnit () {
        this.$emit('unitCompleted')
      },
      changeAnswer (indice) {
        if (this.evaluacion.randQuestions === true) {
          for (var pregunta = 0; pregunta < this.evaluacion.numPreguntas; pregunta++) {
            var dataQuestion = this.questions.PREGUNTAS[pregunta][0]
            var questionSelected = false
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
        }
        else {
          this.corregir = false
        }
      },
      chekAllQuestions () {
        var pregunta, respuesta
        var question
        var isOk = false
        var allIsOk = 0
        if (this.evaluacion.randQuestions === true) {
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
          if (allIsOk === this.evaluacion.numPreguntas){
            this.isAllOk = true
            this.$emit('isCompleted')
          }
          else this.isAllOk = false
        }
        else {
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
      goPin(pin){ 
        this.pin = (pin + 1)
        this.checkNoFeedback()
      },
      topPage () {
      },
      toggleReview () {
        this.chekAllQuestions()
        this.checkNoFeedback()
        this.review = true
        this.tries += 1
        if(this.tries === this.evaluacion.maxIntentos){
         this.$refs.temporizador.timerStop();
        }
      },
      newIntent () {
        this.review = false
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
        noFeedback: false,
        enter: 'fadeInUp',
        leave: 'bounceOutRight',
      }
    }
  }
</script>

<style scoped>
.Evaluador {
  display: flex;
  flex-direction: column;
  width: 100vw;
}
.ContainerInfo {
  display: flex;
  flex-flow: column;
}
.InfoTime {
  color: blue;
}
</style>