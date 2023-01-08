<template>
  <div class="main">
    <div>
      <CharacterSelect :authToken="urlParamsToken"> </CharacterSelect>
    </div>
  </div>
</template>

<style scoped>
.main {
  display: inline-block;
}
</style>

<script>
import CharacterSelect from "@/components/CharacterSelect.vue";

export default {
  components: { CharacterSelect },
  data() {
    return {
      urlParamsToken: null,
    };
  },
  created() {
    //retrieve Auth token from URL (?token='GW2AUTHTOKEN')
    var urlParams = new URLSearchParams(window.location.search);
    //console.log(urlParams.has('token'));
    this.urlParamsToken = urlParams.get("token");
  },
};
</script>
