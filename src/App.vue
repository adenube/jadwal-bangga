<template>
  <div class="app-container">
    
    <div v-if="!isLoggedIn" class="login-wrapper fade-in">
      <div class="login-card modern-login">
        <div class="login-header">
          <div class="logo-circle">📅</div>
          <h1>SmartSchedule</h1>
          <p>Sistem Manajemen Jadwal Terpadu</p>
        </div>
        <div class="login-body">
          <div class="form-group">
            <input 
              v-model="loginInput" 
              @keyup.enter="doLogin"
              type="text" 
              maxlength="4"
              class="login-input-modern" 
              placeholder="KODE GURU"
              autocomplete="off"
            >
          </div>
          <button @click="doLogin" class="btn-login-modern" :disabled="isLoadingLogin">
            {{ isLoadingLogin ? 'MEMERIKSA...' : 'MASUK' }}
          </button>
        </div>
      </div>
    </div>

    <template v-else>
      
      <div v-if="isSidebarOpen" class="sidebar-overlay" @click="isSidebarOpen = false"></div>
      <aside class="sidebar" :class="{ 'open': isSidebarOpen }">
        <div class="sidebar-header">
          <h2>NAVIGASI</h2>
          <button @click="isSidebarOpen = false" class="btn-close-sidebar">✕</button>
        </div>
        <div class="sidebar-menu">
          <template v-if="currentUser.role === 'admin'">
            <div class="menu-label">Jadwal Harian</div>
            <button v-for="hari in ['Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat']" :key="hari" @click="pilihMenu(hari)" :class="{'aktif': menuAktif === hari}">
              <span class="menu-icon">📅</span> Hari {{ hari }}
            </button>
            
            <div class="menu-label">Master Data</div>
            <button @click="pilihMenu('Daftar Guru')" :class="{'aktif': menuAktif === 'Daftar Guru'}">
              <span class="menu-icon">👨‍🏫</span> Daftar Guru
            </button>
            <button @click="pilihMenu('Rombel Kelas')" :class="{'aktif': menuAktif === 'Rombel Kelas'}">
              <span class="menu-icon">🏫</span> Rombel Kelas
            </button>
            <button @click="pilihMenu('Pengaturan Jam')" :class="{'aktif': menuAktif === 'Pengaturan Jam'}">
              <span class="menu-icon">⏱️</span> Pengaturan Waktu
            </button>
            
            <div class="menu-label">Laporan & Ekspor</div>
            <button @click="pilihMenu('Rekap Evaluasi')" :class="{'aktif': menuAktif === 'Rekap Evaluasi'}">
              <span class="menu-icon">📊</span> Rekap Evaluasi
            </button>
            <button @click="pilihMenu('Pengaturan Cetak')" :class="{'aktif': menuAktif === 'Pengaturan Cetak'}">
              <span class="menu-icon">🖨️</span> Pengaturan Cetak
            </button>
          </template>
          <template v-else>
            <div class="menu-label">Menu Guru</div>
            <button @click="pilihMenu('Dashboard Saya')" :class="{'aktif': menuAktif === 'Dashboard Saya'}">
              <span class="menu-icon">🏠</span> Dashboard Personal
            </button>
          </template>
        </div>
      </aside>

      <header class="navbar-modern">
        <div class="nav-left">
          <button @click="isSidebarOpen = true" class="btn-hamburger">☰</button>
          <div class="brand">
            <span class="logo-icon">📅</span>
            <div class="brand-text">
              <h1>SmartSchedule</h1>
              <p>SMP Mode</p>
            </div>
          </div>
        </div>

        <div class="user-profile">
          <div class="user-info">
            <span class="user-name">Halo, <strong>{{ currentUser.nama }}</strong> ({{ currentUser.kode }})</span>
            <span class="user-role badge-micro" :class="currentUser.role === 'admin' ? 'bg-danger-light text-danger' : 'bg-light border-subtle'">
              {{ currentUser.role.toUpperCase() }}
            </span>
          </div>
          <button @click="doLogout" class="btn-outline btn-sm">Logout</button>
        </div>
      </header>

      <div v-if="menuAktif === 'Dashboard Saya'" class="page-content fade-in">
        <div class="panel full-height-panel" style="max-width: 800px; margin: 0 auto;">
          <div class="panel-header compact-header">
            <h2 class="section-title">Dashboard Personal: {{ currentUser.nama }}</h2>
          </div>
          <div class="table-responsive" style="padding: 20px;">
            
            <div class="info-banner" style="margin-bottom: 20px; border-radius: 6px;">
              👨‍🏫 Berikut adalah rincian penugasan mengajar Anda. Pastikan jumlah total JP sesuai dengan Target SK.
            </div>

            <h3 style="font-size: 0.85rem; color: #0f172a; margin-bottom: 8px;">1. Distribusi Beban Harian</h3>
            <table class="minimal-table text-center-table" style="margin-bottom: 30px;">
              <thead>
                <tr><th>SENIN</th><th>SELASA</th><th>RABU</th><th>KAMIS</th><th>JUMAT</th><th class="bg-light">TOTAL JP</th><th>TARGET SK</th></tr>
              </thead>
              <tbody>
                <tr v-if="myRekap">
                  <td><span :class="{'text-danger fw-bold': myRekap.Senin > 6, 'text-muted': myRekap.Senin === 0}">{{ myRekap.Senin }}</span></td>
                  <td><span :class="{'text-danger fw-bold': myRekap.Selasa > 6, 'text-muted': myRekap.Selasa === 0}">{{ myRekap.Selasa }}</span></td>
                  <td><span :class="{'text-danger fw-bold': myRekap.Rabu > 6, 'text-muted': myRekap.Rabu === 0}">{{ myRekap.Rabu }}</span></td>
                  <td><span :class="{'text-danger fw-bold': myRekap.Kamis > 6, 'text-muted': myRekap.Kamis === 0}">{{ myRekap.Kamis }}</span></td>
                  <td><span :class="{'text-danger fw-bold': myRekap.Jumat > 6, 'text-muted': myRekap.Jumat === 0}">{{ myRekap.Jumat }}</span></td>
                  <td class="bg-light"><strong :class="{'text-danger': myRekap.Total > myRekap.target_jam}">{{ myRekap.Total }}</strong></td>
                  <td><span class="badge-micro bg-success-light text-success">{{ myRekap.target_jam }}</span></td>
                </tr>
              </tbody>
            </table>

            <h3 style="font-size: 0.85rem; color: #0f172a; margin-bottom: 8px;">2. Rincian Jadwal Mengajar Kelas</h3>
            <table class="minimal-table">
              <thead><tr><th style="width: 80px;">JAM KE</th><th style="width: 150px;">RENTANG WAKTU</th><th>KELAS MENGAJAR</th></tr></thead>
              <tbody>
                <template v-for="(jadwals, hari) in groupedJadwalGuru" :key="hari">
                  <tr class="mapel-group-header">
                    <td colspan="3">
                      <div class="mapel-group-title">
                        <span class="mapel-name">{{ hari.toUpperCase() }}</span>
                        <span class="badge-micro bg-light text-muted border-subtle">{{ jadwals.length }} JP</span>
                      </div>
                    </td>
                  </tr>
                  <tr v-for="item in jadwals" :key="item.id">
                    <td><span class="badge-micro bg-light text-dark border-subtle">Jam {{ item.jam_ke }}</span></td>
                    <td class="text-muted" style="font-family: monospace;">{{ item.waktu }}</td>
                    <td>
                      <span class="assigned-card-clean" :class="getCardColorClass(item.rombel)" style="display: inline-flex; width: max-content; padding: 3px 12px; font-weight: 700; font-size: 0.75rem;">
                        {{ item.rombel }}
                      </span>
                    </td>
                  </tr>
                </template>
                <tr v-if="Object.keys(groupedJadwalGuru).length === 0">
                  <td colspan="3" class="empty-state" style="padding: 2rem;">Belum ada jadwal mengajar yang dialokasikan untuk Anda.</td>
                </tr>
              </tbody>
            </table>

          </div>
        </div>
      </div>

      <template v-if="currentUser.role === 'admin'">
        
        <div v-if="menuAktif === 'Rekap Evaluasi'" class="page-content fade-in">
          <div class="panel full-height-panel">
            <div class="panel-header compact-header" style="flex-direction: column; align-items: flex-start; gap: 10px;">
              <div style="display: flex; justify-content: space-between; width: 100%; align-items: center;">
                <h2 class="section-title">Pusat Evaluasi & Audit Jadwal</h2>
                <button @click="generateJadwalCerdas()" class="btn-magic btn-sm">✨ Auto-Draft Sesuai SK</button>
              </div>
              
              <div class="sub-nav-pills">
                <button @click="subRekap = 'audit'" :class="{ 'aktif': subRekap === 'audit' }">Audit Kekurangan SK</button>
                <button @click="subRekap = 'harian'" :class="{ 'aktif': subRekap === 'harian' }">Distribusi Beban Harian</button>
                <button @click="subRekap = 'jadwal'" :class="{ 'aktif': subRekap === 'jadwal' }">Jadwal Per Guru</button>
              </div>
            </div>

            <div v-if="subRekap === 'audit'">
              <div class="info-banner">💡 <strong>Insight:</strong> Tabel ini menyoroti guru yang jam mengajarnya masih di bawah target SK.</div>
              <div class="table-responsive">
                <table class="minimal-table">
                  <thead>
                    <tr><th style="width: 60px;">KODE</th><th>NAMA GURU</th><th>TARGET</th><th>TERJADWAL</th><th>STATUS</th><th>RINCIAN KELAS (BELUM TERPENUHI)</th></tr>
                  </thead>
                  <tbody>
                    <template v-for="(guruList, mapel) in groupedRekapKekurangan" :key="mapel">
                      <tr class="mapel-group-header">
                        <td colspan="6"><div class="mapel-group-title"><span class="mapel-name">{{ mapel.toUpperCase() }}</span><span class="badge-micro bg-danger-light text-danger">{{ guruList.length }} Guru Kurang Jam</span></div></td>
                      </tr>
                      <tr v-for="rekap in guruList" :key="rekap.id">
                        <td><span class="fw-bold text-primary">{{ rekap.kode }}</span></td>
                        <td><strong class="text-dark">{{ rekap.nama }}</strong></td>
                        <td class="text-muted">{{ rekap.target_jam }} JP</td>
                        <td><span class="badge-micro bg-success-light text-success">{{ rekap.totalJam }} JP</span></td>
                        <td><span class="badge-micro bg-danger-light text-danger">Kurang {{ rekap.totalKurang }} JP</span></td>
                        <td><div class="assigned-badges"><span v-for="(detail, idx) in rekap.detailKurang" :key="idx" class="badge-micro bg-warning-light text-warning">{{ detail }}</span></div></td>
                      </tr>
                    </template>
                    <tr v-if="rekapKekurangan.length === 0 && teachers.length > 0"><td colspan="6" class="empty-state"><h3>Aman Terkendali!</h3><p>Seluruh jadwal guru sudah terpenuhi sesuai Target SK.</p></td></tr>
                  </tbody>
                </table>
              </div>
            </div>

            <div v-if="subRekap === 'harian'">
              <div class="info-banner">💡 <strong>Tips:</strong> Angka <span class="text-danger fw-bold">merah</span> menandakan beban lebih dari 6 JP dalam sehari.</div>
              <div class="table-responsive">
                <table class="minimal-table text-center-table">
                  <thead>
                    <tr><th style="text-align: left; width: 60px;">KODE</th><th style="text-align: left;">NAMA GURU</th><th>SENIN</th><th>SELASA</th><th>RABU</th><th>KAMIS</th><th>JUMAT</th><th class="bg-light">TOTAL</th><th>SK</th></tr>
                  </thead>
                  <tbody>
                    <template v-for="(guruList, mapel) in groupedRekapHarian" :key="mapel">
                      <tr class="mapel-group-header"><td colspan="9" style="text-align: left;"><div class="mapel-group-title"><span class="mapel-name">{{ mapel.toUpperCase() }}</span></div></td></tr>
                      <tr v-for="rekap in guruList" :key="rekap.kode">
                        <td style="text-align: left;"><span class="fw-bold text-primary">{{ rekap.kode }}</span></td>
                        <td style="text-align: left;"><strong class="text-dark">{{ rekap.nama }}</strong></td>
                        <td><span :class="{'text-danger fw-bold': rekap.Senin > 6, 'text-muted': rekap.Senin === 0}">{{ rekap.Senin }}</span></td>
                        <td><span :class="{'text-danger fw-bold': rekap.Selasa > 6, 'text-muted': rekap.Selasa === 0}">{{ rekap.Selasa }}</span></td>
                        <td><span :class="{'text-danger fw-bold': rekap.Rabu > 6, 'text-muted': rekap.Rabu === 0}">{{ rekap.Rabu }}</span></td>
                        <td><span :class="{'text-danger fw-bold': rekap.Kamis > 6, 'text-muted': rekap.Kamis === 0}">{{ rekap.Kamis }}</span></td>
                        <td><span :class="{'text-danger fw-bold': rekap.Jumat > 6, 'text-muted': rekap.Jumat === 0}">{{ rekap.Jumat }}</span></td>
                        <td class="bg-light"><strong :class="{'text-danger': rekap.Total > rekap.target_jam}">{{ rekap.Total }}</strong></td>
                        <td><span class="badge-micro bg-success-light text-success">{{ rekap.target_jam }}</span></td>
                      </tr>
                    </template>
                    <tr v-if="Object.keys(groupedRekapHarian).length === 0"><td colspan="9" class="empty-state">Belum ada data.</td></tr>
                  </tbody>
                </table>
              </div>
            </div>

            <div v-if="subRekap === 'jadwal'">
              <div class="info-banner" style="display: flex; gap: 15px; align-items: center; border-bottom: none;">
                <strong style="white-space: nowrap; color: #0f172a;">Pilih Guru:</strong>
                <select v-model="selectedGuruJadwal" class="input-clean" style="max-width: 300px; padding: 6px 10px;">
                  <option value="">-- Pilih Guru --</option>
                  <option v-for="guru in teachers" :key="guru.kode" :value="guru.kode">{{ guru.kode }} - {{ guru.nama }}</option>
                </select>
              </div>

              <div class="table-responsive" v-if="selectedGuruJadwal">
                <table class="minimal-table" style="max-width: 600px;">
                  <thead><tr><th style="width: 80px;">JAM KE</th><th style="width: 150px;">RENTANG WAKTU</th><th>KELAS MENGAJAR</th></tr></thead>
                  <tbody>
                    <template v-for="(jadwals, hari) in groupedJadwalGuru" :key="hari">
                      <tr class="mapel-group-header"><td colspan="3"><div class="mapel-group-title"><span class="mapel-name">{{ hari.toUpperCase() }}</span><span class="badge-micro bg-light text-muted border-subtle">{{ jadwals.length }} JP</span></div></td></tr>
                      <tr v-for="item in jadwals" :key="item.id">
                        <td><span class="badge-micro bg-light text-dark border-subtle">Jam {{ item.jam_ke }}</span></td>
                        <td class="text-muted" style="font-family: monospace;">{{ item.waktu }}</td>
                        <td><span class="assigned-card-clean" :class="getCardColorClass(item.rombel)" style="display: inline-flex; width: max-content; padding: 2px 12px; font-weight: 700; font-size: 0.7rem;">{{ item.rombel }}</span></td>
                      </tr>
                    </template>
                    <tr v-if="Object.keys(groupedJadwalGuru).length === 0"><td colspan="3" class="empty-state" style="padding: 2rem;">Guru ini belum memiliki jadwal mengajar.</td></tr>
                  </tbody>
                </table>
              </div>
              <div v-else class="empty-state" style="padding: 4rem;">Silakan pilih nama guru pada kotak pilihan di atas untuk melihat rincian jadwalnya.</div>
            </div>
          </div>
        </div>

        <div v-else-if="menuAktif === 'Daftar Guru'" class="page-content fade-in">
          <div class="panel full-height-panel">
            <div class="panel-header compact-header">
              <h2 class="section-title">Kelola Penugasan Guru</h2>
              <div class="action-buttons">
                <button @click="bukaModalImportGuru()" class="btn-outline btn-sm">Import Excel</button>
                <button @click="bukaModalGuru()" class="btn-primary btn-sm">Tambah Guru</button>
              </div>
            </div>
            <div class="table-responsive">
              <table class="minimal-table">
                <thead>
                  <tr><th style="width: 60px;">KODE</th><th>NAMA & MAPEL</th><th style="width: 80px;">DURASI</th><th>PENUGASAN KELAS</th><th style="width: 80px;">SK</th><th style="width: 80px;">AKSI</th></tr>
                </thead>
                <tbody>
                  <template v-for="(guruList, mapel) in groupedTeachers" :key="mapel">
                    <tr class="mapel-group-header"><td colspan="6"><div class="mapel-group-title"><span class="mapel-name">{{ mapel }}</span></div></td></tr>
                    <tr v-for="guru in guruList" :key="guru.id">
                      <td><span class="fw-bold text-primary">{{ guru.kode }}</span></td>
                      <td><strong class="text-dark" style="display: block;">{{ guru.nama }}</strong><span class="text-muted" style="font-size: 0.65rem;">{{ guru.mapel }}</span></td>
                      <td class="text-sm text-muted">{{ guru.durasi_mapel }} JP</td>
                      <td>
                        <div class="assigned-badges">
                          <span v-for="tugas in getPenugasanGuru(guru.kode)" :key="tugas.id" class="badge-micro bg-light text-dark border-subtle">{{ tugas.nama_rombel }}</span>
                          <button @click="bukaModalAturKelas(guru)" class="btn-add-mini">+</button>
                        </div>
                      </td>
                      <td><strong class="text-dark">{{ guru.target_jam }} JP</strong></td>
                      <td><button @click="bukaModalGuru(guru)" class="btn-icon">✏️</button><button @click="hapusGuru(guru.id)" class="btn-icon text-danger">🗑️</button></td>
                    </tr>
                  </template>
                </tbody>
              </table>
            </div>
          </div>
        </div>

        <div v-else-if="menuAktif === 'Pengaturan Cetak'" class="page-content fade-in">
          <div class="panel full-height-panel" style="max-width: 800px; margin: 0 auto;">
            <div class="panel-header compact-header">
              <h2 class="section-title">Format Cetak & Download Excel</h2>
              <div class="action-buttons">
                <button @click="exportKeExcel()" class="btn-primary btn-sm" style="background-color: #16a34a;">📥 Download Excel</button>
              </div>
            </div>
            <div class="table-responsive" style="padding: 20px;">
              <div class="info-banner" style="margin-bottom: 20px; border-radius: 8px;">💡 <strong>Keterangan:</strong> Data di bawah ini akan digunakan sebagai Kop Surat dan Tanda Tangan pada file Excel yang didownload.</div>
              <div class="form-group"><label>Nama Sekolah</label><input v-model="pengaturanCetak.namaSekolah" class="input-clean"></div>
              <div class="form-group"><label>Tahun Pelajaran</label><input v-model="pengaturanCetak.tahunPelajaran" class="input-clean"></div>
              <div class="form-group"><label>Tanggal Mulai Berlaku</label><input v-model="pengaturanCetak.tanggalBerlaku" class="input-clean"></div>
              <hr style="border-top: 1px solid #e2e8f0; margin: 20px 0;">
              <div class="form-group"><label>Tempat & Tanggal TTD</label><input v-model="pengaturanCetak.tempatTanggal" class="input-clean"></div>
              <div class="form-group"><label>Nama Kepala Sekolah</label><input v-model="pengaturanCetak.namaKepsek" class="input-clean"></div>
              <div class="form-group"><label>NIP Kepala Sekolah</label><input v-model="pengaturanCetak.nipKepsek" class="input-clean"></div>
              <div style="margin-top: 20px;"><button @click="simpanPengaturanCetak()" class="btn-primary" style="width: 100%;">💾 Simpan Pengaturan Cetak</button></div>
            </div>
          </div>
        </div>

        <div v-else-if="['Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat'].includes(menuAktif)" class="main-layout fade-in full-width-layout">
          <main class="schedule-area panel full-width-panel border-0">
            <div class="panel-header grid-header compact-header">
              <h3 class="section-title">Jadwal: {{ menuAktif }}</h3>
              <div class="action-buttons">
                <button @click="resetSemuaJadwal()" class="btn-outline text-danger border-danger btn-sm">Kosongkan</button>
                <button @click="generateJadwalCerdas()" class="btn-magic btn-sm">✨ Auto-Draft</button>
                <button @click="exportKeExcel()" class="btn-primary btn-sm" style="background-color: #16a34a; border-color: #16a34a;">📥 Excel</button>
              </div>
            </div>

            <div class="top-teacher-pool-container">
              <div class="pool-header text-muted fw-bold">DAFTAR GURU (DRAG KE SLOT BAWAH)</div>
              <div class="grouped-pools-container">
                <template v-for="(guruList, mapel) in groupedTeachers" :key="mapel">
                  <div class="mapel-pool-group">
                    <div class="mapel-pool-label">{{ mapel }}</div>
                    <div class="top-teacher-pool">
                      <div v-for="element in guruList" :key="element.id" class="teacher-pill" draggable="true" @dragstart="startDrag($event, element.kode)" title="Tarik ke jadwal">
                        <span class="kode-fw">{{ element.kode }}</span>
                        <span class="jp-badge-mini" :class="{ 'text-success': element.totalJam === element.target_jam, 'text-danger': element.totalJam > element.target_jam, 'text-muted': element.totalJam < element.target_jam }">
                          {{ element.totalJam }}/{{ element.target_jam }}
                        </span>
                      </div>
                    </div>
                  </div>
                </template>
              </div>
            </div>
            
            <div class="schedule-scroll">
              <table class="grid-table-clean">
                <thead>
                  <tr>
                    <th class="sticky-col th-waktu">WAKTU</th>
                    <th v-for="rombel in rombels" :key="rombel" :class="getKelasColorClass(rombel)">{{ rombel }}</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="sesi in getJadwalHariIni()" :key="sesi.id">
                    <td class="time-label sticky-col">
                      <div class="time-content">
                        <span v-if="sesi.tipe === 'pelajaran'" class="jam-ke-clean">JAM {{ sesi.jam_ke }}</span>
                        <span class="waktu-teks-clean">{{ sesi.waktu }}</span>
                      </div>
                    </td>
                    <td v-if="sesi.tipe === 'khusus'" :colspan="rombels.length" class="special-row-clean">
                      <span class="special-text-clean">{{ sesi.nama_kegiatan }}</span>
                    </td>
                    <td v-else v-for="rombel in rombels" :key="rombel" class="slot-clean" @dragover.prevent @dragenter.prevent @drop="onDrop($event, rombel, sesi.jam_ke)">
                      <div class="drop-zone-clean">
                        <div v-for="element in getScheduleData(rombel, sesi.jam_ke)" :key="element.id" class="assigned-card-clean" :class="getCardColorClass(rombel)">
                          <span class="assigned-kode-clean">{{ element.guru_mapel.split(' - ')[0] }}</span>
                          <button @click="removeJadwal(element.id)" class="btn-remove-clean">×</button>
                        </div>
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </main>
        </div>

        <div v-else-if="menuAktif === 'Pengaturan Jam'" class="page-content fade-in">
          <div class="panel full-height-panel">
            <div class="panel-header compact-header">
              <h2 class="section-title">Kelola Jam & Waktu</h2>
              <div class="action-buttons"><button @click="bukaModalImportWaktu()" class="btn-outline btn-sm">Import Excel</button><button @click="bukaModalWaktu()" class="btn-primary btn-sm">Tambah Jam</button></div>
            </div>
            <div class="table-responsive">
              <table class="minimal-table">
                <thead><tr><th>HARI</th><th>WAKTU</th><th>TIPE</th><th>KEGIATAN/JAM</th><th>AKSI</th></tr></thead>
                <tbody><tr v-for="waktu in masterWaktu" :key="waktu.id"><td><strong class="text-dark">{{ waktu.hari }}</strong></td><td>{{ waktu.waktu }}</td><td><span class="text-muted">{{ waktu.tipe }}</span></td><td><span v-if="waktu.tipe==='pelajaran'">Jam ke-{{waktu.jam_ke}}</span><strong v-else>{{waktu.nama_kegiatan}}</strong></td><td><button @click="bukaModalWaktu(waktu)" class="btn-icon">✏️</button><button @click="hapusWaktu(waktu.id)" class="btn-icon text-danger">🗑️</button></td></tr></tbody>
              </table>
            </div>
          </div>
        </div>
        
        <div v-else-if="menuAktif === 'Rombel Kelas'" class="page-content fade-in">
          <div class="panel full-height-panel">
            <div class="panel-header compact-header">
              <h2 class="section-title">Kelola Rombel Kelas</h2>
              <div class="action-buttons"><button @click="bukaModalRombel()" class="btn-primary btn-sm">Generate Otomatis</button><button @click="bukaModalRombelManual()" class="btn-outline btn-sm">Tambah Manual</button></div>
            </div>
            <div class="rombel-grid-container">
              <div v-if="masterRombel.length === 0" class="empty-state">Belum ada rombel.</div>
              <div v-for="rombel in masterRombel" :key="rombel.id" class="rombel-card-clean">
                <span class="fw-bold text-dark">{{ rombel.nama_rombel }}</span>
                <button @click="hapusRombel(rombel.id, rombel.nama_rombel)" class="btn-remove-rombel">×</button>
              </div>
            </div>
          </div>
        </div>

        <div v-if="modalAturKelas.tampil" class="modal-overlay" @click.self="modalAturKelas.tampil = false">
          <div class="modal-box">
            <div class="modal-header"><h3>Pilih Kelas: {{ modalAturKelas.guru.nama }}</h3></div>
            <div class="rombel-selector">
              <label v-for="r in rombels" :key="r" class="rombel-checkbox">
                <input type="checkbox" :value="r" v-model="modalAturKelas.terpilih">
                <span class="checkbox-visual">{{ r }}</span>
              </label>
            </div>
            <div class="modal-actions"><button @click="modalAturKelas.tampil = false" class="btn-text">Batal</button><button @click="simpanPenugasan()" class="btn-primary">Simpan</button></div>
          </div>
        </div>

        <div v-if="modalGuru.tampil" class="modal-overlay" @click.self="modalGuru.tampil = false">
          <div class="modal-box">
            <div class="modal-header"><h3>{{ modalGuru.form.id ? 'Edit Guru' : 'Tambah Guru' }}</h3></div>
            <div class="form-group"><label>Kode (Maks 4 Huruf)</label><input v-model="modalGuru.form.kode" maxlength="4" class="input-clean"></div>
            <div class="form-group"><label>Nama Lengkap</label><input v-model="modalGuru.form.nama" class="input-clean"></div>
            <div class="form-group"><label>Mata Pelajaran</label><input v-model="modalGuru.form.mapel" class="input-clean"></div>
            <div style="display: flex; gap: 15px;">
              <div class="form-group" style="flex: 1;"><label>Durasi JP/Kelas</label><input v-model="modalGuru.form.durasi_mapel" type="number" min="1" class="input-clean"></div>
              <div class="form-group" style="flex: 1;"><label>Target SK (JP)</label><input v-model="modalGuru.form.target_jam" type="number" min="0" class="input-clean"></div>
            </div>
            <div class="modal-actions"><button @click="modalGuru.tampil = false" class="btn-text">Batal</button><button @click="simpanGuru()" class="btn-primary">Simpan</button></div>
          </div>
        </div>

        <div v-if="modalImport.tampil" class="modal-overlay" @click.self="modalImport.tampil = false">
          <div class="modal-box import-box">
            <div class="modal-header"><h3>Import via Excel</h3></div>
            <textarea v-model="modalImport.teksData" rows="6" class="input-clean textarea-clean" placeholder="Paste data (CTRL+V)..."></textarea>
            <div class="modal-actions"><button @click="modalImport.tampil = false" class="btn-text">Batal</button><button @click="prosesImport()" class="btn-primary">Proses</button></div>
          </div>
        </div>

        <div v-if="modalRombel.tampil" class="modal-overlay" @click.self="modalRombel.tampil = false">
          <div class="modal-box">
            <div class="modal-header"><h3>{{ modalRombel.isManual ? 'Tambah Rombel' : 'Generate Rombel' }}</h3></div>
            <div v-if="modalRombel.isManual" class="form-group"><label>Nama Rombel</label><input v-model="modalRombel.form.nama" class="input-clean"></div>
            <div v-else>
              <div class="form-group"><label>Tingkat (Cth: 7)</label><input v-model="modalRombel.form.tingkat" class="input-clean"></div>
              <div class="form-group"><label>Jumlah Rombel (Cth: 7)</label><input v-model="modalRombel.form.jumlah" type="number" class="input-clean"></div>
            </div>
            <div class="modal-actions"><button @click="modalRombel.tampil = false" class="btn-text">Batal</button><button @click="simpanRombel()" class="btn-primary">Simpan</button></div>
          </div>
        </div>
        
        <div v-if="modalWaktu.tampil" class="modal-overlay" @click.self="modalWaktu.tampil = false">
          <div class="modal-box">
            <div class="modal-header"><h3>{{ modalWaktu.form.id ? 'Edit Jam' : 'Tambah Jam' }}</h3></div>
            <div class="form-group"><label>Hari</label><select v-model="modalWaktu.form.hari" class="input-clean"><option value="Senin">Senin</option><option value="Selasa">Selasa</option><option value="Rabu">Rabu</option><option value="Kamis">Kamis</option><option value="Jumat">Jumat</option></select></div>
            <div class="form-group"><label>Tipe Waktu</label><select v-model="modalWaktu.form.tipe" class="input-clean"><option value="pelajaran">Jam Pelajaran</option><option value="khusus">Kegiatan Khusus</option></select></div>
            <div class="form-group"><label>Rentang Waktu</label><input v-model="modalWaktu.form.waktu" class="input-clean"></div>
            <div class="form-group" v-if="modalWaktu.form.tipe === 'pelajaran'"><label>Jam Ke-</label><input v-model="modalWaktu.form.jam_ke" class="input-clean"></div>
            <div class="form-group" v-if="modalWaktu.form.tipe === 'khusus'"><label>Nama Kegiatan</label><input v-model="modalWaktu.form.nama_kegiatan" class="input-clean"></div>
            <div class="modal-actions"><button @click="modalWaktu.tampil = false" class="btn-text">Batal</button><button @click="simpanWaktu()" class="btn-primary">Simpan</button></div>
          </div>
        </div>

      </template>
      </template>
  </div>
