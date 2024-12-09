<template>
  <div>
    <h1>Faktur</h1>
    <div class="row">
      <div class="col">
        <v-col>
          <v-row>
            <v-text-field v-model="form.kodePelanggan" id="kode-pelanggan" label="Kode Pelanggan" required append-icon="mdi-magnify"
            @change="isiPelanggan" @keydown.enter="kodePelanggan"></v-text-field>
          </v-row>
          <div>
            <v-row>
              <v-text-field v-model="form.namaPelanggan" label="Nama Pelanggan" disabled></v-text-field>
            </v-row>
            <v-row>
              <v-text-field v-model="form.alamatPelanggan" label="Alamat Pelanggan" disabled></v-text-field>
            </v-row>
            <v-row>
              <v-text-field v-model="form.kotaPelanggan" label="Kota Pelanggan" disabled></v-text-field>
            </v-row>
            <v-row>
              <v-text-field v-model="form.noTelp" label="No. Telp. Pelanggan" disabled></v-text-field>
            </v-row>
          </div>
        </v-col>
      </div>
      <div class="col">

      </div>
      <div class="col">
        <v-col>
          <v-row>
            <v-text-field v-model="form.noFaktur" label="No. Faktur" disabled></v-text-field>
          </v-row>
          <v-row>
            <v-text-field v-model="form.tanggalJual" id="tanggal-jual" @keydown.enter="tanggalJual" type="date" label="Tanggal Jual"></v-text-field>
          </v-row>
          <v-row>
            <v-text-field v-model="form.tanggalJatuhTempo" id="tanggal-jatuh-tempo" @keydown.enter="tanggalJatuhTempo" type="date" label="Tanggal jatuh tempo"></v-text-field>
          </v-row>
          <v-row>
            <v-text-field v-model="form.keteranganBayar" id="keterangan-bayar" @keydown.enter="keteranganBayar" label="Ketarangan Bayar"></v-text-field>
          </v-row>
        </v-col>
      </div>
    </div>
    <v-container v-show="fieldhiden">
      <v-simple-table>
        <template>
          <thead>
            <tr>
              <th>No.</th>
              <th>Kode Barang</th>
              <th>Nama Barang</th>
              <th>Harga</th>
              <th>Jumlah</th>
              <th>Total</th>
              <th>ACT</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, i) in kdbrg" :key="item.id">
              <td>{{ i + 1 }}</td>
              <td>{{ item.kodebarang }}</td>
              <td>{{ item.namaBarang }}</td>
              <td>{{ item.hargaJual }}</td>
              <td>{{ item.jumlah }}</td>
              <td>{{ item.total }}</td>
              <td>
                <v-btn @click="hapusbrg(item.index)" class="mx-2" fab dark small color="primary">
                  <v-icon dark>
                    mdi-minus
                  </v-icon>
                </v-btn>
              </td>
            </tr>

            <tr>
              <td></td>
              <td>
                <v-text-field label="Kode Barang" id="kode-barang" v-model="barang.kodeBarang" @keydown.enter="kodeBarang" append-icon="mdi-magnify"></v-text-field>
              </td>
              <td>
                <v-text-field label="Nama Barang" id="nama-barang" v-model="barang.namaBarang" disabled></v-text-field>
              </td>
              <td>
                <v-text-field label="Harga Barang" id="harga-jual" v-model="barang.hargaJual" disabled></v-text-field>
              </td>
              <td>
                <v-text-field label="Jumlah Barang" id="jumlah-barang" v-model="barang.jumlah" @keydown.enter="additem"
                  type="number"></v-text-field>
              </td>
            </tr>

          </tbody>
        </template>
      </v-simple-table>
      <v-row>
        <v-col cols="12" md="9"></v-col>
        <v-col cols="12" md="3">
          <v-text-field label="Total Belanja" id="total-belanja" v-model="form.totalBelanja" disabled></v-text-field>
        </v-col>
      </v-row>
    </v-container>
    <v-container>
      <v-row>
        <v-col class="d-flex flex-row mb-6 bg-surface-variant mt-5" cols="12" ms="6" md="6">
          <v-btn class="mr-4" color="success" width="150px" @click="baru">baru</v-btn>
          <v-btn class="mr-4" width="150px" color="primary" type="button" name="button" v-show="!updateSubmit"
            @click="save" :disabled="dialog" :loading="dialog">simpan</v-btn>
          <v-btn color="error" class="mr-4" width="150px" @click="batal">batal</v-btn>
        </v-col>
      </v-row>
      <div>
        <h2>Data Penjualan</h2>
        <v-text-field v-model="cari" append-icon="mdi-magnify" label="Pencarian" single-line></v-text-field>
        <v-data-table :headers="headpenjualan" :items="dtpenjualan" :search="cari" :items-per-page="5" class="elevation-1">
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
  </div>
