<template>
  <div class="container mt-5">
    <TransactionForm />
    <div v-if="transactionResult.show" class="alert mt-4" :class="transactionResult.type" role="alert">
      {{ transactionResult.message }}
    </div>
    <div v-if="paymentReceipt.show" class="card mt-4">
      <div class="card-body">
        <h5 class="card-title">Comprobante de Pago</h5>
        <p>Producto: {{ paymentReceipt.details.producto }}</p>
        <p>Precio: {{ paymentReceipt.details.precio }}</p>
        <p>Referencia: {{ paymentReceipt.details.referencia }}</p>
        <p>PIN: {{ paymentReceipt.details.PIN || 'N/A' }}</p>
        <p>Instrucciones: {{ paymentReceipt.details.legend }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import TransactionForm from '../components/TransactionForm.vue';

export default {
  components: {
    TransactionForm
  },
  data() {
    return {
      transactionResult: {
        show: false,
        type: '',
        message: ''
      },
      paymentReceipt: {
        show: false,
        details: {}
      }
    };
  },
  methods: {
    handleTransactionResult(result) {
      this.transactionResult.show = true;
      this.transactionResult.type = result.type;
      this.transactionResult.message = result.message;

      if (result.type === 'alert-success') {
        this.paymentReceipt.show = true;
        this.paymentReceipt.details = result.details;
      } else {
        this.paymentReceipt.show = false;
      }
    }
  }
};
</script>

<style scoped>
.container {
  max-width: 600px;
}
</style>