</template>

<script setup>
import { ref, onMounted, reactive, computed } from 'vue'
import { supabase } from './lib/supabase' 

// =========================================================================
// SISTEM OTENTIKASI & SIDEBAR MENU
// =========================================================================
const isLoggedIn = ref(false)
const currentUser = ref(null)
const loginInput = ref('')
const isLoadingLogin = ref(false)

const isSidebarOpen = ref(false) 
const pilihMenu = (menu) => {
  menuAktif.value = menu;
  isSidebarOpen.value = false;
}

const menuAdmin = ['Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat', 'Daftar Guru', 'Rombel Kelas', 'Pengaturan Jam', 'Pengaturan Cetak', 'Rekap Evaluasi']
const menuGuru = ['Dashboard Saya']

const daftarMenuTerlihat = computed(() => {
  if (!currentUser.value) return [];
  return currentUser.value.role === 'admin' ? menuAdmin : menuGuru;
})

const menuAktif = ref('Senin')
const subRekap = ref('audit')
const urutanHari = { 'Senin': 1, 'Selasa': 2, 'Rabu': 3, 'Kamis': 4, 'Jumat': 5 }

const doLogin = async () => {
  if (!loginInput.value) return;
  isLoadingLogin.value = true;
  const kode = loginInput.value.trim().toUpperCase();

  const { data, error } = await supabase.from('guru').select('*').eq('kode', kode).single();

  if (data) {
    const userRole = kode === 'BE' ? 'admin' : 'guru';
    currentUser.value = { kode: data.kode, nama: data.nama, role: userRole, target_jam: data.target_jam };
    localStorage.setItem('smartschedule_user', JSON.stringify(currentUser.value));
    
    isLoggedIn.value = true;
    menuAktif.value = userRole === 'admin' ? 'Senin' : 'Dashboard Saya';
    subRekap.value = userRole === 'admin' ? 'audit' : 'harian';
    selectedGuruJadwal.value = data.kode; 
    
    fetchData();
  } else {
    alert('❌ Kode Guru tidak ditemukan! Pastikan kode dimasukkan dengan benar.');
  }
  isLoadingLogin.value = false;
}

