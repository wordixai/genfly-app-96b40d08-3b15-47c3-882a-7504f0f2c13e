<template>
  <v-app>
    <v-main>
      <v-container fluid class="pa-4">
        <v-row justify="center">
          <v-col cols="12" md="8" lg="6">
            <ShopProfileCard />
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import ShopProfileCard from './components/ShopProfileCard.vue'
</script>