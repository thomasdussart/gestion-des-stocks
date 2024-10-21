<template>
  <v-container class="fill-height" fluid>
    <v-row align="center" justify="center">
      <v-col cols="12" sm="8" md="4">
        <v-card class="elevation-12">
          <v-toolbar color="primary" dark flat>
            <v-toolbar-title>Connexion</v-toolbar-title>
          </v-toolbar>
          <v-card-text @keyup.enter="login()">
            <v-form>
              <v-text-field
                label="Nom d'utilisateur"
                name="login"
                v-model="username"
                prepend-icon="mdi-account"
                type="text"
              ></v-text-field>

              <v-text-field
                id="password"
                label="Mot de passe"
                name="password"
                v-model="password"
                prepend-icon="mdi-lock"
                type="password"
              ></v-text-field>
            </v-form>
          </v-card-text>
          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" @click="login">Connexion</v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
import swal from "sweetalert2";

export default {
  data: () => ({
    username: "",
  }),
  methods: {
    login() {
      axios
        .post("https://stockapp-server-eight.vercel.app/login", {
          username: this.username,
          password: this.password,
        })
        .then((response) => {
          const user = {
            id: response.data.user[0].id,
            username: response.data.user[0].username,
            role: response.data.user[0].role,
          };
          localStorage.setItem("user", JSON.stringify(user));
          localStorage.setItem("role", response.data.user[0].role);
          window.location.reload();
        })
        .catch((error) => {
          console.log(error);
          swal.fire({
            toast: true,
            timer: 3000,
            timerProgressBar: true,
            position: "top-end",
            icon: "error",
            text: "Nom d'utilisateur ou mot de passe incorrect",
          });
        });
    },
  },
};
</script>
