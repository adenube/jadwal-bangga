<template>
  <div class="app-container">
    
    <div v-if="!isLoggedIn" class="login-wrapper fade-in">
      <div class="login-card modern-login">
        <div class="login-header">
          <div class="logo-circle">
            <svg width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="#0f172a" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="4" width="18" height="18" rx="2" ry="2"></rect><line x1="16" y1="2" x2="16" y2="6"></line><line x1="8" y1="2" x2="8" y2="6"></line><line x1="3" y1="10" x2="21" y2="10"></line></svg>
          </div>
          <h1>SmartSchedule</h1>
          <p>Manajemen Jadwal Cerdas</p>
        </div>
        <div class="login-body">
          <div class="form-group">
            <input 
              v-model="loginInput" 
              @keyup.enter="doLogin"
              type="text" 
              class="input-underline login-input-modern" 
              placeholder="KODE GURU / ADMIN"
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
          <button @click="isSidebarOpen = false" class="btn-close-sidebar">
            <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
          </button>
        </div>
        
        <div class="sidebar-menu">
          <template v-if="currentUser.role === 'admin'">
            <div class="menu-label">Dashboard</div>
            <button @click="pilihMenu('Home')" :class="{'aktif': menuAktif === 'Home'}">
               Home
            </button>

            <div class="menu-label">Jadwal Harian</div>
            <button v-for="hari in ['Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat']" :key="hari" @click="pilihMenu(hari)" :class="{'aktif': menuAktif === hari}">
               Hari {{ hari }}
            </button>
            
            <div class="menu-label">Master Data</div>
            <button @click="pilihMenu('Daftar Guru')" :class="{'aktif': menuAktif === 'Daftar Guru'}">Daftar Guru</button>
            <button @click="pilihMenu('Rombel Kelas')" :class="{'aktif': menuAktif === 'Rombel Kelas'}">Rombel Kelas</button>
            <button @click="pilihMenu('Pengaturan Jam')" :class="{'aktif': menuAktif === 'Pengaturan Jam'}">Pengaturan Waktu</button>
            <button @click="pilihMenu('Pengecualian Khusus')" :class="{'aktif': menuAktif === 'Pengecualian Khusus'}">Pengecualian Aturan</button>
            
            <div class="menu-label">Laporan & Ekspor</div>
            <button @click="pilihMenu('Rekap Evaluasi')" :class="{'aktif': menuAktif === 'Rekap Evaluasi'}">Rekap Evaluasi</button>
            <button @click="pilihMenu('Pengaturan Cetak')" :class="{'aktif': menuAktif === 'Pengaturan Cetak'}">Pengaturan Cetak</button>
          </template>
          <template v-else>
            <div class="menu-label">Menu Guru</div>
            <button @click="pilihMenu('Dashboard Saya')" :class="{'aktif': menuAktif === 'Dashboard Saya'}">Dashboard Personal</button>
          </template>
        </div>

        <div class="sidebar-footer">
          <div class="user-info-clean">
            <span class="u-name">{{ currentUser.kode }}</span>
            <span class="u-role">{{ currentUser.role.toUpperCase() }}</span>
          </div>
          <button @click="doLogout" class="btn-logout-full">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" style="margin-right:6px;"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"></path><polyline points="16 17 21 12 16 7"></polyline><line x1="21" y1="12" x2="9" y2="12"></line></svg>
            Keluar Aplikasi
          </button>
        </div>
      </aside>

      <header class="navbar-modern">
        <div class="nav-left">
          <button @click="isSidebarOpen = true" class="btn-hamburger">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><line x1="3" y1="12" x2="21" y2="12"></line><line x1="3" y1="6" x2="21" y2="6"></line><line x1="3" y1="18" x2="21" y2="18"></line></svg>
          </button>
          <div class="brand" @click="pilihMenu(currentUser.role === 'admin' ? 'Home' : 'Dashboard Saya')" title="Kembali ke Dashboard Utama">
            <span class="brand-title">SmartSchedule</span>
            <span class="brand-subtitle">{{ pengaturanCetak.namaSekolah }}</span>
          </div>
        </div>
      </header>

      <div v-if="menuAktif === 'Dashboard Saya'" class="page-content fade-in">
        <div class="content-wrapper" style="max-width: 800px;">
          <div class="panel-header-clean">
            <h2 class="section-title">Dashboard Personal</h2>
            <p class="section-subtitle">Rincian penugasan mengajar Anda. Pastikan total JP sesuai SK.</p>
          </div>
          <div class="panel-body-clean">
            
            <h3 class="subsection-title">Distribusi Beban Harian</h3>
            <div class="table-card">
              <table class="modern-clean-table text-center-table">
                <thead><tr><th>SENIN</th><th>SELASA</th><th>RABU</th><th>KAMIS</th><th>JUMAT</th><th class="highlight-col">TOTAL JP</th><th>TARGET SK</th></tr></thead>
                <tbody>
                  <tr v-if="myRekap">
                    <td><span class="jp-val" :class="{'danger': myRekap.Senin > 6, 'muted': myRekap.Senin === 0}">{{ myRekap.Senin }}</span></td>
                    <td><span class="jp-val" :class="{'danger': myRekap.Selasa > 6, 'muted': myRekap.Selasa === 0}">{{ myRekap.Selasa }}</span></td>
                    <td><span class="jp-val" :class="{'danger': myRekap.Rabu > 6, 'muted': myRekap.Rabu === 0}">{{ myRekap.Rabu }}</span></td>
                    <td><span class="jp-val" :class="{'danger': myRekap.Kamis > 6, 'muted': myRekap.Kamis === 0}">{{ myRekap.Kamis }}</span></td>
                    <td><span class="jp-val" :class="{'danger': myRekap.Jumat > 6, 'muted': myRekap.Jumat === 0}">{{ myRekap.Jumat }}</span></td>
                    <td class="highlight-col"><strong class="total-val" :class="{'text-success': myRekap.Total === myRekap.target_jam, 'text-danger': myRekap.Total > myRekap.target_jam}">{{ myRekap.Total }}</strong></td>
                    <td><span class="badge-success">{{ myRekap.target_jam }}</span></td>
                  </tr>
                </tbody>
              </table>
            </div>

            <h3 class="subsection-title" style="margin-top: 2rem;">Rincian Jadwal Kelas</h3>
            <div class="table-card">
              <table class="modern-clean-table">
                <thead><tr><th style="width: 80px;">JAM</th><th style="width: 150px;">WAKTU</th><th>KELAS</th></tr></thead>
                <tbody>
                  <template v-for="(jadwals, hari) in groupedJadwalGuru" :key="hari">
                    <tr class="row-group-clean"><td colspan="3">{{ hari.toUpperCase() }} <span class="badge-neutral">{{ jadwals.length }} JP</span></td></tr>
                    <tr v-for="item in jadwals" :key="item.id">
                      <td><span class="time-badge">Jam {{ item.jam_ke }}</span></td>
                      <td class="time-text">{{ item.waktu }}</td>
                      <td><span class="class-pill" :class="getKelasColorClass(item.rombel)">{{ item.rombel }}</span></td>
                    </tr>
                  </template>
                  <tr v-if="Object.keys(groupedJadwalGuru).length === 0"><td colspan="3" class="empty-state">Belum ada jadwal mengajar.</td></tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>

      <template v-if="currentUser.role === 'admin'">
        
        <div v-if="menuAktif === 'Home'" class="page-content fade-in">
          <div class="content-wrapper" style="height: 100%; display: flex; flex-direction: column;">
            <div class="panel-header-clean flex-between" style="flex-shrink: 0;">
              <div>
                <h2 class="section-title">Dashboard Utama</h2>
                <p class="section-subtitle">Distribusi mengajar harian guru</p>
              </div>
            </div>
            
            <div class="table-card table-scroll-container">
              <table class="modern-clean-table text-center-table freeze-header-table">
                <thead><tr><th style="text-align: left; width: 60px;">KODE</th><th style="text-align: left;">NAMA GURU</th><th>SENIN</th><th>SELASA</th><th>RABU</th><th>KAMIS</th><th>JUMAT</th><th class="highlight-col">TOTAL</th><th>SK</th></tr></thead>
                <tbody>
                  <template v-for="(guruList, mapel) in groupedRekapHarian" :key="mapel">
                    <tr class="row-group-clean"><td colspan="9" style="text-align: left;">{{ mapel.toUpperCase() }}</td></tr>
                    <tr v-for="rekap in guruList" :key="rekap.kode">
                      <td style="text-align: left;"><span class="fw-bold text-primary">{{ rekap.kode }}</span></td><td style="text-align: left;">{{ rekap.nama }}</td>
                      <td><span :class="{'text-danger fw-bold': rekap.Senin > 6, 'text-muted': rekap.Senin === 0}">{{ rekap.Senin }}</span></td>
                      <td><span :class="{'text-danger fw-bold': rekap.Selasa > 6, 'text-muted': rekap.Selasa === 0}">{{ rekap.Selasa }}</span></td>
                      <td><span :class="{'text-danger fw-bold': rekap.Rabu > 6, 'text-muted': rekap.Rabu === 0}">{{ rekap.Rabu }}</span></td>
                      <td><span :class="{'text-danger fw-bold': rekap.Kamis > 6, 'text-muted': rekap.Kamis === 0}">{{ rekap.Kamis }}</span></td>
                      <td><span :class="{'text-danger fw-bold': rekap.Jumat > 6, 'text-muted': rekap.Jumat === 0}">{{ rekap.Jumat }}</span></td>
                      <td class="highlight-col fw-bold" :class="{'text-danger': rekap.Total > rekap.target_jam, 'text-success': rekap.Total === rekap.target_jam}">{{ rekap.Total }}</td>
                      <td>{{ rekap.target_jam }}</td>
                    </tr>
                  </template>
                </tbody>
              </table>
            </div>
          </div>
        </div>

        <div v-else-if="menuAktif === 'Pengecualian Khusus'" class="page-content fade-in">
          <div class="content-wrapper">
            <div class="panel-header-clean">
              <h2 class="section-title">Kendali Aturan</h2>
              <div class="segmented-control">
                <button @click="subPengecualian = 'guru'" :class="{ 'active': subPengecualian === 'guru' }">Larangan Guru</button>
                <button @click="subPengecualian = 'mapel'" :class="{ 'active': subPengecualian === 'mapel' }">Hari Mapel</button>
              </div>
            </div>

            <div v-if="subPengecualian === 'guru'" class="panel-body-clean">
              <div class="flex-between" style="margin-bottom: 1rem;">
                <p class="section-subtitle" style="margin:0;">Larang guru mengajar pada jam & hari tertentu</p>
                <button @click="bukaModalLaranganGuru()" class="btn-solid">Tambah Larangan</button>
              </div>
              <div class="table-card">
                <table class="modern-clean-table">
                  <thead><tr><th>KODE GURU</th><th>HARI DILARANG</th><th>JAM KE-</th><th style="text-align:right;">AKSI</th></tr></thead>
                  <tbody>
                    <tr v-for="rule in rulesRestrictedSlots" :key="rule.id">
                      <td><strong class="text-primary">{{ rule.kode_guru }}</strong></td><td>{{ rule.hari }}</td><td>Jam {{ rule.jam_ke }}</td>
                      <td style="text-align:right;">
                        <button @click="hapusLaranganGuru(rule.id)" class="btn-icon-outline text-danger" title="Hapus"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg></button>
                      </td>
                    </tr>
                    <tr v-if="rulesRestrictedSlots.length === 0"><td colspan="4" class="empty-state">Belum ada aturan larangan guru.</td></tr>
                  </tbody>
                </table>
              </div>
            </div>

            <div v-if="subPengecualian === 'mapel'" class="panel-body-clean">
              <div class="flex-between" style="margin-bottom: 1rem;">
                <p class="section-subtitle" style="margin:0;">Atur mapel agar HANYA muncul di hari pilihan</p>
                <button @click="bukaModalHariMapel()" class="btn-solid">Atur Hari Mapel</button>
              </div>
              <div class="table-card">
                <table class="modern-clean-table">
                  <thead><tr><th>MATA PELAJARAN</th><th>HARI YANG DIIZINKAN</th><th style="text-align:right;">AKSI</th></tr></thead>
                  <tbody>
                    <tr v-for="rule in rulesSubjectDays" :key="rule.id">
                      <td><strong class="text-primary">{{ rule.mapel }}</strong></td>
                      <td><span v-for="h in rule.allowed_days" :key="h" class="badge-success" style="margin-right:6px;">{{ h }}</span></td>
                      <td style="text-align:right;">
                        <button @click="hapusHariMapel(rule.id)" class="btn-icon-outline text-danger" title="Hapus"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg></button>
                      </td>
                    </tr>
                    <tr v-if="rulesSubjectDays.length === 0"><td colspan="3" class="empty-state">Semua mapel bebas mengajar di semua hari.</td></tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>

        <div v-else-if="['Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat'].includes(menuAktif)" class="main-layout fade-in full-width-layout">
          <main class="schedule-area border-0">
            
            <div class="schedule-toolbar-modern">
              <h3 class="schedule-title">Jadwal: {{ menuAktif }}</h3>
              <div class="action-buttons">
                <button @click="resetSemuaJadwal()" class="btn-outline-danger">Kosongkan</button>
                <button @click="generateJadwalCerdas()" class="btn-solid-accent">✨ Auto-Draft</button>
                <button @click="exportKeExcel()" class="btn-solid-success">📥 Excel</button>
              </div>
            </div>

            <div class="top-teacher-pool-container">
              <div class="pool-header">DAFTAR GURU (DRAG KE SLOT BAWAH)</div>
              <div class="grouped-pools-container">
                <template v-for="(guruList, mapel) in groupedTeachers" :key="mapel">
                  <div class="mapel-pool-group">
                    <div class="mapel-pool-label">{{ mapel }}</div>
                    <div class="top-teacher-pool">
                      <div v-for="element in guruList" :key="element.id" class="teacher-pill" draggable="true" @dragstart="startDrag($event, element.kode, mapel)" title="Tarik ke jadwal">
                        <span class="kode-fw">{{ element.kode }}</span>
                        <span class="jp-badge-mini" :class="{ 'ok': element.totalJam === element.target_jam, 'over': element.totalJam > element.target_jam, 'text-muted': element.totalJam < element.target_jam }">
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
                          <button @click="removeJadwal(element.id)" class="btn-remove-clean">✕</button>
                        </div>
                      </div>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </main>
        </div>

        <div v-else-if="menuAktif === 'Daftar Guru'" class="page-content fade-in">
          <div class="content-wrapper">
            <div class="panel-header-clean flex-between">
              <div>
                <h2 class="section-title">Kelola Penugasan Guru</h2>
                <p class="section-subtitle">Target Jam dan Tugas Mengajar</p>
              </div>
              <div class="action-buttons">
                <button @click="bukaModalImportGuru()" class="btn-outline">Import Excel</button>
                <button @click="bukaModalGuru()" class="btn-solid">Tambah Guru</button>
              </div>
            </div>
            
            <div class="table-card">
              <table class="modern-clean-table">
                <thead><tr><th style="width: 60px;">KODE</th><th>NAMA & MAPEL</th><th style="width: 80px;">DURASI</th><th>PENUGASAN SK</th><th style="width: 80px; text-align:center;">SK</th><th style="width: 80px; text-align:right;">AKSI</th></tr></thead>
                <tbody>
                  <template v-for="(guruList, mapel) in groupedTeachers" :key="mapel">
                    <tr class="row-group-clean"><td colspan="6">{{ mapel }}</td></tr>
                    <tr v-for="guru in guruList" :key="guru.id">
                      <td><span class="fw-bold text-primary">{{ guru.kode }}</span></td>
                      <td><div class="fw-bold text-dark">{{ guru.nama }}</div><div class="text-muted font-small">{{ guru.mapel }}</div></td>
                      <td class="text-muted font-small">{{ guru.durasi_mapel }} JP</td>
                      <td>
                        <div class="assigned-badges">
                          <span v-for="tugas in getPenugasanGuru(guru.kode)" :key="tugas.id" class="chip-label">{{ tugas.nama_rombel }}</span>
                          <button @click="bukaModalAturKelas(guru)" class="btn-circle-add">+</button>
                        </div>
                      </td>
                      <td style="text-align:center;">
                        <span class="fw-bold" :class="{'text-success': guru.totalJam === guru.target_jam, 'text-danger': guru.totalJam > guru.target_jam, 'text-muted': guru.totalJam < guru.target_jam}">
                          {{ guru.totalJam }}/{{ guru.target_jam }}
                        </span>
                      </td>
                      <td style="text-align:right;">
                        <button @click="bukaModalGuru(guru)" class="btn-icon-outline" title="Edit"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path></svg></button>
                        <button @click="hapusGuru(guru.id)" class="btn-icon-outline text-danger" title="Hapus" style="margin-left:4px;"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg></button>
                      </td>
                    </tr>
                  </template>
                </tbody>
              </table>
            </div>
          </div>
        </div>

        <div v-else-if="menuAktif === 'Rombel Kelas'" class="page-content fade-in">
           <div class="content-wrapper">
             <div class="panel-header-clean flex-between">
               <div>
                 <h2 class="section-title">Master Rombel Kelas</h2>
                 <p class="section-subtitle">Daftar kelas berdasar tingkatan angkatan</p>
               </div>
               <div class="action-buttons"><button @click="bukaModalRombel()" class="btn-solid">Generate Otomatis</button><button @click="bukaModalRombelManual()" class="btn-outline">Tambah Manual</button></div>
             </div>
             
             <div class="rombel-card-layout">
               <template v-for="(rombelsGroup, grade) in groupedRombels" :key="grade">
                 <div class="rombel-grade-card">
                   <div class="rombel-grade-header">
                     <h3>Kelas {{ grade }}</h3>
                     <span class="rombel-count">{{ rombelsGroup.length }} Rombel</span>
                   </div>
                   <div class="rombel-grade-body">
                     <div v-for="rombel in rombelsGroup" :key="rombel.id" class="rombel-pill">
                       <span class="fw-bold">{{ rombel.nama_rombel }}</span>
                       <button @click="hapusRombel(rombel.id, rombel.nama_rombel)" class="btn-remove-rombel">✕</button>
                     </div>
                   </div>
                 </div>
               </template>
               <div v-if="Object.keys(groupedRombels).length === 0" class="empty-state" style="width: 100%;">Belum ada data rombel.</div>
             </div>
           </div>
        </div>

        <div v-else-if="menuAktif === 'Pengaturan Jam'" class="page-content fade-in">
          <div class="content-wrapper">
            <div class="panel-header-clean flex-between">
              <div>
                <h2 class="section-title">Master Waktu & Sesi</h2>
                <p class="section-subtitle">Atur rentang jam pelajaran dan jam khusus</p>
              </div>
              <div class="action-buttons"><button @click="bukaModalImportWaktu()" class="btn-outline">Import Excel</button><button @click="bukaModalWaktu()" class="btn-solid">Tambah Jam</button></div>
            </div>
            <div class="table-card">
              <table class="modern-clean-table">
                <thead><tr><th>HARI</th><th>WAKTU</th><th>TIPE</th><th>KETERANGAN</th><th style="text-align:right;">AKSI</th></tr></thead>
                <tbody>
                  <tr v-for="waktu in masterWaktu" :key="waktu.id">
                    <td><strong class="text-dark">{{ waktu.hari }}</strong></td><td class="font-mono">{{ waktu.waktu }}</td>
                    <td><span class="text-muted font-small" style="text-transform:uppercase;">{{ waktu.tipe }}</span></td>
                    <td><span v-if="waktu.tipe==='pelajaran'" class="time-badge">Jam ke-{{waktu.jam_ke}}</span><strong v-else class="text-primary">{{waktu.nama_kegiatan}}</strong></td>
                    <td style="text-align:right;">
                      <button @click="bukaModalWaktu(waktu)" class="btn-icon-outline" title="Edit"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path></svg></button>
                      <button @click="hapusWaktu(waktu.id)" class="btn-icon-outline text-danger" title="Hapus" style="margin-left:4px;"><svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg></button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>

        <div v-else-if="menuAktif === 'Rekap Evaluasi'" class="page-content fade-in">
          <div class="content-wrapper">
            <div class="panel-header-clean">
              <div class="flex-between" style="margin-bottom:1rem;">
                <h2 class="section-title">Pusat Evaluasi & Audit</h2>
                <button @click="generateJadwalCerdas()" class="btn-solid-accent">✨ Auto-Draft SK</button>
              </div>
              <div class="segmented-control">
                <button @click="subRekap = 'audit'" :class="{ 'active': subRekap === 'audit' }">Audit Kekurangan SK</button>
                <button @click="subRekap = 'jadwal'" :class="{ 'active': subRekap === 'jadwal' }">Jadwal Per Guru</button>
              </div>
            </div>
            
            <div v-if="subRekap === 'audit'" class="panel-body-clean">
              <div class="table-card">
                <table class="modern-clean-table">
                  <thead><tr><th style="width: 60px;">KODE</th><th>NAMA GURU</th><th>TARGET</th><th>TERJADWAL</th><th>STATUS</th><th>RINCIAN KELAS</th></tr></thead>
                  <tbody>
                    <template v-for="(guruList, mapel) in groupedRekapKekurangan" :key="mapel">
                      <tr class="row-group-clean"><td colspan="6">{{ mapel.toUpperCase() }} - {{ guruList.length }} Guru Kurang Jam</td></tr>
                      <tr v-for="rekap in guruList" :key="rekap.id">
                        <td><span class="fw-bold text-primary">{{ rekap.kode }}</span></td><td>{{ rekap.nama }}</td>
                        <td class="text-muted">{{ rekap.target_jam }} JP</td><td><span class="fw-bold">{{ rekap.totalJam }} JP</span></td>
                        <td class="text-danger fw-bold">Kurang {{ rekap.totalKurang }} JP</td>
                        <td><div style="display:flex;gap:4px;flex-wrap:wrap;"><span v-for="(detail, idx) in rekap.detailKurang" :key="idx" class="chip-label-warning">{{ detail }}</span></div></td>
                      </tr>
                    </template>
                    <tr v-if="rekapKekurangan.length === 0 && teachers.length > 0"><td colspan="6" class="empty-state">Aman Terkendali! Seluruh jadwal SK sudah terpenuhi.</td></tr>
                  </tbody>
                </table>
              </div>
            </div>
            
            <div v-if="subRekap === 'jadwal'" class="panel-body-clean">
              <div style="margin-bottom: 1.5rem; display: flex; align-items: center; gap: 10px;">
                <select v-model="selectedGuruJadwal" class="input-underline" style="max-width: 300px; margin: 0; padding-top:4px; padding-bottom:4px;">
                  <option value="">Pilih Guru...</option>
                  <option v-for="guru in teachers" :key="guru.kode" :value="guru.kode">{{ guru.kode }} - {{ guru.nama }}</option>
                </select>
              </div>
              <div class="table-card" v-if="selectedGuruJadwal">
                <table class="modern-clean-table" style="max-width: 600px;">
                  <thead><tr><th style="width: 100px;">JAM</th><th style="width: 150px;">WAKTU</th><th>KELAS</th></tr></thead>
                  <tbody>
                    <template v-for="(jadwals, hari) in groupedJadwalGuru" :key="hari">
                      <tr class="row-group-clean"><td colspan="3">{{ hari.toUpperCase() }} ({{ jadwals.length }} JP)</td></tr>
                      <tr v-for="item in jadwals" :key="item.id">
                        <td><span class="time-badge">Jam {{ item.jam_ke }}</span></td><td class="font-mono text-muted">{{ item.waktu }}</td><td><span class="chip-label">{{ item.rombel }}</span></td>
                      </tr>
                    </template>
                    <tr v-if="Object.keys(groupedJadwalGuru).length === 0"><td colspan="3" class="empty-state">Belum ada jadwal mengajar.</td></tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </div>

        <div v-else-if="menuAktif === 'Pengaturan Cetak'" class="page-content fade-in">
          <div class="content-wrapper" style="max-width: 600px;">
            <div class="panel-header-clean flex-between">
              <div>
                <h2 class="section-title">Format Cetak Excel</h2>
                <p class="section-subtitle">Kop surat dan TTD File Jadwal</p>
              </div>
              <button @click="exportKeExcel()" class="btn-solid-success">📥 Unduh Excel</button>
            </div>
            <div class="panel-body-clean" style="padding-top: 1rem;">
              <div class="form-group"><label>Nama Sekolah</label><input v-model="pengaturanCetak.namaSekolah" class="input-underline"></div>
              <div class="form-group"><label>Tahun Pelajaran</label><input v-model="pengaturanCetak.tahunPelajaran" class="input-underline"></div>
              <div class="form-group"><label>Tanggal Berlaku</label><input v-model="pengaturanCetak.tanggalBerlaku" class="input-underline"></div>
              <div style="height: 1px; background: #e2e8f0; margin: 1.5rem 0;"></div>
              <div class="form-group"><label>Tempat & Tanggal TTD</label><input v-model="pengaturanCetak.tempatTanggal" class="input-underline"></div>
              <div class="form-group"><label>Nama Kepala Sekolah</label><input v-model="pengaturanCetak.namaKepsek" class="input-underline"></div>
              <div class="form-group"><label>NIP Kepala Sekolah</label><input v-model="pengaturanCetak.nipKepsek" class="input-underline"></div>
              <button @click="simpanPengaturanCetak()" class="btn-solid" style="width: 100%; margin-top: 10px;">Simpan Pengaturan</button>
            </div>
          </div>
        </div>

        <div v-if="modalAturKelas.tampil" class="modal-overlay" @click.self="modalAturKelas.tampil = false"><div class="modal-box"><div class="modal-header"><h3>Pilih Kelas: {{ modalAturKelas.guru.nama }}</h3><button @click="modalAturKelas.tampil = false" class="btn-close-modal">✕</button></div><div class="modal-body"><div class="rombel-selector"><label v-for="r in rombels" :key="r" class="rombel-checkbox"><input type="checkbox" :value="r" v-model="modalAturKelas.terpilih"><span class="checkbox-visual">{{ r }}</span></label></div></div><div class="modal-actions"><button @click="modalAturKelas.tampil = false" class="btn-outline">Batal</button><button @click="simpanPenugasan()" class="btn-solid">Simpan</button></div></div></div>
        <div v-if="modalGuru.tampil" class="modal-overlay" @click.self="modalGuru.tampil = false"><div class="modal-box"><div class="modal-header"><h3>{{ modalGuru.form.id ? 'Edit Guru' : 'Tambah Guru' }}</h3><button @click="modalGuru.tampil = false" class="btn-close-modal">✕</button></div><div class="modal-body"><div class="form-group"><label>Kode (Maks 4 Huruf)</label><input v-model="modalGuru.form.kode" maxlength="4" class="input-underline"></div><div class="form-group"><label>Nama Lengkap</label><input v-model="modalGuru.form.nama" class="input-underline"></div><div class="form-group"><label>Mata Pelajaran</label><input v-model="modalGuru.form.mapel" class="input-underline"></div><div style="display: flex; gap: 15px;"><div class="form-group" style="flex: 1;"><label>Durasi JP</label><input v-model="modalGuru.form.durasi_mapel" type="number" min="1" class="input-underline"></div><div class="form-group" style="flex: 1;"><label>Target SK</label><input v-model="modalGuru.form.target_jam" type="number" min="0" class="input-underline"></div></div></div><div class="modal-actions"><button @click="modalGuru.tampil = false" class="btn-outline">Batal</button><button @click="simpanGuru()" class="btn-solid">Simpan</button></div></div></div>
        <div v-if="modalImport.tampil" class="modal-overlay" @click.self="modalImport.tampil = false"><div class="modal-box import-box"><div class="modal-header"><h3>Import via Excel</h3><button @click="modalImport.tampil = false" class="btn-close-modal">✕</button></div><div class="modal-body"><textarea v-model="modalImport.teksData" rows="6" class="input-underline font-mono" placeholder="Paste data dari Excel di sini..."></textarea></div><div class="modal-actions"><button @click="modalImport.tampil = false" class="btn-outline">Batal</button><button @click="prosesImport()" class="btn-solid">Proses</button></div></div></div>
        <div v-if="modalRombel.tampil" class="modal-overlay" @click.self="modalRombel.tampil = false"><div class="modal-box"><div class="modal-header"><h3>{{ modalRombel.isManual ? 'Tambah Rombel' : 'Generate Rombel' }}</h3><button @click="modalRombel.tampil = false" class="btn-close-modal">✕</button></div><div class="modal-body"><div v-if="modalRombel.isManual" class="form-group"><label>Nama Rombel</label><input v-model="modalRombel.form.nama" class="input-underline"></div><div v-else><div class="form-group"><label>Tingkat (Contoh: 7)</label><input v-model="modalRombel.form.tingkat" class="input-underline"></div><div class="form-group"><label>Jumlah Rombel</label><input v-model="modalRombel.form.jumlah" type="number" class="input-underline"></div></div></div><div class="modal-actions"><button @click="modalRombel.tampil = false" class="btn-outline">Batal</button><button @click="simpanRombel()" class="btn-solid">Simpan</button></div></div></div>
        <div v-if="modalWaktu.tampil" class="modal-overlay" @click.self="modalWaktu.tampil = false"><div class="modal-box"><div class="modal-header"><h3>{{ modalWaktu.form.id ? 'Edit Jam' : 'Tambah Jam' }}</h3><button @click="modalWaktu.tampil = false" class="btn-close-modal">✕</button></div><div class="modal-body"><div class="form-group"><label>Hari</label><select v-model="modalWaktu.form.hari" class="input-underline"><option v-for="h in ['Senin','Selasa','Rabu','Kamis','Jumat']" :key="h" :value="h">{{ h }}</option></select></div><div class="form-group"><label>Tipe Waktu</label><select v-model="modalWaktu.form.tipe" class="input-underline"><option value="pelajaran">Jam Pelajaran</option><option value="khusus">Kegiatan Khusus</option></select></div><div class="form-group"><label>Rentang Waktu</label><input v-model="modalWaktu.form.waktu" class="input-underline" placeholder="Misal: 07.00 - 07.40"></div><div class="form-group" v-if="modalWaktu.form.tipe === 'pelajaran'"><label>Jam Ke-</label><input v-model="modalWaktu.form.jam_ke" type="number" class="input-underline"></div><div class="form-group" v-if="modalWaktu.form.tipe === 'khusus'"><label>Nama Kegiatan</label><input v-model="modalWaktu.form.nama_kegiatan" class="input-underline" placeholder="Misal: Upacara"></div></div><div class="modal-actions"><button @click="modalWaktu.tampil = false" class="btn-outline">Batal</button><button @click="simpanWaktu()" class="btn-solid">Simpan</button></div></div></div>
        
        <div v-if="modalLaranganGuru.tampil" class="modal-overlay" @click.self="modalLaranganGuru.tampil = false"><div class="modal-box"><div class="modal-header"><h3>Larangan Jam Guru</h3><button @click="modalLaranganGuru.tampil = false" class="btn-close-modal">✕</button></div><div class="modal-body"><div class="form-group"><label>Pilih Guru</label><select v-model="modalLaranganGuru.form.kode_guru" class="input-underline"><option v-for="g in teachers" :key="g.kode" :value="g.kode">{{ g.kode }} - {{ g.nama }}</option></select></div><div class="form-group"><label>Hari</label><select v-model="modalLaranganGuru.form.hari" class="input-underline"><option v-for="h in ['Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat']" :key="h" :value="h">{{ h }}</option></select></div><div class="form-group"><label>Dilarang pada Jam Ke-</label><input type="number" v-model="modalLaranganGuru.form.jam_ke" class="input-underline" placeholder="Misal: 1"></div></div><div class="modal-actions"><button @click="modalLaranganGuru.tampil = false" class="btn-outline">Batal</button><button @click="simpanLaranganGuru()" class="btn-solid">Kunci Slot</button></div></div></div>
        <div v-if="modalHariMapel.tampil" class="modal-overlay" @click.self="modalHariMapel.tampil = false"><div class="modal-box"><div class="modal-header"><h3>Hari Khusus Mapel</h3><button @click="modalHariMapel.tampil = false" class="btn-close-modal">✕</button></div><div class="modal-body"><div class="form-group"><label>Nama Mapel</label><input v-model="modalHariMapel.form.mapel" class="input-underline" placeholder="Contoh: OR"></div><div class="form-group"><label>Pilih Hari Yang Boleh</label><div style="display:flex; flex-wrap:wrap; gap:12px; margin-top:8px;"><label v-for="h in ['Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat']" :key="h" style="display:flex; align-items:center; gap:6px; font-size:0.8rem; cursor:pointer;"><input type="checkbox" :value="h" v-model="modalHariMapel.form.allowed_days"> {{ h }}</label></div></div></div><div class="modal-actions"><button @click="modalHariMapel.tampil = false" class="btn-outline">Batal</button><button @click="simpanHariMapel()" class="btn-solid">Simpan</button></div></div></div>

      </template>
    </template>
  </div>
