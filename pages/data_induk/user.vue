<template>
  <v-container>
    <h1>Users</h1>
    <v-form v-model="valid" ref="form">
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="name" id="name" v-model="form.name" :rules="nameRulers" label="Name"
            required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="email" id="email" v-model="form.email" :rules="emailRulers" label="E-mail"
            required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="pass" id="pass" v-model="form.pass" :rules="passRulers" label="Password"
            required></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col class="d-flex flex-row mb-6 bg-surface-variant mt-4" cols="12" ms="6" md="6">
          <v-btn class="mr-4" color="success" width="150px" @click="baru">baru</v-btn>
          <v-btn class="mr-4" width="150px" color="primary" type="button" name="button" v-show="!updateSubmit"
            @click="save" :disabled="dialog" :loading="dialog">simpan</v-btn>
          <v-btn class="mr-4" width="150px" color="primary" type="button" name="button" v-show="updateSubmit"
            v-on:click="update(form)" isvalid="true">simpan</v-btn>
          <v-btn color="error" class="mr-4" width="150px" @click="batal">batal</v-btn>
        </v-col>
      </v-row>
    </v-form>
    <div>
      <h2>Data Users</h2>
      <v-text-field v-model="cari" append-icon="mdi-magnify" label="Pencarian" single-line></v-text-field>
      <v-data-table :headers="headuser" :items="dtuser" :search="cari" :items-per-page="5" class="elevation-1">
        <template v-slot:[`item.tools`]="{ item }">
          <v-icon color="error" @click="hapus(item)">
            mdi-delete
          </v-icon>
        </template>
      </v-data-table>
    </div>
    <v-dialog v-model="dialog" :scrim="false" persistent width="150">
      <v-card color="secondary">
        <v-card-text>
          &nbsp;
          <v-progress-linear indeterminate color="white" class="mb-0"></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-container>
</template>
<script>
import Swal from 'sweetalert2'
export default {
  data() {
    return {
      apiurl: "http://localhost:8000/api",
      dialog: false,
      valid: false,
      updateSubmit: false,
      cari: "",
      dtuser: [],
      form: {
        id: "",
        name: "",
        email: "",
        pass: ""
      },
      dtedit: {
        id: "",
        name: "",
        email: "",
        pass: ""
      },
      headuser: [
        { text: "No", value: "id" },
        { text: "Name", value: "name" },
        { text: "email", value: "email" },
        { text: "Password", value: "password" },
        { text: "Keterangan", value: "akses" },
        { text: "Tools", value: "tools" }
      ],
      passRulers: [
        (v) => !!v || 'Nama Pemasok tidak boleh kosong!',
      ],
      nameRulers: [
        (v) => !!v || 'Name tidak boleh kosong!',
      ],
      emailRulers: [
        value => {
          if (value) return true

          return 'E-mail tidak boleh kosong!'
        },
        value => {
          if (/.+@.+\..+/.test(value)) return true

          return 'E-mail belum valid!'
        },
      ],
    }
  },
  computed: {

  },
  watch: {
    dialog(val){
      if (!val) return
      setTimeout(()=>(this.dialog = false), 4000)
    }
  },
  methods: {
    hapus(item) {
      Swal.fire({
        title: 'Anda Yakin?',
        text: "Data yang dihapus tidak bisa dikembalikan!",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#3085d6',
        cancelButtonColor: '#d33',
        confirmButtonText: 'Yes'
      }).then((result) => {
        if (result.isConfirmed) {
          this.dialog = true
          this.$axios
          .delete(this.apiurl + `/delete-user/${item.id}`)
          .then(() => {
            this.loadUser()
            Swal.fire({
              position: 'top-end',
              icon: 'success',
              title: 'Berhasil menghapus User',
              showConfirmButton: false,
              timer: 1000
            })
          })
        }
      })
    },
    save(){
      if (this.form.name == null || this.form.email == null || this.form.pass == null) {
        Swal.fire({
          icon: 'error',
          title: 'Data belum lengkap!',
          showConfirmButton: false,
          timer: 1000
        })
      } else {
        Swal.fire({
          title: 'Anda Yakin?',
          text: "Pastikan data yang diisi sudah benar!",
          icon: 'question',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'Yes'
        }).then((result) => {
          if (result.isConfirmed) {
            this.dialog = true
            this.$axios
            .post(this.apiurl + '/insert-user', this.form)
            .then(() => {
              this.loadUser()
              this.$refs.form.reset()
              Swal.fire({
                position: 'top-end',
                icon: 'success',
                title: 'Berhasil manambah User baru',
                showConfirmButton: false,
                timer: 1000
              })
            })
          }
        })
      }
    },
    loadUser(){
      this.$axios.get(this.apiurl + '/load-user')
      .then((ress)=>{
        this.dtuser = ress.data.data
      })
      .catch(() => {
        console.log(err);
      })
    },
    baru(){
      this.$refs.form.reset()
      document.getElementById('name').focus()
      this.updateSubmit = false
    },
    batal(){
      this.$refs.form.reset()
      this.updateSubmit = false
    },
    email(){
      document.getElementById('pass').focus()
    },
    name(){
      document.getElementById('email').focus()
    },
  },
  mounted() {
    this.loadUser()
  }
}
</script>
