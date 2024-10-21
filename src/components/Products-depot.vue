<template>
  <Fragment>
    <v-btn class="mt-4 ml-4" color="primary" @click="logout">Déconnexion</v-btn>
    <v-btn class="mt-4 ml-4" @click="changePassword"
      >Changer mot de passe</v-btn
    >

    <v-tabs v-model="tab" background-color="primary" align-tabs="center">
      <v-tab value="product">Dépôt</v-tab>
      <v-tab value="commande">Commande</v-tab>
    </v-tabs>

    <v-window v-model="tab">
      <v-window-item value="product">
        <v-col cols="12">
          <!-- Search bar -->
          <v-data-table
            :headers="headers"
            :items="products"
            item-key="id"
            class="elevation-1"
          >
            <template v-slot:item.history="{ item }">
              <v-list dense>
                <v-list-item-group>
                  <v-list-item
                    v-for="(history, index) in getLastThreeChanges(
                      item.history
                    )"
                    :key="index"
                  >
                    <v-list-item-content>
                      <v-list-item-title
                        >{{ history.date }}
                        {{ history.updatedBy }}</v-list-item-title
                      >
                      <v-list-item-subtitle>
                        {{ history.count }}
                      </v-list-item-subtitle>
                    </v-list-item-content>
                  </v-list-item>
                </v-list-item-group>
              </v-list>
            </template>
            <template v-slot:item.action="{ item }">
              <div class="d-flex justify-center align-center">
                <v-btn color="primary" @click="openUpdateDialog(item)"
                  ><v-icon>mdi-pencil</v-icon></v-btn
                >
                <v-btn
                  color="green darken-1"
                  class="ml-2"
                  @click="addToOrder(item)"
                  ><v-icon>mdi-plus</v-icon></v-btn
                >
              </div>
            </template>
          </v-data-table>
        </v-col>
      </v-window-item>

      <v-window-item value="commande">
        <v-col cols="12">
          <v-data-table
            :headers="orderHeaders"
            :items="selectedProductsDetails"
            item-key="id"
            class="elevation-1"
          >
            <template v-slot:item.orderCount="{ item }">
              <div class="d-flex justify-center align-center">
                <v-text-field
                  class="mt-4"
                  v-model="item.orderCount"
                  type="number"
                  min="1"
                  :max="item.count"
                  label="Quantité"
                  dense
                ></v-text-field>
              </div>
            </template>
            <template v-slot:item.action="{ item }">
              <v-btn color="red" @click="removeFromOrder(item)"
                ><v-icon>mdi-delete</v-icon></v-btn
              >
            </template>
          </v-data-table>

          <v-btn color="primary " class="mt-4" @click="orderProducts"
            >Commander
          </v-btn>
        </v-col>
      </v-window-item>
    </v-window>

    <!-- Dialog for updating product count -->
    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title class="headline text-center"
          >Mettre à jour la quantité</v-card-title
        >
        <v-card-text>
          <v-form ref="form">
            <v-text-field
              v-model="updatedCount"
              label="Nouvelle quantité"
              type="number"
              min="0"
              required
              @keypress.enter="updateProductCount"
            ></v-text-field>
          </v-form>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="red darken-1" text @click="closeDialog">Annuler</v-btn>
          <v-btn color="green darken-1" text @click="updateProductCount"
            >Sauvegarder</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
  </Fragment>
</template>

<script>
import axios from "axios";
import Swal from "sweetalert2";

