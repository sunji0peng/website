<template>
  <div>
    <v-row>
      <v-col cols="5" md="12">
        <v-tooltip top v-bind:disabled="!avatarDescription">
          <template v-slot:activator="{ on }">
            <v-card-text
              v-on="on"
              style="cursor: pointer"
              @click.stop="iconsDialogOpened = true"
              class="player-avatar text-center"
              :style="{'background-image': 'url(' + picture(avatarCategory, avatarIcon) + ')'}"
            />
          </template>
          <span>{{ avatarDescription }}</span>
        </v-tooltip>

        <v-tooltip bottom>
          <template v-slot:activator="{ on }">
            <div
              v-on="on"
              class="country__container clickable"
              @click="goToCountryRankings()"
            >
              <country-flag
                v-if="selectedCountryCode != ''"
                class="player-country"
                :country="selectedCountryCode"
                size="normal"
              />
            </div>
          </template>
          <span>{{ selectedCountry }}</span>
        </v-tooltip>
      </v-col>
    </v-row>
    <player-socials :userProfile="userProfile" />

    <v-dialog v-model="iconsDialogOpened" max-width="1150px" class="scroll-v-dialog">
      <v-card>
        <v-checkbox
          style="margin-left: 25px"
          v-model="useClassicIcons"
          :label="$t('components_player_playeravatar.useClassicIcons')"
        ></v-checkbox>

        <!-- Starter Icons -->
        <v-row
          class="pb-3"
          align="center"
          justify="center">
          <v-card-text class="avatar-choose-headers pa-0 ma-0" align="center">Starter</v-card-text>
          <v-col
            cols="auto"
            v-for="number in starterPicNumbers"
            :key="number"
          >
          <v-tooltip top>
              <template v-slot:activator="{ on }">
                <v-card-text
                  v-on="on"
                  class="player-avatar-choosing"
                  @click="isLoggedInPlayer ? savePicture(EAvatarCategory.STARTER, number) : null"
                  :style="{'background-image': 'url(' + picture(EAvatarCategory.STARTER, number) + ')'}"
                />
              </template>
              <span>Starter</span>
            </v-tooltip>
          </v-col>
        </v-row>

        <!-- Race Icons Grid -->
        <v-row
          class="mt-0 mb-2"
          style="padding-left: 25px; padding-right: 25px"
          v-for="race in races"
          :key="race"
          align="center"
          justify="center"
        >
          <v-card-text
            class="avatar-choose-headers pa-0 ma-0" align="center">
            {{ enumToString(race) }}
          </v-card-text>
          <v-col cols="auto" v-for="number in racePicNumbers" :key="number">
            <v-tooltip top>
              <template v-slot:activator="{ on }">
                <v-card-text
                  v-on="on"
                  :class="getCorrectClasses(raceToAvatar(race), number)"
                  @click="isLoggedInPlayer ? savePicture(raceToAvatar(race), number) : null"
                  :style="{'background-image': 'url(' + picture(raceToAvatar(race), number) + ')'}"
                />
              </template>
              <span>{{ winsOf(winsOfRace(race), number, race) }}</span>
            </v-tooltip>
          </v-col>
        </v-row>

        <!-- Special Icons -->
        <v-row v-if="specialPictures.length > 0" class="pb-3" align="center" justify="center">
          <v-card-text class="avatar-choose-headers pa-0 ma-0" align="center">Specials</v-card-text>
          <v-col cols="auto" v-for="specialPicture in specialPictures" :key="specialPicture.pictureId">
            <v-tooltip top>
              <template v-slot:activator="{ on }">
                <v-card-text
                  v-on="on"
                  class="player-avatar-choosing"
                  v-bind:class="{ pointer: isLoggedInPlayer }"
                  @click="
                    isLoggedInPlayer
                      ? savePicture(EAvatarCategory.SPECIAL, specialPicture.pictureId, specialPicture.description)
                      : null
                  "
                  :style="{'background-image': 'url(' + picture(EAvatarCategory.SPECIAL, specialPicture.pictureId) + ')'}"
                />
              </template>
              <span>{{ specialPicture.description }}</span>
            </v-tooltip>
          </v-col>
        </v-row>
      </v-card>
    </v-dialog>

    <v-row>
      <v-col style="margin-top: -5px">
        <h3>
          {{ $t("components_player_playeravatar.games") }}
          {{ playerGames }}
        </h3>
      </v-col>
    </v-row>

    <v-row>
      <v-col style="margin-top: -15px">
        <h3>
          {{ $t("components_player_playeravatar.wins") }}
          {{ totalWins() }}
        </h3>
      </v-col>
    </v-row>

    <v-row>
      <v-col>
        <h3>{{ $t("components_player_playeravatar.homepage") }}</h3>
        <div v-if="homePageLinks && homePageLinks.length > 0">
          <a
            class="d-block"
            v-for="homePageLink in homePageLinks"
            rel="noopener noreferrer nofollow"
            target="_blank"
            :href="homePageLink"
            :key="homePageLink"
          >
            {{ homePage }}
          </a>
        </div>
        <div v-else>{{ homePage }}</div>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <h3>{{ $t("components_player_playeravatar.about") }}</h3>
        <div>{{ savedMessageValue ? savedMessageValue : "-" }}</div>
      </v-col>
    </v-row>
    <template>
      <v-row v-if="isLoggedInPlayer">
        <v-col>
          <v-dialog v-model="personalSettingsDialogOpened" persistent max-width="600px">
            <template v-slot:activator="{ on }">
              <v-btn
                @click="personalSettingsDialogOpened = true"
                small
                class="ma-0"
                outlined
                v-on="on"
                color="primary"
              >
                <v-icon left>{{ mdiPencil }}</v-icon>
                {{ $t("components_player_playeravatar.edit") }}
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="text-h5">
                  {{ $t("components_player_playeravatar.userprofile") }}
                </span>
              </v-card-title>
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-text-field
                      :prepend-icon="mdiTwitch"
                      color="purple accent-4"
                      dense
                      clearable
                      single-line
                      shaped
                      prefix="https://twitch.tv/"
                      :hint="$t('components_player_playeravatar.entertwitchname')"
                      v-model="userProfile.twitch"
                    ></v-text-field>
                    <v-text-field
                      :prepend-icon="mdiYoutube"
                      color="red darken-2"
                      dense
                      clearable
                      single-line
                      shaped
                      :hint="$t('components_player_playeravatar.enterytname')"
                      prefix="https://www.youtube.com/"
                      v-model="userProfile.youtube"
                    ></v-text-field>
                    <v-text-field
                      :prepend-icon="mdiTwitter"
                      color="blue darken-2"
                      dense
                      clearable
                      single-line
                      shaped
                      :hint="$t('components_player_playeravatar.entertwittername')"
                      prefix="https://www.twitter.com/"
                      v-model="userProfile.twitter"
                    ></v-text-field>
                    <v-text-field
                      prepend-icon="$trovo"
                      color="green darken-3"
                      dense
                      clearable
                      single-line
                      shaped
                      :hint="$t('components_player_playeravatar.entertrovoname')"
                      prefix="https://trovo.live/"
                      v-model="userProfile.trovo"
                    ></v-text-field>
                    <v-text-field
                      :prepend-icon="mdiHome"
                      color="blue darken-2"
                      dense
                      :rules="[rules.maxLength(50)]"
                      single-line
                      clearable
                      v-model="userProfile.homePage"
                      shaped
                      :hint="$t('components_player_playeravatar.entercustomhp')"
                      label="Homepage"
                    ></v-text-field>
                    <v-container>
                      <v-checkbox
                        dense
                        class="alias-checkbox"
                        :prepend-icon="mdiAccountCheck"
                        v-model="userProfile.aliasSettings.showAka"
                        :label="$t('components_player_playeravatar.showalias')"
                      ></v-checkbox>
                      <!-- THIS FEATURE IS WAITING ON BETTER ICONS - DO NOT USE UNTIL NEW ICONS -->
                      <!-- <v-checkbox dense class="alias-checkbox"
                        prepend-icon="$w3info"
                        v-model="userProfile.aliasSettings.showW3info"
                        :label="`Show Warcraft3.info Profile Link`"
                      ></v-checkbox>
                      <v-checkbox dense class="alias-checkbox"
                        prepend-icon="$liquipedia"
                        v-model="userProfile.aliasSettings.showLiquipedia"
                        :label="`Show Liquipedia Page Link`"
                      ></v-checkbox> -->
                    </v-container>
                  </v-row>
                  <v-row no-gutters class="countryInput">
                    <v-col md="12">
                      <v-autocomplete
                        :prepend-icon="mdiFlag"
                        clearable
                        :item-value="countryCode"
                        :items="countries"
                        :filter="countryFilter"
                        :label="$t('components_player_playeravatar.selectcountry')"
                        item-text="country"
                        v-model="selectedCountry"
                        :return-object="false"
                      >
                        <template v-slot:item="{ item }">
                          <country-flag :country="item.countryCode" size="normal" />
                          {{ item.country }}
                          <v-spacer></v-spacer>
                        </template>
                        <template v-slot:selection="{ item }">
                          <country-flag :country="item.countryCode" size="normal" />
                          <span class="pr-2">{{ item.country }}</span>
                        </template>
                      </v-autocomplete>
                    </v-col>
                  </v-row>
                  <v-row>
                    <v-col>
                      <v-textarea
                        outlined
                        name="input-7-1"
                        label="About"
                        clearable
                        :rules="[rules.maxLength(300)]"
                        v-model="userProfile.about"
                        :hint="$t('components_player_playeravatar.aboutdesc')"
                      ></v-textarea>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="resetUserProfile">
                  {{ $t("components_player_playeravatar.close") }}
                </v-btn>
                <v-btn color="blue darken-1" text @click="saveUserProfile">
                  {{ $t("components_player_playeravatar.save") }}
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-col>
      </v-row>
    </template>
  </div>