</template>

<script setup>
import { ref, onMounted, reactive, computed } from 'vue'
import { supabase } from './lib/supabase' 

// =========================================================================
// 1. CONSTANTS
// =========================================================================
const urutanHari = { 'Senin': 1, 'Selasa': 2, 'Rabu': 3, 'Kamis': 4, 'Jumat': 5 }
const urutanMapel = ['AGM', 'MTK', 'IND', 'IPA', 'OR', 'ING', 'IPS', 'PKN', 'JWA', 'PRA', 'INF', 'BK']

// =========================================================================
// 2. STATE & VARIABLES
// =========================================================================
const isLoggedIn = ref(false)
const currentUser = ref(null)
const loginInput = ref('')
const isLoadingLogin = ref(false)

const isSidebarOpen = ref(false) 
const menuAktif = ref('Home') 
const subRekap = ref('audit')
const subPengecualian = ref('guru')

const teachers = ref([])
const masterWaktu = ref([])
const masterRombel = ref([])
const rombels = ref([])
const penugasanSK = ref([])
const allSchedules = ref([])
const rulesRestrictedSlots = ref([])
const rulesSubjectDays = ref([])
const selectedGuruJadwal = ref('')

const pengaturanCetak = reactive({
  namaSekolah: localStorage.getItem('cetak_sekolah') || 'SMP NEGERI 3 BANGUNTAPAN',
  tahunPelajaran: localStorage.getItem('cetak_tapel') || '2025/2026 SEMESTER 2',
  tanggalBerlaku: localStorage.getItem('cetak_tanggal') || '05 Januari 2026',
  tempatTanggal: localStorage.getItem('cetak_ttd_tempat') || 'Banguntapan, 2 Januari 2026',
  namaKepsek: localStorage.getItem('cetak_kepsek') || 'Fulan Nur fulan, M.Pd.',
  nipKepsek: localStorage.getItem('cetak_nip') || '19700xxxxxxxxxxxx'
})

