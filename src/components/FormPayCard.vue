<template>
  <form @submit.prevent="payMethod()" class="form-pay" id="pay" name="pay">
    <div class="form-pay__header">
      <button @click="$emit('hide')" type="button" class="close">
        &times;
      </button>
      <div class="form-pay__logo">
        <img src="https://www.joinnus.com/static/images/joinnus-iso-v2.svg" width="100" alt="Joinnus">
      </div>
      <div v-if="email" style="opacity: 0.8; line-height: 1.2">
        <span>Enviaremos tus entradas a:</span><br>
        <b>{{ email }}</b>
      </div>
    </div>
    <div class="form-pay__body">
      <div class="mb-3 d-none">
        <card
          v-model="card"
          :placeholders="cardPlaceholders"
          @validate="onCardValidate($event)"
          @type="onCardType($event)"
        >
        </card>
      </div>
      <div class="row">
        <div class="col-12 text-center">
          <b-form-group label="Selecciona el tipo de tarjeta">
            <b-form-radio-group
              name="radio-inline"
              v-model="card.type"
              required
            >
              <b-form-radio value="debit">Débito</b-form-radio>
              <b-form-radio value="credit">Crédito</b-form-radio>
            </b-form-radio-group>
          </b-form-group>
        </div>
        <div class="col-12">
          <b-input-group class="mt-2 mb-4">
            <b-input-group-text slot="prepend">
              <i class='fa fa-credit-card'></i>
            </b-input-group-text>
            <input
              type="text"
              v-model="card.number"
              name="number"
              required
              placeholder="Número de tarjeta"
              autocomplete="off"
              class="form-control"
              :class="{'is-invalid': !cardValidate.number}"
              data-checkout="cardNumber"
            />
          </b-input-group>
        </div>
        <div v-if="cardIssuers.length" class="col-12">
          <b-input-group class="mb-3">
            <b-input-group-text slot="prepend">
              <i class="fa fa-university"></i>
            </b-input-group-text>
            <select
              id="issuerId"
              v-model="card.issuerId"
              class="form-control"
              required
            >
              <option v-for="i in cardIssuers" :value="i.id" :key="i.id">
                {{ i.name }}
              </option>
            </select>
          </b-input-group>
        </div>
        <div class="col-6">
          <b-input-group class="mb-4">
            <b-input-group-text slot="prepend">
              <i class='fa fa-calendar'></i>
            </b-input-group-text>
            <input
              type="text"
              v-model="card.expiry"
              name="expiry"
              required
              placeholder="Mes/Año"
              autocomplete="off"
              maxlength="7"
              class="form-control"
              :class="{'is-invalid': !cardValidate.expiry && !cardMP}"
              @keyup="onKeyUpExpiry($event)"
            />
          </b-input-group>
          <input
            v-model="cardMP.month"
            type="text"
            class="d-none"
            data-checkout="cardExpirationMonth"
          >
          <input
            v-model="cardMP.year"
            type="text"
            class="d-none"
            data-checkout="cardExpirationYear"
          >
        </div>
        <div class="col-6">
          <b-input-group  class="mb-4">
            <b-input-group-text slot="prepend">
              <i class='fa fa-lock'></i>
            </b-input-group-text>
            <input
              type="text"
              v-model.trim="card.cvc"
              name="cvc"
              required
              placeholder="CVC"
              autocomplete="off"
              class="form-control"
              :class="{'is-invalid': !cardValidate.cvc}"
              data-checkout="securityCode"
            />
          </b-input-group>
        </div>
        <div class="col-12">
          <b-form-group>
            <input
              type="text"
              v-model.trim="card.name"
              name="name"
              required
              placeholder="Titular de la tarjeta"
              autocomplete="off"
              class="form-control"
              :class="{'is-invalid': !cardValidate.name}"
              data-checkout="cardholderName"
            />
          </b-form-group>
        </div>
        <div class="col-12">
          <b-form-group class="mt-1">
            <select
              v-model="card.documentType"
              class="form-control"
              required
              data-checkout="docType"
            >
              <option value="">Tipo de documento</option>
              <option v-for="i in documentTypes" :value="i.value" :key="i.value">
                {{ i.text }}
              </option>
            </select>
          </b-form-group>
        </div>
        <div class="col-12">
          <b-form-group>
            <input
              type="text"
              v-model="card.document"
              required
              placeholder="N° Documento"
              autocomplete="off"
              maxlength="12"
              class="form-control"
              data-checkout="docNumber"
            />
          </b-form-group>
        </div>
        <div class="col-12" v-if="card.type === 'credit'">
          <b-form-group>
            <b-form-select v-model="card.quota" :disabled="disabledQuota">
              <option value="null">Cuotas</option>
              <option :value="elem" v-for="elem in 12" :key="elem">{{ elem }}</option>
            </b-form-select>
          </b-form-group>
        </div>
      </div>
      <div v-if="msgError" class="small text-danger">
        * {{ msgError }}
      </div>
    </div>
    <div class="form-pay__footer">
      <button
        type="submit"
        class="btn btn-primary btn-lg btn-block"
      >
        <b-spinner v-if="loading" small class="mr-1"></b-spinner>
        <span v-if="!loading">Pagar ahora:</span>
        <span v-else>Procesando pago:</span>
        {{ currency.symbol }} {{ paymentTotal }} {{ currency.iso }}
      </button>
    </div>
    <input
      v-model="paymentMethodId"
      name="paymentMethodId"
      type="hidden"
    >
  </form>
