<template>
  <v-container fluid v-show="isAdmin" style="height: 100%">
    <admin-check-jwt-lifetime />
    <div class="admin-page-wrapper">
      <admin-navigation></admin-navigation>
      <v-card tile>
        <router-view></router-view>
      </v-card>
    </div>
  </v-container>
</template>

<script lang="ts">
import Vue from "vue";
import { Component } from "vue-property-decorator";
import AdminNavigation from "@/components/admin/AdminNavigation.vue";
import AdminCheckJwtLifetime from "@/components/admin/AdminCheckJwtLifetime.vue";
import { useOauthStore } from "@/store/oauth/store";

@Component({
  components: {
    AdminNavigation,
    AdminCheckJwtLifetime,
  },
})
export default class Admin extends Vue {
  private oauthStore = useOauthStore();

  get isAdmin(): boolean {
    return this.oauthStore.isAdmin;
  }
}
</script>

<style lang="scss" scoped>
.admin-page-wrapper {
  height: 100%;
  display: grid;
  grid-template-columns: 256px auto;
}
</style>
