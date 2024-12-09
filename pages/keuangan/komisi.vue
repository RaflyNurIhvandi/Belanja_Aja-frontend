<template>
  <v-container>
    <h1>Komisi</h1>
    <v-form v-model="valid" ref="form">
      <v-row>
        <v-col cols="12" md="4">
          <v-combobox v-model="form.namaKaryawan" id="nama-karyawan" label="Nama Karyawan"
              :items="pemItm" variant="underlined" @keydown.enter="karyawan"></v-combobox>
        </v-col>
        <v-col cols="12" md="3">
          <v-text-field v-model="form.komisi" id="komisi" label="Komisi" required></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col class="d-flex flex-row mb-6 bg-surface-variant mt-4" cols="12" ms="6" md="6">
          <v-btn class="mr-4" color="success" width="150px" @click="baru">baru</v-btn>
          <v-btn class="mr-4" width="150px" color="primary" type="button" name="button"
            @click="save" :disabled="dialog" :loading="dialog">simpan</v-btn>
          <v-btn color="error" class="mr-4" width="150px" @click="batal">batal</v-btn>
        </v-col>
      </v-row>
    </v-form>
    <div>
      <h2>Data Komisi</h2>
      <v-text-field v-model="cari" append-icon="mdi-magnify" label="Pencarian" single-line></v-text-field>
      <v-data-table :headers="headkomisi" :items="dtkomisi" :search="cari" :items-per-page="5" class="elevation-1">
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
      cari: "",
      dtkomisi: [],
      headkomisi: [
        { text: "No", value: 'id' },
        { text: "Nama Karyawan", value: "nama_karyawan" },
        { text: "Komisi", value: "komisi" },
      ],
      dtkodes: [],
      pemItm: [],
      valid: false,
      form: {
        namaKaryawan: "",
        komisi: ""
      }
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
    karyawan(){
      document.getElementById('komisi').focus()
    },
    baru(){
      document.getElementById('nama-karyawan').focus()
      this.$refs.form.reset()
    },
    save() {
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
            .post(this.apiurl + '/insert-komisi', this.form)
            .then(() => {
              this.loadKaryawan()
              this.loadKomisi()
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
    },
    loadKomisi(){
      this.$axios.get(this.apiurl + '/load-komisi')
      .then((ress)=>{
        this.dtkomisi = ress.data.data
      })
      .catch(() => {
        console.log(err);
      })
    },
    loadKaryawan(){
      this.$axios.get(this.apiurl + '/load-karyawan')
      .then((ress)=>{
        this.dtkodes = ress.data.data
      })
      .then(()=>{
        for (let i = 0; i < this.dtkodes.length; i++) {
          this.pemItm.push(this.dtkodes[i].name)
        }
      })
      .catch(() => {
        console.log(err);
      })
    },
  },
  mounted() {
    this.loadKaryawan()
    this.loadKomisi()
  }
}
</script>
