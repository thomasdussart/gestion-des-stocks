<template>
  <Fragment>
    <v-btn class="mt-4 ml-4" color="primary" @click="logout">Logout</v-btn>

    <v-tabs v-model="tab" background-color="primary" align-tabs="center">
      <v-tab value="product">Économat</v-tab>
      <v-tab value="commande">Commande</v-tab>
    </v-tabs>

    <v-window v-model="tab">
      <v-window-item value="product">
        <v-col cols="12">
          <v-data-table
            :headers="headers"
            :items="products"
            item-key="id"
            class="elevation-1"
          >
            <template v-slot:item.history="item">
              <v-list dense>
                <v-list-item-group>
                  <v-list-item
                    v-for="(history, index) in getLastThreeChanges(
                      item.history
                    )"
                    :key="index"
                  >
                    <v-list-item-content>
                      <v-list-item-title>
                        {{ history.date }} {{ history.updatedBy }}
                      </v-list-item-title>
                      <v-list-item-subtitle>
                        {{ history.count }}
                      </v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>
                </v-list-item-group>
              </v-list>
            </template>
            <template v-slot:item.action="item">
              <div class="d-flex justify-center align-center">
                <v-btn color="primary" @click="openUpdateDialog(item)"
                  >Update Count</v-btn
                >
                <v-btn
                  color="green darken-1"
                  class="ml-2"
                  @click="addToOrder(item)"
                  >Add to Order</v-btn
                >
              </div>
            </template>
          </v-data-table>
        </v-col>
      </v-window-item>

      <v-window-item value="commande">
        <v-col cols="12">
          <!-- Add the content for the "Commande" tab here -->
        </v-col>
      </v-window-item>
    </v-window>
  </Fragment>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      products: [],
      tab: "product",
      headers: [
        { text: "Product", value: "name" },
        { text: "Current Stock", value: "count" },
        { text: "History", value: "history" },
        { text: "Actions", value: "action", sortable: false },
      ],
    };
  },
  mounted() {
    axios
      .get("http://localhost:1337/products-economat")
      .then(() => {
        this.products = response.data;
      })
      .catch((error) => {
        console.log(error);
      });
  },
  methods: {
    getLastThreeChanges(history) {
      return history.slice(-3);
    },
    openUpdateDialog(product) {
      this.currentProduct = product;
      this.updatedCount = product.count;
      this.dialog = true;
    },
    closeDialog() {
      this.dialog = false;
      this.currentProduct = null;
      this.updatedCount = 0;
    },
    addToOrder(product) {
      if (!this.selectedProducts.includes(product.id)) {
        this.selectedProducts.push(product.id);
        Swal.fire({
          icon: "success",
          title: "Product added to order",
          toast: true,
          timer: 3000,
          timerProgressBar: true,
          position: "top-end",
          showConfirmButton: false,
        });
      }
    },
    change(id, count) {
      const user = localStorage.getItem("user");
      const parsedUser = JSON.parse(user);

      axios
        .patch(`http://localhost:1337/products-economat/${id}`, {
          count: count,
          user: parsedUser,
        })
        .then(() => {
          const index = this.products.findIndex((product) => product.id === id);
          this.products[index].count = count;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    remove(id) {
      axios
        .delete(`http://localhost:1337/products-economat/${id}`)
        .then(() => {
          const index = this.products.findIndex((product) => product.id === id);
          this.products.splice(index, 1);
        })
        .catch((error) => {
          console.log(error);
        });
    },
    logout() {
      localStorage.clear();
      window.location.reload();
    },
  },
};
</script>

<style scoped>
/* Add any component-specific styles here */
</style>