</template>

<script lang="ts">
import { computed, defineComponent, onMounted, ref, watch } from "vue";
import { useI18n } from "vue-i18n-bridge";
import { ERaceEnum, EAvatarCategory } from "@/store/types";
import { ECountries } from "@/store/countries";
import { AkaSettings, PersonalSetting, ProfilePlayerSocials, SpecialPicture } from "@/store/personalSettings/types";
import PlayerSocials from "./PlayerSocials.vue";
import { getAvatarUrl } from "@/helpers/url-functions";
import { enumKeys } from "@/helpers/general";
import { usePersonalSettingsStore } from "@/store/personalSettings/store";
import { usePlayerStore } from "@/store/player/store";
import { mdiAccountCheck, mdiFlag, mdiHome, mdiPencil, mdiTwitch, mdiTwitter, mdiYoutube } from "@mdi/js";
import { useRouter } from "vue-router/composables";
import { CountryType } from "@/store/ranking/types";

// Lazy load.
const CountryFlag = () => import(/* webpackChunkName: "country-flag" */ "vue-country-flag");

export default defineComponent({
  name: "PlayerAvatar",
  components: {
    CountryFlag,
    PlayerSocials,
  },
  props: {
    isLoggedInPlayer: {
      type: Boolean,
      required: true,
    },
  },
  setup(props) {
    const { t } = useI18n();
    const router = useRouter();
    const personalSettingsStore = usePersonalSettingsStore();
    const playerStore = usePlayerStore();
    const races = [ERaceEnum.HUMAN, ERaceEnum.ORC, ERaceEnum.NIGHT_ELF, ERaceEnum.UNDEAD, ERaceEnum.RANDOM, ERaceEnum.TOTAL];
    const countries = ref<CountryType[]>([]);
    const iconsDialogOpened = ref<boolean>(false);
    const personalSettingsDialogOpened = ref<boolean>(false);
    const useClassicIcons = ref<boolean>(false);

    const enumToString = (race: ERaceEnum) => t(`races.${ERaceEnum[race]}`);

    const personalSetting = computed<PersonalSetting>(() => personalSettingsStore.personalSettings);

    const starterPicNumbers = Array.from({ length: 5 }, (_, i) => i + 1);
    const racePicNumbers = Array.from({ length: 18 }, (_, i) => i + 1);

    const playerGames = computed<number>(() => {
      return personalSetting.value.winLosses?.reduce((sum, stat) => {
        return sum + stat.games;
      }, 0);
    });

    const homePage = computed<string>(() => personalSetting.value.homePage || "-");
    const countryCode = computed<string>(() => personalSetting.value.countryCode || "");
    const twitch = computed<string>(() => personalSetting.value.twitch || "");
    const youtube = computed<string>(() => personalSetting.value.youTube || "");
    const twitter = computed<string>(() => personalSetting.value.twitter || "");
    const trovo = computed<string>(() => personalSetting.value.trovo || "");
    const douyu = computed<string>(() => personalSetting.value.douyu || "");

    const aliasSettings = computed<AkaSettings>(() => {
      return (
        personalSetting.value.aliasSettings || {
          showAka: true,
          showW3info: true,
          showLiquipedia: true,
        } satisfies AkaSettings
      );
    });

    const homePageLinks = computed<string[]>(() => {
      if (!homePage.value || !homePage.value.includes("http")) {
        return [];
      }

      return homePage.value
        .split(" ")
        .filter((url) => !!url)
        .map((url) => url.trim());
    });

    const specialPictures = computed<SpecialPicture[]>(() => personalSetting.value.specialPictures || []);
    const savedMessageValue = computed<string>(() => personalSetting.value.profileMessage);
    const avatarIcon = computed<number>(() => personalSetting.value?.profilePicture?.pictureId ?? 0);
    const avatarCategory = computed<EAvatarCategory>(() => personalSetting.value?.profilePicture?.race ?? EAvatarCategory.TOTAL);

    const rules = {
      maxLength: (len: number) => (v: string) => (v || "").length < len || `Can not exceed ${len} characters`,
    };

    const selectedCountry = ref<string>("");
    const selectedCountryCode = ref<string>("");

    const userProfile = computed<ProfilePlayerSocials>(() => {
      return {
        twitch: twitch.value,
        youtube: youtube.value,
        twitter: twitter.value,
        trovo: trovo.value,
        douyu: douyu.value,
        aliasSettings: aliasSettings.value,
        about: savedMessageValue.value,
        homePage: homePage.value,
      } satisfies ProfilePlayerSocials;
    });

    function countryFilter(item: CountryType, queryText: string): boolean {
      const textOne = item.country.toLowerCase();
      const searchText = queryText.toLowerCase();
      return textOne.includes(searchText);
    }

    async function resetUserProfile(): Promise<void> {
      // userProfile.value = {
      //   twitch: twitch.value,
      //   youtube: youtube.value,
      //   twitter: twitter.value,
      //   trovo: trovo.value,
      //   douyu: douyu.value,
      //   aliasSettings: aliasSettings.value,
      //   about: savedMessageValue.value,
      //   homePage: homePage.value,
      // };
      personalSettingsDialogOpened.value = false;
    }

    async function saveUserProfile(): Promise<void> {
      const personalSettings = personalSetting.value;
      personalSettings.twitch = userProfile.value.twitch;
      personalSettings.youTube = userProfile.value.youtube;
      personalSettings.twitter = userProfile.value.twitter;
      personalSettings.trovo = userProfile.value.trovo;
      personalSettings.douyu = userProfile.value.douyu;
      personalSettings.aliasSettings = userProfile.value.aliasSettings;
      personalSettings.profileMessage = userProfile.value.about;
      personalSettings.homePage = userProfile.value.homePage;

      countries.value.map((c) => {
        if (c.country == selectedCountry.value) {
          selectedCountry.value = c.country;
          selectedCountryCode.value = c.countryCode;
        }
      });

      if (!selectedCountry.value) {
        selectedCountryCode.value = "";
      }

      personalSettings.country = selectedCountry.value || "";
      personalSettings.countryCode = selectedCountryCode.value || "";

      await personalSettingsStore.saveUserProfile(personalSettings);
      personalSettingsDialogOpened.value = false;
    }

    function raceToAvatar(race: ERaceEnum): EAvatarCategory {
      return {
        [ERaceEnum.RANDOM]: EAvatarCategory.RANDOM,
        [ERaceEnum.HUMAN]: EAvatarCategory.HUMAN,
        [ERaceEnum.ORC]: EAvatarCategory.ORC,
        [ERaceEnum.NIGHT_ELF]: EAvatarCategory.NIGHTELF,
        [ERaceEnum.UNDEAD]: EAvatarCategory.UNDEAD,
        [ERaceEnum.TOTAL]: EAvatarCategory.TOTAL,
        [ERaceEnum.STARTER]: EAvatarCategory.STARTER,
      }[race];
    }

    function getCorrectClasses(category: EAvatarCategory, iconId: number): string[] {
      const classes = ["player-avatar-choosing"];
      if (props.isLoggedInPlayer && enabledIfEnoughWins(category, iconId)) classes.push("pointer");

      if (!enabledIfEnoughWins(category, iconId)) classes.push("player-avatar-choosing-disabled");

      return classes;
    }

    const avatarDescription = computed<string>(() => {
      if (avatarCategory.value != EAvatarCategory.SPECIAL) return "";
      const specialPicture = specialPictures.value.find((x) => x.pictureId == avatarIcon.value);
      return specialPicture?.description ?? "";
    });

    function enabledIfEnoughWins(category: EAvatarCategory, iconId: number): boolean {
      if (category == EAvatarCategory.SPECIAL || category == EAvatarCategory.STARTER) return true;

      if (category == EAvatarCategory.TOTAL) {
        return personalSetting.value.pickablePictures?.filter((r) => r.avatarType == ERaceEnum.TOTAL)[0].max >= iconId;
      }

      const raceCategory = category as unknown as ERaceEnum;

      return personalSetting.value.pickablePictures?.filter((r) => r.avatarType == raceCategory)[0].max >= iconId;
    }

    function winsOfRace(race: ERaceEnum): number {
      if (race == ERaceEnum.TOTAL) return totalWins();
      return personalSetting.value.winLosses?.filter((w) => w.race == race)[0]?.wins ?? 0;
    }

    const winsOf = (wins: number, iconId: number, race: ERaceEnum): string => `${wins}/${winsTransformed(iconId, race)}`;

    function totalWins(): number {
      return personalSetting.value.winLosses?.map((x) => x.wins).reduce((partial, y) => partial + y, 0) ?? 0;
    }

    function winsTransformed(iconId: number, race: ERaceEnum): number {
      if (race == ERaceEnum.TOTAL) {
        return personalSetting.value.totalPictureRange?.filter((p) => p.pictureId == iconId)[0]?.neededWins;
      }
      return personalSetting.value.racePictureRange?.filter((p) => p.pictureId == iconId)[0]?.neededWins;
    }

    const picture = (category: EAvatarCategory, picId: number) => getAvatarUrl(category, picId, useClassicIcons.value);

    async function savePicture(avatarCategory: EAvatarCategory, picture: number, description?: string): Promise<void> {
      if (!enabledIfEnoughWins(avatarCategory, picture)) return;
      await personalSettingsStore.saveAvatar({
        race: avatarCategory,
        pictureId: picture,
        isClassic: useClassicIcons.value,
        description,
      });

      iconsDialogOpened.value = false;
    }

    function goToCountryRankings(): void {
      router.push(`/Countries?country=${selectedCountryCode.value}`);
    }

    onMounted((): void => {
      init();
    });

    // Watch for battleTag changes and reload personal settings
    watch(
      () => playerStore.battleTag,
      (newBattleTag, oldBattleTag) => {
        if (newBattleTag && newBattleTag !== oldBattleTag) {
          init();
        }
      }
    );

    async function init() {
      await personalSettingsStore.loadPersonalSetting();

      useClassicIcons.value = personalSetting.value?.profilePicture?.isClassic ?? false;

      // populate countries dropdown for combobox
      enumKeys(ECountries).map((key) => {
        const country = {
          country: key,
          countryCode: ECountries[key],
        };

        if (countryCode.value && countryCode.value == country.countryCode) {
          selectedCountry.value = country.country;
          selectedCountryCode.value = country.countryCode;
        }

        countries.value.push(country);
      });

      if (!selectedCountryCode.value && personalSetting.value?.location) {
        selectedCountryCode.value = personalSetting.value.location;

        enumKeys(ECountries).map((key) => {
          const element = ECountries[key] as string;
          if (element == selectedCountryCode.value) {
            selectedCountry.value = key;
          }
        });
      }
    }

    return {
      mdiAccountCheck,
      mdiFlag,
      mdiHome,
      mdiPencil,
      mdiTwitch,
      mdiTwitter,
      mdiYoutube,
      races,
      avatarDescription,
      iconsDialogOpened,
      personalSettingsDialogOpened,
      picture,
      avatarCategory,
      avatarIcon,
      goToCountryRankings,
      selectedCountryCode,
      selectedCountry,
      userProfile,
      useClassicIcons,
      starterPicNumbers,
      savePicture,
      EAvatarCategory,
      enumToString,
      racePicNumbers,
      getCorrectClasses,
      winsOf,
      winsOfRace,
      specialPictures,
      playerGames,
      totalWins,
      homePageLinks,
      homePage,
      savedMessageValue,
      countryCode,
      countries,
      countryFilter,
      rules,
      resetUserProfile,
      saveUserProfile,
      raceToAvatar,
    };
  },
});
</script>

