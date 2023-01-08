<template>
  <div class="main-frame">
    <div v-if="!loading" class="inventory-frame">
      <InventoryBag
        v-for="(bag, index) in this.inv.bags"
        :key="index"
        :bagContents="bag.inventory"
      >
      </InventoryBag>
    </div>
    <div v-else class="inventory-frame">
      <InventoryBag
        v-for="(bag, index) in this.emptyInv.bags"
        :key="index"
        :bagContents="bag.inventory"
      >
      </InventoryBag>
      <div class="loading-overlay">
        <div class="loading-spinner"></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.inventory-frame {
  position: relative;
  display: inline-block;
}
.loading-overlay {
  background-color: var(--c-tooltip-background);
  border: 2px solid var(--c-tooltip-frame);
  top: 0;
  left: 0;
  width: 99%;
  height: 100%;
  position: absolute;
  z-index: 1;
}
.loading-spinner {
  height: 64px;
  width: 64px;
  margin: auto;
  margin-top: 25%;
  border: 8px solid white;
  border-bottom-color: transparent;
  border-radius: 50%;

  animation-name: spin;
  animation-duration: 1500ms;
  animation-iteration-count: infinite;
  animation-timing-function: linear;
}
@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(359deg);
  }
}
</style>

<script>
import axios from "axios";
import InventoryBag from "@/components/InventoryBag.vue";
import defaultInv from "@/assets/data/inventoryDefault.json";

export default {
  components: { InventoryBag },
  props: {
    authToken: String,
    characterName: String,
  },
  data() {
    return {
      inv: null,
      loading: true,
      emptyInv: defaultInv,
    };
  },
  created() {
    this.requestInventory();
  },
  watch: {
    characterName: function (newValue, oldValue) {
      this.requestInventory();
    },
  },
  methods: {
    async requestInventory() {
      this.loading = true;
      if (this.characterName == null) {
        return;
      }
      var resp = await axios
        .get(
          "https://api.guildwars2.com/v2/characters/" +
            this.characterName.replace(" ", "%20") +
            "/inventory?access_token=" +
            this.authToken
        )
        .then(function (response) {
          return response;
        })
        .catch((error) => console.log(error));
      this.inv = resp.data;
      this.loading = false;
    },
  },
};
</script>