</template>

<script>
import card from 'vue-credit-card/src/components/Card.vue';

function clearNumber(value = '') {
  return value.replace(/\D+/g, '');
}

const cardPlaceholders = {
  number: '•••• •••• •••• ••••',
  name: 'Nombre completo',
  expiry: '••/••',
  cvc: '•••',
};

export default {
  components: { card },
  props: {
    documentTypes: Array,
    email: String,
    currency: Object,
    quantity: Number,
    paymentTotal: String,
    msgError: {
      type: String,
      default: '',
    },
    loading: {
      type: Boolean,
      default: false,
    },
    card: {
      type: Object,
      default() {
        return {};
      },
    },
    paymentMethodId: String,
    cardIssuers: Array,
  },
  data() {
    return {
      cardPlaceholders,
      cardValidate: {
        number: true,
        name: true,
        expiry: true,
        cvc: true,
      },
      disabledQuota: false,
    };
  },
  computed: {
    validCard() {
      const {
        number,
        name,
        expiry,
        cvc,
      } = this.cardValidate;
      return number
        && name
        && expiry
        && cvc;
    },
    cardMP() {
      const { expiry } = this.card;
      if (expiry.length < 7) return false;
      const strExpiry = expiry.replace(/ /g, '');
      const char = strExpiry.substr(2, 1);
      if (char === '-' && char !== '/') return false;
      const splitExpiry = strExpiry.split('/');
      const month = splitExpiry[0];
      const year = splitExpiry[1].substr(-2);
      return { month, year };
    },
  },
  methods: {
    onCardValidate(obj) {
      this.cardValidate = obj;
    },
    payMethod() {
      this.$emit('pay-method');
    },
    onCardType(type) {
      this.card.cardType = type;
      this.disabledQuota = false;
      this.card.quota = null;

      if (type !== 'visa') {
        this.card.quota = 1;
        this.disabledQuota = true;
      }
    },
    onKeyUpExpiry(e) {
      const clearValue = clearNumber(this.card.expiry);
      if (e.keyCode !== 8 && clearValue.length === 2) {
        this.card.expiry = `${clearValue} / `;
      }
      if (clearValue.length >= 3) {
        this.card.expiry = `${clearValue.slice(0, 2)} / ${clearValue.slice(2, 4)}`;
      }
    },
  },
};
</script>
