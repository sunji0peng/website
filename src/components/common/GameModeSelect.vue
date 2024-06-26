<template>
  <v-menu offset-x>
    <template v-slot:activator="{ on }">
      <v-btn tile v-on="on" class="transparent">
        <v-icon style="margin-right: 5px">{{ mdiControllerClassic }}</v-icon>
        {{ gameModeName() }}
      </v-btn>
    </template>
    <v-card>
      <v-card-text>
        <v-list>
          <v-list-item-content>
            <v-list-item-title>
              {{ $t("components_common_gamemodeselect.selectgamemode") }}
            </v-list-item-title>
          </v-list-item-content>
        </v-list>
        <v-divider></v-divider>
        <v-list dense>
          <v-list-item
            v-for="mode in gameModes()"
            :key="mode.id"
            @click="selectGameMode(mode.id)"
          >
            <v-list-item-content>
              <v-list-item-title>{{ mode.name }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-card-text>
    </v-card>
  </v-menu>
</template>

<script lang="ts">
import { defineComponent, PropType } from "vue";
import { activeGameModesWithAT, loadActiveGameModes } from "@/mixins/GameModesMixin";
import { LocaleMessage } from "vue-i18n";
import { EGameMode } from "@/store/types";
import { mdiControllerClassic } from "@mdi/js";

export default defineComponent({
  name: "GameModeSelect",
  components: {},
  props: {
    gameMode: {
      type: Number as PropType<EGameMode>,
      required: false,
    },
    disabledModes: {
      type: Array<EGameMode> as PropType<EGameMode[]>,
      required: false,
    }
  },
  setup: (props, context) => {
    function gameModes(): Array<{ name: LocaleMessage; id: number }> {
      let modes = activeGameModesWithAT();

      if (props.disabledModes) {
        modes = modes?.filter((x) => !props.disabledModes?.includes(x.id));
      }

      return modes;
    }

    function gameModeName(): LocaleMessage {
      if (!props.gameMode) {
        return "";
      }

      const mode = activeGameModesWithAT()?.filter((g) => g.id == props.gameMode)[0];

      if (!mode) {
        return "Not Supported";
      }

      return mode.name;
    }

    function selectGameMode(gameMode: EGameMode): void {
      context.emit("gameModeChanged", gameMode);
    }

    return {
      mdiControllerClassic,
      gameModes,
      gameModeName,
      selectGameMode,
    };
  },
  mounted: async (): Promise<void> => {
    await loadActiveGameModes();
  },
});
</script>

<style></style>