const doLogout = () => {
  if(confirm('Yakin ingin keluar dari aplikasi?')) {
    localStorage.removeItem('smartschedule_user');
    isLoggedIn.value = false;
    currentUser.value = null;
    loginInput.value = '';
    isSidebarOpen.value = false;
  }
}

const checkLoginSession = () => {
  const saved = localStorage.getItem('smartschedule_user');
  if (saved) {
    currentUser.value = JSON.parse(saved);
    isLoggedIn.value = true;
    menuAktif.value = currentUser.value.role === 'admin' ? 'Senin' : 'Dashboard Saya';
    subRekap.value = currentUser.value.role === 'admin' ? 'audit' : 'harian';
    selectedGuruJadwal.value = currentUser.value.kode;
    fetchData();
  }
}

// =========================================================================
// DATA & LOGIKA APLIKASI UTAMA
// =========================================================================
const urutanMapel = ['AGM', 'MTK', 'IND', 'IPA', 'OR', 'ING', 'IPS', 'PKN', 'JWA', 'PRA', 'INF', 'BK'];

const teachers = ref([])
const masterWaktu = ref([])
const masterRombel = ref([])
const rombels = ref([])
const penugasanSK = ref([])
const allSchedules = ref([])

const selectedGuruJadwal = ref('')

