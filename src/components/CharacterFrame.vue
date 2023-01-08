<template>
  <div>
    <div
      class="main-container"
      :class="{ active: isActive }"
      :style="{ 'border-right-color': classColor }"
    >
      <div class="text-name">{{ characterName }}</div>
      <div class="sub-container">
        <div class="text-container">
          <div class="text-level">
            {{ characterCore.level }}
          </div>
          <div class="text-profession" :style="{ color: classColor }">
            {{ characterCore.profession }}
          </div>
        </div>
        <div class="sub-container">
          <div
            v-for="(item, index) in this.characterCrafting"
            :key="index"
            :class="craftingIcon(index)"
            class="crafting-icon crafting-tooltip"
          >
            <span class="crafting-tooltip-container">
              <span class="crafting-tooltip-text">{{
                characterCrafting[index].rating
              }}</span>
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.main-container {
  background: var(--color-background-dark);
  margin: 5px 8px;
  max-width: 312px;
  min-width: 212px;
  min-height: 64px;
  display: block;

  border: 2px solid transparent;
  border-right: 8px solid var(--c-text-item-title);
}
.main-container:hover {
  border-left-color: var(--c-accent-darkred);
}
.main-container .active {
  border-left-color: var(--c-text-item-title);
}

.sub-container {
  display: flex;
  padding: 0px 3px;
}
.text-container {
  display: flex;
  padding: 0px 3px;
  width: 80%;
}
.text-name {
  color: var(--c-text-item-title);
  padding: 4px 0px 4px 6px;
  font-weight: 600;
  font-size: 14pt;
}
.text-level {
  color: var(--c-text-item-title);
  font-weight: 600;
}
.text-profession {
  margin-left: 3pt;
  font-weight: 500;
}
.crafting-icon {
  scale: 0.9;
}

.crafting-tooltip {
  position: relative;
  display: inline-block;
}
.crafting-tooltip .crafting-tooltip-container {
  visibility: hidden;
  background-color: var(--c-tooltip-background);
  border: 2px solid var(--c-tooltip-frame);
  color: var(--color-text);

  position: absolute;
  top: 110%;
  left: 110%;
  z-index: 1;

  padding: 2px 8px;
}
.crafting-tooltip:hover .crafting-tooltip-container {
  visibility: visible;
}
.crafting-tooltip-text {
  display: block;
  font-size: 10pt;
}
</style>

<script>
import axios from "axios";
export default {
  props: {
    authToken: String,
    characterName: String,
    isActive: Boolean,
  },
  data() {
    return {
      characterCore: {
        profession: ". . .",
        level: "??",
      },
      characterCrafting: [],
    };
  },
  async created() {
    //Get character core info
    var resp = await axios
      .get(
        "https://api.guildwars2.com/v2/characters/" +
          this.characterName.replace(" ", "%20") +
          "/core?access_token=" +
          this.authToken
      )
      .then(function (response) {
        return response;
      })
      .catch((error) => console.log(error));
    this.characterCore = resp.data;

    //Get crafting profession info
    var resp2 = await axios
      .get(
        "https://api.guildwars2.com/v2/characters/" +
          this.characterName.replace(" ", "%20") +
          "/crafting?access_token=" +
          this.authToken
      )
      .then(function (response) {
        return response;
      })
      .catch((error) => console.log(error));
    //Remove all crafting professions that ar inactive
    this.characterCrafting = resp2.data.crafting.filter((item) => item.active);
  },
  computed: {
    //compute string for item class color
    classColor() {
      switch (this.characterCore.profession) {
        case "Guardian":
          return "var(--c-accent-class-guardian)";
        case "Warrior":
          return "var(--c-accent-class-warrior)";
        case "Revenant":
          return "var(--c-accent-class-revenant)";
        case "Ranger":
          return "var(--c-accent-class-ranger)";
        case "Engineer":
          return "var(--c-accent-class-engineer)";
        case "Thief":
          return "var(--c-accent-class-thief)";
        case "Elementalist":
          return "var(--c-accent-class-elementalist)";
        case "Mesmer":
          return "var(--c-accent-class-mesmer)";
        case "Necromancer":
          return "var(--c-accent-class-necromancer)";
        default:
          return "var(--c-text-item-title)";
      }
    },
    //compute string for class background gradient
    classGradient() {
      return "linear-gradient(270deg," + this.classColor + " 0%, #0000 12%)";
    },
  },
  methods: {
    craftingIcon(i) {
      return (
        "crafting-icon-" +
        this.characterCrafting[i].discipline.toLocaleLowerCase()
      );
    },
  },
};
</script>
