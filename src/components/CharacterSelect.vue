<template>
  <div class="main-container">
    <div class="character-frames">
      <CharacterFrame
        v-for="(character, index) in this.characterList"
        :key="index"
        :characterName="character"
        :authToken="this.authToken"
        @click="selectInventory(character, index)"
        :isActive="this.selectedIndex == index"
      >
      </CharacterFrame>
    </div>
    <div class="sub-frame">
      <Inventory
        :characterName="this.selectedCharacter"
        :authToken="this.authToken"
      >
      </Inventory>
    </div>
  </div>
</template>

<style scoped>
.main-container {
  display: flex;
  min-width: 232px;
  background: var(--color-background);
}
</style>

<script>
import axios from "axios";
import Inventory from "@/components/Inventory.vue";
import CharacterFrame from "@/components/CharacterFrame.vue";
export default {
  components: { Inventory, CharacterFrame },
  props: {
    authToken: String,
  },
  data() {
    return {
      characterList: null,
      selectedCharacter: null,
      selectedIndex: 0,
      errorCode: 0,
    };
  },
  async created() {
    var resp = await axios
      .get(
        "https://api.guildwars2.com/v2/characters?access_token=" +
          this.authToken
      )
      .then(function (response) {
        return response;
      })
      .catch(function (error) {
        return error.response;
      });
    this.errorCode = resp.status;
    if (this.errorCode == 200 || this.errorCode == 0) {
      this.characterList = resp.data;
      this.selectedCharacter = resp.data[0];
    }
    console.log(this.errorCode);
  },
  methods: {
    selectInventory(c, i) {
      this.selectedCharacter = c;
      this.selectedIndex = i;
    },
  },
};
</script>
