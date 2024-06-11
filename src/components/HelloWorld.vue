<template>
  <div id="app" class="container mt-5">
    <div class="card">
      <div class="card-body">
        <h5 class="card-title">Realizar Transacción</h5>
        <form @submit.prevent="sendTransaction">
          <div class="mb-3">
            <label for="productSelect" class="form-label">Selecciona un producto</label>
            <select v-model="selectedProduct" class="form-select" required>
              <option disabled value="">Seleccione un producto</option>
              <option v-for="product in products" :key="product.idProducto" :value="product">
                {{ product.producto }}
              </option>
            </select>
          </div>
          <div class="mb-3">
            <label for="phoneNumber" class="form-label">Número de teléfono</label>
            <input v-model="phoneNumber" type="tel" class="form-control" placeholder="Ingresa el número de teléfono" required>
          </div>
          <button type="submit" class="btn btn-primary">Realizar Transacción</button>
        </form>
      </div>
    </div>
    <div v-if="message" :class="`alert mt-4 ${message.type}`" role="alert">
      {{ message.text }}
    </div>
    <div v-if="receipt" class="card mt-4">
      <div class="card-body">
        <h5 class="card-title">Comprobante de Pago</h5>
        <p>Producto: {{ receipt.producto }}</p>
        <p>Precio: {{ receipt.precio }}</p>
        <p>Referencia: {{ receipt.referencia }}</p>
        <p>PIN: {{ receipt.pin }}</p>
        <p>Instrucciones: {{ receipt.legend }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      products: [
        { idProducto: 298, idCatTipoServicio: 3, precio: 50.0, producto: 'Recarga AT-T $50' },
        { idProducto: 200, idCatTipoServicio: 10, precio: 100.0, producto: 'Amazon $100' },
        { idProducto: 7124, idCatTipoServicio: 10, precio: 59.0, producto: 'Free Fire 310 Diamonds' },
        { idProducto: 10397, idCatTipoServicio: 11, precio: 109.0, producto: 'Paramount+ 1 mes' }
      ],
      selectedProduct: '',
      phoneNumber: '',
      message: null,
      receipt: null,
    };
  },
  methods: {
    async sendTransaction() {
      const payload = {
        idProducto: this.selectedProduct.idProducto,
        idServicio: this.selectedProduct.idCatTipoServicio,
        telefono: this.phoneNumber,
        horaLocal: new Date().toISOString().replace(/[-:.TZ]/g, ""),
        montoPago: this.selectedProduct.precio,
        referencia: '25720381',
        upc: 'UPC_Test_XX',
        unidad: '083'
      };

      const myHeaders = new Headers();
      myHeaders.append("Authorization", `Bearer qmzAAEYFmqQbIT/Ktxme8FDV343iYQPwVJM4T39cEinWq1yjc1pp9oxSo4Rjmo5Fk27YzkCDaCr5RM4xtGZNCBZk1MH1tMNbykJ/WzfVjxSEXc9FERnZJPsVuFcAgfsfHmEYEfsLSmLZgovuGmnpDA==`);
      myHeaders.append("Content-Type", "application/x-www-form-urlencoded");

      const urlencoded = new URLSearchParams();
      urlencoded.append("random", "random_string");
      urlencoded.append("signed", btoa(JSON.stringify(payload)));

      const requestOptions = {
        method: "POST",
        headers: myHeaders,
        body: urlencoded,
        redirect: "follow"
      };

      try {
        const response = await fetch("https://gestopago.portalventas.net/sistema/service/sendTx.do", requestOptions);
        const result = await response.json();
        if (result.CODIGO === '01') {
          this.message = { text: result.TEXTO, type: 'alert-success' };
          this.confirmTransaction(payload);
        } else {
          this.message = { text: result.TEXTO, type: 'alert-danger' };
        }
      } catch (error) {
        console.error('Error:', error);
        this.message = { text: 'Error al realizar la transacción. Por favor, inténtalo de nuevo más tarde.', type: 'alert-danger' };
      }
    },
    async confirmTransaction(payload) {
      await new Promise(resolve => setTimeout(resolve, 62000)); // Esperar 62 segundos

      const myHeaders = new Headers();
      myHeaders.append("Authorization", `Bearer qmzAAEYFmqQbIT/Ktxme8FDV343iYQPwVJM4T39cEinWq1yjc1pp9oxSo4Rjmo5Fk27YzkCDaCr5RM4xtGZNCBZk1MH1tMNbykJ/WzfVjxSEXc9FERnZJPsVuFcAgfsfHmEYEfsLSmLZgovuGmnpDA==`);
      myHeaders.append("Content-Type", "application/x-www-form-urlencoded");

      const urlencoded = new URLSearchParams();
      urlencoded.append("random", "random_string");
      urlencoded.append("signed", btoa(JSON.stringify(payload)));

      const requestOptions = {
        method: "POST",
        headers: myHeaders,
        body: urlencoded,
        redirect: "follow"
      };

      try {
        const response = await fetch("https://gestopago.portalventas.net/sistema/service/confirmTx.do", requestOptions);
        const result = await response.json();
        if (result.CODIGO === '01') {
          this.receipt = {
            producto: this.selectedProduct.producto,
            precio: this.selectedProduct.precio,
            referencia: payload.referencia,
            pin: result.PIN,
            legend: this.selectedProduct.legend
          };
        } else {
          this.message = { text: result.TEXTO, type: 'alert-danger' };
        }
      } catch (error) {
        console.error('Error:', error);
        this.message = { text: 'Error al confirmar la transacción. Por favor, inténtalo de nuevo más tarde.', type: 'alert-danger' };
      }
    }
  }
};
</script>

<style>
body {
  background-color: #f8f9fa;
}

.container {
  max-width: 600px;
  padding-top: 20px;
}

.card {
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.card, .alert {
  margin-bottom: 20px;
}
</style>
