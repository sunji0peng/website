<template>
  <div>
    <v-card-title>
      Global Mute
    </v-card-title>
    <v-container>
      <v-data-table
        :headers="headers"
        :items-per-page="-1"
        :items="globallyMutedPlayers"
        hide-default-footer
        sort-by="id"
        :sort-desc="true"
      >
        <template v-slot:top>
          <v-toolbar flat color="transparent">
          <template>
            <v-text-field
              v-model="searchQuery"
              label="Search mute"
              :prepend-icon="mdiMagnify"
              @keydown.enter="loadMutes"
            ></v-text-field>
          </template>
            <v-spacer></v-spacer>
            <v-dialog v-model="dialog" max-width="500px">
              <template v-slot:activator="{ on, attrs }">
                <v-btn
                  color="primary"
                  class="mb-2 w3-race-bg--text"
                  v-bind="attrs"
                  v-on="on"
                >
                  {{ $t(`views_admin.mutePlayer`) }}
                </v-btn>
              </template>

              <v-card>
                <v-card-title>
                  {{ $t(`views_admin.mutePlayer`) }}
                </v-card-title>

                <v-card-text>
                  <v-container>
                    <v-row>
                      <player-search
                        @searchCleared="searchCleared"
                        @playerFound="playerFound"
                        classes="ml-5 mr-5"
                      ></player-search>
                    </v-row>
                    <v-row v-if="showConfirmation" class="ma-2">
                      <v-menu
                        v-model="dateMenu"
                        :close-on-content-click="false"
                        min-width="290px"
                      >
                        <template #activator="{ on, attrs }">
                          <v-text-field
                            v-model="banExpiry"
                            readonly
                            :label="$t(`views_admin.banenddate`)"
                            v-bind="attrs"
                            v-on="on"
                          />
                        </template>
                        <v-date-picker v-model="banExpiry" no-title scrollable max="2099-01-01">
                          <v-spacer />
                          <v-btn
                            text
                            @click="
                              banExpiry = '';
                              dateMenu = false;
                            "
                          >
                            {{ $t(`views_admin.cancel`) }}
                          </v-btn>
                          <v-btn
                            color="primary"
                            class="w3-race-bg--text"
                            @click="dateMenu = false"
                          >
                            {{ $t(`views_admin.ok`) }}
                          </v-btn>
                        </v-date-picker>
                      </v-menu>

                      <v-card-text>
                        Are you sure you want to mute this player?
                      </v-card-text>
                      <v-card-title>
                        {{ player }}
                        <v-spacer></v-spacer>
                      </v-card-title>
                    </v-row>

                    <v-row>
                      <v-col>
                        <v-btn
                          v-if="showConfirmation && banDateSet"
                          color="primary"
                          class="w3-race-bg--text"
                          @click="save"
                        >
                          {{ $t(`views_admin.ok`) }}
                        </v-btn>
                      </v-col>
                      <v-col class="text-right">
                        <v-btn
                          color="primary"
                          class="w3-race-bg--text"
                          @click="close"
                        >
                          {{ $t(`views_admin.cancel`) }}
                        </v-btn>
                      </v-col>
                    </v-row>
                  </v-container>
                </v-card-text>
              </v-card>
            </v-dialog>
          </v-toolbar>
        </template>

        <template #[`item.actions`]="{ item }">
          <v-icon small @click="deleteItem(item)">{{ mdiDelete }}</v-icon>
        </template>
      </v-data-table>
      <v-row class="ma-2">
        <v-spacer />
        <v-btn color="primary" class="w3-race-bg--text" v-if="!searchQuery" @click="loadMutes">
          Next
        </v-btn>
        <v-spacer />
      </v-row>
    </v-container>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, onMounted, ref, watch } from "vue";
import { GloballyMutedPlayer, GlobalMute } from "@/store/admin/types";
import PlayerSearch from "@/components/common/PlayerSearch.vue";
import { useAdminStore } from "@/store/admin/store";
import { useOauthStore } from "@/store/oauth/store";
import { usePlayerSearchStore } from "@/store/playerSearch/store";
import { mdiMagnify, mdiDelete } from "@mdi/js";

export default defineComponent({
  name: "AdminGlobalMute",
  components: {
    PlayerSearch,
  },
  setup() {
    const adminStore = useAdminStore();
    const oauthStore = useOauthStore();
    const playerSearchStore = usePlayerSearchStore();

    const banExpiry = ref<string>("");
    const dateMenu = ref<boolean>(false);
    const dialog = ref<boolean>(false);
    const showConfirmation = ref<boolean>(false);
    const player = ref<string>("");
    const searchQuery = ref<string | undefined>(undefined);
    const mutesNextId = computed<number | null>(() => adminStore.mutesNextId);

    const globallyMutedPlayers = computed<GloballyMutedPlayer[]>(() => adminStore.globallyMutedPlayers);
    const banDateSet = computed<boolean>(() => banExpiry.value != "");
    const author = computed<string>(() => oauthStore.blizzardVerifiedBtag);
    const isAdmin = computed<boolean>(() => oauthStore.isAdmin);

    async function deleteItem(item: GloballyMutedPlayer): Promise<void> {
      confirm("Are you sure you want to delete this item?") &&
      await adminStore.deleteGlobalMute(item);
      loadMutes();
    }

    function close(): void {
      dialog.value = false;
      showConfirmation.value = false;
    }

    async function save(): Promise<void> {
      close();
      const mute = { battleTag: player.value, expiresAt: banExpiry.value, author: author.value } as GlobalMute;
      await adminStore.addGlobalMute(mute);
      loadMutes();
    }

    async function loadMutes(): Promise<void> {
      if (isAdmin.value) {
        await adminStore.loadGlobalMutes(searchQuery.value, mutesNextId.value);
      }
    }

    watch(isAdmin, init);

    async function init(): Promise<void> {
      if (!isAdmin.value) return;
      await loadMutes();
    }

    onMounted(async (): Promise<void> => {
      await init();
    });

    function searchCleared(): void {
      showConfirmation.value = false;
      banExpiry.value = "";
    }

    function playerFound(bTag: string): void {
      showConfirmation.value = true;
      player.value = bTag;
    }

    watch(dialog, onDialogToggled);
    function onDialogToggled(): void {
      // Only trigger on dialog close, not dialog open
      if (!dialog.value) {
        searchCleared();
        playerSearchStore.clearPlayerSearch();
      }
    }

    const headers = [
      {
        text: "Flo Ban Id",
        sortable: true,
        value: "id",
      },
      {
        text: "BattleTag",
        sortable: true,
        value: "battleTag",
      },
      {
        text: "Ban Insert Date",
        sortable: true,
        value: "createdAt",
      },
      {
        text: "Ban Expiry Date",
        sortable: true,
        value: "expiresAt",
      },
      {
        text: "Author",
        sortable: true,
        value: "author",
      },
      {
        text: "Actions",
        sortable: false,
        value: "actions",
      },
    ];

    return {
      mdiMagnify,
      mdiDelete,
      headers,
      globallyMutedPlayers,
      searchQuery,
      dialog,
      searchCleared,
      playerFound,
      showConfirmation,
      dateMenu,
      banExpiry,
      player,
      banDateSet,
      save,
      close,
      deleteItem,
      loadMutes,
    };
  },
});
</script>