// =========================================================================
// 3. HELPER FUNCTIONS
// =========================================================================
const getPenugasanGuru = (kode) => penugasanSK.value.filter(p => p.kode_guru === kode)
const getJadwalHariIni = () => masterWaktu.value.filter(w => w.hari === menuAktif.value)
const getScheduleData = (rombel, jamKe) => allSchedules.value.filter(s => s.id_rombel === rombel && s.jam_ke === `${menuAktif.value}-${jamKe}`)

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

const checkMappingLegality = (kodeGuru, mapel, hari, jamKe) => {
  const isBannedSlot = rulesRestrictedSlots.value.some(r => r.kode_guru === kodeGuru && r.hari === hari && parseInt(r.jam_ke) === parseInt(jamKe))
  if (isBannedSlot) return { legal: false, msg: `🚫 GURU ${kodeGuru} dilarang mengajar pada ${hari} Jam Ke-${jamKe}!` }
  
  const mapelRule = rulesSubjectDays.value.find(r => r.mapel.toUpperCase() === mapel.toUpperCase())
  if (mapelRule && !mapelRule.allowed_days.includes(hari)) return { legal: false, msg: `📅 MAPEL ${mapel} hanya boleh diajarkan pada hari: ${mapelRule.allowed_days.join(', ')}!` }
  
  return { legal: true }
}

