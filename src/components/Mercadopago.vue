<template>
  <div>
    <div>paymentMethodToken {{ paymentMethodToken }} </div>
    <div>resSdk {{ resSdk }} </div>
    <div>
      <h4>Configuración</h4>
      <div class="row">
        <div class="col-6">
          <p> mercadoPagoKeyCo </p>
          <b-input-group  class="mb-4">
            <b-input-group-text slot="prepend">
              <i class='fa fa-lock'></i>
            </b-input-group-text>
            <input
              type="text"
              v-model="mercadoPagoKeyCo"
              name="cvc"
              required
              placeholder="mercadoPagoKeyCo"
              class="form-control"
              data-checkout="securityCode"
            />
          </b-input-group>
        </div>
        <div class="col-6">
          <p> ID del plan </p>
          <b-input-group  class="mb-4">
            <b-input-group-text slot="prepend">
              <i class='fa fa-lock'></i>
            </b-input-group-text>
            <input
              type="text"
              v-model="planInfo.plan.id"
              name="cvc"
              required
              placeholder="ID del plan"
              class="form-control"
              data-checkout="securityCode"
            />
          </b-input-group>
        </div>
        <div class="col-6">
          <p> payer Email </p>
          <b-input-group  class="mb-4">
            <b-input-group-text slot="prepend">
              <i class='fa fa-lock'></i>
            </b-input-group-text>
            <input
              type="text"
              v-model="payerEmail"
              name="cvc"
              required
              placeholder="payer Email"
              class="form-control"
              data-checkout="securityCode"
            />
          </b-input-group>
        </div>
        <div class="col-6">
          <p> Collector Token </p>
          <b-input-group  class="mb-4">
            <b-input-group-text slot="prepend">
              <i class='fa fa-lock'></i>
            </b-input-group-text>
            <input
              type="text"
              v-model="collectorToken"
              name="cvc"
              required
              placeholder="Collector Token"
              class="form-control"
              data-checkout="securityCode"
            />
          </b-input-group>
        </div>
      </div>
    </div>
    <CardMethod recomended>
      <span slot="title">
        <b>Paga de forma segura con tarjetas nacionales e internacionales</b>
      </span>
      <div slot="description" class="small mb-3">
        Paga de forma segura con cualquier tarjeta de crédito o débito.
        Visa, Mastercard, Amex, Diners y Tarjetas Internacionales.
        <!-- <div v-if="discountBines.length" class="mt-2">
          Para descuentos con tarjeta
          <b v-for="i in discountBines" :key="i">{{ i }}, </b>
          usa este método.
        </div> -->
      </div>
      <div class="row justify-content-between align-items-end px-3">
        <div class="col-auto">
          <img
            height="40"
            class="rounded border mt-2 mr-2"
            src="https://s3-us-west-2.amazonaws.com/joinnus.com/files/metodos/PE/mercadopago.png"
            alt="methodimg"
          >
        </div>
        <div class="col-auto mt-3">
          <a
            href="#"
            @click.prevent="$emit('ok')"
            class="btn btn-primary text-uppercase"
          >
            Suscríbete aquí
          </a>
        </div>
      </div>
    </CardMethod>
    <!-- modal-pay -->
    <b-modal
      v-model="modalPay"
      no-close-on-backdrop
      modal-class="modal-pay"
      hide-footer
      hide-header
      centered
    >
      <FormPayCard
        :currency="currency"
        :document-types="documentTypes"
        :quantity="1"
        :paymentTotal="paymentTotal"
        :loading="loading"
        :card="card"
        :paymentMethodId="paymentMethodId"
        :cardIssuers="cardIssuers"
        :msg-error="error"
        @hide="modalPay = false"
        @pay-method="submitOk()"
      >
      </FormPayCard>
    </b-modal>
  </div>
</template>

<script>
/* global Mercadopago */
import axios from 'axios'
import CardMethod from './CardMethod.vue'
import FormPayCard from './FormPayCard.vue'

// const apiCheckout = process.env.VUE_APP_API_CHECKOUT
// const mercadoPagoKeyCo = process.env.VUE_APP_MERCADOPAGO_KEY_CO

const apiCheckout = 'https://devcheckout.joinnus.com/api/v1'
// const mercadoPagoKeyCo = 'APP_USR-b4296116-d367-4d0f-97fa-8c9260553f8a'

const docTypes = {
  PE: [
    { key: 'DNI', name: 'DNI' },
    { key: 'CE', name: 'C.E' },
    { key: 'RUC', name: 'RUC' },
    { key: 'PASS', name: 'PASS' },
    { key: 'OTHER', name: 'OTRO' }
  ],
  CO: [
    { key: 'CC', name: 'Cédula de Ciudadanía' },
    { key: 'CE', name: 'Cédula de Extranjería' },
    { key: 'TI', name: 'Tarjeta de identidad' },
    { key: 'NIT', name: 'Número de identificación tributaria' },
    { key: 'PASS', name: 'Pasaporte' }
  ],
  EC: [
    { key: 'CC', name: 'Cédula de Ciudadanía' },
    { key: 'CE', name: 'Cédula de Extranjería' },
    { key: 'TI', name: 'Tarjeta de identidad' },
    { key: 'PASS', name: 'Pasaporte' }
  ],
  MX: [
    { key: 'CC', name: 'Cédula de Ciudadanía' },
    { key: 'CE', name: 'Cédula de Extranjería' },
    { key: 'TI', name: 'Tarjeta de identidad' },
    { key: 'PASS', name: 'Pasaporte' }
  ],
}

