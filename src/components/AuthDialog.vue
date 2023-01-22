<template>
  <div class="main-container">
    <div class="frame">
      <div class="sub-container">
        <input class="input-override input-textarea" v-model="tokenInput" placeholder="OAuth Token"/>
      </div>
      <div class="sub-container">
        <button @click="authenticate">Verify</button>
        <a href="https://account.arena.net/applications" target="_blank">Get your GW2 API-Key</a>
      </div>
      <div v-if="errcode!=200&&errcode!=0">
        <p>Invalid Token</p>
      </div>
      <div v-else>
        <ul class="permissions-list">
          <li class="permission-li" :class="[permissions.account?'active':'inactive']">Account</li>
          <li class="permission-li" :class="[permissions.characters?'active':'inactive']">Characters</li>
          <li class="permission-li" :class="[permissions.inventories?'active':'inactive']">Inventories</li>
          <li class="permission-li" :class="[permissions.tradingpost?'active':'inactive']">Tradingpost</li>
          <li class="permission-li" :class="[permissions.wallet?'active':'inactive']">Wallet</li>
          <li class="permission-li" :class="[permissions.builds?'active':'inactive']">Builds</li>
          <li class="permission-li" :class="[permissions.unlocks?'active':'inactive']">Unlocks</li>
          <li class="permission-li" :class="[permissions.progression?'active':'inactive']">Progression</li>
          <li class="permission-li" :class="[permissions.pvp?'active':'inactive']">PVP</li>
          <li class="permission-li" :class="[permissions.guilds?'active':'inactive']">Guilds</li>
        </ul>
      </div>
      <div>
        <button v-if="errcode==200&&this.sufficientPermissions==true" @click="confirmAction">Confirm</button>
        <p v-if="errcode==200&&this.sufficientPermissions==false">Your token requires permissions to 'characters' and 'inventories'</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.main-container{
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100vw;
  height: 100vh;
}
.frame{
  width: 700px;
  height: 400px;
  background-color: var(--color-background-dark);
  border: 3px solid var(--c-accent-darkred);
  border-radius: 12px;
  display: block;
  padding: 12px;
}
.sub-container{
  display: flex;
  justify-content: center;
  width: 100%;
}
.input-textarea{
  width: 100%;
  font-size: 11pt;
  margin-top: 32px;
}
.permissions-list{
  list-style: none;
}
.permission-li{
  
}

.permission-li.active::before{
  content: 'o  ';
  color:green;
  font-weight:bold;
}
.permission-li.inactive::before{
  content: 'X  ';
  color:red;
  font-weight:bold;
}
a a:link, a:visited, a:hover, a:active{
  color: var(--c-accent-darkred)
}

</style>

<script>
import axios from "axios";

export default {
  emits: ['validated'],
  data(){
    return {
      tokenInput: "",
      token: "",
      permissions: {
        tradingpost: false,
        characters: false,
        pvp: false,
        progression: false,
        wallet: false,
        guilds: false,
        builds: false,
        account: false,
        inventories: false,
        unlocks: false
      },
      errcode: 0,
      sufficientPermissions: false,
      loading: false
    }
  },
  methods: {
    async authenticate() {
      if(this.loading){return}
      this.loading=true
      this.errcode=0
      this.sufficientPermissions=false
      this.token=""
      this.permissions = {
        tradingpost: false,
        characters: false,
        pvp: false,
        progression: false,
        wallet: false,
        guilds: false,
        builds: false,
        account: false,
        inventories: false,
        unlocks: false
      }

      if(this.tokenInput.length==0){return}
      var resp = await axios
        .get("https://api.guildwars2.com/v2/tokeninfo?access_token="+this.tokenInput)
        .then(res=>{return res})
        .catch(err=>{return err.response})
      if(resp.status == 200){
        if(resp.data.permissions.includes('tradingpost')){this.permissions.tradingpost=true}
        if(resp.data.permissions.includes('characters')){this.permissions.characters=true}
        if(resp.data.permissions.includes('pvp')){this.permissions.pvp=true}
        if(resp.data.permissions.includes('progression')){this.permissions.progression=true}
        if(resp.data.permissions.includes('wallet')){this.permissions.wallet=true}
        if(resp.data.permissions.includes('guilds')){this.permissions.guilds=true}
        if(resp.data.permissions.includes('builds')){this.permissions.builds=true}
        if(resp.data.permissions.includes('account')){this.permissions.account=true}
        if(resp.data.permissions.includes('inventories')){this.permissions.inventories=true}
        if(resp.data.permissions.includes('unlocks')){this.permissions.unlocks=true}
        if(this.permissions.characters==true&&this.permissions.inventories==true){this.sufficientPermissions=true}
        this.token=this.tokenInput
        this.errcode=200
      }
      else{
        this.errcode=resp.status
      }
      this.loading=false
    },
    confirmAction() {
      this.$emit('validated', this.token, this.permissions)
    }
  }
}
</script>