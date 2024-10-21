<template>
  <Fragment>
    <!-- <v-btn class="mt-4 ml-4" color="primary" @click="logout"
      >Se déconnecter</v-btn
    >
    <v-btn class="mt-4 ml-4" @click="changePassword"
      >Changer le mot de passe</v-btn
    > -->

    <v-row justify="end" class="mr-8 mt-4">
      <v-menu
        offset-y
        :close-on-content-click="false"
        transition="scale-transition"
        left
      >
        <!-- Activator: Profile picture with first letter of username -->
        <template #activator="{ props }">
          <v-btn v-bind="props" class="profile-avatar" icon>
            <v-avatar color="primary" size="40">
              <span class="white--text text-h6">{{ firstLetter }}</span>
            </v-avatar>
          </v-btn>
        </template>

        <!-- Dropdown menu content -->
        <v-list>
          <v-list-item @click="logout">
            <v-list-item-title>Se déconnecter</v-list-item-title>
          </v-list-item>
          <v-list-item @click="changePassword">
            <v-list-item-title>Modifier le mot de passe</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>
    </v-row>
    <!-- MENU -->

    <v-card class="mt-10">
      <v-tabs v-model="tab" background-color="primary" align-tabs="center">
        <v-tab value="product">Produits</v-tab>
        <v-tab value="commande">Commande </v-tab
        ><v-badge :content="selectedProducts.length" color="red" class="mt-4">
        </v-badge>
      </v-tabs>

      <!-- SELECTION PRODUIT -->
      <v-card-text>
        <v-window v-model="tab">
          <v-window-item value="product">
            <!-- DATATABLE ENTRETIEN -->
            <v-col cols="12">
              <v-data-table
                :headers="entretienHeaders"
                :items="products"
                class="elevation-1"
                item-key="id"
              >
                <!-- <template v-slot:item.history="{ item }">
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
                </template> -->
                <template v-slot:[`item.actions`]="{ item }">
                  <v-btn color="blue darken-1" @click="editProduct(item)">
                    <v-icon small class=""> mdi-pencil </v-icon>
                  </v-btn>
                  <v-btn
                    color="green darken-1"
                    class="ml-xl-2"
                    @click="addToOrder(item)"
                    ><v-icon>mdi-plus</v-icon></v-btn
                  >
                </template>
              </v-data-table>
            </v-col>
          </v-window-item>

          <!-- Edit Product Dialog -->
          <v-dialog v-model="editProductDialog" max-width="600px">
            <v-card class="p-4">
              <v-card-title class="headline">Modifier le produit</v-card-title>
              <v-card-text>
                <v-form ref="editProductForm">
                  <v-number-input
                    v-model="editProductForm.count"
                    label="Quantité"
                    required
                  ></v-number-input>
                </v-form>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="red" @click="closeEditProductDialog"
                  >Annuler</v-btn
                >
                <v-btn color="green" @click="saveProduct">Sauvegarder</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>

          <!-- Commande Tab -->
          <v-window-item value="commande">
            <v-col cols="12">
              <v-data-table
                :headers="orderHeaders"
                :items="selectedProductsDetails"
                item-key="id"
                class="elevation-1"
              >
                <template v-slot:[`item.orderCount`]="{ item }">
                  <div class="d-flex justify-center align-center">
                    <v-text-field
                      v-model="item.orderCount"
                      type="number"
                      min="1"
                      :max="item.count"
                      label="Quantité à commander"
                      dense
                    ></v-text-field>
                  </div>
                </template>
                <template v-slot:[`item.action`]="{ item }">
                  <v-btn color="red" @click="removeFromOrder(item)"
                    ><v-icon>mdi-minus</v-icon></v-btn
                  >
                </template>
              </v-data-table>

              <v-btn color="primary" class="mt-4" @click="orderProducts"
                >Commander</v-btn
              >
            </v-col>
          </v-window-item>
        </v-window>
      </v-card-text>
    </v-card>
  </Fragment>
</template>

<script>
import axios from "axios";
import Swal from "sweetalert2";

