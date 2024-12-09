<template>
  <v-container>
    <h1>Barang</h1>
    <v-form v-model="valid" ref="form">
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="namaBarang" id="nama-barang" v-model="form.namaBarang" :rules="namaBarangRulers"
            label="Nama Barang" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="hargaBeli" @keypress="isNumber($event)" id="harga-beli" v-model="form.hargaBeli" :rules="hargaBeliRulers"
            label="Harga Beli" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="hargaJual" @keypress="isNumber($event)" id="harga-jual" v-model="form.hargaJual" :rules="hargaJualRulers"
            label="Harga Jual" required></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="5">
          <div v-for="kod in filterPemasok" :key="kod.id" v-show="isKode">
            <h2 style="font-weight:normal;">{{ kod.nama_pemasok }}</h2>
            <h5 style="font-weight:normal;">{{ kod.alamat }}<br>Kota. {{ kod.kota_pemasok }}, Telp. {{ kod.no_telp
            }}</h5>
          </div>
        </v-col>
        <v-col cols="12" md="3">
        </v-col>
        <v-col cols="12" md="4">
          <v-combobox v-model="form.pemSel" id="kode-pemasok" @change="kodePemasok" label="Kode Pemasok"
              :items="pemItm" variant="underlined" :rules="kodeRulers"></v-combobox>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="jumlahBarang" @keypress="isNumber($event)" id="jumlah-barang" v-model="form.jumlahBarang" :rules="jumlahBarangRulers"
            label="Jumlah Barang" required></v-text-field>
        </v-col>
        <v-col cols="12" md="4">
        </v-col>
        <v-col cols="12" md="4">
          <v-text-field @keydown.enter="tanggalMasuk" id="tanggal-masuk" v-model="form.tanggalMasuk" :rules="tanggalRulers"
            label="Tanggal Masuk" type="date" required></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="12" md="12">
          <v-text-field id="deskripsi" v-model="form.deskripsi" :rules="deskripsiRulers"
            label="Deskripsi" required></v-text-field>
        </v-col>
      </v-row>
      <v-row>
        <v-col class="d-flex flex-row mb-6 bg-surface-variant mt-5" cols="12" ms="6" md="6">
          <v-btn class="mr-4" color="success" width="150px" @click="baru">baru</v-btn>
          <v-btn class="mr-4" width="150px" color="primary" type="button" name="button" v-show="!updateSubmit"
            @click="save" :disabled="dialog" :loading="dialog">simpan</v-btn>
          <v-btn class="mr-4" width="150px" color="primary" type="button" name="button" v-show="updateSubmit"
            v-on:click="update(form)" isvalid="true">simpan</v-btn>
          <v-btn color="error" class="mr-4" width="150px" @click="batal">batal</v-btn>
        </v-col>
      </v-row>
    </v-form>
    {{ dtedit.jumlah_barang }}
    <div>
      <h2>Data Barangs</h2>
      <v-text-field v-model="cari" append-icon="mdi-magnify" label="Pencarian" single-line></v-text-field>
      <v-data-table :headers="headbarang" :items="dtbarangs" :search="cari" :items-per-page="5" class="elevation-1">
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
      cari: "",
      dialog: false,
      valid: false,
      isKode: false,
      updateSubmit: false,
      form: {
        id: "",
        namaBarang: "",
        hargaBeli: "",
        hargaJual: "",
        pemSel: "",
        jumlahBarang: "",
        tanggalMasuk: "",
        deskripsi: "",
        user: this.$auth.user.data.email
      },
      dtedit: {
        id: "",
        nama_barang: "",
        harga_beli: "",
        harga_jual: "",
        kode_pemasok: "",
        jumlah_barang: "",
        tanggal_masuk: "",
        deskripsi: "",
        user: this.$auth.user.data.email
      },
      dtkodes: [],
      pemItm: [],
      dtpemasoks: [],
      dtbarangs: [],
      headbarang: [
        { text: "No", value: 'id' },
        { text: "Nama Barang", value: "nama_barang" },
        { text: "Kode Barang", value: "kode_barang" },
        { text: "Harga Beli", value: "harga_beli" },
        { text: "Harga Jual", value: "harga_jual" },
        { text: "Pemasok", value: "nama_pemasok" },
        { text: "Jumlah Barang", value: "jumlah_barang" },
        { text: "Tanggal Masuk", value: "tanggal_masuk" },
        { text: "Tools", value: "tools" }
      ],
      namaBarangRulers: [
        (v) => !!v || 'Nama Barang tidak boleh kosong!',
      ],
      hargaBeliRulers: [
        (v) => !!v || 'Harga Beli tidak boleh kosong!',
      ],
      hargaJualRulers: [
        (v) => !!v || 'Harga Jual tidak boleh kosong!',
      ],
      kodeRulers: [
        (v) => !!v || 'Kode Pemasok tidak boleh kosong!',
      ],
      jumlahBarangRulers: [
        (v) => !!v || 'Jumlah Barang tidak boleh kosong!',
      ],
      tanggalMasukRulers: [
        (v) => !!v || 'Tanggal Masuk tidak boleh kosong!',
      ],
      tanggalMasukRulers: [
        (v) => !!v || 'Tanggal Masuk tidak boleh kosong!',
      ],
      deskripsiRulers: [
        (v) => !!v || 'Deskripsi tidak boleh kosong!',
      ],
    }
  },
  computed: {
    filterPemasok: function () {
      return this.dtpemasoks.filter((pm) => {
        return pm.kode_pemasok.match(this.form.pemSel)
      })
    },
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
          .delete(this.apiurl + `/delete-barang/${item.id}`)
          .then(() => {
            this.loadBarang()
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
            .put(this.apiurl + '/update-barang/' + form.id, {
              'namaBarang': this.form.namaBarang,
              'hargaBeli': this.form.hargaBeli,
              'hargaJual': this.form.hargaJual,
              'pemSel': this.form.pemSel,
              'jumlahBarang': this.form.jumlahBarang,
              'tanggalMasuk': this.form.tanggalMasuk,
              'deskripsi': this.form.deskripsi,
            })
            .then(() => {
              this.loadBarang()
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
      this.form.namaBarang = this.dtedit.nama_barang
      this.form.hargaBeli = this.dtedit.harga_beli
      this.form.hargaJual = this.dtedit.harga_jual
      this.form.pemSel = this.dtedit.kode_pemasok
      this.form.jumlahBarang = this.dtedit.jumlah_barang
      this.form.tanggalMasuk = this.dtedit.tanggal_masuk
      this.form.deskripsi = this.dtedit.deskripsi
      this.isKode = true
    },
    save() {
      if (this.form.namaBarang == null || this.form.hargaBeli == null || this.form.hargaJual == null || this.form.pemSel == null || this.form.jumlahBarang == null || this.form.tanggalMasuk == null || this.form.deskripsi == null) {
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
            .post(this.apiurl + '/insert-barang', this.form)
            .then(() => {
              this.loadBarang()
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
    loadBarang(){
      this.$axios.get(this.apiurl + '/load-barang')
      .then((ress)=>{
        this.dtbarangs = ress.data.data
      })
      .catch(() => {
        console.log(err);
      })
    },
    loadPemasoks(){
      this.$axios.get(this.apiurl + '/load-pemasok-barang')
      .then((ress)=>{
        this.dtpemasoks = ress.data.data
      })
      .catch(() => {
        console.log(err);
      })
    },
    loadKodes(){
      this.$axios.get(this.apiurl + '/load-kode-pemasok')
      .then((ress)=>{
        this.dtkodes = ress.data.data
      })
      .then(()=>{
        for (let i = 0; i < this.dtkodes.length; i++) {
          this.pemItm.push(this.dtkodes[i].kode_pemasok)
        }
      })
      .catch(() => {
        console.log(err);
      })
    },
    batal() {
      this.isKode = false
      this.$refs.form.reset()
      this.updateSubmit = false
    },
    baru() {
      this.isKode = false
      this.$refs.form.reset()
      document.getElementById('nama-barang').focus()
      this.updateSubmit = false
    },
    tanggalMasuk(){
      document.getElementById('deskripsi').focus()
    },
    jumlahBarang(){
      document.getElementById('tanggal-masuk').focus()
    },
    kodePemasok(){
      this.isKode = true
      document.getElementById('jumlah-barang').focus()
    },
    hargaJual() {
      document.getElementById('kode-pemasok').focus()
    },
    hargaBeli() {
      document.getElementById('harga-jual').focus()
    },
    namaBarang() {
      document.getElementById('harga-beli').focus()
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
    this.loadKodes()
    this.loadPemasoks()
    this.loadBarang()
  }
}
</script>
