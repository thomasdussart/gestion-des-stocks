<template>
  <Fragment>
    <v-responsive>
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
      <v-row justify="start" class="ml-8">
        <v-btn color="primary" @click="handleNotificationsClick">
          NOTIFICATIONS
        </v-btn>
        <v-badge
          v-if="notificationCount > 0"
          :content="notificationCount"
          color="red"
          overlap
        >
        </v-badge>
      </v-row>

      <!-- Notifications Dialog -->
      <v-dialog v-model="dialog" max-width="1250">
        <v-card>
          <v-card-title class="headline">Notifications</v-card-title>

          <v-card-text>
            <!-- Data Table for Notifications -->
            <v-data-table
              :headers="notificationHeaders"
              :items="notifications"
              :items-per-page="5"
              class="elevation-1"
              :footer-props="{
                'items-per-page-options': [5, 10, 15],
              }"
            >
              <template v-slot:item.text="{ item }">
                <span :class="{ 'strike-through': item.isRead }">{{
                  item.text
                }}</span>
              </template>

              <!-- Action buttons column -->
              <template v-slot:item.actions="{ item, index }">
                <v-btn icon @click="notificationMarkAsRead(item.id, index)">
                  <v-icon v-if="item.isRead">mdi-check-circle-outline</v-icon>
                  <v-icon v-else>mdi-checkbox-blank-circle-outline</v-icon>
                </v-btn>
              </template>
            </v-data-table>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" @click="dialog = false">Fermer</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <!-- MENU -->

      <v-card class="mt-10">
        <v-tabs v-model="tab" background-color="primary" align-tabs="center">
          <v-tab value="product">Produits</v-tab>
          <v-tab value="users">Utilisateurs</v-tab>
          <v-tab value="delivery">Livraisons</v-tab>
        </v-tabs>

        <!-- SELECTION PRODUIT -->
        <v-card-text>
          <v-window v-model="tab">
            <v-window-item value="product">
              <v-select
                v-model="selected"
                :items="items"
                label="Produit"
                outlined
                dense
                @update:model-value="selectedProducts"
              />

              <br />
              <v-btn
                v-if="this.selected"
                @click="editProductDialog = true"
                class=""
                color="green"
                >Ajouter un produit</v-btn
              >

              <!-- DATATABLE ENTRETIEN -->
              <v-col cols="12">
                <v-data-table
                  :headers="entretienHeaders"
                  :items="products"
                  class="elevation-1"
                  item-key="id"
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
                  <template v-slot:[`item.actions`]="{ item }">
                    <v-btn color="blue darken-1" @click="editProduct(item)">
                      <v-icon small class=""> mdi-pencil </v-icon>
                    </v-btn>
                    <v-btn
                      color="red darken-1"
                      class="ml-xl-2"
                      click="deleteProduct(item)"
                    >
                      <v-icon small> mdi-delete </v-icon>
                    </v-btn>
                  </template>
                </v-data-table>
              </v-col>
            </v-window-item>

            <!-- Edit Product Dialog -->
            <v-dialog v-model="editProductDialog" max-width="600px">
              <v-card>
                <v-card-title class="headline">Edit Product</v-card-title>
                <v-card-text>
                  <v-form ref="editProductForm">
                    <v-text-field
                      v-model="editProductForm.type"
                      label="Type"
                      required
                    ></v-text-field>
                    <v-text-field
                      v-model="editProductForm.name"
                      label="Name"
                      required
                    ></v-text-field>
                    <v-number-input
                      v-model="editProductForm.count"
                      label="Quantité"
                      required
                    ></v-number-input>
                    <v-text-field
                      v-model="editProductForm.conditionnement"
                      label="Conditionnement"
                      required
                    ></v-text-field>
                    <v-text-field
                      v-model="editProductForm.reference"
                      label="Référence"
                      required
                    ></v-text-field>
                  </v-form>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="red" @click="closeEditProductDialog"
                    >Cancel</v-btn
                  >
                  <v-btn color="green" @click="saveProduct">Save</v-btn>
                </v-card-actions>
              </v-card>
            </v-dialog>

            <!-- Delivery tab -->
            <v-window-item value="delivery">
              <v-data-table
                :headers="deliveryHeaders"
                :items="livraisons"
                class="elevation-1"
                :items-per-page="10"
                item-key="product"
              >
                <template v-slot:[`item.status`]="{ item }">
                  <v-chip
                    :color="item.status === 'En attente' ? 'red' : 'green'"
                    dark
                  >
                    {{ item.status }}
                  </v-chip>
                </template>
                <!-- Actions column with edit button -->
                <template v-slot:[`item.actions`]="{ item }">
                  <v-btn @click="openDeliveryDialog(item)" color="primary">
                    Edit
                  </v-btn>
                  <v-btn
                    color="red darken-1"
                    class="ml-2"
                    @click="deleteDelivery(item)"
                  >
                    <v-icon small> mdi-delete </v-icon>
                  </v-btn>
                </template>
              </v-data-table>
            </v-window-item>

            <!-- Edit Delivery Dialog -->
            <v-dialog v-model="editDeliveredByDialog" max-width="500px">
              <v-card>
                <v-card-title> Edit Order </v-card-title>
                <v-card-text>
                  <v-form ref="form">
                    <!-- Status dropdown -->
                    <v-select
                      v-model="selectedItem.status"
                      :items="['En attente', 'Effectuée']"
                      label="Status"
                    ></v-select>

                    <!-- Delivered By dropdown -->
                    <v-select
                      v-model="selectedItem.whoDelivering"
                      :items="deliveredBy"
                      label="Delivered By"
                    ></v-select>
                  </v-form>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn color="green darken-1" text @click="saveDelivery"
                    >Save</v-btn
                  >
                  <v-btn color="red darken-1" text @click="closeDeliveryDialog"
                    >Cancel</v-btn
                  >
                </v-card-actions>
              </v-card>
            </v-dialog>

            <!-- Users Tab -->
            <v-window-item value="users">
              <v-btn @click="saveUserDialog = true" color="green"
                >Ajouter un utilisateur</v-btn
              >
              <v-data-table
                :headers="userHeaders"
                :items="users"
                class="elevation-1"
                :items-per-page="10"
                item-key="id"
              >
                <template v-slot:[`item.actions`]="{ item }">
                  <v-btn color="blue darken-1" @click="editUser(item)">
                    <v-icon small class="mr-2"> mdi-pencil </v-icon>
                  </v-btn>
                  <v-btn
                    color="green darken-1"
                    class="ml-xl-2 ml-l-2 ml-md-2 ml-sm-2 ml-xs-2"
                    @click="resetUserPassword(item)"
                  >
                    <v-icon small class=""> mdi-lock-reset </v-icon>
                  </v-btn>
                  <v-btn
                    color="red darken-1"
                    class="ml-xl-2 ml-l-2 ml-md-2 ml-sm-2 ml-xs-2"
                    @click="deleteUser(item)"
                  >
                    <v-icon small> mdi-delete </v-icon>
                  </v-btn>
                </template>
              </v-data-table>
            </v-window-item>
          </v-window>
        </v-card-text>
      </v-card>

      <!-- Edit User Dialog -->
      <v-dialog v-model="editUserDialog" max-width="600px">
        <v-card>
          <v-card-title class="headline">Edit User</v-card-title>
          <v-card-text>
            <v-form ref="editUserForm">
              <v-text-field
                v-model="editUserForm.name"
                label="Name"
                required
              ></v-text-field>
              <v-select
                v-model="editUserForm.role"
                :items="roles"
                label="Role"
                required
              ></v-select>
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="green" @click="saveEditedUser">Save</v-btn>
            <v-btn color="red" @click="closeEditUserDialog">Cancel</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
      <!-- Save User Dialog -->
      <v-dialog v-model="saveUserDialog" max-width="600px">
        <v-card>
          <v-card-title class="headline">Nouvel utilisateur</v-card-title>
          <v-card-text>
            <v-form ref="saveUserForm">
              <v-text-field
                v-model="saveUserForm.name"
                label="Name"
                required
              ></v-text-field>
              <v-select
                v-model="saveUserForm.role"
                :items="roles"
                label="Role"
                required
              ></v-select>
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="green" @click="saveUser">Save</v-btn>
            <v-btn color="red" @click="closeSaveUserDialog">Cancel</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-responsive>
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
      selected: null,
      items: [
        "Stock",
        "Depot",
        "LSR",
        "Avernas",
        "GH",
        "Moxhe",
        "Thisnes",
        "Merdorp",
        "AC",
        "AHDV",
        "MDS",
        "Bibliotheque",
        "Emploi",
        "Saline",
      ],
      deliveredBy: ["Bob", "Alice", "John", "Doe", "Jane", "Smith", "Doe"],
      name: "",
      isRead: false,
      notificationCount: 0,
      notifications: [],
      livraisons: [],
      count: 0,
      tab: null,
      tabEntretien: null,
      entretien: false,
      dialog: false,
      user: { name: JSON.parse(localStorage.getItem("user")).username },

      userHeaders: [
        { title: "Nom", value: "username", align: "center" },
        { title: "Role", value: "role", align: "center" },
        { title: "Actions", value: "actions", align: "center" },
      ],
      entretienHeaders: [
        { title: "Type", value: "type", align: "center" },
        { title: "Nom", value: "name", align: "center" },
        { title: "Quantité", value: "count", align: "center" },
        { title: "Historique", value: "history", align: "center" },
        { title: "Conditionnement", value: "conditionnement", align: "center" },
        { title: "Référence", value: "reference", align: "center" },
        { title: "Actions", value: "actions", align: "center" },
      ],
      notificationHeaders: [
        { title: "Date", value: "date", align: "center" },
        { title: "Notification", value: "text", align: "center" },
        { title: "Action", value: "actions", align: "center" },
      ],
      deliveryHeaders: [
        { title: "Produit", value: "product", align: "center" },
        { title: "Demandé par", value: "whoAsked", align: "center" },
        { title: "Date de la demande", value: "dateOfAsking", align: "center" },
        { title: "Statut", value: "status", align: "center" },
        { title: "Livraison par", value: "whoDelivering", align: "center" },
        { title: "Lieu de livraison", value: "location", align: "center" },
        {
          title: "Date de livraison",
          value: "dateOfDelivery",
          align: "center",
        },
        { title: "Actions", value: "actions", align: "center" },
      ],

      // Delivery Dialog
      editDeliveredByDialog: false,
      selectedItem: {
        status: "",
        whoDelivering: "",
      },

      // Edit Product Dialog

      editProductDialog: false,
      editProductForm: {
        name: "",
        count: 0,
        type: "",
        conditionnement: "",
        reference: "",
      },

      // Edit User Dialog
      editUserDialog: false,
      saveUserDialog: false,

      editUserForm: {
        id: null,
        name: "",
        password: "",
        role: "",
      },

      saveUserForm: {
        name: "",
        password: "",
        role: "",
      },

      roles: [
        "Admin",
        "Depot",
        "LSR",
        "Avernas",
        "GH",
        "Moxhe",
        "Thisnes",
        "Merdorp",
        "AC",
        "AHDV",
        "MDS",
        "Bibliotheque",
        "Emploi",
        "Saline",
        "Economat",
      ],
    };
  },
  computed: {
    async selectedProducts() {
      if (!this.selected) return;
      try {
        const response = await axios.get(
          `https://stockapp-server-eight.vercel.app/products-${this.selected.toLowerCase()}`
        );
        console.log(this.selected);
        return (this.products = response.data);
      } catch (error) {
        console.error(error);
      }
    },
    firstLetter() {
      return this.user.name.charAt(0).toUpperCase();
    },
  },

  mounted() {
    this.fetchUsers();
    this.getDeliveries();
    this.fetchNotifications();
    setInterval(() => {
      this.fetchNotifications();
    }, 30000);
  },
  methods: {
    getDeliveries() {
      axios
        .get("https://stockapp-server-eight.vercel.app/livraisons")
        .then((response) => {
          this.livraisons = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },

    openDeliveryDialog(item) {
      this.editDeliveredByDialog = true;
      this.selectedItem = { ...item };
      console.log(this.selectedItem);
    },
    closeDeliveryDialog() {
      this.editDeliveredByDialog = false;
    },
    saveDelivery() {
      console.log(this.selectedItem.id);
      axios
        .patch(
          `https://stockapp-server-eight.vercel.app/livraisons/${this.selectedItem.id}`,
          {
            whoDelivering: this.selectedItem.whoDelivering,
            status: this.selectedItem.status,
          }
        )
        .then(() => {
          this.getDeliveries();
          this.closeDeliveryDialog();
        })
        .catch((error) => {
          console.log(error);
        });
    },

    fetchUsers() {
      axios
        .get("https://stockapp-server-eight.vercel.app/users")
        .then((response) => {
          this.users = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },

    getLastThreeChanges(history) {
      return history.slice(-3).reverse();
    },
    fetchNotifications() {
      axios
        .get("https://stockapp-server-eight.vercel.app/notifications")
        .then((response) => {
          let numberOfNotifications = response.data.filter(
            (notification) => notification.isRead === false
          );
          this.notificationCount = numberOfNotifications.length;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    flushNotifications() {
      this.notifications = this.notifications.filter(
        (notification) => !notification.isRead
      );
    },
    async notificationMarkAllAsRead() {
      try {
        await axios.patch(
          "https://stockapp-server-eight.vercel.app/notifications",
          { isRead: 1 }
        );
        await this.fetchNotifications();
        this.notifications.forEach((notification) => {
          notification.isRead = true;
        });
      } catch (error) {
        console.log(error);
      }
    },
    async notificationMarkAsRead(id, index) {
      try {
        await axios.patch(
          `https://stockapp-server-eight.vercel.app/notifications/${id}`,
          {
            isRead: 1,
          }
        );
        await this.fetchNotifications();
        this.notifications[index].isRead = true;
      } catch (error) {
        console.log(error);
      }
    },

    handleNotificationsClick() {
      this.showNotifications();
    },
    showNotifications() {
      this.dialog = true; // Open the notifications dialog
      axios
        .get("https://stockapp-server-eight.vercel.app/notifications")
        .then((response) => {
          this.notifications = response.data.sort(
            (a, b) => new Date(b.date) - new Date(a.date)
          );
        })
        .catch((error) => {
          console.log(error);
        });
    },
    deleteProduct(product) {
      Swal.fire({
        title: "Etes-vous certain de vouloir supprimer le produit?",
        text: "Vous ne pourrez pas revenir en arrière !",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Oui, supprimez-le !",
        cancelButtonText: "Annuler",
      }).then((result) => {
        if (result.isConfirmed) {
          axios
            .delete(
              `https://stockapp-server-eight.vercel.app/products-${this.selected.toLowerCase()}/${
                product.id
              }`
            )
            .then(() => {
              this.$forceUpdate();
              axios
                .get(
                  `https://stockapp-server-eight.vercel.app/products-${this.selected.toLowerCase()}`
                )
                .then((response) => {
                  this.products = response.data;
                });
              Swal.fire("Supprimé!", "Le produit a été supprimé", "success");
            });
        }
      });
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

    deleteDelivery(delivery) {
      Swal.fire({
        title: "Etes-vous certain de vouloir supprimer la livraison?",
        text: "Vous ne pourrez pas revenir en arrière !",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Oui, supprimez-la !",
        cancelButtonText: "Annuler",
      }).then((result) => {
        if (result.isConfirmed) {
          axios
            .delete(
              `https://stockapp-server-eight.vercel.app/livraisons/${delivery.id}`
            )
            .then(() => {
              this.$forceUpdate();
              axios
                .get("https://stockapp-server-eight.vercel.app/livraisons")
                .then((response) => {
                  this.livraisons = response.data;
                });
              Swal.fire("Supprimé!", "La livraison a été supprimée", "success");
            });
        }
      });
    },

    deleteUser(user) {
      Swal.fire({
        title: "Etes-vous certain de vouloir supprimer l'utilisateur?",
        text: "Vous ne pourrez pas revenir en arrière !",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Oui, supprimez-le !",
        cancelButtonText: "Annuler",
      }).then((result) => {
        if (result.isConfirmed) {
          axios
            .delete(`https://stockapp-server-eight.vercel.app/users/${user.id}`)
            .then(() => {
              axios
                .get("https://stockapp-server-eight.vercel.app/users")
                .then((response) => {
                  this.users = response.data;
                  this.$forceUpdate();
                });
              Swal.fire("Supprimé!", "L'utilisateur a été supprimé", "success");
            });
        }
      });
    },

    editUser(user) {
      this.editUserForm.id = user.id;
      this.editUserForm.name = user.username;
      this.editUserForm.password = "";
      this.editUserForm.role = user.role;
      this.editUserDialog = true;
    },
    closeEditUserDialog() {
      this.editUserDialog = false;
    },
    saveUser(user) {
      this.saveUserForm.id = user.id;
      this.saveUserForm.name = user.username;
      this.saveUserForm.password = "";
      this.saveUserForm.role = user.role;
      this.savetUserDialog = true;
    },
    closeSaveUserDialog() {
      this.saveUserDialog = false;
    },
    saveUser() {
      console.log(
        this.editUserForm.name,
        this.editUserForm.password,
        this.editUserForm.role
      );
      axios
        .post(`https://stockapp-server-eight.vercel.app/users`, {
          username: this.saveUserForm.name,
          password: this.saveUserForm.password,
          role: this.saveUserForm.role,
        })
        .then((response) => {
          Swal.fire(
            "Utilisateur ajouté!",
            `Mot de passe de l'utilisateur : ${response.data.clearPassword}`,
            "success"
          );
          this.closeEditUserDialog();
          axios
            .get("https://stockapp-server-eight.vercel.app/users")
            .then((response) => {
              this.users = response.data;
            });
        })
        .catch((error) => {
          console.log(error);
        });
      this.closeSaveUserDialog();
    },
    saveEditedUser() {
      if (this.$refs.editUserForm.validate()) {
        console.log(this.editUserForm.role);
        axios
          .patch(
            `https://stockapp-server-eight.vercel.app/users-edit/${this.editUserForm.id}`,
            {
              username: this.editUserForm.name,
              role: this.editUserForm.role,
            }
          )
          .then((response) => {
            Swal.fire("Modifié!", "L'utilisateur a été modifié", "success");
            this.closeEditUserDialog();
            axios
              .get("https://stockapp-server-eight.vercel.app/users")
              .then((response) => {
                this.users = response.data;
              });
          })
          .catch((error) => {
            console.log(error);
          });
        this.closeEditUserDialog();
      }
    },
    resetUserPassword(user) {
      Swal.fire({
        title:
          "Etes-vous certain de vouloir réinitialiser le mot de passe de l'utilisateur?",
        text: "Vous ne pourrez pas revenir en arrière !",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        confirmButtonText: "Oui, réinitialisez-le !",
        cancelButtonText: "Annuler",
      }).then((result) => {
        if (result.isConfirmed) {
          let newRandomPassword = (Math.random() + 1).toString(36).substring(2);
          axios
            .patch(
              `https://stockapp-server-eight.vercel.app/users-reset-password/${user.id}`,
              {
                password: newRandomPassword,
              }
            )
            .then((response) => {
              this.users = response.data;
            })
            .catch((error) => {
              console.log(error);
            });
          Swal.fire(
            "Réinitialisé!",
            `Nouveau mot de passe : ${newRandomPassword}`,
            "success"
          );
        }
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
