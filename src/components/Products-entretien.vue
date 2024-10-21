<template>
  <Fragment
    ><v-btn class="mt-4 ml-4" color="primary" @click="logout">Logout</v-btn>
    <v-tabs v-model="tab" background-color="primary" align-tabs="center">
      <v-tab value="product">Entretien</v-tab>
      <!-- <v-tab value="commande">Commande</v-tab> -->
    </v-tabs>
    <v-window>
      <v-window-item value="product">
        <div v-if="products" class="d-flex justify-center">
          <v-col cols="12">
            <v-table>
              <template #default>
                <thead>
                  <tr>
                    <th class="text-center">Type</th>
                    <th class="text-center">Nom</th>
                    <th class="text-center">Quantité</th>
                    <th class="text-center">Conditionnement</th>
                    <th class="text-center">Référence</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="product in products" :key="product.id">
                    <td class="text-center">{{ product.type }}</td>
                    <td class="text-center">{{ product.name }}</td>
                    <td class="text-center">
                      <v-number-input
                        v-model="product.count"
                        :reverse="false"
                        controlVariant="solo-filled"
                        label=""
                        :hideInput="false"
                        :inset="false"
                        variant="outlined"
                        min="0"
                        max="100"
                        @update:model-value="change(product.id, product.count)"
                      ></v-number-input>
                    </td>
                    <td class="text-center">{{ product.conditionnement }}</td>
                    <td class="text-center">{{ product.reference }}</td>
                  </tr>
                </tbody>
              </template>
            </v-table>
            <br />
          </v-col>
        </div>
      </v-window-item>
    </v-window>
    <!-- <v-window v-model="tab">
      <v-window-item value="commande">
        <div v-if="products" class="d-flex justify-center">
          <v-col cols="12">
            <v-table>
              <template #default>
                <thead>
                  <tr>
                    <th class="text-center">Nom</th>
                    <th class="text-center">Quantité</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="product in products" :key="product.id">
                    <td class="text-center">{{ product.name }}</td>
                    <td class="text-center">
                      <v-number-input
                        v-model="product.commandeCount"
                        :reverse="false"
                        controlVariant="solo-filled"
                        label=""
                        :hideInput="false"
                        :inset="false"
                        variant="outlined"
                        min="0"
                        max="100"
                      ></v-number-input>
                    </td>
                  </tr>
                </tbody>
              </template>
            </v-table>
            <br />
            <v-btn
              class="text-center mt-4 ml-4"
              color="primary"
              @click="addCommande(products)"
              >Commander</v-btn
            >
          </v-col>
        </div>
      </v-window-item>
    </v-window> -->
  </Fragment>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      products: null,
      tab: "null",
    };
  },
  mounted() {
    axios
      .get("http://localhost:1337/products-entretien")
      .then((response) => {
        this.products = response.data;
        console.log(response.data);
      })
      .catch((error) => {
        console.log(error);
      });
  },
  methods: {
    remove(id) {
      axios
        .delete(`http://localhost:1337/products-entretien/${id}`)
        .then((response) => {
          const index = this.products.findIndex((product) => product.id === id);
          this.products.splice(index, 1);
        })
        .catch((error) => {
          console.log(error);
        });
    },
    change(id, count) {
      const user = localStorage.getItem("user");
      const parsedUser = JSON.parse(user);

      axios
        .patch(`http://localhost:1337/products-entretien/${id}`, {
          count: count,
          user: parsedUser,
        })
        .then((response) => {
          const index = this.products.findIndex((product) => product.id === id);
          this.products[index].count = count;
          console.log(response.data);
        })
        .catch((error) => {
          console.log(error);
        });
    },
    addCommande(products) {
      products.forEach((product) => {
        if (product.commandeCount !== 0) {
          return console.log(
            `Pour : ${product.name}, il faut en commander ${product.commandeCount} car il en reste ${product.count}`
          );
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
