<template>
  <div class="bag-container">
    <InventorySlot
      v-for="item in this.bagContents"
      :key="item.index"
      :itemData="item"
    >
    </InventorySlot>
  </div>
</template>

<style scoped>
.bag-container {
  padding: 0px 0px 2px 2px;
  padding: 5px;
  display: flex;
  flex-flow: row wrap;
  max-width: 670px;
}
</style>

<script>
import InventorySlot from "@/components/InventorySlot.vue";

export default {
  components: { InventorySlot },
  props: {
    bagContents: Object,
  },
  data() {
    return {
      isLoading: true,
    };
  },
  created() {
    //Replace 'null'-Strings with empty Objects to prevent null-references
    //Note: Each InventorySlot makes its own API call to retrieve item data
    for (let i = 0; i < this.bagContents.length; i++) {
      if (this.bagContents[i] == null) {
        this.bagContents[i] = new Object();
      }
    }
  },
};
</script>
