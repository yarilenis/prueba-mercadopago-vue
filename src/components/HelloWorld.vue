<template>
  <div class="container hello">
    <div class="text-center mb-3">
      <h1>Suscríbete ahora</h1>
    </div>
    <div class="my-2">
      <mercadopago
        ref="mercadopago"
        @ok="mercadopagoOk()"
      ></mercadopago>
    </div>
  </div>
</template>

<script>
import Mercadopago from './Mercadopago.vue'

export default {
  name: 'HelloWorld',
  components: {
    Mercadopago
  },
  props: {
    msg: String
  },
  methods: {
    mercadopagoOk() {
      this.$refs.mercadopago.submit();
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