</template>

<script>
import Swal from 'sweetalert2'
export default {
  data(){
    return{
      apiurl: "http://localhost:8000/api",
      dialog: false,
      cari: "",
      dtpenjualan: [],
      itemPelanggan: [],
      itemBarang:[],
      kdbrg: [],
      fieldhiden: false,
      headpenjualan: [
        { text: "No", value: 'id' },
        { text: "Nama Pelanggan", value: "nama_pelanggan" },
        { text: "Alamat", value: "alamat_pelanggan" },
        { text: "Kota", value: "kota_pelanggan" },
        { text: "No Telp", value: "no_telp" },
        { text: "No Faktur", value: "no_faktur" },
        { text: "Tanggal Jual", value: "tanggal_jual" },
        { text: "Tanggal Jatuh Tempo", value: "tanggal_jatuh_tempo" },
        { text: "Keterangan", value: "keterangan_bayar" }
      ],
      item: {
        kodeBarang: "",
        namaBarang: "",
        hargaJual: "",
        jumlah: "",
      },
      barang: {
        kodeBarang: "",
        namaBarang: "",
        hargaJual: "",
        jumlah: "",
      },
      form: {
        kodePelanggan: "",
        namaPelanggan: "",
        alamatPelanggan: "",
        kotaPelanggan: "",
        noTelp: "",
        noFaktur: "",
        tanggalJual: "",
        tanggalJatuhTempo: "",
        keteranganBayar: "",
        totalBelanja: ""
      }
    }
  },
  computed:{
    filterBarang: function () {
      return this.itemBarang.filter((ib) => {
        return ib.kode_barang.match(this.barang.kodeBarang)
      })
    },
    filterPelanggan: function () {
      return this.itemPelanggan.filter((ip) => {
        return ip.kode_pelanggan.match(this.form.kodePelanggan)
      })
    },
  },
  watch:{
    dialog(val){
      if (!val) return
      setTimeout(()=>(this.dialog = false), 4000)
    }
  },
  methods:{
    save(){
      let objek = {
        penjualans : this.kdbrg,
        penjualanDetails : this.form
      }
      if (this.form.kodePelanggan == 0 || this.form.totalBelanja == 0) {
        alert("Data Salah")
      } else {
        Swal.fire({
          title: 'Anda Yakin?',
          text: "Pastikan Data Sudah Benar",
          icon: 'warning',
          showCancelButton: true,
          confirmButtonColor: '#3085d6',
          cancelButtonColor: '#d33',
          confirmButtonText: 'Yes'
        })
          .then((result) => {
            if (result.isConfirmed) {
              this.dialog = true
              this.$axios.post(this.apiurl + '/faktur/save-faktur', objek)
              .then(() =>{
                this.batal()
                this.loadPenjualan()
                Swal.fire({
                  position: "top-end",
                  icon: "success",
                  title: "Data berhasil disimpan",
                  showConfirmButton: false,
                  timer: 1500
                });
              })
            }
          })
      }
    },
    batal(){
      this.kdbrg.splice(0)
      this.fieldhiden = false
      this.item.kodeBarang = ""
      this.item.namaBarang = ""
      this.item.hargaJual = ""
      this.item.jumlah = ""
      this.barang.kodeBarang = ""
      this.barang.namaBarang = ""
      this.barang.hargaJual = ""
      this.barang.jumlah = ""
      this.form.kodePelanggan = ""
      this.form.namaPelanggan = ""
      this.form.alamatPelanggan = ""
      this.form.kotaPelanggan = ""
      this.form.noTelp = ""
      this.form.noFaktur = ""
      this.form.tanggalJual = ""
      this.form.tanggalJatuhTempo = ""
      this.form.keteranganBayar = ""
      this.form.totalBelanja = ""
      this.loadPelanggan()
      this.loadBarang()
    },
    baru(){
      document.getElementById('kode-pelanggan').focus()
      this.kdbrg.splice(0)
      this.fieldhiden = false
      this.item.kodeBarang = ""
      this.item.namaBarang = ""
      this.item.hargaJual = ""
      this.item.jumlah = ""
      this.barang.kodeBarang = ""
      this.barang.namaBarang = ""
      this.barang.hargaJual = ""
      this.barang.jumlah = ""
      this.form.kodePelanggan = ""
      this.form.namaPelanggan = ""
      this.form.alamatPelanggan = ""
      this.form.kotaPelanggan = ""
      this.form.noTelp = ""
      this.form.noFaktur = ""
      this.form.tanggalJual = ""
      this.form.tanggalJatuhTempo = ""
      this.form.keteranganBayar = ""
      this.form.totalBelanja = ""
      this.loadPelanggan()
      this.loadBarang()
    },
    hapusbrg(index) {
      let idx = this.kdbrg.findIndex(x => x.index === index)
      this.kdbrg.splice(idx, 1);
      this.hitungTotalBelanja()
    },
    kodeBarang() {
      this.filterBarang.forEach((fb) => {
        this.barang.namaBarang = fb.nama_barang
        this.barang.hargaJual = fb.harga_jual
      })
      document.getElementById('jumlah-barang').focus()
    },
    ambilBrg(){
      document.getElementById('kode-barang').focus()
    },
    keteranganBayar(){
      this.fieldhiden = true
      this.ambilBrg()
    },
    hitungTotalBelanja(){
      var tl = 0
      this.kdbrg.forEach(e => {
          tl += e.total
      });
      this.form.totalBelanja = tl
    },
    additem() {
      if (this.barang.jumlah == "" || this.barang.jumlah == 0) {
        alert('jumlah kosong')
      } else {
        this.pushobject();
        this.barang.kodeBarang = ""
        this.barang.namaBarang = ""
        this.barang.hargaJual = ""
        this.barang.jumlah = ""
        this.hitungTotalBelanja()
      }
    },
    pushobject() {
      // ----------------push action-------------------
      let idx = this.kdbrg.length
      let ttl = (this.barang.hargaJual * this.barang.jumlah)
      var my_object = {
        index: idx,
        noFaktur: this.form.noFaktur,
        kodeBarang: this.barang.kodeBarang,
        namaBarang: this.barang.namaBarang,
        hargaJual: this.barang.hargaJual,
        jumlah: this.barang.jumlah,
        total: ttl,
      };
      this.kdbrg.push(my_object)
      document.getElementById('kode-barang').focus()
      // --------------------
    },
    tanggalJatuhTempo(){
      document.getElementById('keterangan-bayar').focus()
    },
    tanggalJual(){
      document.getElementById('tanggal-jatuh-tempo').focus()
    },
    kodePelanggan(){
      document.getElementById('tanggal-jual').focus()
    },
    isiNoFaktur(){
      var date = new Date()
      var year = date.getFullYear()
      var month = date.getMonth()
      var day = date.getDay()
      var hours = date.getHours()
      var minutes = date.getMinutes()
      var seconds = date.getSeconds()
      var miliseconds = date.getMilliseconds()
      this.form.noFaktur = "FK"+year+month+day+hours+minutes+seconds+miliseconds
    },
    isiPelanggan() {
      this.filterPelanggan.forEach((fp) => {
        this.form.namaPelanggan = fp.nama_pelanggan
        this.form.alamatPelanggan = fp.alamat_pelanggan
        this.form.kotaPelanggan = fp.kota_pelanggan
        this.form.noTelp = fp.no_telp_pelanggan
      })
      this.isiNoFaktur()
    },
    loadBarang(){
      this.$axios.get(this.apiurl + '/faktur/load-barang')
      .then((ress)=>{
        this.itemBarang = ress.data.data
      })
      .catch(() => {
        console.log(err);
      })
    },
    loadPelanggan(){
      this.$axios.get(this.apiurl + '/faktur/load-pelanggan')
      .then((ress)=>{
        this.itemPelanggan = ress.data.data
      })
      .catch(() => {
        console.log(err);
      })
    },
    loadPenjualan(){
      this.$axios.get(this.apiurl + '/faktur/load-penjualan')
      .then((ress)=>{
        this.dtpenjualan = ress.data.data
      })
      .catch(() => {
        console.log(err);
      })
    },
  },
  mounted() {
    this.loadPelanggan()
    this.loadBarang()
    this.loadPenjualan()
  }
}
</script>
