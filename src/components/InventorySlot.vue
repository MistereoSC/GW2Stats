<template>
  <div>
    <div v-if="isLoading" class="inventory-slot">
      <div class="loading-overlay">
        <div class="loading-spinner"></div>
      </div>
    </div>
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
      @click.exact="lClick()"
      @click.ctrl.exact="lClick()"
      @click.alt.prevent="copyToClipboard()"
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
        <div class="slot-tooltip-text">{{levelTextIfNecessary}}{{ specificType }}</div>
        <div class="slot-tooltip-text">{{ item.description }}</div>
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
  user-select: none;
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
/* #region loading overlay CSS */
.loading-overlay {
  background-color: var(--c-tooltip-background);
  border: 2px solid var(--c-tooltip-frame);
  width: 100%;
  height: 100%;
  z-index: 1;
}
.loading-spinner {
  height: 24px;
  width: 24px;
  margin: auto;
  margin-top: 25%;
  border: 3px solid white;
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
/* #endregion */

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
.slot-tooltip-text {
  margin:0px;
  margin-top: 5px;
  padding:0px;
  white-space: pre-wrap;
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
/* #endregion */
</style>

<script>
import axios from "axios";
import statChoices from "@/assets/data/itemStats.json"

export default {
  props: {
    itemData: Object,
  },
  data() {
    return {
      statChoices: statChoices,
      isEmpty: true,
      errorCode: 0,
      isLoading: true,
      item: {
        icon: null,
        count: 0,
        rarity: "Basic",
        name: "?",
        description: "",
        chatLink: "",
        type: "",
        details: null
      },
    };
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
          level: resp.data.level,
          icon: resp.data.icon,
          rarity: resp.data.rarity,
          name: resp.data.name,
          chatLink: resp.data.chat_link,
          type: resp.data.type,
          details: resp.data.details,
          description: resp.data.description
        };
        this.item.description = this.buildDescription(this.item.description,this.item.type)
      }
      this.isLoading = false;
    }
    else{this.isLoading=false;this.errorCode=200}
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
    levelTextIfNecessary() {
      var out=""
      if(this.item.level > 0) {out += "("+this.item.level+") "}
      if(this.item.type==="Armor") {out += this.item.details.weight_class+" "}
      return out
    },
    specificType() {
      if(this.item.type==="Weapon"||this.item.type==="Armor"||this.item.type=="Trinket") {
        if(this.item.details.type==="HelmAquatic") return "Aquatic Gear"
        else if(this.item.details.type==="LongBow") return "Longbow"
        else if(this.item.details.type==="ShortBow") return "Shortbow"
        else return this.item.details.type
      }
      else if (this.item.type==="CraftingMaterial") return "Material"
      else if (this.item.type==="MiniPet") return "Mini"
      else if (this.item.type==="UpgradeComponent") return "Component"
      else return this.item.type
    }
  },
  methods: {
    lClick(){
      var url = "https://wiki.guildwars2.com/index.php?search=%5B%26"+this.item.chatLink.substring(2,this.item.chatLink.length-1)+"%5D"
      window.open(url, '_blank')
    },
    copyToClipboard(){
      navigator.clipboard.writeText(this.item.chatLink)
    },
    buildDescription(txt,type){
      var out=""
      //Weapon/Armor/Back/Trinket defaults
      if (type==="Weapon"||type==="Armor"||type==="Back"||type==="Trinket"){
        //Armor Defense
        if(this.item.type==="Armor"){out+=this.item.details.defense+" Armor\n"}
        //Weapon Damage
        if(this.item.type==="Weapon"){out+=this.item.details.min_power+" - "+this.item.details.max_power+" Damage\n"}
        //Stat Selectable
        if(this.item.details.stat_choices!==undefined){
          const sq = this.item.details.stat_choices
          out += "Stat selectable:\n"
          for(let i=0;i<sq.length;i++){
            if(i===3){out+="  ( . . . )\n";break}
            out+="  "+this.statChoices.find(itm => itm.id==sq[i]).name+"\n"
          }
        }
        else {
          const sq = this.item.details.infix_upgrade
          out += this.statChoices.find(itm => itm.id==sq.id).name+"\n"
          for(let i=0;i<sq.attributes.length;i++){
            out+="  "+sq.attributes[i].modifier+"  "
            switch(sq.attributes[i].attribute){
              case "ConditionDamage":
                out+="Condition Damage"
                break
              case "Healing":
                out+="Healing Power"
                break
              case "CritDamage":
                out+="Ferocity"
                break
              case "BoonDuration":
                out+="Concentration"
                break
              case "ConditionDuration":
                out+="Expertise"
                break
              default:
                out+=sq.attributes[i].attribute+"\n"
            }
          }
        }
      }
      else if(type ==="Consumable"){
        if(this.item.description==undefined){out=this.item.details.description}
        else{out=this.item.description}
        
      }
      //Flavor Text filter html tags
      else if (txt!==undefined && txt!="") {
        return txt.replace(/(<[^>]*>)/g,'\n')
      }
      return out
    }
  }
};
</script>
