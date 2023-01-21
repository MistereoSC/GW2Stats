<template>
  <div class="main">
    <div v-if="authenticated==true">
      <CharacterSelect :authToken="urlParamsToken"> </CharacterSelect>
    </div>
    <div v-else>
      <AuthDialog @validated="(token,permissions) => userValidated(token,permissions)"></AuthDialog>
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
import AuthDialog from "@/components/AuthDialog.vue";

export default {
  components: { CharacterSelect, AuthDialog },
  data() {
    return {
      urlParamsToken: null,
      authenticated: false,
      permissions: {}
    };
  },
  created() {
    //retrieve Auth token from URL (?token='GW2AUTHTOKEN')
    // var urlParams = new URLSearchParams(window.location.search);
    // this.urlParamsToken = urlParams.get("token");
  },
  methods: {
    userValidated(t,p) {
      console.log(t)
      this.urlParamsToken=t
      this.permissions=p
      this.authenticated=true
      
    }
  }
};
</script>
