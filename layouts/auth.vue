<template>
  <v-app dark>
    <v-navigation-drawer v-model="drawer" :mini-variant="miniVariant" :clipped="clipped" fixed app color="blue darken-1" dark>
      <v-list>
        <v-list-item v-for="(item, i) in items" :key="i" :to="item.to" router exact>
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar :clipped-left="clipped" fixed app color="blue lighten-5">
      <v-app-bar-nav-icon @click.stop="drawer = !drawer" />
      <v-btn icon @click.stop="miniVariant = !miniVariant">
        <v-icon>mdi-{{ `chevron-${miniVariant ? 'right' : 'left'}` }}</v-icon>
      </v-btn>
      <v-btn icon @click.stop="clipped = !clipped">
        <v-icon>mdi-application</v-icon>
      </v-btn>
      <v-btn icon @click.stop="fixed = !fixed">
        <v-icon>mdi-minus</v-icon>
      </v-btn>
      <v-toolbar-title>{{ title }}</v-toolbar-title>
      <v-spacer />
      <v-btn icon @click.stop="rightDrawer = !rightDrawer">
        <v-icon>mdi-account-tie</v-icon>
      </v-btn>
    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
    <v-navigation-drawer v-model="rightDrawer" :right="right" temporary fixed>
      <v-list>
        <v-list-item @click.native="right = !right">
          <v-list-item-action>
            <v-icon light> mdi-repeat </v-icon>
          </v-list-item-action>
          <v-list-item-title>Switch drawer (click me)</v-list-item-title>
        </v-list-item>
        <div style="padding-left: 10%; padding-right: 10%;">
          <div style="text-align: center;" class="mt-5">
            <v-icon class="mdi-48px">mdi-account-alert</v-icon><br>
            <p style="color: red;">{{ user }}</p>
            <v-btn id="out" color="secondary" @click.prevent="logout"> Log Out </v-btn>
          </div>
        </div>
      </v-list>
    </v-navigation-drawer>
    <v-footer :absolute="!fixed" app color="blue lighten-5">
      <span>&copy; {{ new Date().getFullYear() }} Copyright <b>
        <a target="_blank" href="https://instagram.com/raflynurihvandi?igshid=OGQ5ZDc2ODk2ZA==" style="text-decoration:none">
          Rafly Nur Ihvandi
        </a></b> . All Rights Reserved.</span>
    </v-footer>
  </v-app>
</template>
<script>
export default {
  name: 'DefaultLayout',
  data() {
    return {
      user: "",
      clipped: false,
      drawer: false,
      fixed: false,
      items: [
        {
          icon: 'mdi-login',
          title: 'Log In',
          to: '/',
        },
      ],
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'Belanja Aja',
    }
  },
  watch:{
  },
  methods:{
    async logout(){
      await this.$auth.logout()
      this.$router.push('/login')
      this.pushuser()
    },
    pushuser(){
      try {
        this.user = this.$auth.user.data.name
        document.getElementById('out').style.display = 'block'
      } catch (error) {
        document.getElementById('out').style.display = 'none'
        this.user = "Anda Harus Log In Terlebih Dahulu"
      }
    }
  },
  mounted() {
    this.pushuser()
  },
}
</script>