export default {
  data() {
    return {
      products: [],
      productsAdmin: [],
      selectedProducts: [],
      tab: "product",
      dialog: false,
      updatedCount: 0,
      currentProduct: null,
      user: { name: JSON.parse(localStorage.getItem("user")).username }, // Replace with actual user information
      headers: [
        { title: "Nom", value: "name", align: "center" },
        { title: "Quantité", value: "count", align: "center" },
        // {
        //   title: "Historique",
        //   value: "history",
        //   sortable: false,
        //   align: "center",
        // },
        { title: "Actions", value: "action", sortable: false, align: "center" },
      ],
      orderHeaders: [
        { title: "Nom", value: "name" },
        // { title: "Quantité ", value: "count", align: "center" },
        {
          title: "Quantité à Commander",
          value: "orderCount",
          sortable: false,
          align: "center",
        },
        { title: "Actions", value: "action", sortable: false, align: "center" },
      ],
    };
  },
  mounted() {
    axios
      .get("https://stockapp-server-eight.vercel.app/products-depot")
      .then((response) => {
        this.products = response.data.map((product) => ({
          ...product,
          orderCount: 1,
        }));
      });
  },
  computed: {
    selectedProductsDetails() {
      return this.products.filter((product) =>
        this.selectedProducts.includes(product.id)
      );
    },
  },
  filteredProducts() {
    return this.products.filter((product) => {
      return product.name.toLowerCase().includes(this.search.toLowerCase());
    });
  },

  methods: {
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
    updateProductCount() {
      if (this.updatedCount === this.currentProduct.count) {
        this.closeDialog();
        return;
      }

      if (this.currentProduct.count === undefined) {
        this.currentProduct.count = 0;
      }

      const updatedProduct = {
        id: this.currentProduct.id,
        name: this.currentProduct.name,
        previousCount: this.currentProduct.count,
        newCount: this.updatedCount,
        username: this.user.name,
        _id: this.currentProduct._id,
        history: this.currentProduct.history,
      };

      console.log("Updated product", updatedProduct);

      axios
        .patch(
          `https://stockapp-server-eight.vercel.app/products-depot/${this.currentProduct.id}`,
          {
            count: this.newCount,
            previousCount: this.currentProduct.count,
            newCount: this.updatedCount,
            username: this.user.name,
            name: this.currentProduct.name,
            _id: this.currentProduct._id,
            history: this.currentProduct.history,
          }
        )
        .then((response) => {
          const index = this.products.findIndex(
            (product) => product.id === this.currentProduct.id
          );
          this.products[index] = response.data;
          console.log("Product count updated successfully", response.data);
          this.closeDialog();
          Swal.fire({
            icon: "success",
            title: "Product count updated successfully",
            toast: true,
            timer: 3000,
            timerProgressBar: true,
            position: "center",
            showConfirmButton: false,
          });
        })
        .catch((error) => {
          console.error("Error updating product count", error);
        });
    },
    getLastThreeChanges(history) {
      return history.slice(-3).reverse();
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
    removeFromOrder(product) {
      const index = this.selectedProducts.indexOf(product.id);
      if (index > -1) {
        this.selectedProducts.splice(index, 1);
        Swal.fire({
          icon: "success",
          title: "Product removed from order",
          toast: true,
          timer: 3000,
          timerProgressBar: true,
          position: "top-end",
          showConfirmButton: false,
        });
      }
    },
    orderProducts() {
      const user = JSON.parse(localStorage.getItem("user"));
      console.log("User", user);
      if (this.selectedProducts.length === 0) {
        Swal.fire({
          icon: "error",
          title: "Pas de produit sélectionné",
          toast: true,
          timer: 3000,
          timerProgressBar: true,
          position: "center",
          showConfirmButton: false,
        });
        return;
      }
      const orders = this.selectedProductsDetails.map((product) => ({
        id: product.id,
        commandCount: product.orderCount,
        role: user.role,
        user: user.username,
        name: product.name,
        count: product.count,
        type: product.type,
        conditionnement: product.conditionnement,
        reference: product.reference,
        history: product.history,
      }));

      console.log("Ordering products", orders);

      axios
        .post("https://stockapp-server-eight.vercel.app/commandes", orders)
        .then((response) => {
          console.log("Products ordered successfully", response.data);
          this.selectedProducts = [];
          Swal.fire({
            icon: "success",
            title:
              orders.length > 1
                ? "Produits commandés avec succès"
                : "Produit commandé avec succès",
            toast: true,
            timer: 3000,
            timerProgressBar: true,
            toast: true,
            position: "top-right",
            showConfirmButton: false,
          });
        })
        .catch((error) => {
          console.error("Error ordering products", error);
        });
    },
    changePassword() {
      const user = JSON.parse(localStorage.getItem("user"));
      const id = user[0].id;
      Swal.fire({
        title: "Changer le mot de passe",
        html:
          '<input id="swal-input1" class="swal2-input" placeholder="Ancien mot de passe">' +
          '<input id="swal-input2" class="swal2-input" placeholder="Nouveau mot de passe">' +
          '<input id="swal-input3" class="swal2-input" placeholder="Confirmer le nouveau mot de passe">',
        focusConfirm: false,
        preConfirm: () => {
          return [
            document.getElementById("swal-input1").value,
            document.getElementById("swal-input2").value,
            document.getElementById("swal-input3").value,
          ];
        },
      }).then((result) => {
        if (result.value) {
          const [oldPassword, newPassword, confirmPassword] = result.value;
          if (newPassword !== confirmPassword) {
            Swal.fire(
              "Erreur",
              "Les mots de passe ne correspondent pas",
              "error"
            );
            return;
          }
          axios
            .patch(
              `https://stockapp-server-eight.vercel.app/users-change-password/${id}`,
              {
                oldPassword,
                newPassword,
              }
            )
            .then((response) => {
              Swal.fire("Succès", "Mot de passe changé avec succès", "success");
            })
            .catch((error) => {
              Swal.fire("Erreur", "Ancien mot de passe incorrect", "error");
            });
        }
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
.centered {
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
