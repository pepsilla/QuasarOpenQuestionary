<template>
  <a  v-on:click="goPin(index)" :class="getFeedbackClass " >
    {{ (index + 1) }}
  </a>
</template>
<script>
export default {
  props: {
    corregir:Boolean,
    index:Number,
    pin:Number,
    pregunta: {
      type: Object,
      required: true,
      validator: function (value) {
        if (value.feedback && value.isTrue && value.isSelected && value.tittle) {
          if (typeof (value.feedback) !== 'string') return null
          if (typeof (value.isTrue) !== 'boolean') return null
          if (typeof (value.isSelected) !== 'boolean') return null
          if (typeof (value.tittle) !== 'string') return null
        }
        return value
      }
    } 
   ,
    indice: Number
  },
  data () {
    return {
    }
  },
  components: {
 
  },
  computed: {
    getFeedbackClass () {
      if ( this.corregir &&  this.pregunta.isOk  )  {
        return 'isOk'
      }
      else {
        return 'isNoOk'
      }
    },
 
  },
  methods: {
    goPin (i) { 
      this.$emit('goPin', (this.index + 1))
    },
    changeSelected () {
      this.$emit('userSel', this.indice)
    }
  }
}
</script> 