<style lang="scss" scoped>
.countryInput {
  margin-left: -11px;
}

.player-country {
  position: absolute;
  border-color: white;
  border-style: solid;
  border-width: thin;
  bottom: 10px;
  right: 10px;
}

.country__container {
  position: relative;
  max-width: 120px;
}

.socialIcon {
  padding-top: 0px;
  padding-left: 2px;
}

.twitchIcon {
  margin-top: 2px;
}

@media (min-width: 960px) {
  .player-avatar {
    height: 185px !important;
  }

  .country__container {
    max-width: 185px !important;
  }
}

.player-avatar {
  max-width: 185px;
  height: 120px;
  background-repeat: no-repeat;
  background-size: contain;
}

.player-avatar-choosing {
  padding-top: 100%;
  width: 150%;
  background-repeat: no-repeat;
  background-size: contain;
}

.player-avatar-choosing-disabled {
  opacity: 0.5;
  filter: alpha(opacity=50);
}

.player-league {
  width: 182px;

  .player-league-rank {
    font-size: 20px;
  }

  .player-league-points {
    font-size: 13px;
  }
}

.special-icons {
  .col {
    margin-left: 10px;
  }

  .col:first-child {
    margin-left: 0;
  }
}

.alias-checkbox {
  margin-top: 0px;
  padding-top: 0px;
}

.avatar-choose-headers {
  padding-left: 25px;
  padding-right: 25px;
}

.clickable {
  cursor: pointer;
}
</style>