const calculateTeachingHours = () => { teachers.value.forEach(t => { t.totalJam = allSchedules.value.filter(s => s.guru_mapel.startsWith(t.kode)).length }) }

// =========================================================================
// 4. COMPUTED PROPERTIES
// =========================================================================
const groupedTeachers = computed(() => {
  const groups = {}; 
  const guruAsli = teachers.value.filter(t => t.mapel.toUpperCase() !== 'SISTEM');
  guruAsli.forEach(guru => { const mapel = guru.mapel ? guru.mapel.toUpperCase() : 'LAINNYA'; if (!groups[mapel]) groups[mapel] = []; groups[mapel].push(guru); });
  const sortedKeys = Object.keys(groups).sort((a, b) => { let indexA = urutanMapel.indexOf(a); let indexB = urutanMapel.indexOf(b); if (indexA === -1) indexA = 999; if (indexB === -1) indexB = 999; return indexA - indexB || a.localeCompare(b); });
  const sortedGroups = {}; sortedKeys.forEach(key => sortedGroups[key] = groups[key]); return sortedGroups;
});

const groupedTeachersAll = computed(() => {
  const groups = {}; 
  teachers.value.forEach(guru => { const mapel = guru.mapel ? guru.mapel.toUpperCase() : 'LAINNYA'; if (!groups[mapel]) groups[mapel] = []; groups[mapel].push(guru); });
  const sortedKeys = Object.keys(groups).sort((a, b) => { let indexA = urutanMapel.indexOf(a); let indexB = urutanMapel.indexOf(b); if (indexA === -1) indexA = 999; if (indexB === -1) indexB = 999; return indexA - indexB || a.localeCompare(b); });
  const sortedGroups = {}; sortedKeys.forEach(key => sortedGroups[key] = groups[key]); return sortedGroups;
});

const groupedRombels = computed(() => {
  const groups = {};
  masterRombel.value.forEach(r => {
    const grade = r.nama_rombel.charAt(0);
    if (!groups[grade]) groups[grade] = [];
    groups[grade].push(r);
  });
  const sorted = {};
  Object.keys(groups).sort().forEach(k => {
    sorted[k] = groups[k].sort((a,b) => a.nama_rombel.localeCompare(b.nama_rombel));
  });
  return sorted;
});

const rekapKekurangan = computed(() => {
  return teachers.value.map(guru => {
    const penugasan = getPenugasanGuru(guru.kode); const jadwalGuruIni = allSchedules.value.filter(s => s.guru_mapel.startsWith(guru.kode)); let totalKurang = 0; let detailKurang = [];
    penugasan.forEach(tugas => { const aktualJpKelas = jadwalGuruIni.filter(s => s.id_rombel === tugas.nama_rombel).length; const kurang = tugas.durasi_jp - aktualJpKelas; if (kurang > 0) { totalKurang += kurang; detailKurang.push(`${tugas.nama_rombel} (-${kurang} JP)`); } });
    return { ...guru, totalKurang, detailKurang };
  }).filter(g => g.totalKurang > 0); 
});

const groupedRekapKekurangan = computed(() => {
  const groups = {}; rekapKekurangan.value.forEach(guru => { const mapel = guru.mapel ? guru.mapel.toUpperCase() : 'LAINNYA'; if (!groups[mapel]) groups[mapel] = []; groups[mapel].push(guru); });
  const sortedKeys = Object.keys(groups).sort((a, b) => { let indexA = urutanMapel.indexOf(a); let indexB = urutanMapel.indexOf(b); if (indexA === -1) indexA = 999; if (indexB === -1) indexB = 999; return indexA - indexB || a.localeCompare(b); });
  const sortedGroups = {}; sortedKeys.forEach(key => sortedGroups[key] = groups[key]); return sortedGroups;
});

const rekapHarian = computed(() => {
  return teachers.value
    .filter(guru => guru.mapel.toUpperCase() !== 'SISTEM') // Misi 1: Sembunyikan SISTEM dari rekap
    .map(guru => {
      const jadwalGuruIni = allSchedules.value.filter(s => s.guru_mapel.startsWith(guru.kode)); const hitungan = { Senin: 0, Selasa: 0, Rabu: 0, Kamis: 0, Jumat: 0 };
      jadwalGuruIni.forEach(j => { const hari = j.jam_ke.split('-')[0]; if (hitungan[hari] !== undefined) hitungan[hari]++; });
      return { kode: guru.kode, nama: guru.nama, mapel: guru.mapel, target_jam: guru.target_jam, Senin: hitungan.Senin, Selasa: hitungan.Selasa, Rabu: hitungan.Rabu, Kamis: hitungan.Kamis, Jumat: hitungan.Jumat, Total: jadwalGuruIni.length };
    }).sort((a, b) => b.Total - a.Total);
});

