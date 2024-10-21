<template>
  <div v-if="role === 'admin'">
    <products-admin />
  </div>
  <div v-else-if="role === 'economat'">
    <products-economat />
  </div>
  <div v-else-if="role === 'ac'">
    <products-entretien-ac />
  </div>
  <div v-else-if="role === 'lsr'">
    <products-entretien-lsr />
  </div>
  <div v-else-if="role === 'gh'">
    <products-entretien-gh />
  </div>
  <div v-else-if="role === 'avernas'">
    <products-entretien-avernas />
  </div>
  <div v-else-if="role === 'moxhe'">
    <products-entretien-moxhe />
  </div>
  <div v-else-if="role === 'thisnes'">
    <products-entretien-thisnes />
  </div>
  <div v-else-if="role === 'merdorp'">
    <products-entretien-merdorp />
  </div>
  <div v-else-if="role === 'depot'">
    <products-entretien-depot />
  </div>
  <div v-else-if="role === 'mds'">
    <products-entretien-mds />
  </div>
  <div v-else-if="role === 'bibliotheque'">
    <products-entretien-bibli />
  </div>
  <div v-else-if="role === 'ahdv'">
    <products-entretien-ahdv />
  </div>
  <div v-else-if="role === 'academie'">
    <products-entretien-academie />
  </div>
  <div v-else-if="role === 'saline'">
    <products-entretien-saline />
  </div>
  <div v-else-if="role === 'emploi'">
    <products-entretien-emploi />
  </div>
  <div v-else>
    <Login />
  </div>
</template>

<!-- LSR - 60qv2x4xsw -->

<script setup>
const token = localStorage.getItem("token");
const role = localStorage.getItem("role");
</script>