export default {
  components: {
    CardMethod,
    FormPayCard
  },
  data() {
    return {
      mercadopago: null,
      modalPay: false,
      loading: false,
      isVerifyBin: false,
      error: '',
      paymentMethodId: '',
      paymentMethodToken: '',
      resSdk: {},
      cardIssuers: [],
      card: {
        number: '',
        name: '',
        expiry: '',
        cvc: '',
        documentType: '',
        document: '',
        type: '',
        cardType: '',
        quota: null,
        issuerId: '',
      },
      phone: {
        iso: '',
        number: '',
      },
      discountBank: null,
      planInfo: {
        plan: {
          id: '2c9380847e90980f017e9c4bcf25031a',
          createdAt: '2022-01-14 10:30:27',
          currency: 'COP',
          dateEnd: null,
          dateStart: null,
          description: null,
          frequency: 'ANNUAL',
          information: null,
          metadata: null,
          name: 'Plan Anual',
          organizerId: 909453,
          price: 10000,
          status: 1,
          type: 'basic',
          updatedAt: null,
          visanetProductId: ""
        },
        addressId: '',
        deliveryType: '',
        carnetName: 'junior canaless',
      },
      mercadoPagoKeyCo: 'APP_USR-b4296116-d367-4d0f-97fa-8c9260553f8a',
      payerEmail: 'test_user_63507395@testuser.com',
      collectorToken: 'APP_USR-466114512001040-012716-5be1eb8463b48d2e887b485eb88a7d2f-1064103377',
    };
  },
  computed: {
    documentTypes() {
      return docTypes.CO
    },
    paymentTotal() {
      let total = this.planInfo.plan.price || 0
      if (this.discountBank) {
        total -= this.discountBank
      }
      return total.toFixed(2)
    },
    currency() {
      return {
        iso: 'COP',
        symbol: '$',
      }
    },
  },
  methods: {
    setError(msg) {
      this.error = msg
      setTimeout(() => {
        this.error = ''
      }, 9000)
    },
    submit() {
      console.log("mercadoPagoKeyCo", this.mercadoPagoKeyCo)
      Mercadopago.setPublishableKey(this.mercadoPagoKeyCo)
      Mercadopago.getIdentificationTypes()
      this.modalPay = true
    },
    submitOk() {
      console.log('CARD values', this.card);
      console.log('CONF mercadoPagoKeyCo', this.mercadoPagoKeyCo);
      console.log('CONF payerEmail', this.payerEmail);
      console.log('CONF plan', this.planInfo.plan.id);
      this.loading = true
      const bin = this.card.number.substring(0, 7)
      Mercadopago.getPaymentMethod({ bin }, this.setPaymentMethodInfo)
    },
    setPaymentMethodInfo(status, res) {
      if (status === 200) {
        this.paymentMethodId = res[0].id
        const issuerM = res[0].additional_info_needed.includes('issuer_id')
        if (issuerM) {
          Mercadopago.getIssuers(res[0].id, (_, issuers) => {
            this.cardIssuers = issuers
          });
        }
        const $form = document.querySelector('#pay')
        setTimeout(() => {
          Mercadopago.createToken($form, this.sdkResponseHandler)
        }, 250)
      } else {
        this.loading = false
        // eslint-disable-next-line
        console.log(`Payment method info error: ${response}`)
      }
    },
    sdkResponseHandler(status, res) {
      if (status !== 200 && status !== 201) {
        this.loading = false
        this.setError('Verifica que los datos sean correctos.')
      } else {
        console.log('sdkResponseHandler res', res)
        this.paymentMethodToken = res.id
        setTimeout(() => {
          console.log('sdkResponseHandler res', res)
          this.payProccess()
        }, 250)
      }
    },
    async payProccess() {
      this.loading = true
      try {
        const body = {
          country: 'CO', // this.country,
          preapproval_plan_id: this.planInfo.plan.id,
          card_token_id: this.paymentMethodToken,
          payer_email: this.payerEmail, // userAuth.email,
          collector_token: this.collectorToken,
        }
        console.log('payProccess body',body)
        const url = '/payment/recurrent/mercadopago/subscription/create'
        const res = await axios.post(apiCheckout + url, body)
        console.log(res)
      } catch (err) {
        this.loading = false
        this.toastError('Error, inténtalo en un momento')
      }
    }
  },
};
</script>