const pengaturanCetak = reactive({
  namaSekolah: localStorage.getItem('cetak_sekolah') || 'SMP NEGERI 3 BANGUNTAPAN',
  tahunPelajaran: localStorage.getItem('cetak_tapel') || '2025/2026 SEMESTER 2',
  tanggalBerlaku: localStorage.getItem('cetak_tanggal') || '05 Januari 2026',
  tempatTanggal: localStorage.getItem('cetak_ttd_tempat') || 'Banguntapan, 2 Januari 2026',
  namaKepsek: localStorage.getItem('cetak_kepsek') || 'Fulan Nur fulan, M.Pd.',
  nipKepsek: localStorage.getItem('cetak_nip') || '19700xxxxxxxxxxxx'
});

const simpanPengaturanCetak = () => {
  localStorage.setItem('cetak_sekolah', pengaturanCetak.namaSekolah);
  localStorage.setItem('cetak_tapel', pengaturanCetak.tahunPelajaran);
  localStorage.setItem('cetak_tanggal', pengaturanCetak.tanggalBerlaku);
  localStorage.setItem('cetak_ttd_tempat', pengaturanCetak.tempatTanggal);
  localStorage.setItem('cetak_kepsek', pengaturanCetak.namaKepsek);
  localStorage.setItem('cetak_nip', pengaturanCetak.nipKepsek);
  alert('Pengaturan Cetak Berhasil Disimpan!');
};

