<template>
    <div class="card">
      <div class="card-body">
        <h5 class="card-title">Realizar Transacción</h5>
        <form @submit.prevent="handleSubmit">
          <div class="mb-3">
            <label for="productSelect" class="form-label">Selecciona un producto</label>
            <select v-model="selectedProduct" class="form-select" required>
              <option value="" disabled>Seleccione un producto</option>
              <option v-for="product in products" :key="product.idProducto" :value="product">
                {{ product.producto }} - {{ product.precio }}
              </option>
            </select>
          </div>
          <div class="mb-3">
            <label for="phoneNumber" class="form-label">Número de teléfono</label>
            <input v-model="phoneNumber" type="tel" class="form-control" placeholder="Ingresa el número de teléfono">
          </div>
          <button type="submit" class="btn btn-primary">Realizar Transacción</button>
        </form>
      </div>
    </div>
  </template>
  
  <script>
import { encrypt } from '@/encryption';

  export default {
    data() {
      return {
        selectedProduct: null,
        phoneNumber: '',
        products: [
          { idProducto: 298, idCatTipoServicio: 3, precio: 50.0, producto: 'Recarga AT-T $50' },
          { idProducto: 200, idCatTipoServicio: 10, precio: 100.0, producto: 'Amazon $100' },
          { idProducto: 7124, idCatTipoServicio: 10, precio: 59.0, producto: 'Free Fire 310 Diamonds' },
          { idProducto: 10397, idCatTipoServicio: 11, precio: 109.0, producto: 'Paramount+ 1 mes' }
        ]
      };
    },
    methods: {
      async handleSubmit() {
        if (!this.selectedProduct) {
          this.$emit('transaction-result', {
            type: 'alert-warning',
            message: 'Por favor, selecciona un producto.'
          });
          return;
        }
  
        const payload = {
          idProducto: this.selectedProduct.idProducto,
          idServicio: this.selectedProduct.idCatTipoServicio,
          telefono: this.phoneNumber,
          horaLocal: new Date().toISOString().replace(/[-:.TZ]/g, ""),
          montoPago: this.selectedProduct.precio,
          referencia: '25720381', // Ejemplo de referencia, se debe generar dinámicamente
          upc: 'UPC_Test_XX',
          unidad: '083'
        };
  
        const myHeaders = new Headers();
        myHeaders.append("Authorization", `Bearer ${this.BEARER_TOKEN}`);
        myHeaders.append("Content-Type", "application/x-www-form-urlencoded");
  
        const urlencoded = new URLSearchParams();
        urlencoded.append("random", "random_string"); // Cambiar a una cadena aleatoria generada
        urlencoded.append("signed", btoa(JSON.stringify(payload))); // Encriptar payload como se requiera
  
        const requestOptions = {
          method: "POST",
          headers: myHeaders,
          body: urlencoded,
          redirect: "follow"
        };
  
        try {
          const response = await fetch(`${this.API_URL}sendTx.do`, requestOptions);
          const result = await response.json();
          this.handleResponse(result);
        } catch (error) {
          console.error('Error:', error);
          this.$emit('transaction-result', {
            type: 'alert-danger',
            message: 'Error al realizar la transacción. Por favor, inténtalo de nuevo más tarde.'
          });
        }
      },
      handleResponse(response) {
        if (response.CODIGO === '01') {
          this.$emit('transaction-result', {
            type: 'alert-success',
            message: response.TEXTO,
            details: {
              producto: this.selectedProduct.producto,
              precio: this.selectedProduct.precio,
              referencia: this.payload.referencia,
              PIN: response.PIN,
              legend: this.selectedProduct.legend
            }
          });
        } else {
          this.$emit('transaction-result', {
            type: 'alert-danger',
            message: response.TEXTO
          });
        }
      }
    },
    computed: {
      BEARER_TOKEN() {
        return 'qmzAAEYFmqQbIT/Ktxme8FDV343iYQPwVJM4T39cEinWq1yjc1pp9oxSo4Rjmo5Fk27YzkCDaCr5RM4xtGZNCBZk1MH1tMNbykJ/WzfVjxSEXc9FERnZJPsVuFcAgfsfHmEYEfsLSmLZgovuGmnpDA==';
      },
      API_URL() {
        return 'https://gestopago.portalventas.net/sistema/service/';
      }
    }
  };
  </script>
  
  <style scoped>
  .card {
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }
  </style>
  