<template lang="html">
  <div class="container">

    <div class="pantalla">
      <p class="font-weight-bold" style="color:white; font-size:18px;">Pantalla</p>
    </div>

    <div class="asientos">
      <div class="row">
        <div class="col asiento"
          v-for="asiento in asientos"
          v-text="asiento.id"
          v-bind:id="asiento.id"
          v-bind:class="{disponible: asiento.disponible, ocupado: !asiento.disponible, pointer: asientoDiponible(asiento)}"
          @click="seleccionarAsiento"
          >
        </div>
      </div>
    </div>

    <div class="botones">
      <b-button :disabled="contador == 0" variant="success" @click="guardar">Guardar</b-button>
      <b-button style="margin-left:10px;" :disabled="contador == 0" variant="danger" @click="cancelar">Cancelar</b-button>
      <b-button style="margin-left:10px;" @click="liberar">Liberar</b-button>
    </div>

    <div style="margin-top:20px;">
      <strong>Asientos seleccionados {{ contador }}</strong>
    </div>

  </div>
</template>

<script>
import firebase from 'firebase'

const path = 'salas'
const pathId = '1'

export default {
  created(){
    // this.actualizarElementos()

    this.id = firebase.database().ref('/users').push().key

    firebase.database()
    .ref(path)
    .child(pathId)
    .on('value', snapshot => this.cargarElementos(snapshot.val()) )
  },

  data(){
    return {
      id: '',
      contador: 0,
      asientos: []
    }
  },

  methods: {
    seleccionarAsiento: function(event){
      let asiento = this.asientos.find(a => a.id == event.target.id)
      if(asiento.adquirido || (asiento.user_id != null && asiento.user_id != this.id) ){

        this.$notify({
          group:'foo', type:'error', title:'Error', text:'No es posible seleccionar el asiento ' + asiento.id
        })

        return
      }

      asiento.disponible = !asiento.disponible
      asiento.user_id = this.id
      this.actualizarElementos()

      this.contador = this.asientosSeleccionados().length
    },

    actualizarElementos: function(){
      firebase.database()
      .ref(path)
      .child(pathId)
      .set(this.asientos)
    },

    validarRespuesta: function(error, committed, snapshot){
      if(error){
        this.$notify({
          group:'foo', type:'error', title:'Error', text:'No es posible completar la operación'
        })
      }

      if(committed){
        this.$notify({
          group:'foo', type:'success', title:'Exito', text:'Asientos adquiridos exitosamente'
        })
      }

      console.log(snapshot.val())
    },

    cargarElementos: function(data){
      this.asientos = data
    },

    guardar: function(){
      firebase.database()
      .ref(path)
      .child(pathId)
      .transaction(
        valoresDB => this.validarCompra(valoresDB),
        (error, committed, snapshot) => this.validarRespuesta(error, committed, snapshot)
      )
      this.contador = 0

    },

    validarCompra: function(valoresDB){
      this.asientosSeleccionados().forEach(function(asiento){

        if(valoresDB.find(a => a.id == asiento.id).adquirido){
          return
        }
        asiento.adquirido = true
      })

      return this.asientos
    },

    cancelar: function(){
      this.asientosSeleccionados().forEach(function(asiento){
        asiento.user_id = null
        asiento.disponible = true
      })

      this.actualizarElementos()
      this.contador = 0
    },

    validarAsientos: function(){
      this.asientosSeleccionados().forEach(function(asiento){
        asiento.adquirido = true
      })
    },

    asientoDiponible: function(asiento){
      return !asiento.adquirido && (asiento.user_id == null || asiento.user_id == this.id)
    },

    asientosSeleccionados: function(){
      return this.asientos.filter(a => !a.disponible && !a.adquirido)
    },

    liberar: function(){
      this.asientos.forEach(function(asiento){
        asiento.adquirido = false
        asiento.disponible = true
        asiento.user_id = null
      })
      this.actualizarElementos()
      this.contador = 0
    }
  }
}
</script>

<style lang="css" scoped>
  .pantalla{
    background-color: #2b6282;
  }

  .asientos{
    margin-top: 60px;
  }

  .asiento{
    color:white;
    margin:3px;
    font-weight:bold;
  }

  .pointer{
    cursor: pointer;
  }

  .disponible{
    background-color: #2d4d86;
  }

  .ocupado{
    background-color: #73264f;
  }

  .botones{
    margin-top:60px;
  }
</style>
