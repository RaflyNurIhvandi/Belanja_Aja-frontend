<template>
  <v-container>
    <h1 style="color: red;">Warning!</h1>
    <p>User Name <b>{{ user.name }}</b> Belum Terdaftar!</p>
  </v-container>
</template>
<script>
import Swal from 'sweetalert2'
import axios from 'axios'
export default {
  data() {
    return {
      user: this.$auth.user.data,
    }
  },
  computed: {

  },
  watch: {

  },
  methods: {
  },
  mounted() {
  }
}
</script>
