<template>
  <v-app dark>
    <v-navigation-drawer v-model="drawer" :mini-variant="miniVariant" :clipped="clipped" fixed app color="blue darken-1" dark>
      <v-list>
        <v-list-item v-for="(item, i) in dashboard" :key="i" :to="item.to" router exact>
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>

        <v-list-subheader class="ms-3" style="color: #CFD8DC;">MASTER DATA</v-list-subheader>
        <v-list-item v-for="(item, i) in masterdata" :key="i" :to="item.to" router exact>
          <v-list-item-action>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>{{ item.title }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-list-subheader class="ms-3" style="color: #CFD8DC;">TRANSAKSI</v-list-subheader>
        <v-list-item v-for="(item, i) in transaksi" :key="i" :to="item.to" router exact>
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
            <v-icon class="mdi-48px">mdi-account-circle</v-icon><br>
            <b>{{ user }}</b>
            <p>Status : <b>{{ akses }}</b></p>
            <v-btn block id="out" app color="primary" @click.prevent="logout"> Log Out </v-btn>
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
import axios from 'axios'
export default {
  name: 'DefaultLayout',
  data() {
    return {
      apiurl: "http://localhost:8000/api",
      akses: "",
      user: "",
      id_user: "",
      clipped: false,
      drawer: false,
      fixed: true,
      miniVariant: false,
      right: true,
      rightDrawer: false,
      title: 'Belanja Aja',
      dashboard: "",
      masterdata: "",
      transaksi: "",
      laporan: "",
    }
  },
  computed:{
  },
  watch:{
  },
  methods:{
    async logout(){
      await this.$auth.logout()
      this.$router.push('/login')
    },
    pushuser(){
      try {
        this.user = this.$auth.user.data.name
        this.id_user = this.$auth.user.data.id
        document.getElementById('out').style.display = 'block'
      } catch (error) {
        document.getElementById('out').style.display = 'none'
        this.user = "Anda Harus Log In Terlebih Dahulu"
      }
    },
    getakses(){
      axios.get(this.apiurl + '/akses')
      .then((ress)=>{
        const user = ress.data.filter((user) => user.id_user === this.id_user);
        this.akses = user[0].akses
      })
      .then(()=>{
        if (this.akses == "kepala") {
          this.$router.push('/')
          var dhs = [
            {icon: 'mdi-apps', title: 'Dashboard', to: '/'},
          ]
          var md = [
            {icon: 'mdi-package-variant', title: 'Barang', to: '/data_induk/barang'},
            {icon: 'mdi-all-inclusive', title: 'Pemasok', to: '/data_induk/pemasok'},
            {icon: 'mdi-human-male-female', title: 'Pelanggan', to: '/data_induk/pelanggan'},
            {icon: 'mdi-account', title: 'Users', to: '/data_induk/user'},
          ]
          var tr = [
            {icon: 'mdi-calculator', title: 'Faktur', to: '/transaksi/registrasi'},
            {icon: 'mdi-credit-card-plus', title: 'Komisi', to: '/keuangan/komisi'},
          ]
          this.dashboard = dhs
          this.masterdata = md
          this.transaksi = tr
        } else if (this.akses == "karyawan") {
          this.$router.push('/dashboard_karyawan')
          var dhs = [
            {icon: 'mdi-apps', title: 'Dashboard', to: '/dashboard_karyawan'},
          ]
          var md = [
            {icon: 'mdi-package-variant', title: 'Barang', to: '/data_induk/barang'},
            {icon: 'mdi-all-inclusive', title: 'Pemasok', to: '/data_induk/pemasok'},
            {icon: 'mdi-human-male-female', title: 'Pelanggan', to: '/data_induk/pelanggan'},
          ]
          var tr = [
            {icon: 'mdi-calculator', title: 'Faktur', to: '/transaksi/registrasi'},
          ]
          this.dashboard = dhs
          this.masterdata = md
          this.transaksi = tr
        } else {
          this.$router.push('/belum_terdaftar')
        }
      })
      .catch(()=>{
        console.log(err);
      })
    },
  },
  mounted() {
    this.pushuser()
    this.getakses()
  },
}
</script>
