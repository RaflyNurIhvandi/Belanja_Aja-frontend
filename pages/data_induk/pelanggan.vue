<template>
  <v-container>
    <h1>Pelanggan</h1>
    <v-form v-model="valid" ref="form">
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="namaPelanggan" id="nama-pelanggan" v-model="form.namaPelanggan" :rules="namaPelangganRulers"
            label="Nama Pelanggan" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="noTelp" @keypress="isNumber($event)" id="no-telp" v-model="form.noTelp" :rules="noTelpRulers"
            label="No. Telp" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="kotaPelanggan" id="kota-pelanggan" v-model="form.kotaPelanggan" :rules="kotaPelangganRulers"
            label="Kota Pelanggan" required></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="email" id="email" v-model="form.email" :rules="emailRulers"
            label="E-mail" required></v-text-field>
        </v-col>
        <v-col cols="12" md="8">
          <v-text-field id="alamat" v-model="form.alamat" :rules="alamatRulers"
            label="Alamat" required></v-text-field>
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
      <h2>Data Pelanggan</h2>
      <v-text-field v-model="cari" append-icon="mdi-magnify" label="Pencarian" single-line></v-text-field>
      <v-data-table :headers="headpelanggan" :items="dtpelanggan" :search="cari" :items-per-page="5" class="elevation-1">
        <template v-slot:[`item.tools`]="{ item }">
          <v-icon color="success" @click="edit(item)">
            mdi-pencil
          </v-icon>
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
import axios from 'axios'
export default {
  data() {
    return {
      apiurl: "http://localhost:8000/api",
      dialog: false,
      valid: false,
      cari: "",
      dtpelanggan: [],
      updateSubmit: false,
      form: {
        id: "",
        namaPelanggan: "",
        noTelp: "",
        kotaPelanggan: "",
        email: "",
        alamat: "",
        user: this.$auth.user.data.email
      },
      dtedit: {
        id: "",
        nama_pelanggan: "",
        no_telp_pelanggan: "",
        kota_pelanggan: "",
        email_pelanggan: "",
        alamat_pelanggan: "",
        user: this.$auth.user.data.email
      },
      headpelanggan: [
        { text: "No", value: "id" },
        { text: "Nama Pelanggan", value: "nama_pelanggan" },
        { text: "Kode Pelanggan", value: "kode_pelanggan" },
        { text: "No. Telp", value: "no_telp_pelanggan" },
        { text: "Kota Pemasok", value: "kota_pelanggan" },
        { text: "E-mail", value: "email_pelanggan" },
        { text: "Alamat", value: "alamat_pelanggan" },
        { text: "Tools", value: "tools" }
      ],
      namaPelangganRulers: [
        (v) => !!v || 'Nama Pelanggan tidak boleh kosong!',
      ],
      noTelpRulers: [
        (v) => !!v || 'No. Telp tidak boleh kosong!',
      ],
      kotaPelangganRulers: [
        (v) => !!v || 'Kota Pelanggan tidak boleh kosong!',
      ],
      alamatRulers: [
        (v) => !!v || 'Alamat Jual tidak boleh kosong!',
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
          .delete(this.apiurl + `/delete-pelanggan/${item.id}`)
          .then(() => {
            this.loadPelanggan()
            Swal.fire({
              position: 'top-end',
              icon: 'success',
              title: 'Berhasil menghapus data',
              showConfirmButton: false,
              timer: 1000
            })
          })
        }
      })
    },
    update(form) {
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
            .put(this.apiurl + '/update-pelanggan/' + form.id, {
              'namaPelanggan': this.form.namaPelanggan,
              'noTelp': this.form.noTelp,
              'kotaPelanggan': this.form.kotaPelanggan,
              'email': this.form.email,
              'alamat': this.form.alamat,
              'user': this.form.user,
            })
            .then(() => {
              this.loadPelanggan()
              this.$refs.form.reset()
              Swal.fire({
                position: 'top-end',
                icon: 'success',
                title: 'Berhasil mengupdate data',
                showConfirmButton: false,
                timer: 1000
              })
              this.updateSubmit = false
            })
        }
      })
    },
    edit(item){
      this.updateSubmit = true
      this.dtedit = Object.assign({}, item)
      this.form.id = this.dtedit.id
      this.form.namaPelanggan = this.dtedit.nama_pelanggan
      this.form.noTelp = this.dtedit.no_telp_pelanggan
      this.form.kotaPelanggan = this.dtedit.kota_pelanggan
      this.form.email = this.dtedit.email_pelanggan
      this.form.alamat = this.dtedit.alamat_pelanggan
    },
    save() {
      if (this.form.namaPelanggan == null || this.form.noTelp == null || this.form.kotaPelanggan == null || this.form.email == null || this.form.alamat == null) {
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
            .post(this.apiurl + '/insert-pelanggan', this.form)
            .then(() => {
              this.loadPelanggan()
              this.$refs.form.reset()
              Swal.fire({
                position: 'top-end',
                icon: 'success',
                title: 'Berhasil menyimpan data',
                showConfirmButton: false,
                timer: 1000
              })
            })
          }
        })
      }
    },
    loadPelanggan(){
      this.$axios.get(this.apiurl + '/load-pelanggan')
      .then((ress)=>{
        this.dtpelanggan = ress.data.data
      })
      .catch(() => {
        console.log(err);
      })
    },
    baru(){
      this.$refs.form.reset()
      document.getElementById('nama-pelanggan').focus()
      this.updateSubmit = false
    },
    batal(){
      this.$refs.form.reset()
      this.updateSubmit = false
    },
    namaPelanggan(){
      document.getElementById('no-telp').focus()
    },
    noTelp(){
      document.getElementById('kota-pelanggan').focus()
    },
    kotaPelanggan(){
      document.getElementById('email').focus()
    },
    email(){
      document.getElementById('alamat').focus()
    },
    isNumber: function (evt) {
      evt = (evt) ? evt : window.event;
      var charCode = (evt.which) ? evt.which : evt.keyCode;
      if ((charCode > 31 && (charCode < 48 || charCode > 57)) && charCode !== 46) {
        evt.preventDefault();;
      } else {
        return true;
      }
    },
  },
  mounted() {
    this.loadPelanggan()
  }
}
</script>
