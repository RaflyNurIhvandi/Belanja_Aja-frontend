<template>
  <v-container>
    <h1>Dashboard</h1>
    <v-row>
      <v-col cols="12" md="4">
        <v-date-picker show-adjacent-months elevation="3" v-model="date" :show-current="true" width="100%"></v-date-picker>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import Swal from 'sweetalert2'
import axios from 'axios'

export default {
  data() {
    return {
      date: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),
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