const groupedRekapHarian = computed(() => {
  const groups = {}; rekapHarian.value.forEach(guru => { const mapel = guru.mapel ? guru.mapel.toUpperCase() : 'LAINNYA'; if (!groups[mapel]) groups[mapel] = []; groups[mapel].push(guru); });
  const sortedKeys = Object.keys(groups).sort((a, b) => { let indexA = urutanMapel.indexOf(a); let indexB = urutanMapel.indexOf(b); if (indexA === -1) indexA = 999; if (indexB === -1) indexB = 999; return indexA - indexB || a.localeCompare(b); });
  const sortedGroups = {}; sortedKeys.forEach(key => sortedGroups[key] = groups[key]); return sortedGroups;
});

const groupedJadwalGuru = computed(() => {
  if (!selectedGuruJadwal.value) return {}; const teacher = teachers.value.find(t => t.kode === selectedGuruJadwal.value); if (!teacher) return {};
  const jadwalGuru = allSchedules.value.filter(s => s.guru_mapel.startsWith(teacher.kode)); const groups = {};
  jadwalGuru.forEach(j => { const [hari, jamKe] = j.jam_ke.split('-'); const waktuObj = masterWaktu.value.find(w => w.hari === hari && w.jam_ke == jamKe && w.tipe === 'pelajaran'); if (!groups[hari]) groups[hari] = []; groups[hari].push({ id: j.id, hari: hari, jam_ke: parseInt(jamKe), waktu: waktuObj ? waktuObj.waktu : '-', rombel: j.id_rombel }); });
  const sortedKeys = Object.keys(groups).sort((a, b) => urutanHari[a] - urutanHari[b]); const sortedGroups = {}; sortedKeys.forEach(key => { sortedGroups[key] = groups[key].sort((a, b) => a.jam_ke - b.jam_ke); });
  return sortedGroups;
});

const myRekap = computed(() => {
  if (!currentUser.value) return null;
  return rekapHarian.value.find(r => r.kode === currentUser.value.kode) || { Senin: 0, Selasa: 0, Rabu: 0, Kamis: 0, Jumat: 0, Total: 0, target_jam: currentUser.value.target_jam || 0 };
});

// =========================================================================
// 5. FETCH DATA DENGAN FILTER KAMAR (KODE_SEKOLAH)
// =========================================================================
const fetchData = async () => {
  if(!currentUser.value) return;
  const ks = currentUser.value.kode_sekolah; // Ambil kunci kamar dari user yg login

  const { data: rData } = await supabase.from('master_rombel').select('*').eq('kode_sekolah', ks).order('nama_rombel'); masterRombel.value = rData || []; rombels.value = masterRombel.value.map(r => r.nama_rombel);
  const { data: gData } = await supabase.from('guru').select('*').eq('kode_sekolah', ks).order('kode'); teachers.value = gData || [];
  
  if (currentUser.value && currentUser.value.role === 'guru') {
     const me = teachers.value.find(t => t.kode === currentUser.value.kode);
     if (me) currentUser.value.target_jam = me.target_jam;
  }

  const { data: wData } = await supabase.from('master_waktu').select('*').eq('kode_sekolah', ks); 
  masterWaktu.value = (wData || []).sort((a,b) => urutanHari[a.hari] - urutanHari[b.hari] || a.waktu.localeCompare(b.waktu));
  
  const { data: pData } = await supabase.from('penugasan_sk').select('*').eq('kode_sekolah', ks); penugasanSK.value = pData || [];
  const { data: jData } = await supabase.from('jadwal').select('*').eq('kode_sekolah', ks); allSchedules.value = jData || [];
  
  try {
    const { data: rules1, error: e1 } = await supabase.from('rules_restricted_slots').select('*').eq('kode_sekolah', ks);
    if (!e1) rulesRestrictedSlots.value = rules1 || [];
    const { data: rules2, error: e2 } = await supabase.from('rules_subject_days').select('*').eq('kode_sekolah', ks);
    if (!e2) rulesSubjectDays.value = rules2 || [];
  } catch (error) { console.log("Tabel aturan belum ada di Supabase, fallback local."); }
  
  calculateTeachingHours();
}

// =========================================================================
// 6. ACTIONS & LOGIC (MODALS, AUTO-DRAFT, EXCEL, LOGIN)
// =========================================================================
const pilihMenu = (menu) => { menuAktif.value = menu; isSidebarOpen.value = false; }
const doLogin = async () => {
  if (!loginInput.value) return; isLoadingLogin.value = true; const kode = loginInput.value.trim().toUpperCase();
  const { data } = await supabase.from('guru').select('*').eq('kode', kode).single();
  if (data) {
    const userRole = data.mapel.toUpperCase() === 'SISTEM' ? 'admin' : 'guru';
    // Simpan kode_sekolah ke memori agar tau masuk kamar mana
    currentUser.value = { kode: data.kode, nama: data.nama, role: userRole, target_jam: data.target_jam, kode_sekolah: data.kode_sekolah };
    localStorage.setItem('smartschedule_user', JSON.stringify(currentUser.value));
    isLoggedIn.value = true; menuAktif.value = userRole === 'admin' ? 'Home' : 'Dashboard Saya';
    selectedGuruJadwal.value = data.kode; fetchData();
  } else { alert('❌ Kode Guru tidak ditemukan!'); }
  isLoadingLogin.value = false;
}
const doLogout = () => { if(confirm('Yakin ingin logout?')) { localStorage.removeItem('smartschedule_user'); isLoggedIn.value = false; currentUser.value = null; loginInput.value = ''; isSidebarOpen.value = false; } }

const resetSemuaJadwal = async () => { 
  if (!confirm("HAPUS SEMUA JADWAL?")) return; 
  // Hapus hanya yg ada di kamar ini
  const { error } = await supabase.from('jadwal').delete().eq('kode_sekolah', currentUser.value.kode_sekolah).not('id', 'is', null); 
  if (!error) { alert("Jadwal dikosongkan!"); fetchData(); } 
}
const removeJadwal = async (id) => { await supabase.from('jadwal').delete().eq('id', id); fetchData(); }

const startDrag = (evt, kodeGuru, mapel) => { evt.dataTransfer.setData('kodeGuru', kodeGuru); evt.dataTransfer.setData('mapel', mapel); }
const onDrop = async (evt, rombel, jamKe) => {
  const kodeGuru = evt.dataTransfer.getData('kodeGuru'); const mapel = evt.dataTransfer.getData('mapel');
  if(!kodeGuru) return;

  const validasi = checkMappingLegality(kodeGuru, mapel, menuAktif.value, jamKe);
  if (!validasi.legal) { alert(validasi.msg); return; }

  const isDitugaskan = penugasanSK.value.some(p => p.kode_guru === kodeGuru && p.nama_rombel === rombel);
  if (!isDitugaskan) { alert(`❌ Guru ${kodeGuru} tidak ditugaskan untuk kelas ${rombel} di SK.`); return; }

  const penandaWaktu = `${menuAktif.value}-${jamKe}`;
  const isSibuk = allSchedules.value.some(s => s.jam_ke === penandaWaktu && s.guru_mapel.startsWith(kodeGuru));
  if (isSibuk) { alert(`⚠️ Guru ${kodeGuru} sudah mengajar di kelas lain pada jam ini.`); return; }

  await supabase.from('jadwal').insert([{ 
    id_rombel: rombel, 
    jam_ke: penandaWaktu, 
    guru_mapel: `${kodeGuru} - ${mapel}`,
    kode_sekolah: currentUser.value.kode_sekolah 
  }]); 
  fetchData();
}

const generateJadwalCerdas = async () => {
  if(!confirm("Mulai Auto-Draft dengan Aturan Khusus?")) return;
  const jadwalTerisi = new Set(allSchedules.value.map(s => `${s.id_rombel}_${s.jam_ke}`)); 
  const guruSibuk = new Set(allSchedules.value.map(s => `${s.guru_mapel.split(' - ')[0]}_${s.jam_ke}`)); 
  const jamPelajaran = masterWaktu.value.filter(w => w.tipe === 'pelajaran');
  const payload = [];
  let trackerJamGuru = {}; teachers.value.forEach(t => trackerJamGuru[t.kode] = t.totalJam);
  
  const tumpukanTugas = [...penugasanSK.value].sort((a, b) => {
    const gA = teachers.value.find(t => t.kode === a.kode_guru);
    const gB = teachers.value.find(t => t.kode === b.kode_guru);
    const isOrA = gA && gA.mapel.toUpperCase() === 'OR';
    const isOrB = gB && gB.mapel.toUpperCase() === 'OR';
    
    if (isOrA && !isOrB) return -1;
    if (!isOrA && isOrB) return 1;
    return b.durasi_jp - a.durasi_jp;
  });

  for (const tugas of tumpukanTugas) {
    const guru = teachers.value.find(t => t.kode === tugas.kode_guru); if (!guru) continue;
    const sisa = tugas.durasi_jp - allSchedules.value.filter(s => s.id_rombel === tugas.nama_rombel && s.guru_mapel.startsWith(guru.kode)).length;
    if (sisa <= 0) continue;

    let blokJp = sisa === 6 ? [3, 3] : (sisa === 5 ? [3, 2] : (sisa >= 4 ? [2, 2] : [sisa]));
    let hariTerpakaiRombelIni = new Set();

    for (const ukuran of blokJp) {
      let berhasil = false; const urutanHariAcak = ['Senin', 'Selasa', 'Rabu', 'Kamis', 'Jumat'].sort(() => Math.random() - 0.5);
      for (const hari of urutanHariAcak) {
        if (berhasil || hariTerpakaiRombelIni.has(hari)) continue;
        const checkMapel = checkMappingLegality(guru.kode, guru.mapel, hari, 1);
        if(!checkMapel.legal && checkMapel.msg.includes("MAPEL")) continue;

        let slotHariIni = jamPelajaran.filter(j => j.hari === hari); let run = [];
        for (const sesi of slotHariIni) {
          
          if (guru.mapel.toUpperCase() === 'OR' && parseInt(sesi.jam_ke) > 7) {
            run = []; continue;
          }

          const pnd = `${sesi.hari}-${sesi.jam_ke}`;
          const checkSlot = checkMappingLegality(guru.kode, guru.mapel, hari, sesi.jam_ke);

          if (!jadwalTerisi.has(`${tugas.nama_rombel}_${pnd}`) && !guruSibuk.has(`${guru.kode}_${pnd}`) && checkSlot.legal) {
            run.push(pnd);
            if (run.length === ukuran) {
              run.forEach(p => { 
                jadwalTerisi.add(`${tugas.nama_rombel}_${p}`); guruSibuk.add(`${guru.kode}_${p}`); 
                payload.push({ 
                  id_rombel: tugas.nama_rombel, 
                  jam_ke: p, 
                  guru_mapel: `${guru.kode} - ${guru.mapel}`,
                  kode_sekolah: currentUser.value.kode_sekolah 
                }); 
              });
              hariTerpakaiRombelIni.add(hari); berhasil = true; break;
            }
          } else { run = []; }
        }
      }
    }
  }
  if (payload.length > 0) { await supabase.from('jadwal').insert(payload); fetchData(); } 
  else alert("Tugas selesai, sisa jadwal tidak menemukan slot kosong sesuai aturan.");
}

