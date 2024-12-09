<template>
  <div>
    <v-img class="mx-auto my-6" max-width="228"
      src="https://cdn.vuetifyjs.com/docs/images/logos/vuetify-logo-v3-slim-text-light.svg"></v-img>

    <v-card class="mx-auto pa-12 pb-8" elevation="8" max-width="448" rounded="lg">
      <v-form method="post" @submit.prevent="login">
        <div class="text-subtitle-1 text-medium-emphasis">Account</div>
        <v-text-field type="email" v-model="email" required density="compact" placeholder="Email address"
          prepend-inner-icon="mdi-email-outline" variant="outlined"></v-text-field>
        <div class="text-subtitle-1 text-medium-emphasis d-flex align-center justify-space-between">
          Password
        </div>
        <v-text-field v-model="password" :append-inner-icon="visible ? 'mdi-eye-off' : 'mdi-eye'"
          :type="visible ? 'text' : 'password'" density="compact" placeholder="Enter your password"
          prepend-inner-icon="mdi-lock-outline" variant="outlined"
          @click:append-inner="visible = !visible"></v-text-field>
        <v-btn type="submit" block class="mb-5 mt-3" color="primary" size="large" variant="tonal" @click="dialog = true">
          Log In
        </v-btn>

      </v-form>
    </v-card>
    <v-dialog v-model="dialog" :scrim="false" persistent width="150">
      <v-card color="secondary">
        <v-card-text>
          &nbsp;
          <v-progress-linear indeterminate color="white" class="mb-0"></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
export default {
  layout: "auth",
  data() {
    return {
      dialog: false,
      visible: false,
      email: '',
      password: '',
    }
  },
  watch:{
    dialog(val){
      if (!val) return
      setTimeout(()=>(this.dialog = false), 4000)
    }
  },
  methods: {
    async login() {
      await this.$auth.loginWith('laravelSanctum', {
        data: {
          email: this.email,
          password: this.password,
        }
      })
      this.$router.push('/')
    }
  }
}
</script>