export default {
  data() {
    return {
      products: [],
      users: null,
      selectedProducts: [],
      selected: "Avernas",
      name: "",
      isRead: false,
      count: 0,
      tab: null,
      dialog: false,
      user: { name: JSON.parse(localStorage.getItem("user")).username },
      entretienHeaders: [
        { title: "Type", value: "type", align: "center" },
        { title: "Nom", value: "name", align: "center" },
        { title: "Quantité", value: "count", align: "center" },
        // { title: "Historique", value: "history", align: "center" },
        { title: "Conditionnement", value: "conditionnement", align: "center" },
        { title: "Référence", value: "reference", align: "center" },
        { title: "Actions", value: "actions", align: "center" },
      ],
      orderHeaders: [
        { text: "Type", value: "type" },
        { text: "Nom", value: "name" },
        { text: "Quantité à commander", value: "orderCount", align: "center" },
        { text: "Conditionnement", value: "conditionnement" },
        { text: "Référence", value: "reference" },
        { text: "Actions", value: "action" },
      ],
      editProductDialog: false,
      editProductForm: {
        name: "",
        count: 0,
        type: "",
        conditionnement: "",
        reference: "",
      },
    };
  },
  computed: {
    selectedProductsDetails() {
      return this.products.filter((product) =>
        this.selectedProducts.includes(product.id)
      );
    },
    firstLetter() {
      return this.user.name.charAt(0).toUpperCase();
    },
  },

  mounted() {
    this.fetchProducts();
  },
  methods: {
    fetchProducts() {
      axios
        .get(
          `https://stockapp-server-eight.vercel.app/products-${this.selected.toLowerCase()}`
        )
        .then((response) => {
          this.products = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },

    getLastThreeChanges(history) {
      return history.slice(-3).reverse();
    },

    editProduct(product) {
      this.editProductForm.previousCount = product.count;
      this.editProductForm.type = product.type;
      this.editProductForm.conditionnement = product.conditionnement;
      this.editProductForm.reference = product.reference;
      this.editProductForm.id = product.id;
      this.editProductForm.name = product
        ? product.name
        : this.selected === "Depot"
        ? ""
        : "";
      this.editProductDialog = true;
      this.editProductForm.history = product.history;
    },

    closeEditProductDialog() {
      this.editProductDialog = false;
    },

    saveProduct() {
      if (this.$refs.editProductForm.validate()) {
        const user = JSON.parse(localStorage.getItem("user"));
        const username = user.username;

        axios
          .patch(
            `https://stockapp-server-eight.vercel.app/products-${this.selected.toLowerCase()}/${
              this.editProductForm.id
            }`,
            {
              username: username,
              name: this.editProductForm.name,
              count: this.editProductForm.count,
              previousCount: this.editProductForm.previousCount,
              type: this.editProductForm.type,
              conditionnement: this.editProductForm.conditionnement,
              reference: this.editProductForm.reference,
              history: this.editProductForm.history,
            }
          )
          .then(() => {
            axios
              .get(
                `https://stockapp-server-eight.vercel.app/products-${this.selected.toLowerCase()}`
              )
              .then((response) => {
                this.products = response.data;
              });
            Swal.fire("Modifié!", "Le produit a été modifié", "success");
            this.$forceUpdate();
          })
          .catch((error) => {
            console.log(error);
          });
        this.closeEditProductDialog();
      }
    },
    addToOrder(product) {
      if (!this.selectedProducts.includes(product.id)) {
        this.selectedProducts.push(product.id);
        Swal.fire({
          icon: "success",
          title: "Produit ajouté à la commande",
          toast: true,
          timer: 3000,
          timerProgressBar: true,
          position: "top-end",
          showConfirmButton: false,
        });
      } else {
        Swal.fire({
          icon: "error",
          title: "Produit déjà ajouté à la commande",
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
          title: "Produuit retiré de la commande",
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
      const id = user.id;
      Swal.fire({
        title: "Changer le mot de passe",
        html:
          '<input type="password" id="swal-input1" class="swal2-input" placeholder="Ancien mot de passe">' +
          '<input type="password" id="swal-input2" class="swal2-input" placeholder="Nouveau mot de passe">' +
          '<input type="password" id="swal-input3" class="swal2-input" placeholder="Confirmer le nouveau mot de passe">',
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
<style>
.v-list-item__content {
  display: flex;
  justify-content: center;
  align-items: center;
}
.line-through {
  text-decoration: line-through;
}
.icon-gray {
  color: gray;
}
.icon-default {
  color: red;
}

.notification-item {
  position: relative;
  display: flex;
  align-items: center;
}

.text-with-icon {
  flex: 1;
  position: relative;
  padding-right: 40px; /* Space for the icon */
}

.text-container {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.icon {
  position: absolute;
  right: 0;
  top: 50%;
  transform: translateY(-50%);
}
</style>