const simpanPengaturanCetak = () => { localStorage.setItem('cetak_sekolah', pengaturanCetak.namaSekolah); localStorage.setItem('cetak_tapel', pengaturanCetak.tahunPelajaran); localStorage.setItem('cetak_tanggal', pengaturanCetak.tanggalBerlaku); localStorage.setItem('cetak_ttd_tempat', pengaturanCetak.tempatTanggal); localStorage.setItem('cetak_kepsek', pengaturanCetak.namaKepsek); localStorage.setItem('cetak_nip', pengaturanCetak.nipKepsek); alert('Pengaturan Cetak Berhasil Disimpan!'); };
const exportKeExcel = () => { /* KODE EXCEL SAMA */ }

// =========================================================================
// 7. MODAL HANDLERS (DENGAN KODE SEKOLAH)
// =========================================================================
const modalLaranganGuru = reactive({ tampil: false, form: { kode_guru: '', hari: 'Senin', jam_ke: 1 } });
const bukaModalLaranganGuru = () => { modalLaranganGuru.tampil = true; };
const simpanLaranganGuru = async () => { 
  const payload = { ...modalLaranganGuru.form, kode_sekolah: currentUser.value.kode_sekolah };
  try { const { data, error } = await supabase.from('rules_restricted_slots').insert([payload]).select(); if (error) throw error; if (data && data.length) rulesRestrictedSlots.value.push(data[0]); } catch (err) { rulesRestrictedSlots.value.push({ ...payload, id: Date.now() }); }
  modalLaranganGuru.tampil = false; 
}
const hapusLaranganGuru = async (id) => { rulesRestrictedSlots.value = rulesRestrictedSlots.value.filter(r => r.id !== id); try { await supabase.from('rules_restricted_slots').delete().eq('id', id); } catch(e) {} }

const modalHariMapel = reactive({ tampil: false, form: { mapel: '', allowed_days: [] } });
const bukaModalHariMapel = () => { modalHariMapel.tampil = true; };
const simpanHariMapel = async () => { 
  const payload = { ...modalHariMapel.form, mapel: modalHariMapel.form.mapel.toUpperCase(), kode_sekolah: currentUser.value.kode_sekolah };
  try { const { data, error } = await supabase.from('rules_subject_days').insert([payload]).select(); if (error) throw error; if (data && data.length) rulesSubjectDays.value.push(data[0]); } catch (err) { rulesSubjectDays.value.push({ ...payload, id: Date.now() }); }
  modalHariMapel.tampil = false; 
}
const hapusHariMapel = async (id) => { rulesSubjectDays.value = rulesSubjectDays.value.filter(r => r.id !== id); try { await supabase.from('rules_subject_days').delete().eq('id', id); } catch(e) {} }

const modalAturKelas = reactive({ tampil: false, guru: {}, terpilih: [] })
const bukaModalAturKelas = (guru) => { modalAturKelas.guru = guru; modalAturKelas.terpilih = getPenugasanGuru(guru.kode).map(p => p.nama_rombel); modalAturKelas.tampil = true; }
const simpanPenugasan = async () => { 
  const kode = modalAturKelas.guru.kode; const durasi = modalAturKelas.guru.durasi_mapel; 
  await supabase.from('penugasan_sk').delete().eq('kode_guru', kode).eq('kode_sekolah', currentUser.value.kode_sekolah); 
  const payload = modalAturKelas.terpilih.map(rombel => ({ kode_guru: kode, nama_rombel: rombel, durasi_jp: durasi, kode_sekolah: currentUser.value.kode_sekolah })); 
  if (payload.length > 0) await supabase.from('penugasan_sk').insert(payload); modalAturKelas.tampil = false; fetchData(); 
}

const modalGuru = reactive({ tampil: false, form: { id: null, kode: '', nama: '', mapel: '', target_jam: 0, durasi_mapel: 2 } })
const bukaModalGuru = (guru = null) => { modalGuru.form = guru ? { ...guru } : { id: null, kode: '', nama: '', mapel: '', target_jam: 0, durasi_mapel: 2 }; modalGuru.tampil = true }
const simpanGuru = async () => { 
  const data = { kode: modalGuru.form.kode.toUpperCase(), nama: modalGuru.form.nama, mapel: modalGuru.form.mapel, target_jam: modalGuru.form.target_jam, durasi_mapel: modalGuru.form.durasi_mapel, kode_sekolah: currentUser.value.kode_sekolah }; 
  if (modalGuru.form.id) await supabase.from('guru').update(data).eq('id', modalGuru.form.id); 
  else await supabase.from('guru').insert([data]); modalGuru.tampil = false; fetchData() 
}
const hapusGuru = async (id) => { if(confirm('Hapus guru?')) { await supabase.from('guru').delete().eq('id', id); fetchData() } }

const modalImport = reactive({ tampil: false, jenis: 'guru', teksData: '' })
const bukaModalImportGuru = () => { modalImport.jenis = 'guru'; modalImport.teksData = ''; modalImport.tampil = true }
const bukaModalImportWaktu = () => { modalImport.jenis = 'waktu'; modalImport.teksData = ''; modalImport.tampil = true }
const prosesImport = async () => { 
  const barisData = modalImport.teksData.split('\n'); const payload = []; 
  barisData.forEach((baris) => { 
    const kolom = baris.split('\t'); 
    if (modalImport.jenis === 'guru' && kolom.length >= 5) { 
      payload.push({ kode: kolom[0].trim().toUpperCase().substring(0,4), nama: kolom[1].trim(), mapel: kolom[2].trim(), durasi_mapel: parseInt(kolom[3].trim()) || 2, target_jam: parseInt(kolom[4].trim()) || 0, kode_sekolah: currentUser.value.kode_sekolah }) 
    } else if (modalImport.jenis === 'waktu' && kolom.length >= 4) { 
      payload.push({ hari: kolom[0].trim(), tipe: kolom[1].trim().toLowerCase(), jam_ke: kolom[2].trim() === '-' ? '' : kolom[2].trim(), waktu: kolom[3].trim(), nama_kegiatan: kolom[4] ? kolom[4].trim() : '', kode_sekolah: currentUser.value.kode_sekolah }) 
    } 
  }); 
  if (payload.length > 0) { const tabel = modalImport.jenis === 'guru' ? 'guru' : 'master_waktu'; await supabase.from(tabel).insert(payload); } modalImport.tampil = false; fetchData(); 
}

const modalRombel = reactive({ tampil: false, isManual: false, form: { tingkat: '', jumlah: 1, nama: '' } })
const bukaModalRombel = () => { modalRombel.isManual = false; modalRombel.form = { tingkat: '7', jumlah: 7, nama: '' }; modalRombel.tampil = true }
const bukaModalRombelManual = () => { modalRombel.isManual = true; modalRombel.form = { tingkat: '', jumlah: 1, nama: '' }; modalRombel.tampil = true }
const simpanRombel = async () => { 
  const payload = []; 
  if (modalRombel.isManual) payload.push({ nama_rombel: modalRombel.form.nama, kode_sekolah: currentUser.value.kode_sekolah }); 
  else for (let i = 0; i < modalRombel.form.jumlah; i++) payload.push({ nama_rombel: modalRombel.form.tingkat + String.fromCharCode(65 + i), kode_sekolah: currentUser.value.kode_sekolah }); 
  await supabase.from('master_rombel').insert(payload); modalRombel.tampil = false; fetchData(); 
}
const hapusRombel = async (id, nama) => { if(confirm(`Hapus ${nama}?`)) { await supabase.from('jadwal').delete().eq('id_rombel', nama).eq('kode_sekolah', currentUser.value.kode_sekolah); await supabase.from('master_rombel').delete().eq('id', id); fetchData(); } }

const modalWaktu = reactive({ tampil: false, form: { id: null, hari: 'Senin', tipe: 'pelajaran', jam_ke: '', waktu: '', nama_kegiatan: '' } })
const bukaModalWaktu = (waktu = null) => { modalWaktu.form = waktu ? { ...waktu } : { id: null, hari: 'Senin', tipe: 'pelajaran', jam_ke: '', waktu: '', nama_kegiatan: '' }; modalWaktu.tampil = true }
const simpanWaktu = async () => { 
  const data = { ...modalWaktu.form, kode_sekolah: currentUser.value.kode_sekolah }; 
  if (data.tipe === 'pelajaran') data.nama_kegiatan = ''; if (data.tipe === 'khusus') data.jam_ke = ''; 
  if (data.id) await supabase.from('master_waktu').update(data).eq('id', data.id); 
  else { delete data.id; await supabase.from('master_waktu').insert([data]) } modalWaktu.tampil = false; fetchData() 
}
const hapusWaktu = async (id) => { await supabase.from('master_waktu').delete().eq('id', id); fetchData() }