const exportKeExcel = () => {
  if(allSchedules.value.length === 0) { alert('Jadwal masih kosong!'); return; }
  
  let tableHtml = `
  <html xmlns:o="urn:schemas-microsoft-com:office:office" xmlns:x="urn:schemas-microsoft-com:office:excel" xmlns="http://www.w3.org/TR/REC-html40">
  <head><meta charset="UTF-8"><style>td, th { border: 1px solid black; }</style></head>
  <body style="font-family: 'Times New Roman', serif;">
  `;

  const colKelas7 = '#ffff00'; 
  const colKelas8 = '#ffc000'; 
  const colKelas9 = '#00b0f0'; 
  const colJam = '#92d050'; 
  const colHari = '#92d050'; 

  const hariList = ['Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat'];
  const totalCols = rombels.value.length + 2;

  hariList.forEach((hari) => {
     tableHtml += `
     <table>
        <tr><td colspan="${totalCols}" style="text-align:center; font-size:16pt; font-weight:bold; border:none;">JADWAL PEMBELAJARAN</td></tr>
        <tr><td colspan="${totalCols}" style="text-align:center; font-size:14pt; font-weight:bold; border:none;">${pengaturanCetak.namaSekolah}</td></tr>
        <tr><td colspan="${totalCols}" style="text-align:center; font-size:12pt; font-weight:bold; border:none;">TAHUN PELAJARAN ${pengaturanCetak.tahunPelajaran}</td></tr>
        <tr><td colspan="${totalCols}" style="text-align:center; font-size:11pt; font-weight:bold; border:none;">(Jadwal berlaku mulai tanggal ${pengaturanCetak.tanggalBerlaku})</td></tr>
        <tr><td colspan="${totalCols}" style="border:none;"></td></tr>
     </table>
     <table border="1" style="border-collapse: collapse; text-align: center; vertical-align: middle;">
     <tr>
       <th rowspan="3" style="background-color:${colJam}; font-weight:bold; width: 40px;">JAM<br>KE</th>
       <th rowspan="3" style="background-color:${colJam}; font-weight:bold; width: 120px;">WAKTU</th>
       <th colspan="${rombels.value.length}" style="background-color:${colHari}; font-weight:bold; font-size:12pt;">${hari.toUpperCase()}</th>
     </tr>`;

     const kelompokKelas = { '7': [], '8': [], '9': [] };
     rombels.value.forEach(r => { const t = r.charAt(0); if(kelompokKelas[t]) kelompokKelas[t].push(r); });

     tableHtml += `<tr>`;
     if(kelompokKelas['7'].length > 0) tableHtml += `<th colspan="${kelompokKelas['7'].length}" style="background-color:${colKelas7}; font-weight:bold;">7</th>`;
     if(kelompokKelas['8'].length > 0) tableHtml += `<th colspan="${kelompokKelas['8'].length}" style="background-color:${colKelas8}; font-weight:bold;">8</th>`;
     if(kelompokKelas['9'].length > 0) tableHtml += `<th colspan="${kelompokKelas['9'].length}" style="background-color:${colKelas9}; font-weight:bold;">9</th>`;
     tableHtml += `</tr><tr>`;
     
     ['7', '8', '9'].forEach(t => {
         const bg = t === '7' ? colKelas7 : (t === '8' ? colKelas8 : colKelas9);
         kelompokKelas[t].forEach(r => { tableHtml += `<th style="background-color:${bg}; font-weight:bold;">${r.substring(1)}</th>`; });
     });
     tableHtml += `</tr>`;

     const jadwalHariIni = masterWaktu.value.filter(w => w.hari === hari);
     jadwalHariIni.forEach(sesi => {
         tableHtml += `<tr>`;
         if (sesi.tipe === 'pelajaran') {
             tableHtml += `<td>${sesi.jam_ke}</td><td style="font-family: monospace;">${sesi.waktu}</td>`;
             rombels.value.forEach(rombel => {
                 const jadwalArr = allSchedules.value.filter(s => s.id_rombel === rombel && s.jam_ke === `${hari}-${sesi.jam_ke}`);
                 const mapelKode = jadwalArr.length > 0 ? jadwalArr[0].guru_mapel.split(' - ')[0] : '';
                 tableHtml += `<td style="font-weight:bold;">${mapelKode}</td>`;
             });
         } else {
             tableHtml += `<td></td><td style="font-family: monospace;">${sesi.waktu}</td>`;
             tableHtml += `<td colspan="${rombels.value.length}" style="font-weight:bold; background-color:#e0e7ff;">${sesi.nama_kegiatan}</td>`;
         }
         tableHtml += `</tr>`;
     });
     tableHtml += `</table><br><br>`;
  });

  tableHtml += `
  <table>
    <tr><td colspan="${rombels.value.length - 2}" style="border:none;"></td><td colspan="4" style="text-align:left; font-size:12pt; border:none;">${pengaturanCetak.tempatTanggal}</td></tr>
    <tr><td colspan="${rombels.value.length - 2}" style="border:none;"></td><td colspan="4" style="text-align:left; font-size:12pt; border:none;">Kepala Sekolah</td></tr>
    <tr><td colspan="${totalCols}" style="border:none;"></td></tr><tr><td colspan="${totalCols}" style="border:none;"></td></tr><tr><td colspan="${totalCols}" style="border:none;"></td></tr>
    <tr><td colspan="${rombels.value.length - 2}" style="border:none;"></td><td colspan="4" style="text-align:left; font-size:12pt; font-weight:bold; border:none;">${pengaturanCetak.namaKepsek}</td></tr>
    <tr><td colspan="${rombels.value.length - 2}" style="border:none;"></td><td colspan="4" style="text-align:left; font-size:12pt; border:none;">NIP. ${pengaturanCetak.nipKepsek}</td></tr>
  </table></body></html>`;

  const blob = new Blob([tableHtml], { type: 'application/vnd.ms-excel' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url;
  a.download = `Jadwal_Pelajaran_${pengaturanCetak.tahunPelajaran.replace(/[^a-zA-Z0-9]/g, '_')}.xls`;
  document.body.appendChild(a);
  a.click();
  document.body.removeChild(a);
  URL.revokeObjectURL(url);
}

const getKelasColorClass = (rombel) => {
  if (!rombel) return '';
  if (rombel.startsWith('7')) return 'header-kelas-7';
  if (rombel.startsWith('8')) return 'header-kelas-8';
  if (rombel.startsWith('9')) return 'header-kelas-9';
  return '';
}

const getCardColorClass = (rombel) => {
  if (!rombel) return '';
  if (rombel.startsWith('7')) return 'card-kelas-7';
  if (rombel.startsWith('8')) return 'card-kelas-8';
  if (rombel.startsWith('9')) return 'card-kelas-9';
  return '';
}

const groupedTeachers = computed(() => {
  const groups = {};
  teachers.value.forEach(guru => { 
    const mapel = guru.mapel ? guru.mapel.toUpperCase() : 'LAINNYA'; 
    if (!groups[mapel]) groups[mapel] = []; 
    groups[mapel].push(guru); 
  });
  const sortedKeys = Object.keys(groups).sort((a, b) => {
    let indexA = urutanMapel.indexOf(a); let indexB = urutanMapel.indexOf(b);
    if (indexA === -1) indexA = 999; if (indexB === -1) indexB = 999;
    return indexA - indexB || a.localeCompare(b);
  });
  const sortedGroups = {}; sortedKeys.forEach(key => sortedGroups[key] = groups[key]); return sortedGroups;
});

const rekapKekurangan = computed(() => {
  return teachers.value.map(guru => {
    const penugasan = getPenugasanGuru(guru.kode); const jadwalGuruIni = allSchedules.value.filter(s => s.guru_mapel.startsWith(guru.kode));
    let totalKurang = 0; let detailKurang = [];
    penugasan.forEach(tugas => {
      const aktualJpKelas = jadwalGuruIni.filter(s => s.id_rombel === tugas.nama_rombel).length; const kurang = tugas.durasi_jp - aktualJpKelas;
      if (kurang > 0) { totalKurang += kurang; detailKurang.push(`${tugas.nama_rombel} (-${kurang} JP)`); }
    });
    return { ...guru, totalKurang, detailKurang };
  }).filter(g => g.totalKurang > 0); 
});

const groupedRekapKekurangan = computed(() => {
  const groups = {}; 
  rekapKekurangan.value.forEach(guru => { 
    const mapel = guru.mapel ? guru.mapel.toUpperCase() : 'LAINNYA'; 
    if (!groups[mapel]) groups[mapel] = []; groups[mapel].push(guru); 
  });
  const sortedKeys = Object.keys(groups).sort((a, b) => {
    let indexA = urutanMapel.indexOf(a); let indexB = urutanMapel.indexOf(b);
    if (indexA === -1) indexA = 999; if (indexB === -1) indexB = 999;
    return indexA - indexB || a.localeCompare(b);
  });
  const sortedGroups = {}; sortedKeys.forEach(key => sortedGroups[key] = groups[key]); return sortedGroups;
});

const rekapHarian = computed(() => {
  return teachers.value.map(guru => {
    const jadwalGuruIni = allSchedules.value.filter(s => s.guru_mapel.startsWith(guru.kode)); const hitungan = { Senin: 0, Selasa: 0, Rabu: 0, Kamis: 0, Jumat: 0 };
    jadwalGuruIni.forEach(j => { const hari = j.jam_ke.split('-')[0]; if (hitungan[hari] !== undefined) hitungan[hari]++; });
    return { kode: guru.kode, nama: guru.nama, mapel: guru.mapel, target_jam: guru.target_jam, Senin: hitungan.Senin, Selasa: hitungan.Selasa, Rabu: hitungan.Rabu, Kamis: hitungan.Kamis, Jumat: hitungan.Jumat, Total: jadwalGuruIni.length };
  }).sort((a, b) => b.Total - a.Total);
});

const groupedRekapHarian = computed(() => {
  const groups = {}; 
  rekapHarian.value.forEach(guru => { 
    const mapel = guru.mapel ? guru.mapel.toUpperCase() : 'LAINNYA'; 
    if (!groups[mapel]) groups[mapel] = []; groups[mapel].push(guru); 
  });
  const sortedKeys = Object.keys(groups).sort((a, b) => {
    let indexA = urutanMapel.indexOf(a); let indexB = urutanMapel.indexOf(b);
    if (indexA === -1) indexA = 999; if (indexB === -1) indexB = 999;
    return indexA - indexB || a.localeCompare(b);
  });
  const sortedGroups = {}; sortedKeys.forEach(key => sortedGroups[key] = groups[key]); return sortedGroups;
});

const groupedJadwalGuru = computed(() => {
  if (!selectedGuruJadwal.value) return {};
  const teacher = teachers.value.find(t => t.kode === selectedGuruJadwal.value);
  if (!teacher) return {};

  const jadwalGuru = allSchedules.value.filter(s => s.guru_mapel.startsWith(teacher.kode));
  const groups = {};

  jadwalGuru.forEach(j => {
    const [hari, jamKe] = j.jam_ke.split('-');
    const waktuObj = masterWaktu.value.find(w => w.hari === hari && w.jam_ke == jamKe && w.tipe === 'pelajaran');
    if (!groups[hari]) groups[hari] = [];
    groups[hari].push({ id: j.id, hari: hari, jam_ke: parseInt(jamKe), waktu: waktuObj ? waktuObj.waktu : '-', rombel: j.id_rombel });
  });

  const sortedKeys = Object.keys(groups).sort((a, b) => urutanHari[a] - urutanHari[b]);
  const sortedGroups = {};
  sortedKeys.forEach(key => { sortedGroups[key] = groups[key].sort((a, b) => a.jam_ke - b.jam_ke); });
  return sortedGroups;
});

// KHUSUS UNTUK DASHBOARD GURU YANG SEDANG LOGIN
const myRekap = computed(() => {
  if (!currentUser.value) return null;
  return rekapHarian.value.find(r => r.kode === currentUser.value.kode) || {
    Senin: 0, Selasa: 0, Rabu: 0, Kamis: 0, Jumat: 0, Total: 0, target_jam: currentUser.value.target_jam || 0
  };
});

const fetchData = async () => {
  const { data: rombelData } = await supabase.from('master_rombel').select('*').order('nama_rombel')
  masterRombel.value = rombelData || []
  rombels.value = masterRombel.value.map(r => r.nama_rombel)
  
  const { data: guruData } = await supabase.from('guru').select('*').order('kode')
  teachers.value = guruData || []
  
  if (currentUser.value && currentUser.value.role === 'guru') {
     const me = teachers.value.find(t => t.kode === currentUser.value.kode);
     if (me) currentUser.value.target_jam = me.target_jam;
  }

  const { data: waktuData } = await supabase.from('master_waktu').select('*')
  masterWaktu.value = (waktuData || []).sort((a,b) => urutanHari[a.hari] - urutanHari[b.hari] || a.waktu.localeCompare(b.waktu))
  const { data: mappingData } = await supabase.from('penugasan_sk').select('*')
  penugasanSK.value = mappingData || []
  const { data: jadwalData } = await supabase.from('jadwal').select('*')
  allSchedules.value = jadwalData || []
  calculateTeachingHours()
}

const getPenugasanGuru = (kode) => penugasanSK.value.filter(p => p.kode_guru === kode)
const getJadwalHariIni = () => masterWaktu.value.filter(w => w.hari === menuAktif.value)

const resetSemuaJadwal = async () => {
  if (!confirm("HAPUS SEMUA JADWAL?")) return;
  const { error } = await supabase.from('jadwal').delete().not('id', 'is', null);
  if (!error) { alert("Jadwal dikosongkan!"); fetchData(); }
}

const generateJadwalCerdas = async () => {
  if(!confirm("Mulai Auto-Draft sesuai SK?")) return;
  const jadwalTerisi = new Set(allSchedules.value.map(s => `${s.id_rombel}_${s.jam_ke}`)); 
  const guruSibuk = new Set(allSchedules.value.map(s => `${s.guru_mapel.split(' - ')[0]}_${s.jam_ke}`)); 
  const jamPelajaran = masterWaktu.value.filter(w => w.tipe === 'pelajaran');
  const payload = [];
  let trackerJamGuru = {}; teachers.value.forEach(t => trackerJamGuru[t.kode] = t.totalJam);
  const tumpukanTugas = [...penugasanSK.value].sort((a, b) => b.durasi_jp - a.durasi_jp);

  for (const tugas of tumpukanTugas) {
    const guru = teachers.value.find(t => t.kode === tugas.kode_guru); if (!guru) continue;
    const durasi = tugas.durasi_jp; const rombel = tugas.nama_rombel;
    if (trackerJamGuru[guru.kode] + durasi > guru.target_jam) continue; 
    const jamSudahAdaDiKelasIni = allSchedules.value.filter(s => s.id_rombel === rombel && s.guru_mapel.startsWith(guru.kode)).length;
    if (jamSudahAdaDiKelasIni >= durasi) continue;
    let sisaDibutuhkan = durasi - jamSudahAdaDiKelasIni;
    let blokJp = sisaDibutuhkan === 6 ? [3, 3] : (sisaDibutuhkan === 5 ? [3, 2] : (sisaDibutuhkan >= 4 ? [2, 2] : [sisaDibutuhkan]));
    let hariTerpakaiRombelIni = new Set(); let kelasBerhasil = true; let tempPayload = [];

    for (const ukuran of blokJp) {
      let berhasil = false; const urutanHariAcak = ['Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat'].sort(() => Math.random() - 0.5);
      for (const hari of urutanHariAcak) {
        if (berhasil || hariTerpakaiRombelIni.has(hari)) continue;
        let slotHariIni = jamPelajaran.filter(j => j.hari === hari); let run = [];
        for (const sesi of slotHariIni) {
          const penanda = `${sesi.hari}-${sesi.jam_ke}`;
          if (!jadwalTerisi.has(`${rombel}_${penanda}`) && !guruSibuk.has(`${guru.kode}_${penanda}`)) {
            run.push(penanda);
            if (run.length === ukuran) {
              run.forEach(p => { jadwalTerisi.add(`${rombel}_${p}`); guruSibuk.add(`${guru.kode}_${p}`); tempPayload.push({ id_rombel: rombel, jam_ke: p, guru_mapel: `${guru.kode} - ${guru.mapel}` }); });
              hariTerpakaiRombelIni.add(hari); berhasil = true; break;
            }
          } else { run = []; }
        }
      }
      if (!berhasil) kelasBerhasil = false; 
    }
    if (kelasBerhasil) { payload.push(...tempPayload); trackerJamGuru[guru.kode] += sisaDibutuhkan; } 
    else { tempPayload.forEach(p => { jadwalTerisi.delete(`${rombel}_${p.jam_ke}`); guruSibuk.delete(`${guru.kode}_${p.jam_ke}`); }); }
  }
  if (payload.length > 0) { const { error } = await supabase.from('jadwal').insert(payload); if (!error) { fetchData(); } } 
  else alert("Tidak ada slot yang cocok. Cek Audit.");
}

const modalAturKelas = reactive({ tampil: false, guru: {}, terpilih: [] })
const bukaModalAturKelas = (guru) => { modalAturKelas.guru = guru; modalAturKelas.terpilih = getPenugasanGuru(guru.kode).map(p => p.nama_rombel); modalAturKelas.tampil = true; }
const simpanPenugasan = async () => { const kode = modalAturKelas.guru.kode; const durasi = modalAturKelas.guru.durasi_mapel; await supabase.from('penugasan_sk').delete().eq('kode_guru', kode); const payload = modalAturKelas.terpilih.map(rombel => ({ kode_guru: kode, nama_rombel: rombel, durasi_jp: durasi })); if (payload.length > 0) await supabase.from('penugasan_sk').insert(payload); modalAturKelas.tampil = false; fetchData(); }
const modalGuru = reactive({ tampil: false, form: { id: null, kode: '', nama: '', mapel: '', target_jam: 0, durasi_mapel: 2 } })
const bukaModalGuru = (guru = null) => { modalGuru.form = guru ? { ...guru } : { id: null, kode: '', nama: '', mapel: '', target_jam: 0, durasi_mapel: 2 }; modalGuru.tampil = true }
const simpanGuru = async () => { const data = { kode: modalGuru.form.kode.toUpperCase(), nama: modalGuru.form.nama, mapel: modalGuru.form.mapel, target_jam: modalGuru.form.target_jam, durasi_mapel: modalGuru.form.durasi_mapel }; if (modalGuru.form.id) await supabase.from('guru').update(data).eq('id', modalGuru.form.id); else await supabase.from('guru').insert([data]); modalGuru.tampil = false; fetchData() }
const hapusGuru = async (id) => { if(confirm('Hapus guru?')) { await supabase.from('guru').delete().eq('id', id); fetchData() } }
const modalImport = reactive({ tampil: false, jenis: 'guru', teksData: '' })
const bukaModalImportGuru = () => { modalImport.jenis = 'guru'; modalImport.teksData = ''; modalImport.tampil = true }
const bukaModalImportWaktu = () => { modalImport.jenis = 'waktu'; modalImport.teksData = ''; modalImport.tampil = true }
const prosesImport = async () => { const barisData = modalImport.teksData.split('\n'); const payload = []; barisData.forEach((baris) => { const kolom = baris.split('\t'); if (modalImport.jenis === 'guru' && kolom.length >= 5) { payload.push({ kode: kolom[0].trim().toUpperCase().substring(0,4), nama: kolom[1].trim(), mapel: kolom[2].trim(), durasi_mapel: parseInt(kolom[3].trim()) || 2, target_jam: parseInt(kolom[4].trim()) || 0 }) } else if (modalImport.jenis === 'waktu' && kolom.length >= 4) { payload.push({ hari: kolom[0].trim(), tipe: kolom[1].trim().toLowerCase(), jam_ke: kolom[2].trim() === '-' ? '' : kolom[2].trim(), waktu: kolom[3].trim(), nama_kegiatan: kolom[4] ? kolom[4].trim() : '' }) } }); if (payload.length > 0) { const tabel = modalImport.jenis === 'guru' ? 'guru' : 'master_waktu'; await supabase.from(tabel).insert(payload); } modalImport.tampil = false; fetchData(); }
const modalRombel = reactive({ tampil: false, isManual: false, form: { tingkat: '', jumlah: 1, nama: '' } })
const bukaModalRombel = () => { modalRombel.isManual = false; modalRombel.form = { tingkat: '7', jumlah: 7, nama: '' }; modalRombel.tampil = true }
const bukaModalRombelManual = () => { modalRombel.isManual = true; modalRombel.form = { tingkat: '', jumlah: 1, nama: '' }; modalRombel.tampil = true }
const simpanRombel = async () => { const payload = []; if (modalRombel.isManual) payload.push({ nama_rombel: modalRombel.form.nama }); else for (let i = 0; i < modalRombel.form.jumlah; i++) payload.push({ nama_rombel: modalRombel.form.tingkat + String.fromCharCode(65 + i) }); await supabase.from('master_rombel').insert(payload); modalRombel.tampil = false; fetchData(); }
const hapusRombel = async (id, nama) => { if(confirm(`Hapus ${nama}?`)) { await supabase.from('jadwal').delete().eq('id_rombel', nama); await supabase.from('master_rombel').delete().eq('id', id); fetchData(); } }
const modalWaktu = reactive({ tampil: false, form: { id: null, hari: 'Senin', tipe: 'pelajaran', jam_ke: '', waktu: '', nama_kegiatan: '' } })
const bukaModalWaktu = (waktu = null) => { modalWaktu.form = waktu ? { ...waktu } : { id: null, hari: 'Senin', tipe: 'pelajaran', jam_ke: '', waktu: '', nama_kegiatan: '' }; modalWaktu.tampil = true }
const simpanWaktu = async () => { const data = { ...modalWaktu.form }; if (data.tipe === 'pelajaran') data.nama_kegiatan = ''; if (data.tipe === 'khusus') data.jam_ke = ''; if (data.id) await supabase.from('master_waktu').update(data).eq('id', data.id); else { delete data.id; await supabase.from('master_waktu').insert([data]) } modalWaktu.tampil = false; fetchData() }
const hapusWaktu = async (id) => { await supabase.from('master_waktu').delete().eq('id', id); fetchData() }
const calculateTeachingHours = () => { teachers.value.forEach(t => { t.totalJam = allSchedules.value.filter(s => s.guru_mapel.startsWith(t.kode)).length }) }

const startDrag = (evt, kodeGuru) => {
  evt.dataTransfer.dropEffect = 'copy';
  evt.dataTransfer.effectAllowed = 'copy';
  evt.dataTransfer.setData('kodeGuru', kodeGuru);
}

const onDrop = async (evt, rombel, jamKe) => {
  const kodeGuru = evt.dataTransfer.getData('kodeGuru');
  if(!kodeGuru) return;
  const teacher = teachers.value.find(t => t.kode === kodeGuru);
  if(!teacher) return;

  const penugasan = getPenugasanGuru(kodeGuru);
  const isDitugaskan = penugasan.some(p => p.nama_rombel === rombel);
  if (!isDitugaskan) {
    alert(`❌ DITOLAK! Guru ${kodeGuru} tidak ditugaskan untuk kelas ${rombel} di dalam SK.`);
    return;
  }

  const penandaWaktu = `${menuAktif.value}-${jamKe}`;
  const isSibuk = allSchedules.value.some(s => s.jam_ke === penandaWaktu && s.guru_mapel.startsWith(kodeGuru));
  if (isSibuk) {
    alert(`⚠️ BENTROK! Guru ${kodeGuru} sudah mengajar di kelas lain pada jam ini.`);
    return;
  }

  await supabase.from('jadwal').insert([{ id_rombel: rombel, jam_ke: penandaWaktu, guru_mapel: `${teacher.kode} - ${teacher.mapel}` }]); 
  fetchData() 
}

const removeJadwal = async (id) => { await supabase.from('jadwal').delete().eq('id', id); fetchData() }
const getScheduleData = (rombel, jamKe) => { return allSchedules.value.filter(s => s.id_rombel === rombel && s.jam_ke === `${menuAktif.value}-${jamKe}`) }

onMounted(() => checkLoginSession())
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');

* { box-sizing: border-box; font-family: 'Inter', system-ui, sans-serif; }
body { margin: 0; padding: 0; background-color: #f1f5f9; color: #334155; font-size: 13px; -webkit-font-smoothing: antialiased;}

/* GLOBAL UTILITIES */
.app-container { height: 100vh; display: flex; flex-direction: column; overflow: hidden; background-color: #ffffff;}
.text-dark { color: #0f172a; }
.text-muted { color: #64748b; font-size: 0.75rem;}
.fw-bold { font-weight: 600; }
.text-primary { color: #4f46e5; }
.text-danger { color: #e11d48; }
.text-success { color: #16a34a; }
.text-warning { color: #d97706; }

.bg-light { background-color: #f8fafc; }
.bg-danger-light { background-color: #fee2e2; }
.bg-success-light { background-color: #dcfce7; }
.bg-warning-light { background-color: #fef3c7; }
.border-subtle { border: 1px solid #e2e8f0; }

/* ==========================================
   STYLING HALAMAN LOGIN MODERN MINIMALIS
   ========================================== */
.login-wrapper {
  display: flex; align-items: center; justify-content: center;
  height: 100vh; background: linear-gradient(135deg, #f8fafc 0%, #e2e8f0 100%);
}
.login-card.modern-login {
  background: #ffffff; padding: 3rem 2rem; border-radius: 20px;
  box-shadow: 0 20px 40px -10px rgba(15, 23, 42, 0.1); width: 100%; max-width: 360px;
  text-align: center; border: none;
}
.login-header { margin-bottom: 2.5rem; }
.logo-circle { 
  font-size: 2.5rem; background: #f1f5f9; width: 80px; height: 80px; 
  line-height: 80px; border-radius: 50%; margin: 0 auto 1.2rem auto; 
}
.login-header h1 { font-size: 1.6rem; color: #0f172a; margin: 0 0 5px 0; font-weight: 800; letter-spacing: -0.5px; }
.login-header p { margin: 0; color: #64748b; font-size: 0.85rem; font-weight: 500;}

.login-input-modern {
  width: 100%; text-transform: uppercase; text-align: center; font-size: 1.2rem;
  font-weight: 700; letter-spacing: 4px; padding: 14px 16px; margin-bottom: 1.5rem;
  background: #f8fafc; border: 2px solid transparent; border-radius: 12px;
  color: #0f172a; transition: all 0.3s ease; outline: none;
}
.login-input-modern::placeholder { color: #94a3b8; font-weight: 500; letter-spacing: normal; font-size: 1rem;}
.login-input-modern:focus { border-color: #4f46e5; background: #ffffff; box-shadow: 0 0 0 4px rgba(79, 70, 229, 0.1); }

.btn-login-modern {
  width: 100%; padding: 14px; font-size: 0.95rem; font-weight: 700; letter-spacing: 1px;
  background: #0f172a; color: #ffffff; border: none; border-radius: 12px;
  cursor: pointer; transition: all 0.3s ease;
}
.btn-login-modern:hover { background: #1e293b; transform: translateY(-2px); box-shadow: 0 10px 15px -3px rgba(15, 23, 42, 0.2); }
.btn-login-modern:disabled { background: #94a3b8; cursor: not-allowed; transform: none; box-shadow: none;}


/* ==========================================
   NEW MODERN NAVBAR DENGAN HAMBURGER
   ========================================== */
.navbar-modern { 
  background: #ffffff; border-bottom: 1px solid #e2e8f0; 
  padding: 0.5rem 1rem; display: flex; justify-content: space-between; align-items: center;
  flex-shrink: 0; width: 100%;
}
.nav-left { display: flex; align-items: center; gap: 15px; }
.btn-hamburger { 
  background: none; border: none; font-size: 1.5rem; color: #0f172a; 
  cursor: pointer; padding: 0 5px; border-radius: 4px; transition: 0.2s;
}
.btn-hamburger:hover { background: #f1f5f9; }

.brand { display: flex; align-items: center; gap: 10px; }
.logo-icon { font-size: 1.2rem; }
.brand-text h1 { margin: 0; font-size: 1.1rem; color: #0f172a; font-weight: 800; letter-spacing: -0.5px;}
.brand-text p { margin: 0; font-size: 0.65rem; color: #64748b;}

/* NAVBAR USER PROFILE */
.user-profile { display: flex; align-items: center; gap: 15px; }
.user-info { text-align: right; display: flex; flex-direction: column; line-height: 1.2; }
.user-name { font-size: 0.75rem; color: #0f172a; }
.user-role { font-size: 0.6rem; align-self: flex-end; padding: 2px 6px; border-radius: 4px; margin-top: 2px;}


/* ==========================================
   MODERN SIDEBAR MENU (SLIDE-IN) - ULTRA CLEAN
   ========================================== */
.sidebar-overlay {
  position: fixed; top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(15, 23, 42, 0.4); backdrop-filter: blur(2px);
  z-index: 999;
}
.sidebar {
  position: fixed; top: 0; left: -220px; width: 220px; /* Dirampingkan */
  height: 100vh; background: #ffffff; border-right: 1px solid #f1f5f9;
  box-shadow: 10px 0 15px -3px rgba(0,0,0,0.02);
  z-index: 1000; transition: left 0.3s ease; display: flex; flex-direction: column;
}
.sidebar.open { left: 0; }
.sidebar-header { padding: 1rem 1.2rem; border-bottom: 1px solid #f8fafc; display: flex; justify-content: space-between; align-items: center; }
.sidebar-header h2 { margin: 0; font-size: 0.8rem; font-weight: 700; text-transform: uppercase; letter-spacing: 0.5px; color: #0f172a; }
.btn-close-sidebar { background: none; border: none; font-size: 0.9rem; cursor: pointer; color: #94a3b8; transition: 0.2s;}
.btn-close-sidebar:hover { color: #e11d48; }

.sidebar-menu { padding: 0.5rem 0; overflow-y: auto; flex: 1; }
.menu-label { 
  font-size: 0.55rem; font-weight: 700; color: #94a3b8; 
  padding: 1rem 1.2rem 0.3rem; letter-spacing: 1px; text-transform: uppercase;
}
.sidebar-menu button {
  display: flex; align-items: center; gap: 10px; width: 100%; text-align: left; 
  padding: 0.5rem 1.2rem; background: none; border: none; font-size: 0.75rem; 
  font-weight: 500; color: #475569; cursor: pointer; transition: 0.2s; 
  border-left: 2px solid transparent;
}
.menu-icon { font-size: 0.85rem; filter: grayscale(100%); opacity: 0.5; transition: 0.2s; }
.sidebar-menu button:hover { background: #f8fafc; color: #0f172a; }
.sidebar-menu button:hover .menu-icon { opacity: 0.8; }
.sidebar-menu button.aktif { background: #f8fafc; color: #0f172a; border-left-color: #0f172a; font-weight: 600; }
.sidebar-menu button.aktif .menu-icon { opacity: 1; filter: grayscale(0%); }

/* NAV PILLS (Hidden due to Hamburger Menu usage) */
.nav-pills { display: none; } 

.sub-nav-pills { display: flex; gap: 12px; border-bottom: 1px solid #e2e8f0; width: 100%;}
.sub-nav-pills button { background: none; border: none; padding: 8px 0; font-size: 0.8rem; font-weight: 600; color: #64748b; cursor: pointer; transition: 0.2s; border-bottom: 2px solid transparent; border-radius: 0;}
.sub-nav-pills button:hover { color: #0f172a; }
.sub-nav-pills button.aktif { color: #0f172a; border-bottom-color: #0f172a; }

/* PANELS */
.page-content { padding: 1.5rem; max-width: 100%; width: 100%; flex: 1; overflow-y: auto;} 
.main-layout { padding: 0; display: flex; flex-direction: column; flex: 1; overflow: hidden; width: 100%; }

.panel { background: #ffffff; border-radius: 0; border: none; display: flex; flex-direction: column; }
.full-height-panel { height: 100%; } 
.full-width-panel { width: 100%; height: 100%; border-radius: 0; border: none; } 

.panel-header { padding: 1rem; border-bottom: 1px solid #f1f5f9; display: flex; justify-content: space-between; align-items: center; flex-shrink: 0;}
.compact-header { padding: 0.5rem 1rem; }
.section-title { margin: 0; font-size: 1rem; color: #0f172a; font-weight: 700; letter-spacing: -0.3px;}
.action-buttons { display: flex; gap: 8px; }

/* BUTTONS */
.btn-primary { background: #0f172a; color: white; border: none; padding: 6px 12px; border-radius: 4px; font-size: 0.75rem; font-weight: 500; cursor: pointer; transition: 0.2s;}
.btn-primary:hover { background: #1e293b; }
.btn-outline { background: white; color: #334155; border: 1px solid #cbd5e1; padding: 6px 12px; border-radius: 4px; font-size: 0.75rem; font-weight: 500; cursor: pointer; transition: 0.2s;}
.btn-outline:hover { background: #f8fafc; border-color: #94a3b8; }
.btn-sm { padding: 4px 10px; font-size: 0.7rem; }
.btn-danger { background: #ffffff; color: #e11d48; border: 1px solid #fecdd3; padding: 6px 12px; border-radius: 4px; font-size: 0.75rem; font-weight: 500; cursor: pointer;}
.btn-danger:hover { background: #fff1f2; }
.btn-magic { background: #4f46e5; color: white; border: none; padding: 6px 12px; border-radius: 4px; font-size: 0.75rem; font-weight: 500; cursor: pointer;}
.btn-magic:hover { background: #4338ca; }
.btn-icon { background: transparent; border: none; font-size: 0.9rem; padding: 4px; cursor: pointer; opacity: 0.6; }
.btn-icon:hover { opacity: 1; }
.btn-text { background: none; border: none; color: #64748b; font-weight: 500; cursor: pointer; padding: 6px 12px;}

.info-banner { padding: 8px 15px; font-size: 0.75rem; background: #f8fafc; border-bottom: 1px solid #e2e8f0; color: #475569; flex-shrink: 0;}

/* TABEL DAFTAR GURU / EVALUASI */
.table-responsive { overflow: auto; width: 100%; flex: 1;}
.minimal-table { width: 100%; border-collapse: collapse; text-align: left; }
.minimal-table th { background: #ffffff; padding: 8px 12px; font-size: 0.65rem; font-weight: 700; color: #94a3b8; text-transform: uppercase; letter-spacing: 0.05em; border-bottom: 1px solid #e2e8f0; position: sticky; top: 0; z-index: 10;}
.minimal-table td { padding: 8px 12px; border-bottom: 1px solid #f8fafc; vertical-align: middle; color: #334155; font-size: 0.75rem;}
.text-center-table th, .text-center-table td { text-align: center; }

.mapel-group-header td { background: #f8fafc; padding: 4px 12px; border-top: 1px solid #f1f5f9; border-bottom: 1px solid #f1f5f9;}
.mapel-group-title { display: flex; align-items: center; gap: 8px; }
.mapel-name { font-size: 0.65rem; font-weight: 700; color: #64748b; letter-spacing: 0.05em; }
.badge-micro { padding: 2px 6px; border-radius: 4px; font-size: 0.65rem; font-weight: 600; display: inline-block; line-height: 1;}

.btn-add-mini { background: #ffffff; border: 1px dashed #cbd5e1; color: #64748b; border-radius: 4px; padding: 0 6px; cursor: pointer; transition: 0.2s;}
.btn-add-mini:hover { border-color: #4f46e5; color: #4f46e5; background: #e0e7ff;}
.assigned-badges { display: flex; flex-wrap: wrap; gap: 4px; align-items: center;}

/* ---------------------------------------------------
   DOCK GURU KANBAN STYLE (TETAP AMAN 2 BARIS LEGA)
   --------------------------------------------------- */
.top-teacher-pool-container { padding: 4px 8px; background: #ffffff; border-bottom: 1px solid #e2e8f0; flex-shrink: 0; width: 100%;}
.pool-header { font-size: 0.65rem; margin-bottom: 4px; text-align: center;}

.grouped-pools-container { 
  display: flex; flex-wrap: nowrap; gap: 4px; overflow: hidden; width: 100%; justify-content: space-between;
}

.mapel-pool-group { 
  display: flex; flex-direction: column; flex: 1; min-width: 0; 
  background: #f8fafc; border: 1px solid #e2e8f0; border-radius: 4px; padding: 4px; 
  padding-bottom: 6px; 
}

.mapel-pool-label { 
  font-size: 0.55rem; font-weight: 800; color: #0f172a; text-transform: uppercase; 
  text-align: center; padding-bottom: 2px; margin-bottom: 2px; border-bottom: 1px dashed #cbd5e1; 
  white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
}

.top-teacher-pool { 
  display: flex; flex-direction: column; gap: 4px; max-height: 68px; overflow-y: auto; padding-right: 2px;
}

.top-teacher-pool::-webkit-scrollbar { width: 4px; }
.top-teacher-pool::-webkit-scrollbar-thumb { background-color: #cbd5e1; border-radius: 4px; }

.teacher-pill { 
  display: flex; justify-content: space-between; align-items: center; 
  background: #ffffff; border: 1px solid #cbd5e1; padding: 2px 4px; 
  border-radius: 4px; cursor: grab; transition: 0.1s; box-shadow: 0 1px 2px rgba(0,0,0,0.02);
}
.teacher-pill:hover { background: #f1f5f9; border-color: #94a3b8; transform: translateY(-1px); }
.kode-fw { font-weight: 700; color: #334155; font-size: 0.65rem;}
.jp-badge-mini { font-size: 0.55rem; font-weight: 600; padding: 0 2px; background: transparent;}


/* ---------------------------------------------------
   GRID JADWAL (FIT TO SCREEN NO SCROLL, KOLOM WAKTU ANTI POTONG & RATA TENGAH)
   --------------------------------------------------- */
.schedule-area { flex: 1; display: flex; flex-direction: column; overflow: hidden; border: none;}
.schedule-scroll { flex: 1; overflow-x: hidden; overflow-y: auto; width: 100%; height: 100%; position: relative;}

.grid-table-clean { width: 100%; table-layout: fixed; border-collapse: separate; border-spacing: 0; }

.grid-table-clean th, .grid-table-clean td { border-right: 1px solid #f1f5f9; border-bottom: 1px solid #f1f5f9; text-align: center; }

.grid-table-clean th { padding: 4px 2px; font-size: 0.6rem; font-weight: 700; position: sticky; top: 0; z-index: 20; background: #ffffff; color: #475569; }

.sticky-col { position: sticky; left: 0; background: #ffffff; z-index: 30; }
th.sticky-col { z-index: 40 !important; background: #ffffff; border-right: 1px solid #e2e8f0; width: 130px; min-width: 130px;}

.time-label { padding: 2px 6px; width: 130px; min-width: 130px; white-space: nowrap; border-right: 1px solid #e2e8f0; text-align: center;}
.time-content { display: flex; flex-direction: row; align-items: center; justify-content: center; gap: 5px; flex-wrap: nowrap;}
.jam-ke-clean { font-size: 0.6rem; color: #4f46e5; font-weight: 800; display: inline-block; white-space: nowrap;}
.waktu-teks-clean { font-size: 0.6rem; color: #64748b; font-weight: 500; white-space: nowrap;}

.header-kelas-7 { color: #0369a1 !important; border-bottom: 2px solid #38bdf8 !important; }
.header-kelas-8 { color: #a16207 !important; border-bottom: 2px solid #facc15 !important; }
.header-kelas-9 { color: #15803d !important; border-bottom: 2px solid #4ade80 !important; }

.card-kelas-7 { border: 1px solid #7dd3fc !important; background-color: #f0f9ff !important; }
.card-kelas-7 .assigned-kode-clean { color: #0284c7 !important; }
.card-kelas-7 .btn-remove-clean { border-color: #7dd3fc !important; color: #0284c7 !important; }

.card-kelas-8 { border: 1px solid #fde047 !important; background-color: #fefce8 !important; }
.card-kelas-8 .assigned-kode-clean { color: #a16207 !important; }
.card-kelas-8 .btn-remove-clean { border-color: #fde047 !important; color: #a16207 !important; }

.card-kelas-9 { border: 1px solid #86efac !important; background-color: #f0fdf4 !important; }
.card-kelas-9 .assigned-kode-clean { color: #166534 !important; }
.card-kelas-9 .btn-remove-clean { border-color: #86efac !important; color: #166534 !important; }

.special-row-clean { background: #f8fafc; padding: 2px;}
.special-text-clean { font-size: 0.55rem; font-weight: 700; letter-spacing: 0.05em; color: #94a3b8; text-transform: uppercase;}

.slot-clean { padding: 1px; background: #ffffff; vertical-align: middle;}
.drop-zone-clean { min-height: 24px; height: 100%; display: flex; align-items: center; justify-content: center; }

.assigned-card-clean { 
  background: transparent; border: 1px solid #cbd5e1; border-radius: 2px; padding: 1px 0; 
  width: 100%; height: 100%; display: flex; justify-content: center; align-items: center; 
  position: relative; transition: 0.1s; box-shadow: none;
}
.assigned-card-clean:hover { background: #f8fafc; border-color: #94a3b8 !important; }
.assigned-kode-clean { font-weight: 700; color: #334155; font-size: 0.6rem; }

.btn-remove-clean { 
  position: absolute; top: -3px; right: -3px; background: #ffffff; color: #94a3b8; 
  border: 1px solid #e2e8f0; width: 12px; height: 12px; border-radius: 50%; 
  display: flex; align-items: center; justify-content: center; cursor: pointer; 
  opacity: 0; transition: 0.2s; font-size: 7px; line-height: 1; font-weight: bold;
}
.assigned-card-clean:hover .btn-remove-clean { opacity: 1; }
.btn-remove-clean:hover { background: #ef4444 !important; color: white !important; border-color: #ef4444 !important;}

/* MODALS */
.modal-overlay { backdrop-filter: blur(2px); position: fixed; inset: 0; background: rgba(0,0,0,0.4); display: flex; align-items: center; justify-content: center; z-index: 1000;}
.modal-box { padding: 1.5rem; border-radius: 12px; box-shadow: 0 20px 25px -5px rgba(0,0,0,0.1); border: 1px solid #e2e8f0; background: #fff; width: 90%; max-width: 400px;}
.modal-header h3 { font-size: 1rem; color: #0f172a; margin-top: 0; margin-bottom: 1rem;}
.input-clean { padding: 8px 10px; border: 1px solid #e2e8f0; border-radius: 6px; font-size: 0.8rem; background: #f8fafc; color: #0f172a; width: 100%;}
.input-clean:focus { border-color: #0f172a; background: #ffffff; outline: none;}
.textarea-clean { min-height: 100px; font-family: monospace;}
.form-group { margin-bottom: 1rem;}
.form-group label { display: block; font-size: 0.75rem; font-weight: 600; color: #64748b; margin-bottom: 4px;}
.modal-actions { display: flex; justify-content: flex-end; gap: 8px; margin-top: 1rem;}

.rombel-card-clean { background: #ffffff; border: 1px solid #e2e8f0; padding: 10px 14px; border-radius: 6px; display: flex; justify-content: space-between; min-width: 100px;}
.btn-remove-rombel { background: none; border: none; color: #94a3b8; cursor: pointer;}
.btn-remove-rombel:hover { color: #ef4444;}
.rombel-grid-container { display: flex; flex-wrap: wrap; gap: 10px; padding: 15px; overflow-y: auto;}

.rombel-selector { display: grid; grid-template-columns: repeat(auto-fill, minmax(60px, 1fr)); gap: 8px; margin: 1rem 0;}
.rombel-checkbox input { display: none; }
.checkbox-visual { display: block; text-align: center; padding: 8px; border: 1px solid #e2e8f0; border-radius: 6px; font-size: 0.8rem; cursor: pointer; font-weight: 600; color: #64748b; background: #f8fafc; transition: 0.2s;}
.rombel-checkbox input:checked + .checkbox-visual { background: #0f172a; color: white; border-color: #0f172a; }

.empty-state { text-align: center; padding: 3rem 2rem; color: #94a3b8; }
.fade-in { animation: fadeIn 0.2s ease; }
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
</style>