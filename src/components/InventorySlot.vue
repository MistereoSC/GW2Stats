<template>
  <div>
    <div v-if="isLoading" class="inventory-slot"></div>
    <div
      v-else-if="errorCode !== 200"
      class="inventory-slot inventory-slot-error slot-tooltip"
    >
      <span class="slot-tooltip-container">
        <span class="slot-tooltip-text-title">Unknown Item</span>
        <span class="slot-tooltip-text"
          >Cannot find Item {{ itemData.id }}</span
        >
      </span>
    </div>
    <div
      v-else-if="!isEmpty"
      class="inventory-slot slot-tooltip"
      :style="{
        background: 'url(\'' + this.item.icon + '\')',
        'background-size': 'contain',
        'border-image': this.rarityBorder,
      }"
    >
      <span class="slot-count">{{ item.count }}</span>
      <span class="slot-tooltip-container">
        <span
          class="slot-tooltip-text-title"
          :style="{ color: this.rarityColor }"
          >{{ item.count }} {{ item.name }}</span
        >
        <span class="slot-tooltip-text">{{ item.description }}</span>
      </span>
    </div>
    <div v-else class="inventory-slot"></div>
  </div>
</template>

<style scoped>
.inventory-slot {
  background-color: var(--color-itemframe-transparent);
  border: 2px solid var(--color-itemframe-transparent);
  box-shadow: inset 2px 2px 10px 5px #000;
  width: 60px;
  height: 60px;
  margin: 1px 3px 0px 0px;
}
.inventory-slot:hover {
  box-shadow: inset 2px 2px 10px 5px var(--color-itemframe);
}
.inventory-slot-error {
  background-color: var(--c-accent-darkred);
}
.slot-count {
  color: var(--c-text-item-title);
  font-weight: 600;
  position: absolute;
  right: 3%;
  top: -3%;
  text-shadow: 1px 1px 1px #000000;
}

/* #region Tooltip CSS */
.slot-tooltip {
  position: relative;
  display: inline-block;
}
.slot-tooltip-text-title {
  display: block;
  font-size: 14pt;
  margin-bottom: 5px;
  white-space: nowrap;
}
.slot-tooltip .slot-tooltip-container {
  visibility: hidden;
  min-width: 256px;
  background-color: var(--c-tooltip-background);
  border: 2px solid var(--c-tooltip-frame);

  color: var(--color-text);
  text-align: left;
  padding: 5px;

  position: absolute;
  top: 75%;
  left: 50%;
  z-index: 1;
}
.slot-tooltip:hover .slot-tooltip-container {
  visibility: visible;
}
/* #endregion Testregion */
</style>

<script>
import axios from "axios";

export default {
  props: {
    itemData: Object,
  },
  async created() {
    //retrieve item data
    if (Object.keys(this.itemData).length > 0) {
      this.isEmpty = false;
      var resp = await axios
        .get(
          "https://api.guildwars2.com/v2/items/" + this.itemData.id + "?lang=en"
        )
        .then(function (response) {
          return response;
        })
        .catch(function (error) {
          return error.response;
        });
      this.errorCode = resp.status;
      if (this.errorCode == 200) {
        this.item = {
          count: this.itemData.count,
          icon: resp.data.icon,
          rarity: resp.data.rarity,
          name: resp.data.name,
          description: resp.data.description,
        };
      }
      this.isLoading = false;
    }
  },
  data() {
    return {
      isEmpty: true,
      errorCode: 0,
      isLoading: true,
      item: {
        icon: null,
        count: 0,
        rarity: "Basic",
        name: "?",
        description: "?",
      },
    };
  },
  computed: {
    //compute string for item rarity color
    rarityColor() {
      switch (this.item.rarity) {
        case "Fine":
          return "var(--c-accent-rarity-fine)";
        case "Masterwork":
          return "var(--c-accent-rarity-masterwork)";
        case "Rare":
          return "var(--c-accent-rarity-rare)";
        case "Exotic":
          return "var(--c-accent-rarity-exotic)";
        case "Ascended":
          return "var(--c-accent-rarity-ascended)";
        case "Legendary":
          return "var(--c-accent-rarity-legendary)";
        case "Junk":
          return "var(--c-accent-rarity-junk)";
        case "Basic":
        default:
          return "var(--c-text-item-title)";
      }
    },
    //compute string for item rarity border gradient
    rarityBorder() {
      var gradientPlaceholder = this.rarityColor;
      if (this.item.rarity == "Basic") {
        gradientPlaceholder = "var(--color-itemframe)";
      }
      return (
        "linear-gradient(" + gradientPlaceholder + ",var(--color-itemframe)) 5"
      );
    },
  },
};
</script>
