<template>
  <div class="Respuesta">
    <q-item >
      <q-item-side>
        <q-checkbox v-model='respuesta.isSelected' :disable="review" @change="changeSelected"/>
      </q-item-side>
      <q-item-main>
        <q-item-tile title><i>{{respuesta.tittle}}</i></q-item-tile>
        <q-item-tile title v-if="getFeedBackResponse">
            <div :class="getFeedbackClass" v-if="!noFeedback">
              <b>{{respuesta.feedback}}</b>
            </div>
			<div class="isYellow" v-else>
              <b> Has seleccionado más respuestas de las esperadas </b>
            </div>
        </q-item-tile>
      </q-item-main>
    </q-item>
  </div>
</template>

<script>
import {
  QItem,
  QItemSide,
  QCheckbox,
  QItemTile,
  QItemMain
} from 'quasar'
export default {
  props: {
    respuesta: {
      type: Object,
      required: true,
      validator: function (value) {
        if (value.feedback && value.isTrue && value.isSelected && value.tittle) {
          if (typeof (value.feedback) !== 'string') return null
          if (typeof (value.isTrue) !== 'boolean') return null
          if (typeof (value.isSelected) !== 'boolean') return null
          if (typeof (value.tittle) !== 'string') return null
        }
        // console.info('from Respuesta.vue', value)
        return value
      }
    },
    review: {
      type: Boolean,
      required: true
    },
    indice: Number,
    noFeedback: Boolean
  },
  data () {
    return {

    }
  },
  components: {
    QItem,
    QItemSide,
    QCheckbox,
    QItemTile,
    QItemMain
  },
  computed: {
    getFeedbackClass () {
      if (this.respuesta.isSelected && this.respuesta.isTrue) return 'isTrue'
      return 'isFalse'
    },
    getFeedBackResponse () {
      if (this.review && this.respuesta.isSelected) return true
      return false
    }
  },
  methods: {
    changeSelected () {
      console.info('This ' + this.indice + 'is: ' + this.respuesta.isSelected)
      this.$emit('userSel', this.indice)
    }
  }
}
</script>

<style scoped>
  .Respuesta {
    padding: 5px;
  }
  .isTrue {
    color: green;
  }
  .isFalse {
    color:red;
  }
  .isYellow {
    color:darkgoldenrod;
  }
</style>