// =========================================================================
// 8. LIFECYCLE
// =========================================================================
onMounted(() => { 
  const saved = localStorage.getItem('smartschedule_user'); 
  if (saved) { currentUser.value = JSON.parse(saved); isLoggedIn.value = true; menuAktif.value = currentUser.value.role === 'admin' ? 'Home' : 'Dashboard Saya'; fetchData(); } 
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Cantarell&display=swap');

* { box-sizing: border-box; font-family: 'Cantarell', sans-serif; }
body { margin: 0; background-color: #f8fafc; color: #334155; }

/* ==========================================
   GLOBAL UTILITIES
   ========================================== */
.app-container { height: 100vh; display: flex; flex-direction: column; overflow: hidden; background-color: #ffffff;}
.text-primary { color: #4f46e5; }
.text-danger { color: #e11d48 !important; }
.text-success { color: #16a34a !important; }
.text-dark { color: #0f172a; }
.text-muted { color: #64748b; }
.fw-bold { font-weight: 700; }
.fade-in { animation: fadeIn 0.3s ease; }
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

/* ==========================================
   STYLING UI LOGIN (MODERN MINIMALIS GHOST)
   ========================================== */
.login-wrapper { display: flex; align-items: center; justify-content: center; height: 100vh; background: #ffffff; }
.modern-login { background: #ffffff; padding: 2.5rem; width: 100%; max-width: 320px; text-align: center; }
.logo-circle { margin-bottom: 1rem; display: flex; justify-content: center;}
.modern-login h1 { font-size: 1.4rem; color: #0f172a; margin: 0 0 4px 0; font-weight: 900; letter-spacing: -0.5px; }
.modern-login p { margin: 0 0 2rem 0; color: #94a3b8; font-size: 0.8rem; font-weight: 500;}
.login-input-modern { width: 100%; text-transform: uppercase; text-align: center; font-size: 1.1rem; font-weight: 700; letter-spacing: 2px; padding: 12px 0; margin-bottom: 1.5rem; background: transparent; border: none; border-bottom: 2px solid #e2e8f0; border-radius: 0; outline: none; transition: 0.2s; color: #0f172a; }
.login-input-modern::placeholder { color: #cbd5e1; font-weight: 500; letter-spacing: normal; font-size: 0.85rem;}
.login-input-modern:focus { border-bottom-color: #0f172a; }
.btn-login-modern { width: 100%; padding: 12px; font-size: 0.85rem; font-weight: 700; letter-spacing: 0.5px; background: #0f172a; color: #ffffff; border: none; border-radius: 8px; cursor: pointer; transition: 0.2s; }
.btn-login-modern:hover { background: #334155; transform: translateY(-1px); box-shadow: 0 4px 6px -1px rgba(15, 23, 42, 0.2); }

/* ==========================================
   HEADER TOP NAVBAR (POLOS & LEGA)
   ========================================== */
.navbar-modern { background: #ffffff; border-bottom: 1px solid #f1f5f9; padding: 0.6rem 1.2rem; display: flex; justify-content: space-between; align-items: center; flex-shrink: 0; width: 100%; z-index: 50;}
.nav-left { display: flex; align-items: center; gap: 12px; }
.btn-hamburger { background: transparent; border: none; cursor: pointer; color: #0f172a; padding: 4px; border-radius: 6px; transition: 0.2s; display: flex; align-items: center; justify-content: center;}
.btn-hamburger:hover { background: #f8fafc; }
.brand { display: flex; align-items: center; gap: 8px; border-left: 1px solid #e2e8f0; padding-left: 12px; cursor: pointer; transition: 0.2s;}
.brand:hover { opacity: 0.7; }
.brand-title { font-size: 1rem; color: #0f172a; font-weight: 900; letter-spacing: -0.3px;}
.brand-subtitle { font-size: 0.75rem; color: #94a3b8; font-weight: 500; text-transform: uppercase;}

/* USER PROFILE IN HEADER DIHILANGKAN, DIPINDAH KE SIDEBAR */
.user-profile { display: none; }

/* ==========================================
   SIDEBAR MENU HAMBURGER (ONE SCREEN - CLEAN)
   ========================================== */
.sidebar { position: fixed; top: 0; left: -240px; width: 240px; height: 100vh; background: #ffffff; z-index: 10000; transition: left 0.3s cubic-bezier(0.4, 0, 0.2, 1); border-right: 1px solid #f1f5f9; display: flex; flex-direction: column; box-shadow: 10px 0 25px rgba(0,0,0,0.03);}
.sidebar.open { left: 0; }
.sidebar-overlay { position: fixed; inset: 0; background: rgba(15, 23, 42, 0.2); z-index: 9999; backdrop-filter: blur(2px); transition: 0.3s;}
.sidebar-header { padding: 1.5rem 1.2rem; display: flex; justify-content: space-between; align-items: center; }
.sidebar-header h2 { font-size: 0.8rem; font-weight: 900; color: #0f172a; margin: 0; letter-spacing: 0.5px;}
.btn-close-sidebar { background: transparent; border: none; cursor: pointer; color: #94a3b8; padding: 0; transition: 0.2s; display: flex; align-items: center;}
.btn-close-sidebar:hover { color: #0f172a; }

.sidebar-menu { padding: 0; overflow-y: auto; flex: 1; }
.menu-label { font-size: 0.6rem; font-weight: 700; color: #cbd5e1; padding: 1rem 1.2rem 0.2rem; text-transform: uppercase; letter-spacing: 1px;}
.sidebar-menu button { display: flex; align-items: center; gap: 10px; width: 100%; padding: 0.45rem 1.2rem; background: none; border: none; font-size: 0.85rem; font-weight: 500; color: #475569; cursor: pointer; transition: 0.2s; text-align: left; border-right: 3px solid transparent;}
.sidebar-menu button:hover { color: #0f172a; }
.sidebar-menu button.aktif { color: #4f46e5; font-weight: 700; border-right-color: #4f46e5; background: #f8fafc;}

/* FOOTER SIDEBAR (PROFIL & LOGOUT - CLEAN VERSION) */
.sidebar-footer { padding: 1.5rem 1.2rem; border-top: 1px solid #f1f5f9; background: #ffffff; text-align: center;}
.user-info-clean { display: flex; flex-direction: column; margin-bottom: 1rem;}
.u-name { font-size: 1.5rem; font-weight: 900; color: #0f172a; letter-spacing: 1px;}
.u-role { font-size: 0.7rem; font-weight: 700; color: #64748b; letter-spacing: 0.5px;}
.btn-logout-full { width: 100%; padding: 10px; display: flex; align-items: center; justify-content: center; background: #ffffff; border: 1px solid #e2e8f0; border-radius: 8px; font-size: 0.8rem; font-weight: 700; color: #64748b; cursor: pointer; transition: 0.2s;}
.btn-logout-full:hover { background: #fef2f2; color: #e11d48; border-color: #fecaca;}

/* ==========================================
   LAYOUT & PANELS (CLEAN SPACING)
   ========================================== */
.page-content { padding: 2rem 1.5rem; flex: 1; overflow-y: auto; background-color: #ffffff;}
.content-wrapper { max-width: 1000px; margin: 0 auto; display: flex; flex-direction: column; gap: 1.5rem;}

.panel-header-clean { margin-bottom: 1.5rem; }
.panel-header-column { display: flex; flex-direction: column; gap: 15px; margin-bottom: 1.5rem;}
.flex-between { display: flex; justify-content: space-between; align-items: flex-end;}

.section-title { font-size: 1.6rem; font-weight: 900; margin: 0 0 4px 0; color: #0f172a; letter-spacing: -0.5px;}
.section-subtitle { font-size: 0.85rem; color: #64748b; margin: 0;}
.info-banner { background: #f8fafc; padding: 12px 16px; font-size: 0.8rem; font-weight: 500; color: #475569; border-radius: 8px;}

/* ==========================================
   BUTTONS & TABS (GHOST / OUTLINE STYLE)
   ========================================== */
.action-buttons { display: flex; gap: 8px; align-items: center; }
.btn-solid { background: #0f172a; color: white; border: none; padding: 8px 16px; border-radius: 8px; font-size: 0.8rem; font-weight: 700; cursor: pointer; transition: 0.2s;}
.btn-solid:hover { background: #334155; }
.btn-outline { background: #ffffff; color: #0f172a; border: 1px solid #e2e8f0; padding: 8px 16px; border-radius: 8px; font-size: 0.8rem; font-weight: 700; cursor: pointer; transition: 0.2s;}
.btn-outline:hover { background: #f8fafc; border-color: #cbd5e1; }
.btn-solid-success { background: #16a34a; color: white; border: none; padding: 8px 16px; border-radius: 8px; font-size: 0.8rem; font-weight: 700; cursor: pointer;}
.btn-solid-success:hover { background: #15803d; }

.btn-icon-outline { background: #ffffff; border: 1px solid #e2e8f0; font-size: 0.8rem; padding: 6px; cursor: pointer; border-radius: 6px; color: #64748b; display: inline-flex; align-items: center; justify-content: center; transition: 0.2s;}
.btn-icon-outline:hover { background: #f8fafc; color: #0f172a; border-color: #cbd5e1;}

/* Segmented Control (iOS Style Tabs) */
.segmented-control { display: inline-flex; background: #f1f5f9; padding: 4px; border-radius: 10px; gap: 4px; }
.segmented-control button { background: transparent; border: none; padding: 6px 16px; font-size: 0.8rem; font-weight: 700; color: #64748b; border-radius: 8px; cursor: pointer; transition: 0.2s; }
.segmented-control button:hover { color: #0f172a; }
.segmented-control button.active { background: #ffffff; color: #0f172a; box-shadow: 0 1px 3px rgba(0,0,0,0.05); }

/* ==========================================
   BADGES & CHIPS
   ========================================== */
.chip-label { background: #f1f5f9; color: #475569; font-size: 0.75rem; font-weight: 700; padding: 4px 10px; border-radius: 999px;}
.chip-label-warning { background: #fffbeb; color: #d97706; font-size: 0.75rem; font-weight: 700; padding: 4px 10px; border-radius: 999px;}
.btn-circle-add { background: #ffffff; border: 1px dashed #cbd5e1; color: #64748b; border-radius: 50%; width: 24px; height: 24px; display: flex; align-items: center; justify-content: center; font-size: 1rem; cursor: pointer; transition: 0.2s; padding: 0;}
.btn-circle-add:hover { border-color: #0f172a; color: #0f172a;}
.badge-success { background: #ecfdf5; color: #059669; font-size: 0.65rem; padding: 2px 8px; border-radius: 999px; border: 1px solid #a7f3d0;}

/* ==========================================
   MODERN CLEAN TABLES (NO ROUGH BORDERS)
   ========================================== */
.table-card { background: #ffffff; border: 1px solid #f1f5f9; border-radius: 12px; overflow: hidden; margin-bottom: 2rem;}
.modern-clean-table { width: 100%; border-collapse: collapse; text-align: left; }
.modern-clean-table th { padding: 12px 16px; font-size: 0.75rem; font-weight: 800; color: #94a3b8; text-transform: uppercase; letter-spacing: 0.5px; border-bottom: 1px solid #f1f5f9;}
.modern-clean-table td { padding: 12px 16px; border-bottom: 1px solid #f8fafc; font-size: 0.85rem; vertical-align: middle; color: #334155;}
.modern-clean-table tbody tr:last-child td { border-bottom: none; }
.modern-clean-table tbody tr:hover td { background: #fafafa; }
.text-center-table th, .text-center-table td { text-align: center; }
.freeze-header-table th { position: sticky; top: 0; z-index: 10; background: #ffffff;}
.table-scroll-container { max-height: calc(100vh - 200px); overflow-y: auto;}

.row-group-clean td { background: #f8fafc; font-size: 0.7rem; font-weight: 800; color: #475569; padding: 8px 16px; border-top: 1px solid #f1f5f9;}
.empty-state { text-align: center; padding: 3rem 1rem !important; color: #94a3b8; font-size: 0.85rem;}
.font-small { font-size: 0.75rem; }

/* ==========================================
   MASTER ROMBEL (CARD LAYOUT)
   ========================================== */
.rombel-card-layout { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 1.5rem; padding: 1rem 0; }
.rombel-grade-card { background: #ffffff; border: 1px solid #e2e8f0; border-radius: 12px; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.02); overflow: hidden; }
.rombel-grade-header { background: #f8fafc; padding: 1rem 1.5rem; border-bottom: 1px solid #e2e8f0; display: flex; justify-content: space-between; align-items: center; }
.rombel-grade-header h3 { margin: 0; font-size: 1.1rem; color: #0f172a; font-weight: 800; }
.rombel-count { font-size: 0.75rem; color: #64748b; font-weight: 700; background: #e2e8f0; padding: 2px 8px; border-radius: 999px; }
.rombel-grade-body { padding: 1.5rem; display: flex; flex-wrap: wrap; gap: 10px; }
.rombel-pill { display: inline-flex; align-items: center; background: #ffffff; border: 1px solid #cbd5e1; padding: 6px 12px; border-radius: 8px; font-size: 0.9rem; color: #334155; transition: 0.2s; box-shadow: 0 1px 2px rgba(0,0,0,0.02);}
.rombel-pill:hover { border-color: #94a3b8; box-shadow: 0 2px 4px rgba(0,0,0,0.05);}
.btn-remove-rombel { background: none; border: none; color: #94a3b8; cursor: pointer; margin-left: 8px; font-size: 0.85rem; padding: 0; transition: 0.2s;}
.btn-remove-rombel:hover { color: #e11d48; }


/* ==========================================
   GRID JADWAL & GURU (CLASSIC COMPACT - FIT TO SCREEN)
   ========================================== */
.main-layout { padding: 0; display: flex; flex-direction: column; flex: 1; overflow: hidden; width: 100%; min-height: 0; background: #ffffff;}
.schedule-area { flex: 1; display: flex; flex-direction: column; overflow: hidden; border: none; min-height: 0; }

.schedule-toolbar-modern { padding: 10px 15px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid #f1f5f9;}
.schedule-title { font-size: 1.2rem; font-weight: 900; margin: 0; color: #0f172a;}

.btn-solid-accent { background: #4f46e5; color: white; border: none; padding: 6px 14px; border-radius: 6px; font-size: 0.8rem; font-weight: 700; cursor: pointer;}
.btn-outline-danger { background: white; color: #e11d48; border: 1px solid #fecaca; padding: 6px 14px; border-radius: 6px; font-size: 0.8rem; font-weight: 700; cursor: pointer;}

.top-teacher-pool-container { padding: 8px 10px; background: #ffffff; border-bottom: 1px solid #e2e8f0; flex-shrink: 0; width: 100%; }
.pool-header { font-size: 0.7rem; font-weight: 800; color: #64748b; text-align: center; margin-bottom: 6px; letter-spacing: 0.5px;}

/* DOCK FIT TO SCREEN NO SCROLL */
.grouped-pools-container { display: flex; flex-wrap: nowrap; gap: 4px; overflow: hidden; width: 100%; justify-content: space-between; }
.mapel-pool-group { display: flex; flex-direction: column; flex: 1; min-width: 0; background: #ffffff; border: 1px solid #e2e8f0; border-radius: 4px; padding: 4px; padding-bottom: 6px;}
.mapel-pool-label { font-size: 0.55rem; font-weight: 800; text-align: center; text-transform: uppercase; margin-bottom: 4px; border-bottom: 1px dashed #cbd5e1; padding-bottom: 2px;}
.top-teacher-pool { display: flex; flex-direction: column; gap: 3px; max-height: 68px; overflow-y: auto; padding-right: 2px;}
.top-teacher-pool::-webkit-scrollbar { width: 4px; }
.top-teacher-pool::-webkit-scrollbar-thumb { background-color: #cbd5e1; border-radius: 4px; }
.teacher-pill { display: flex; justify-content: space-between; align-items: center; background: #fff; border: 1px solid #cbd5e1; padding: 2px 4px; border-radius: 4px; cursor: grab; font-size: 0.75rem; transition: 0.1s;}
.teacher-pill:hover { background: #f8fafc; border-color: #94a3b8; transform: translateY(-1px); }
.kode-fw { font-weight: 800; }

/* KONTROL WARNA HIJAU JP LACI */
.jp-badge-mini { font-size: 0.65rem; font-weight: 700; color: #94a3b8;}
.jp-badge-mini.ok { color: #16a34a; }
.jp-badge-mini.over { color: #e11d48; font-weight: 900;}

/* WADAH JADWAL */
.schedule-scroll { flex: 1; overflow: auto; background: #ffffff; min-height: 0; padding-bottom: 40px;}
.grid-table-clean { width: 100%; table-layout: fixed; border-collapse: separate; border-spacing: 0; }
.grid-table-clean th, .grid-table-clean td { border-right: 1px solid #f1f5f9; border-bottom: 1px solid #f1f5f9; text-align: center; padding: 1px;}
.grid-table-clean th { padding: 4px 2px; font-size: 0.75rem; font-weight: 800; position: sticky; top: 0; z-index: 20; background: #ffffff; color: #475569; border-bottom: 2px solid #e2e8f0;}

/* KOLOM WAKTU 1 BARIS (LEBAR 180px AGAR AMAN TDK WRAP) */
.sticky-col { position: sticky; left: 0; background: #fff; z-index: 30; width: 180px; min-width: 180px;}
th.sticky-col { z-index: 40; border-right: 1px solid #e2e8f0; }

.time-label { padding: 2px 4px; width: 180px; min-width: 180px; border-right: 1px solid #e2e8f0; text-align: center; vertical-align: middle;}
.time-content { display: flex; flex-direction: row; align-items: center; justify-content: center; gap: 8px;}
.jam-ke-clean { font-size: 0.75rem; color: #4f46e5; font-weight: 900; white-space: nowrap;}
.waktu-teks-clean { font-size: 0.75rem; color: #64748b; font-weight: 600; white-space: nowrap; }

/* WARNA KELAS KLASIK BORDER TEGAS */
.header-kelas-7 { color: #0284c7 !important; border-bottom-color: #38bdf8 !important; }
.header-kelas-8 { color: #a16207 !important; border-bottom-color: #facc15 !important; }
.header-kelas-9 { color: #15803d !important; border-bottom-color: #4ade80 !important; }

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
.special-text-clean { font-size: 0.6rem; font-weight: 800; letter-spacing: 0.05em; color: #64748b; text-transform: uppercase;}

.slot-clean { padding: 1px; background: #ffffff; vertical-align: middle;}
.drop-zone-clean { min-height: 22px; height: 100%; display: flex; align-items: center; justify-content: center; }

.assigned-card-clean { 
  background: transparent; border: 1px solid #cbd5e1; border-radius: 2px; padding: 0; 
  width: 100%; height: 100%; min-height: 20px; display: flex; justify-content: center; align-items: center; 
  position: relative; transition: 0.1s; box-shadow: none;
}
.assigned-card-clean:hover { background: #f8fafc; border-color: #94a3b8 !important; }
.assigned-kode-clean { font-weight: 800; color: #334155; font-size: 0.75rem; }

.btn-remove-clean { 
  position: absolute; top: -4px; right: -4px; background: #ffffff; color: #94a3b8; 
  border: 1px solid #e2e8f0; width: 14px; height: 14px; border-radius: 50%; 
  display: flex; align-items: center; justify-content: center; cursor: pointer; 
  opacity: 0; transition: 0.2s; font-size: 9px; line-height: 1; font-weight: bold;
}
.assigned-card-clean:hover .btn-remove-clean { opacity: 1; }
.btn-remove-clean:hover { background: #ef4444 !important; color: white !important; border-color: #ef4444 !important;}

/* ==========================================
   MODALS (GHOST INPUT STYLE)
   ========================================== */
.modal-overlay { position: fixed; inset: 0; background: rgba(15, 23, 42, 0.4); display: flex; align-items: center; justify-content: center; z-index: 10000; backdrop-filter: blur(2px);}
.modal-box { background: #fff; padding: 2rem; border-radius: 16px; width: 90%; max-width: 400px; box-shadow: 0 20px 25px -5px rgba(0,0,0,0.1); border: 1px solid #f1f5f9;}
.modal-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 1.5rem;}
.modal-header h3 { font-size: 1.2rem; color: #0f172a; margin: 0; font-weight: 800;}
.btn-close-modal { background: transparent; border: none; font-size: 1.2rem; display: flex; align-items: center; justify-content: center; cursor: pointer; color: #94a3b8; transition: 0.2s; padding: 0;}
.btn-close-modal:hover { color: #e11d48;}

.form-group { margin-bottom: 1.2rem;}
.form-group label { display: block; font-size: 0.75rem; font-weight: 700; color: #94a3b8; margin-bottom: 4px; text-transform: uppercase; letter-spacing: 0.5px;}
.input-underline { width: 100%; padding: 8px 0; border: none; border-bottom: 2px solid #e2e8f0; font-size: 0.95rem; font-weight: 600; background: transparent; transition: 0.2s; color: #0f172a; border-radius: 0;}
.input-underline:focus { border-bottom-color: #0f172a; outline: none;}

.modal-actions { display: flex; justify-content: flex-end; gap: 10px; margin-top: 1.5rem;}
</style>