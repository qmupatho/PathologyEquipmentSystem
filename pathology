<script>
  // ==========================================
  // Global Variables & State
  // ==========================================
  let currentUser = null;
  let cachedEquipments = [];
  let financialChartInstance = null;
  
  // Dynamic Table Row Counters
  let pipetteRowCount = 0;
  let thermoRowCount = 0;
  let centrifugeRowCount = 0;
  let balanceRowCount = 0;
  let ovenRowCount = 0;
  let thermoHygroRowCount = 0;
  let pcrRowCount = 0;

  // ==========================================
  // UI Helpers & Global Utilities
  // ==========================================
  function showLoader() { 
    const loader = document.getElementById('loader');
    if (loader) loader.classList.remove('hidden'); 
  }
  
  function hideLoader() { 
    const loader = document.getElementById('loader');
    if (loader) loader.classList.add('hidden'); 
  }

  function toggleTheme() {
    const html = document.documentElement;
    if (html.classList.contains('dark')) {
      html.classList.remove('dark'); 
      html.classList.add('light');
    } else {
      html.classList.remove('light'); 
      html.classList.add('dark');
    }
    // รีเฟรชสีกราฟเมื่อเปลี่ยนโหมดความมืด-สว่าง
    if (typeof renderDashboardFinancials === 'function' && document.getElementById('page-dashboard') && !document.getElementById('page-dashboard').classList.contains('hidden')) {
      renderDashboardFinancials(); 
    }
  }

  function showToast(message, type) {
    type = type || 'warning';
    var area = document.getElementById('notification-area');
    if (!area) {
      // fallback: สร้าง notification area ชั่วคราว
      area = document.createElement('div');
      area.id = 'notification-area';
      area.className = 'fixed bottom-6 right-6 z-[999] flex flex-col items-end pointer-events-none';
      document.body.appendChild(area);
    }
    var cfg = {
      success: { bg: 'background:#16a34a', icon: 'check_circle' },
      danger:  { bg: 'background:#dc2626', icon: 'error' },
      warning: { bg: 'background:#d97706', icon: 'warning' },
      info:    { bg: 'background:#2563eb', icon: 'info' }
    };
    var c = cfg[type] || cfg['warning'];
    var toast = document.createElement('div');
    toast.style.cssText = 'pointer-events:auto;display:flex;align-items:center;padding:12px 16px;color:#fff;border-radius:12px;box-shadow:0 4px 16px rgba(0,0,0,0.18);margin-bottom:8px;transition:opacity 0.4s,transform 0.4s;max-width:360px;' + c.bg;
    toast.innerHTML = '<span class="material-symbols-outlined" style="margin-right:10px;font-size:20px;">' + c.icon + '</span>'
      + '<div style="font-size:13px;font-weight:600;flex:1;">' + message + '</div>'
      + '<button onclick="this.parentElement.remove()" style="margin-left:12px;background:none;border:none;color:#fff;cursor:pointer;opacity:0.75;">'
      + '<span class="material-symbols-outlined" style="font-size:16px;">close</span></button>';
    area.appendChild(toast);
    var delay = type === 'success' ? 3000 : 8000;
    setTimeout(function() {
      toast.style.opacity = '0';
      toast.style.transform = 'translateY(12px)';
      setTimeout(function() { if (toast.parentElement) toast.remove(); }, 450);
    }, delay);
  }

  // ==========================================
  // Authentication & Session Management
  // ==========================================
  function handleLogin() {
    const user = document.getElementById('login-user').value;
    const pass = document.getElementById('login-pass').value;
    showLoader();
    
    google.script.run.withSuccessHandler(function(res) {
      hideLoader();
      if(res.success) {
        currentUser = res;
        setupUIForUser();
        loadEquipmentTypes();
        loadLocations(); 
        document.getElementById('login-view').classList.add('hidden');
        document.getElementById('app-layout').classList.remove('hidden');
        switchPage('dashboard');
      } else {
        alert(res.message);
      }
    }).withFailureHandler(function(err){
      hideLoader(); 
      alert('เกิดข้อผิดพลาด: ' + err);
    }).verifyLogin(user, pass);
  }

  function openLogoutModal() { document.getElementById('logout-modal').classList.remove('hidden'); }
  function closeLogoutModal() { document.getElementById('logout-modal').classList.add('hidden'); }
  
  function confirmLogout() {
    closeLogoutModal();
    currentUser = null;
    _clearDashboardCharts(); // ✅ ไม่ให้กราฟของผู้ใช้ก่อนหน้าค้างไปถึงผู้ใช้คนถัดไป
    document.getElementById('login-form').reset();
    document.getElementById('app-layout').classList.add('hidden');
    document.getElementById('login-view').classList.remove('hidden');
    const mobilePanel = document.getElementById('mobile-admin-panel');
    if(mobilePanel && !mobilePanel.classList.contains('translate-y-full')){
      if (typeof toggleAdminPanel === 'function') toggleAdminPanel();
    }
  }


  // ===== Helper: ชื่อผู้บันทึก (fullname หรือ username) =====
  function _getDisplayName() {
    if (!currentUser) return 'System';
    return (currentUser.fullname || '').trim() || currentUser.username || 'System';
  }

  function setupUIForUser() {
    if (!currentUser) return;
    var displayName = (currentUser.fullname || currentUser.username || '').trim() || (_getDisplayName() || currentUser.username);
    var deptLabel   = currentUser.dept || '';
    var roleLabel   = currentUser.role === 'Admin' ? 'Admin (ส่วนกลาง)' : ('User' + (currentUser.position ? ' — ' + currentUser.position : ''));
    document.getElementById('display-name').innerText  = displayName;
    document.getElementById('display-role').innerText  = roleLabel + (deptLabel ? '\n' + deptLabel : '');
    document.getElementById('user-avatar').innerText   = displayName.charAt(0).toUpperCase();
    const isAdmin = (currentUser.role === 'Admin');
    const sidebarAdmin = document.getElementById('admin-sidebar-section');
    if(sidebarAdmin) {
      if(isAdmin) sidebarAdmin.classList.remove('hidden'); else sidebarAdmin.classList.add('hidden');
    }

    const mobileAdminBtn = document.getElementById('mobile-admin-btn');
    if(mobileAdminBtn) {
      if(isAdmin) mobileAdminBtn.classList.remove('hidden'); else mobileAdminBtn.classList.add('hidden');
    }
  }

  // ==========================================
  // Form Page: Panel Open/Close + Summary Table
  // ==========================================
  function openEquipmentFormPanel() {
    const panel = document.getElementById('equipment-form-panel');
    if (!panel) return;
    panel.classList.remove('hidden');
    const summaryCard = document.getElementById('form-summary-card');
    if (summaryCard) summaryCard.classList.add('hidden');
    // reset title/icon กลับเป็นเพิ่มใหม่
    const titleEl = document.getElementById('form-panel-title');
    const iconEl  = document.getElementById('form-panel-icon');
    if (titleEl) titleEl.textContent = 'ลงทะเบียนเครื่องมือใหม่';
    if (iconEl)  iconEl.textContent  = 'add_circle';
    // รีเซ็ตฟอร์มและ hidden field
    const form = document.getElementById('equipment-form');
    if (form) form.reset();
    const rowIndexEl = document.getElementById('eqRowIndex');
    if (rowIndexEl) rowIndexEl.value = '';
    const previewSec = document.getElementById('form-preview-section');
    if (previewSec) previewSec.classList.add('hidden');
    const formActions = document.getElementById('form-actions');
    if (formActions) formActions.classList.remove('hidden');
    const calContainer = document.getElementById('calDateFieldContainer');
    if (calContainer) calContainer.classList.add('hidden');
    panel.scrollIntoView({ behavior: 'smooth', block: 'start' });
    setTimeout(() => { const el = document.getElementById('eqName'); if (el) el.focus(); }, 400);
  }

  function closeEquipmentFormPanel() {
    const panel = document.getElementById('equipment-form-panel');
    if (panel) panel.classList.add('hidden');
    // แสดงตาราง summary กลับมา
    const summaryCard = document.getElementById('form-summary-card');
    if (summaryCard) summaryCard.classList.remove('hidden');
    const form = document.getElementById('equipment-form');
    if (form) form.reset();
    const previewSec = document.getElementById('form-preview-section');
    if (previewSec) previewSec.classList.add('hidden');
    const formActions = document.getElementById('form-actions');
    if (formActions) formActions.classList.remove('hidden');
  }

  function loadFormSummaryTable() {
    const tbody = document.getElementById('form-summary-tbody');
    const countEl = document.getElementById('form-table-count');
    if (!tbody) return;

    tbody.innerHTML = `<tr><td colspan="7" class="px-5 py-10 text-center text-slate-400 text-sm">
      <span class="material-symbols-outlined text-3xl block mb-2 text-slate-300">hourglass_empty</span>กำลังโหลด...</td></tr>`;

    const data = cachedEquipments || [];
    if (data.length === 0) {
      tbody.innerHTML = `<tr><td colspan="7" class="px-5 py-10 text-center text-slate-400 text-sm">
        <span class="material-symbols-outlined text-3xl block mb-2 text-slate-300">inventory_2</span>ยังไม่มีข้อมูลเครื่องมือ</td></tr>`;
      if (countEl) countEl.textContent = 'ไม่พบข้อมูล';
      return;
    }

    const query = (document.getElementById('form-search-input')?.value || '').toLowerCase().trim();
    const filtered = query
      ? data.filter(item =>
          (item['ชื่อเครื่องมือ'] || '').toLowerCase().includes(query) ||
          (item['รหัสครุภัณฑ์'] || '').toLowerCase().includes(query) ||
          (item['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || '').toLowerCase().includes(query)
        )
      : data;

    if (countEl) countEl.textContent = `พบ ${filtered.length} รายการ`;

    if (filtered.length === 0) {
      tbody.innerHTML = `<tr><td colspan="7" class="px-5 py-10 text-center text-slate-400 text-sm">ไม่พบรายการที่ค้นหา</td></tr>`;
      return;
    }

    tbody.innerHTML = filtered.map((item, idx) => {
      const code  = item['รหัสครุภัณฑ์'] || '-';
      const name  = item['ชื่อเครื่องมือ'] || '-';
      const type  = item['ประเภทเครื่องมือ'] || '-';
      const loc   = item['สถานที่ติดตั้ง'] || '-';
      const _compRaw   = item['ชื่อบริษัทผู้แทน/โทรศัพท์'] || '';
      const _compParts = _compRaw.split('/');
      const rep   = (_compParts[0] || '').trim() || '-';   // บริษัท
      const repCell = (_compParts[1] || '').trim();         // เซลล์/ผู้ติดต่อ
      const phone = (_compParts[2] || _compParts[1] || '').trim() || '-'; // เบอร์โทร
      const rowClass = idx % 2 === 0 ? '' : 'bg-slate-50/60 dark:bg-slate-700/20';
      // ใช้ _index ที่ผูกไว้ตอนโหลด (ถูกต้องทั้ง Admin และ User)
      const globalIdx = item._index !== undefined ? item._index : cachedEquipments.findIndex(e => e['ID'] === item['ID']);
      return `
        <tr class="${rowClass} hover:bg-primary-50/40 dark:hover:bg-slate-700/40 transition">
          <td class="px-5 py-3.5 font-bold text-primary-600 dark:text-primary-400 whitespace-nowrap">${code}</td>
          <td class="px-3 py-2 whitespace-nowrap overflow-hidden" style="max-width:0">
            <div class="text-xs font-semibold text-slate-800 dark:text-slate-100 truncate">${name}</div>
            <div class="text-[10px] text-slate-400 mt-0.5 truncate">${item['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || ''}</div>
          </td>
          <td class="px-3 py-2 text-xs text-slate-600 dark:text-slate-300 whitespace-nowrap overflow-hidden truncate" style="max-width:0">${type}</td>
          <td class="px-3 py-2 text-xs text-slate-600 dark:text-slate-300 whitespace-nowrap overflow-hidden truncate" style="max-width:0">${loc}</td>
          <td class="px-3 py-2 text-xs text-slate-600 dark:text-slate-300 overflow-hidden" style="max-width:0">
            <div class="truncate">${rep}</div>
            ${repCell ? `<div class="text-[10px] text-slate-400 mt-0.5 truncate">${repCell}</div>` : ''}
          </td>
          <td class="px-3 py-2 text-xs text-slate-600 dark:text-slate-300 whitespace-nowrap">${phone || '-'}</td>
          <td class="px-5 py-3.5 text-center whitespace-nowrap">
            <button onclick="viewEquipmentHistory(${globalIdx})" title="ดูประวัติเครื่องมือ"
              class="inline-flex items-center justify-center w-8 h-8 rounded-full text-slate-400 hover:text-blue-600 hover:bg-blue-50 dark:hover:bg-blue-900/30 transition">
              <span class="material-symbols-outlined text-[18px]">visibility</span>
            </button>
            <button onclick="editEquipmentRecord(${globalIdx})" title="แก้ไขข้อมูลเครื่องมือ"
              class="inline-flex items-center justify-center w-8 h-8 rounded-full text-slate-400 hover:text-amber-600 hover:bg-amber-50 dark:hover:bg-amber-900/30 transition">
              <span class="material-symbols-outlined text-[18px]">edit</span>
            </button>
            <button onclick="printEquipmentHistory(${globalIdx})" title="พิมพ์ประวัติ PDF"
              class="inline-flex items-center justify-center w-8 h-8 rounded-full text-slate-400 hover:text-red-600 hover:bg-red-50 dark:hover:bg-red-900/30 transition">
              <span class="material-symbols-outlined text-[18px]">picture_as_pdf</span>
            </button>
            </td>
        </tr>`;
    }).join('');
  }

  function handleFormSummaryRowClick(eqId) {
    const item = cachedEquipments ? cachedEquipments.find(e => e['ID'] === eqId) : null;
    if (!item) return;
    showToast(`เลือก: ${item['ชื่อเครื่องมือ']} — กด ✏️ เพื่อแก้ไข หรือ 👁️ ดูประวัติ`, 'info');
  }

  // ==========================================
  // Form Page Search Filter
  // ==========================================
  function filterEquipmentForm(query) {
    const q = query.trim().toLowerCase();
    const suggestionBox = document.getElementById('form-search-suggestions');

    // อัปเดตตารางสรุปตาม query
    loadFormSummaryTable();

    if (!suggestionBox) return;
    if (!q || !cachedEquipments || cachedEquipments.length === 0) {
      suggestionBox.classList.add('hidden');
      suggestionBox.innerHTML = '';
      return;
    }

    const matches = cachedEquipments.filter(item =>
      (item['ชื่อเครื่องมือ'] || '').toLowerCase().includes(q) ||
      (item['รหัสครุภัณฑ์'] || '').toLowerCase().includes(q) ||
      (item['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || '').toLowerCase().includes(q)
    ).slice(0, 8);

    if (matches.length === 0) {
      suggestionBox.innerHTML = '<div class="px-4 py-2 text-sm text-slate-400">ไม่พบรายการ</div>';
      suggestionBox.classList.remove('hidden');
      return;
    }

    suggestionBox.innerHTML = matches.map(item => `
      <div class="px-4 py-2.5 text-sm hover:bg-primary-50 dark:hover:bg-slate-700 cursor-pointer transition border-b dark:border-slate-700 last:border-0 flex flex-col"
           onclick="document.getElementById('form-search-input').value=''; document.getElementById('form-search-suggestions').classList.add('hidden'); loadFormSummaryTable();">
        <span class="font-bold text-slate-800 dark:text-slate-100">${item['ชื่อเครื่องมือ'] || '-'}</span>
        <span class="text-xs text-slate-400">${item['รหัสครุภัณฑ์'] || ''} · ${item['สถานที่ติดตั้ง'] || ''}</span>
      </div>
    `).join('');
    suggestionBox.classList.remove('hidden');
  }

  // ==========================================
  // Navigation Router
  // ==========================================
  // ✅ Refresh currentUser จาก server ก่อนโหลด history (เพื่อให้ isHead ถูกต้องเสมอ)
  function _refreshCurrentUserThenLoad(pageId) {
    if (!currentUser || !currentUser.username) { loadCalHistory(pageId); return; }
    google.script.run
      .withSuccessHandler(function(res) {
        if (res && res.success) {
          // อัปเดตเฉพาะ field ที่อาจเปลี่ยนได้ (isHead, role, position)
          currentUser.isHead    = res.isHead;
          currentUser.role      = res.role;
          currentUser.position  = res.position;
          currentUser.fullname  = res.fullname || currentUser.fullname;
        }
        loadCalHistory(pageId);
      })
      .withFailureHandler(function() { loadCalHistory(pageId); })
      .getUserInfo(currentUser.username);
  }

  function switchPage(pageId) {
    const allPages = [
      'page-dashboard', 'page-inventory-view', 'page-equipment-list',
      'page-cal-report', 'page-form', 'page-table', 'page-thermo',
      'page-pipette', 'page-centrifuge', 'page-balance', 'page-thermohygro',
      'page-phmeter', 'page-oven', 'page-ruler', 'page-bscfume', 'page-pcr', 'page-admin', 'page-cal-cost', 'page-eq-status', 'page-pm', 'page-depr', 'page-impact', 'page-verify', 'page-intercheck', 'page-calreview'
    ];
    allPages.forEach(function(id) {
      const el = document.getElementById(id);
      if (el) el.style.display = 'none';
    });
    // ✅ เมนูผู้ดูแลระบบแยกปุ่ม → ใช้หน้า page-admin เดียวกัน แต่แสดงเฉพาะส่วนที่เลือก
    const _adminSec = { 'admin': 'users', 'admin-location': 'location', 'admin-department': 'department', 'admin-category': 'category', 'admin-execlink': 'execlink' }[pageId];
    const target = document.getElementById(_adminSec ? 'page-admin' : 'page-' + pageId);
    if (_adminSec) _showAdminSection(_adminSec);
    if (target) {
      target.style.display = 'block';
      target.classList.remove('hidden');
    } else {
      console.error('switchPage: ไม่พบ page-' + pageId);
    }

    document.querySelectorAll('.menu-btn').forEach(btn => btn.classList.remove('bg-primary-50','text-primary-700','dark:bg-slate-700','font-semibold'));
    const activeBtn = document.querySelector('.menu-btn[data-target="' + pageId + '"]');
    if (activeBtn) activeBtn.classList.add('bg-primary-50','text-primary-700','dark:bg-slate-700','font-semibold');

    document.querySelectorAll('.nav-btn').forEach(btn => btn.classList.remove('text-primary-600','dark:text-primary-400'));
    const activeNav = document.querySelector('.nav-btn[data-target="' + pageId + '"]');
    if (activeNav) activeNav.classList.add('text-primary-600','dark:text-primary-400');

    window.scrollTo(0, 0);
    var vc = document.getElementById('view-container');
    if (vc) vc.scrollTop = 0;

    try {
      if (pageId === 'dashboard') loadEquipments();
      if (pageId === 'inventory-view') { if (typeof loadEquipmentInventory === 'function') loadEquipmentInventory(); }
      if (pageId === 'equipment-list') {
        loadEquipments();
        // reset filter
        setTimeout(function() { filterListByStatus('all'); }, 900);
      }
      if (pageId === 'table') loadTableData();
      if (pageId === 'thermo') loadEquipmentsForThermo();
      if (pageId === 'pipette') loadEquipmentsForPipette();
      if (pageId === 'centrifuge') loadEquipmentsForCentrifuge();
      if (pageId === 'balance') loadEquipmentsForBalance();
      if (pageId === 'thermohygro') loadEquipmentsForThermoHygro();
      if (pageId === 'phmeter') { if (typeof loadEquipmentsForPHMeter === 'function') loadEquipmentsForPHMeter(); }
      if (pageId === 'oven') loadEquipmentsForOven();
      if (pageId === 'ruler') { if (typeof loadEquipmentsForRuler === 'function') loadEquipmentsForRuler(); }
      if (pageId === 'bscfume') { if (typeof loadEquipmentsForBSCFume === 'function') loadEquipmentsForBSCFume(); }
      if (pageId === 'pcr') loadEquipmentsForPCR();
      if (pageId === 'pm') loadPMPage();
      if (pageId === 'depr') loadDeprPage();
      if (pageId === 'impact') loadImpactPage();
      if (pageId === 'verify') loadVerifyPage();
      if (pageId === 'intercheck') loadIcPage();
      if (pageId === 'calreview') loadCalReview();
      if (pageId === 'cal-report') loadCalibrationReport();
      if (_adminSec) setTimeout(function(){ if(typeof loadAdminData==='function') loadAdminData(); }, 400);
      if (pageId === 'cal-cost') setTimeout(function(){ if(typeof loadCalCost==='function') loadCalCost(); }, 200);
            if (pageId === 'eq-status') { loadEqStatusPage(); }
      if (pageId === 'form') {
        loadLocations();
        loadEquipmentTypes();
        closeEquipmentFormPanel();
        loadFormSummaryTable();
      }
      var calPages = ['thermo','pipette','centrifuge','balance','thermohygro','phmeter','oven','ruler','bscfume','pcr'];
      if (calPages.indexOf(pageId) !== -1) {
        // ปิด form panel ก่อน แล้ว refresh user แล้วโหลด history
        closeCalFormPanel(pageId);
        _refreshCurrentUserThenLoad(pageId);
      }
    } catch (e) {
      console.error('switchPage error:', e);
    }
  }


  // ==========================================
  // Core Data Fetching (Settings & Master Data)
  // ==========================================
  // =====================================================
  // _afterCalSave — เรียกหลังบันทึกผลสอบเทียบสำเร็จ
  // refresh: equipment list, cal-report, dashboard
  // =====================================================
  function _afterCalSave(pageId) {
    // ✅ กรณีบันทึกจากโหมดแก้ไข → กลับไปที่ตารางประวัติของหน้าเดิม ไม่เด้งไปแดชบอร์ด
    if (window._calEditCtx && window._calEditCtx.pageId === pageId) {
      _clearCalEditMode(pageId);
      closeCalFormPanel(pageId);
      loadCalHistory(pageId);
      loadEquipments();
      if (typeof loadCalibrationReport === 'function') setTimeout(function() { loadCalibrationReport(); }, 1500);
      return;
    }
    loadCalHistory(pageId);
    loadEquipments();
    // refresh หน้าแผนและสรุปผลสอบเทียบในพื้นหลัง
    if (typeof loadCalibrationReport === 'function') {
      setTimeout(function() { loadCalibrationReport(); }, 1500);
    }
    switchPage('dashboard');
  }

  function loadLocations(cb) {
    google.script.run.withSuccessHandler(function(locations) {
      const select = document.getElementById('location');
      if (!select) return;
      select.innerHTML = '<option value="">-- เลือกหน่วยงาน/สถานที่ --</option>';
      locations.forEach(function(loc) {
        if(loc && loc.trim() !== "") {
          select.innerHTML += '<option value="' + loc + '">' + loc + '</option>';
        }
      });
      if (typeof cb === 'function') cb();
    }).withFailureHandler(function(err){
      const select = document.getElementById('location');
      if (select) select.innerHTML = '<option value="">โหลดข้อมูลล้มเหลว</option>';
    }).getLocations();
  }

  function loadEquipmentTypes(cb) {
    google.script.run.withSuccessHandler(function(types) {
      const select = document.getElementById('eqType');
      if (!select) return;
      select.innerHTML = '<option value="">-- เลือกประเภทเครื่องมือ --</option>';
      types.forEach(function(type) {
        if(type) select.innerHTML += '<option value="' + type + '">' + type + '</option>';
      });
      if (typeof cb === 'function') cb();
    }).withFailureHandler(function(err){
      const select = document.getElementById('eqType');
      if (select) select.innerHTML = '<option value="">โหลดข้อมูลล้มเหลว</option>';
    }).getEquipmentTypes();
  }

  function loadEquipments() {
    showLoader();
    const dept = currentUser ? currentUser.dept : "";
    const role = currentUser ? currentUser.role : "";

    const deptLabel = document.getElementById('dash-dept-label');
    if (deptLabel) {
      deptLabel.textContent = role === 'Admin' ? 'ภาพรวมทุกห้องปฏิบัติการ (Admin)' : 'ห้องปฏิบัติการ: ' + dept;
    }

    google.script.run.withSuccessHandler(function(data) {
      hideLoader();
      cachedEquipments = data;
      cachedEquipments.forEach((item, index) => { item._index = index; item._rowIndex = index + 2; }); // +2 = skip header row

      const today = new Date(); today.setHours(0,0,0,0);
      let stats = { total: data.length, overdue: 0, nearing30: 0, nearing60: 0, normal: 0, noCal: 0, waiting: 0, sending: 0, broken: 0, repair: 0, retired: 0, suspended: 0, failed: 0 };
      let urgentList = { overdue: [], nearing: [] };

      // คำนวณสถิติ
      data.forEach(item => {
        const lastCalStr   = item['วันที่สอบเทียบล่าสุด'] || item['วันที่ทำการสอบเทียบ (Cal-Date)'] || '';
        const interval     = item['ระยะเวลาการสอบเทียบ'] || '';
        const isCalibrated = item['สถานะการสอบเทียบ'] || item['การสอบเทียบหรือไม่'] || '';
        const needsCal     = _needsSchedule(isCalibrated, item);
        const sendStatus   = item['สถานะการส่งสอบ'] || '';
        const eqCondition  = item['สถานะเครื่องมือ'] || '';
        const eqName       = item['ชื่อเครื่องมือ'] || '-';

        // ✅ สถานะเครื่องมือพิเศษ — นับแยก
        if (eqCondition === 'สอบเทียบไม่ผ่านเกณฑ์') { stats.failed++;    return; }
        if (eqCondition === 'เครื่องมือเสีย')        { stats.broken++;    return; }
        if (eqCondition === 'ส่งซ่อม')               { stats.repair++;    return; }
        if (eqCondition === 'จำหน่ายออก')            { stats.retired++;   return; }
        if (eqCondition === 'ระงับการใช้งาน')         { stats.suspended++; return; }

        // ✅ นับสถานะการส่งสอบก่อน
        if (sendStatus === 'รับเครื่องส่งสอบเทียบ') { stats.sending++; return; }
        if (sendStatus === 'รอส่งการสอบเทียบ')     { stats.waiting++; return; }

        if (needsCal && lastCalStr && interval) {
          // ใช้ Due Date จาก Sheet ก่อน
          const dueDateSheet = item['สอบเทียบครั้งถัดไป'] || item['Due Date'] || '';
          let dObj = dueDateSheet ? new Date(dueDateSheet) : null;
          if ((!dObj || isNaN(dObj)) ) {
            dObj = new Date(lastCalStr);
            if (!isNaN(dObj)) {
              dObj.setMonth(dObj.getMonth() + (parseInt(interval) || 12));
              dObj.setDate(dObj.getDate() - 1);
            }
          }
          if (dObj && !isNaN(dObj)) {
            const diff = Math.ceil((dObj - today) / (1000*60*60*24));
            if (diff < 0)       { stats.overdue++;   urgentList.overdue.push(eqName); }
            else if (diff <= 30){ stats.nearing30++;  urgentList.nearing.push(eqName + ' (' + diff + 'ว)'); }
            else if (diff <= 60){ stats.nearing60++;  urgentList.nearing.push(eqName + ' (' + diff + 'ว)'); }
            else                { stats.normal++; }
          } else { stats.noCal++; }
        } else { stats.noCal++; }
      });

      renderSummary(stats);
      renderDashboardCharts(data, today);

      // Alert strips (dashboard)
      const alertOverdue = document.getElementById('dash-alert-overdue');
      const alertNearing = document.getElementById('dash-alert-nearing');
      if (alertOverdue) {
        alertOverdue.classList.toggle('hidden', urgentList.overdue.length === 0);
        const el = document.getElementById('dash-overdue-list');
        if (el) el.textContent = urgentList.overdue.join(', ');
      }
      if (alertNearing) {
        alertNearing.classList.toggle('hidden', urgentList.nearing.length === 0);
        const el = document.getElementById('dash-nearing-list');
        if (el) el.textContent = urgentList.nearing.join(', ');
      }

      // Render compact table ใน equipment-list
      renderEquipmentListTable(data, today);

    }).withFailureHandler(function(err) {
      hideLoader();
      alert('ดึงข้อมูลเครื่องมือผิดพลาด: ' + err);
    }).getEquipments(dept, role);
  }

  // =====================================================
  // Render ตารางแถวเล็กๆ ใน equipment-list
  // =====================================================
  var _equipmentListData = [];  // global สำหรับ history modal

  function renderEquipmentListTable(data, today) {
    _equipmentListData = data || [];
    const tbody = document.getElementById('equipment-list-tbody');
    const countEl = document.getElementById('list-row-count');
    if (!tbody) return;

    if (!today) { today = new Date(); today.setHours(0,0,0,0); }

    if (!data || data.length === 0) {
      tbody.innerHTML = `<tr><td colspan="9" class="px-5 py-10 text-center text-slate-400">
        <span class="material-symbols-outlined text-3xl block mb-2 text-slate-300">inventory_2</span>
        ยังไม่มีข้อมูลในระบบ</td></tr>`;
      if (countEl) countEl.textContent = 'ไม่พบข้อมูล';
      var _pg = document.getElementById('list-pager'); if (_pg) _pg.innerHTML = '';
      return;
    }

    // นับ counter แต่ละสถานะ
    var cntAll = data.length, cntOverdue = 0, cntNear30 = 0, cntNear60 = 0, cntWaiting = 0, cntSending = 0;
    var cntBroken = 0, cntRepair = 0, cntRetired = 0, cntSuspended = 0, cntFailed = 0;

    tbody.innerHTML = data.map((item, idx) => {
      const eqCode  = item['รหัสครุภัณฑ์'] || '-';
      const eqName  = item['ชื่อเครื่องมือ'] || '-';
      const eqType  = item['ประเภทเครื่องมือ'] || '-';
      const loc     = item['สถานที่ติดตั้ง'] || '-';
      // ✅ อ่าน lastCal และ intv จาก column จริงใน Sheet
      let lastCal = item['วันที่สอบเทียบล่าสุด'] || item['วันที่ทำการสอบเทียบ (Cal-Date)'] || '';
      let intv    = item['ระยะเวลาการสอบเทียบ'] || '';
      // ✅ ตรวจจับข้อมูลบันทึกผิด column จาก bug เดิม:
      //    col "วันที่สอบเทียบล่าสุด" = ว่าง / col "ระยะเวลา" = date string (YYYY-MM-DD)
      //    เกิดจาก saveEquipment ขาด col ค่าเสื่อมราคา ทำให้ข้อมูลเลื่อน 1 col
      const _intvIsDate = /^\d{4}-\d{2}-\d{2}/.test(String(intv).trim());
      if (_intvIsDate) {
        // ดึง lastCal จาก intv (date ที่บันทึกผิด)
        lastCal = intv;
        // ดึง intv จริงจาก col ถัดไป (Due Date ใน sheet อาจเป็นตัวเลข interval จริง)
        // หรือใช้ค่า default 12 ถ้าหาไม่ได้
        const _nextCol = item['สอบเทียบครั้งถัดไป'] || item['Due Date'] || '';
        intv = /^\d{1,3}$/.test(String(_nextCol).trim()) ? String(_nextCol).trim() : '12';
      }
      if (!intv || intv === '') intv = '-';
      const isCal   = _needsSchedule(item['สถานะการสอบเทียบ'] || item['การสอบเทียบหรือไม่'] || '', item);
      // ✅ ใช้ Due Date จาก Sheet ก่อน (อัปเดตโดย code.gs หลังบันทึกสอบเทียบ)
      const dueDateSheet = item['สอบเทียบครั้งถัดไป'] || item['Due Date'] || item['วันที่สอบเทียบรอบถัดไป'] || '';

      let dueText = '-';
      let statusBadge = _noCalLabel(item) === 'บำรุงรักษา'
        ? `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-sky-100 text-sky-700 dark:bg-sky-900/30 dark:text-sky-300">บำรุงรักษา</span>`
        : `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-slate-100 text-slate-400 dark:bg-slate-700 dark:text-slate-500">ไม่สอบ</span>`;
      let dotColor = 'bg-slate-300';
      let dataStatus = 'none';
      let dueDateIso = '';

      const sendStatus  = item['สถานะการส่งสอบ'] || '';
      const eqCondition = item['สถานะเครื่องมือ'] || '';

      // ✅ สถานะเครื่องมือ (จาก openEquipmentStatusModal) — มีความสำคัญสูงสุด
      // ยกเว้น 'พร้อมใช้งาน' และ 'รอสอบเทียบใหม่' ซึ่งให้ใช้ due-date logic แทน
      const _isNormalStatus = eqCondition === 'พร้อมใช้งาน' || eqCondition === 'รอสอบเทียบใหม่' || eqCondition === '';
      if (!_isNormalStatus && eqCondition === 'สอบเทียบไม่ผ่านเกณฑ์') {
        dotColor = 'bg-red-600'; dataStatus = 'failed'; cntFailed++;
        statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-red-100 text-red-800 whitespace-nowrap">⚠️ ไม่ผ่านเกณฑ์</span>`;
      } else if (!_isNormalStatus && eqCondition === 'เครื่องมือเสีย') {
        dotColor = 'bg-stone-600'; dataStatus = 'broken'; cntBroken++;
        statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-stone-100 text-stone-800 whitespace-nowrap">🔧 เครื่องเสีย</span>`;
      } else if (!_isNormalStatus && eqCondition === 'ส่งซ่อม') {
        dotColor = 'bg-orange-600'; dataStatus = 'repair'; cntRepair++;
        statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-orange-100 text-orange-800 whitespace-nowrap">🛠️ ส่งซ่อม</span>`;
      } else if (!_isNormalStatus && eqCondition === 'จำหน่ายออก') {
        dotColor = 'bg-slate-500'; dataStatus = 'retired'; cntRetired++;
        statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-slate-100 text-slate-600 whitespace-nowrap">📦 จำหน่ายออก</span>`;
      } else if (!_isNormalStatus && eqCondition === 'ระงับการใช้งาน') {
        dotColor = 'bg-violet-600'; dataStatus = 'suspended'; cntSuspended++;
        statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-violet-100 text-violet-800 whitespace-nowrap">🚫 ระงับการใช้</span>`;
      } else if (sendStatus === 'รับเครื่องส่งสอบเทียบ') {
        dotColor = 'bg-purple-500'; dataStatus = 'sending'; cntSending++;
        statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-purple-100 text-purple-700 dark:bg-purple-900/30 dark:text-purple-400 whitespace-nowrap">📦 อยู่ระหว่างสอบเทียบ</span>`;
      } else if (sendStatus === 'รอส่งการสอบเทียบ') {
        dotColor = 'bg-blue-400'; dataStatus = 'waiting'; cntWaiting++;
        statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-blue-100 text-blue-700 dark:bg-blue-900/30 dark:text-blue-400 whitespace-nowrap">⏳ รอส่งสอบ</span>`;
      } else if (isCal) {
        // ✅ ใช้ Due Date จาก Sheet ก่อน ถ้าไม่มีค่อยคำนวณ
        let dObj = dueDateSheet ? new Date(dueDateSheet) : null;
        if ((!dObj || isNaN(dObj)) && lastCal && intv !== '-') {
          dObj = new Date(lastCal);
          if (!isNaN(dObj)) {
            dObj.setMonth(dObj.getMonth() + (parseInt(intv) || 12));
            dObj.setDate(dObj.getDate() - 1);
          }
        }
        if (dObj && !isNaN(dObj)) {
          dueDateIso = dObj.toISOString().slice(0,10);
          const diff = Math.ceil((dObj - today) / (1000*60*60*24));
          dueText = dObj.toLocaleDateString('th-TH', { year:'numeric', month:'short', day:'numeric' });

          if (diff < 0) {
            dotColor = 'bg-red-500'; dataStatus = 'overdue'; cntOverdue++;
            statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-red-100 text-red-700 dark:bg-red-900/30 dark:text-red-400 whitespace-nowrap">เกิน ${Math.abs(diff)} วัน</span>`;
          } else if (diff <= 30) {
            dotColor = 'bg-orange-500'; dataStatus = 'near30'; cntNear30++;
            statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-orange-100 text-orange-700 dark:bg-orange-900/30 dark:text-orange-400 whitespace-nowrap">🔴 อีก ${diff} วัน</span>`;
          } else if (diff <= 60) {
            dotColor = 'bg-amber-400'; dataStatus = 'near60'; cntNear60++;
            statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-amber-100 text-amber-700 dark:bg-amber-900/30 dark:text-amber-400 whitespace-nowrap">⚠️ อีก ${diff} วัน</span>`;
          } else {
            dotColor = 'bg-emerald-500'; dataStatus = 'normal';
            statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-emerald-100 text-emerald-700 dark:bg-emerald-900/30 dark:text-emerald-400">ปกติ</span>`;
          }
        } else {
          statusBadge = `<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-slate-100 text-slate-400">ข้อมูลไม่ครบ</span>`;
        }
      }

      // ✅ format lastCal เป็นภาษาไทยเสมอ (ไม่แสดง ISO)
      const lastCalFmt = lastCal && !isNaN(new Date(lastCal))
        ? new Date(lastCal).toLocaleDateString('th-TH', { year:'numeric', month:'short', day:'numeric' }) : '-';

      const rowBg = idx % 2 === 0 ? '' : 'bg-slate-50/50 dark:bg-slate-700/10';

      return `<tr class="${rowBg} hover:bg-primary-50/30 dark:hover:bg-slate-700/30 transition"
                  data-name="${eqName.toLowerCase()}" data-code="${eqCode.toLowerCase()}"
                  data-status="${dataStatus}" data-due="${dueDateIso}">
        <td class="px-5 py-2.5">
          <span class="w-2.5 h-2.5 rounded-full ${dotColor} inline-block"></span>
        </td>
        <td class="px-3 py-2.5 whitespace-nowrap">
          <span class="text-[11px] font-bold text-primary-600 dark:text-primary-400">${eqCode}</span>
        </td>
        <td class="px-3 py-2.5">
          <div class="text-sm font-semibold text-slate-800 dark:text-slate-100 leading-tight">${eqName}</div>
          <div class="text-[10px] text-slate-400 mt-0.5">${item['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || ''}</div>
        </td>
        <td class="px-3 py-2.5 text-xs text-slate-500 whitespace-nowrap">${eqType}</td>
        <td class="px-3 py-2.5 text-xs text-slate-500 whitespace-nowrap">${loc}</td>
        <td class="px-3 py-2.5 text-xs text-slate-600 dark:text-slate-300 text-center whitespace-nowrap">${lastCalFmt}</td>
        <td class="px-3 py-2.5 text-xs text-slate-500 text-center whitespace-nowrap">${intv !== '-' ? intv : '-'}</td>
        <td class="px-3 py-2.5 text-xs font-bold text-slate-700 dark:text-slate-200 text-center whitespace-nowrap">${dueText}</td>
        <td class="px-3 py-2.5 text-center">${statusBadge}</td>
      </tr>`;
    }).join('');

    // อัปเดต counter ปุ่ม filter
    var _setTxt = function(id, n) { var el = document.getElementById(id); if (el) el.textContent = n > 0 ? '(' + n + ')' : ''; };
    _setTxt('cnt-all', cntAll);
    _setTxt('cnt-overdue', cntOverdue);
    _setTxt('cnt-near30', cntNear30);
    _setTxt('cnt-near60', cntNear60);
    _setTxt('cnt-waiting', cntWaiting);
    _setTxt('cnt-sending', cntSending);
    // สถานะเครื่องมือพิเศษ — อัปเดต legend badge ถ้ามี
    var _eqCondTotal = cntFailed + cntBroken + cntRepair + cntRetired + cntSuspended;
    var _eqCondEl = document.getElementById('cnt-eq-condition');
    if (_eqCondEl) _eqCondEl.textContent = _eqCondTotal > 0 ? '(' + _eqCondTotal + ')' : '';

    // อัปเดต alert banner
    var banner = document.getElementById('list-alert-banner');
    var hasAlert = cntOverdue > 0 || cntNear30 > 0 || cntNear60 > 0;
    if (banner) banner.classList.toggle('hidden', !hasAlert);
    var _setBadge = function(id, n, label) {
      var el = document.getElementById(id);
      if (!el) return;
      if (n > 0) { el.textContent = label + ' (' + n + ')'; el.classList.remove('hidden'); }
      else { el.classList.add('hidden'); }
    };
    _setBadge('list-alert-overdue', cntOverdue, '🔴 เกินกำหนด');
    _setBadge('list-alert-near30', cntNear30, '🔴 ≤30 วัน');
    _setBadge('list-alert-near60', cntNear60, '⚠️ 31–60 วัน');

    if (countEl) countEl.textContent = `รายการทั้งหมด ${cntAll} รายการ`;
    _listRefreshPaging(false);   // ✅ แบ่งหน้า (คงตัวกรอง/ค้นหาเดิมไว้)
    _listUpdateCountText();
  }

  function filterEquipmentList(query) {
    _listPager.q = (query || '').toLowerCase().trim();
    _listRefreshPaging(true);
    _listUpdateCountText();
  }

  function filterListByStatus(status) {
    // อัปเดต active button
    document.querySelectorAll('.list-filter-btn').forEach(btn => {
      btn.classList.remove('border-primary-300','bg-primary-50','text-primary-700','dark:bg-primary-900/30','dark:text-primary-400');
      btn.classList.add('border-slate-200','bg-white','text-slate-500','dark:bg-slate-800','dark:text-slate-400');
    });
    const activeBtn = document.getElementById('list-filter-' + status);
    if (activeBtn) {
      activeBtn.classList.remove('border-slate-200','bg-white','text-slate-500');
      activeBtn.classList.add('border-primary-300','bg-primary-50','text-primary-700');
    }
    _listPager.status = status || 'all';
    _listRefreshPaging(true);
    _listUpdateCountText();
  }

  function _listUpdateCountText() {
    const rows = _listRows();
    const visible = rows.filter(r => r.getAttribute('data-match') !== '0').length;
    const total = rows.length;
    const labelMap = {
      all:'ทั้งหมด', overdue:'เกินกำหนด', near30:'≤30 วัน', near60:'31–60 วัน',
      waiting:'รอส่งการสอบเทียบ', sending:'อยู่ระหว่างสอบเทียบ', normal:'ปกติ'
    };
    const parts = [];
    if (_listPager.status !== 'all') parts.push('กรอง: ' + (labelMap[_listPager.status] || _listPager.status));
    if (_listPager.q) parts.push('ค้นหา: "' + _listPager.q + '"');
    const countEl = document.getElementById('list-row-count');
    if (countEl) countEl.textContent = parts.length
      ? parts.join(' · ') + ` — พบ ${visible} จาก ${total} รายการ`
      : `รายการทั้งหมด ${total} รายการ`;
  }

  // ============================================================
  // ✅ แบ่งหน้า (Pagination) — หน้า "รายการบัญชีทั้งหมด"
  //    ทำงานร่วมกับช่องค้นหา + ปุ่มกรองสถานะ (กรองก่อน แล้วค่อยแบ่งหน้า)
  // ============================================================
  var _listPager = { page: 1, size: 10, q: '', status: 'all' };

  function _listRows() { return Array.prototype.slice.call(document.querySelectorAll('#equipment-list-tbody tr[data-status]')); }

  function _listApplyFilters() {
    var q = _listPager.q, st = _listPager.status;
    _listRows().forEach(function (r) {
      var okQ = !q || (r.getAttribute('data-name') || '').indexOf(q) !== -1 || (r.getAttribute('data-code') || '').indexOf(q) !== -1;
      var okS = st === 'all' || r.getAttribute('data-status') === st;
      r.setAttribute('data-match', okQ && okS ? '1' : '0');
    });
  }

  function _listRenderPage() {
    var rows = _listRows(), matched = rows.filter(function (r) { return r.getAttribute('data-match') !== '0'; });
    var total = matched.length, size = _listPager.size, pages = Math.max(1, Math.ceil(total / size));
    if (_listPager.page > pages) _listPager.page = pages;
    if (_listPager.page < 1) _listPager.page = 1;
    var from = (_listPager.page - 1) * size, to = Math.min(from + size, total);
    rows.forEach(function (r) { r.style.display = 'none'; });
    matched.slice(from, to).forEach(function (r) { r.style.display = ''; });

    var box = document.getElementById('list-pager');
    if (!box) return;
    if (!rows.length) { box.innerHTML = ''; return; }
    var p = _listPager.page;
    var btn = function (label, target, opts) {
      opts = opts || {};
      var dis = opts.disabled, act = opts.active;
      return '<button type="button" ' + (dis ? 'disabled' : 'onclick="listGoPage(' + target + ')"') + ' title="' + (opts.title || '') + '" class="min-w-[36px] h-9 px-2.5 rounded-full border text-sm transition flex items-center justify-center '
        + (act ? 'bg-slate-900 text-white border-slate-900 dark:bg-primary-600 dark:border-primary-600 font-bold'
               : dis ? 'border-slate-200 dark:border-slate-700 text-slate-300 dark:text-slate-600 cursor-not-allowed'
                     : 'border-slate-200 dark:border-slate-600 text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-700') + '">' + label + '</button>';
    };
    var icon = function (n) { return '<span class="material-symbols-outlined text-lg">' + n + '</span>'; };
    // เลขหน้า: 1 … p-1 p p+1 … last
    var nums = [], set = {};
    [1, pages, p - 1, p, p + 1].forEach(function (n) { if (n >= 1 && n <= pages && !set[n]) { set[n] = 1; nums.push(n); } });
    nums.sort(function (a, b) { return a - b; });
    var numHtml = '', prev = 0;
    nums.forEach(function (n) {
      if (prev && n - prev > 1) numHtml += '<span class="px-1 text-slate-400">…</span>';
      numHtml += btn(n, n, { active: n === p });
      prev = n;
    });
    var sizes = [10, 25, 50, 100];
    box.innerHTML =
        '<div class="flex flex-col md:flex-row md:items-center justify-between gap-3 px-5 py-3 border-t dark:border-slate-700 bg-slate-50/50 dark:bg-slate-800/50">'
      +   '<div class="flex flex-wrap items-center gap-4 text-sm text-slate-600 dark:text-slate-300">'
      +     '<span>แสดง <b>' + (total ? from + 1 : 0) + '</b> ถึง <b>' + to + '</b> จากทั้งหมด <b>' + total.toLocaleString() + '</b> รายการ'
      +       (total !== rows.length ? ' <span class="text-xs text-slate-400">(กรองจาก ' + rows.length.toLocaleString() + ')</span>' : '') + '</span>'
      +     '<label class="flex items-center gap-2">แสดงต่อหน้า: <select onchange="listSetPageSize(this.value)" class="px-3 py-1.5 border border-slate-200 dark:border-slate-600 rounded-lg bg-white dark:bg-slate-700 text-sm">'
      +       sizes.map(function (s) { return '<option value="' + s + '"' + (s === size ? ' selected' : '') + '>' + s + '</option>'; }).join('')
      +     '</select></label>'
      +   '</div>'
      +   '<div class="flex items-center gap-1.5 flex-wrap">'
      +     btn(icon('keyboard_double_arrow_left'), 1, { disabled: p === 1, title: 'หน้าแรก' })
      +     btn(icon('chevron_left'), p - 1, { disabled: p === 1, title: 'ก่อนหน้า' })
      +     numHtml
      +     btn(icon('chevron_right'), p + 1, { disabled: p === pages, title: 'ถัดไป' })
      +     btn(icon('keyboard_double_arrow_right'), pages, { disabled: p === pages, title: 'หน้าสุดท้าย' })
      +   '</div>'
      + '</div>';
  }

  function listGoPage(n) {
    _listPager.page = n;
    _listRenderPage();
    var t = document.getElementById('equipment-list-tbody');
    if (t && t.closest) { var card = t.closest('.rounded-xl'); if (card && card.scrollIntoView) card.scrollIntoView({ behavior: 'smooth', block: 'start' }); }
  }
  function listSetPageSize(v) { _listPager.size = parseInt(v, 10) || 10; _listPager.page = 1; _listRenderPage(); }
  function _listRefreshPaging(resetPage) { if (resetPage) _listPager.page = 1; _listApplyFilters(); _listRenderPage(); }


  function renderSummary(stats) {
    const c = document.getElementById('dashboard-summary');
    if (!c) return;
    const cards = [
      { label:'ทั้งหมด',            val: stats.total,    icon:'inventory_2',           bg:'bg-primary-100 dark:bg-primary-900/30',  tx:'text-primary-600 dark:text-primary-400', border:'border-slate-100' },
      { label:'เกินกำหนด',          val: stats.overdue,  icon:'error',                 bg:'bg-red-100 dark:bg-red-900/30',         tx:'text-red-600 dark:text-red-400',         border:'border-red-100' },
      { label:'ใกล้ครบ 30 วัน',     val: stats.nearing30,icon:'warning',               bg:'bg-orange-100 dark:bg-orange-900/30',   tx:'text-orange-600 dark:text-orange-400',   border:'border-orange-100' },
      { label:'ใกล้ครบ 60 วัน',     val: stats.nearing60,icon:'notification_important',bg:'bg-amber-100 dark:bg-amber-900/30',     tx:'text-amber-600 dark:text-amber-400',     border:'border-amber-100' },
      { label:'ปกติ / ไม่สอบ / บำรุงรักษา',      val: stats.normal + stats.noCal, icon:'check_circle', bg:'bg-emerald-100 dark:bg-emerald-900/30', tx:'text-emerald-600 dark:text-emerald-400', border:'border-emerald-100' },
      { label:'รอส่งสอบเทียบ',      val: stats.waiting,  icon:'schedule_send',         bg:'bg-blue-100 dark:bg-blue-900/30',       tx:'text-blue-600 dark:text-blue-400',       border:'border-blue-100' },
      { label:'อยู่ระหว่างสอบเทียบ',val: stats.sending,  icon:'sync',                  bg:'bg-purple-100 dark:bg-purple-900/30',   tx:'text-purple-600 dark:text-purple-400',   border:'border-purple-100' },
      { label:'เสีย / ซ่อม / ระงับ', val: stats.broken + stats.repair + stats.suspended + stats.failed, icon:'build',  bg:'bg-stone-100 dark:bg-stone-900/30', tx:'text-stone-600 dark:text-stone-400', border:'border-stone-100', hidden: (stats.broken + stats.repair + stats.suspended + stats.failed) === 0 },
      { label:'จำหน่ายออกแล้ว',     val: stats.retired,  icon:'delete_forever',        bg:'bg-slate-100 dark:bg-slate-700/30',     tx:'text-slate-500 dark:text-slate-400',     border:'border-slate-200', hidden: stats.retired === 0 },
    ].filter(c => !c.hidden);
    c.innerHTML = cards.map(card => `
      <div class="bg-white dark:bg-slate-800 p-4 rounded-2xl shadow-sm border ${card.border} dark:border-slate-700 flex items-center gap-3">
        <div class="w-11 h-11 ${card.bg} rounded-xl flex items-center justify-center shrink-0">
          <span class="material-symbols-outlined text-2xl ${card.tx}">${card.icon}</span>
        </div>
        <div>
          <div class="text-[10px] font-bold text-slate-400 uppercase tracking-wide">${card.label}</div>
          <div class="text-2xl font-black ${card.tx}">${card.val} <span class="text-xs font-normal text-slate-500">เครื่อง</span></div>
        </div>
      </div>`).join('');
  }

  // ==========================================
  // Dashboard Charts
  // ==========================================
  let _donutChart = null;
  let _barChart   = null;

  // ✅ ล้างกราฟแดชบอร์ด (ใช้เมื่อหน่วยไม่มีเครื่องมือ / ออกจากระบบ)
  //    เดิม: ถ้าข้อมูลว่างจะ return ทันที กราฟของผู้ใช้/หน่วยก่อนหน้าจึงค้างอยู่บนจอ
  function _clearDashboardCharts() {
    try { if (typeof renderDashboardExec === 'function') renderDashboardExec([]); } catch (e) {}
    try { if (typeof _donutChart !== 'undefined' && _donutChart) { _donutChart.destroy(); _donutChart = null; } } catch (e) {}
    try { if (typeof _barChart !== 'undefined' && _barChart) { _barChart.destroy(); _barChart = null; } } catch (e) {}
    ['dash-type-donut', 'dash-status-bar'].forEach(function (id) {
      var c = document.getElementById(id);
      if (c && c.getContext) { var ctx = c.getContext('2d'); if (ctx) ctx.clearRect(0, 0, c.width, c.height); }
    });
    var totalEl = document.getElementById('dash-donut-total');
    if (totalEl) totalEl.innerText = '0';
    var legendEl = document.getElementById('dash-type-legend');
    if (legendEl) legendEl.innerHTML = '<div class="text-slate-400 italic py-6 text-center">ยังไม่มีเครื่องมือของหน่วยนี้</div>';
  }

  function renderDashboardCharts(data, today) {
    try { renderDashboardExec(data || []); } catch (e) { console.warn('renderDashboardExec', e); } // ✅ ภาพรวมผู้บริหาร
    try { renderDashOpsAlert(data || []); } catch (e) { console.warn('renderDashOpsAlert', e); }    // ✅ เตือนตรวจระหว่างรอบ / ทวนสอบ
    if (!data || !data.length) { _clearDashboardCharts(); return; }
    if (typeof Chart === 'undefined') return;

    // === 1. นับจำนวนตามประเภท ===
    const typeCount = {};
    data.forEach(item => {
      const t = item['ประเภทเครื่องมือ'] || 'ไม่ระบุ';
      typeCount[t] = (typeCount[t] || 0) + 1;
    });
    const typeLabels = Object.keys(typeCount).sort((a,b) => typeCount[b] - typeCount[a]);
    const typeValues = typeLabels.map(k => typeCount[k]);

    // === 2. นับสถานะตามประเภท ===
    const typeStatus = {}; // { type: { overdue, near60, normal, noCal, waiting, sending } }
    data.forEach(item => {
      const t           = item['ประเภทเครื่องมือ'] || 'ไม่ระบุ';
      const lastCal     = item['วันที่สอบเทียบล่าสุด'] || item['วันที่ทำการสอบเทียบ (Cal-Date)'] || '';
      const intv        = item['ระยะเวลาการสอบเทียบ'] || '';
      const isCal       = _needsSchedule(item['สถานะการสอบเทียบ'] || item['การสอบเทียบหรือไม่'] || '', item);
      const sendStatus  = item['สถานะการส่งสอบ'] || '';
      const eqCondition = item['สถานะเครื่องมือ'] || '';  // ✅ ประกาศที่นี่
      if (!typeStatus[t]) typeStatus[t] = { overdue:0, near60:0, normal:0, noCal:0, waiting:0, sending:0, broken:0 };

      const _badConditions = ['สอบเทียบไม่ผ่านเกณฑ์','เครื่องมือเสีย','ส่งซ่อม','ระงับการใช้งาน','จำหน่ายออก'];
      if (_badConditions.indexOf(eqCondition) >= 0) {
        typeStatus[t].broken++; return;
      }
      if (sendStatus === 'รับเครื่องส่งสอบเทียบ') { typeStatus[t].sending++; return; }
      if (sendStatus === 'รอส่งการสอบเทียบ')     { typeStatus[t].waiting++; return; }

      if (isCal && lastCal && intv) {
        const dueSheet = item['สอบเทียบครั้งถัดไป'] || item['Due Date'] || '';
        let dObj = dueSheet ? new Date(dueSheet) : null;
        if (!dObj || isNaN(dObj)) {
          dObj = new Date(lastCal);
          if (!isNaN(dObj)) {
            dObj.setMonth(dObj.getMonth() + (parseInt(intv) || 12));
            dObj.setDate(dObj.getDate() - 1);
          }
        }
        if (dObj && !isNaN(dObj)) {
          const diff = Math.ceil((dObj - today)/(1000*60*60*24));
          if (diff < 0)      typeStatus[t].overdue++;
          else if (diff<=60) typeStatus[t].near60++;
          else               typeStatus[t].normal++;
          return;
        }
      }
      typeStatus[t].noCal++;
    });

    // === Palette ===
    const PALETTE = [
      '#3b82f6','#8b5cf6','#06b6d4','#f59e0b','#10b981',
      '#ef4444','#f97316','#ec4899','#6366f1','#84cc16',
      '#14b8a6','#a855f7','#0ea5e9','#eab308','#22c55e',
    ];

    // === Donut Chart ===
    const donutCtx = document.getElementById('dash-type-donut');
    if (donutCtx) {
      if (_donutChart) { _donutChart.destroy(); _donutChart = null; }
      _donutChart = new Chart(donutCtx, {
        type: 'doughnut',
        data: {
          labels: typeLabels,
          datasets: [{
            data: typeValues,
            backgroundColor: typeLabels.map((_,i) => PALETTE[i % PALETTE.length]),
            borderWidth: 2,
            borderColor: '#fff',
            hoverOffset: 6,
          }]
        },
        options: {
          cutout: '68%',
          plugins: {
            legend: { display: false },
            tooltip: {
              callbacks: {
                label: ctx => ` ${ctx.label}: ${ctx.parsed} เครื่อง (${Math.round(ctx.parsed/data.length*100)}%)`,
              }
            }
          },
          animation: { animateRotate: true, duration: 600 },
        }
      });

      // อัปเดต total ตรงกลาง
      const totalEl = document.getElementById('dash-donut-total');
      if (totalEl) totalEl.textContent = data.length;
    }

    // === Legend ===
    const legendEl = document.getElementById('dash-type-legend');
    if (legendEl) {
      legendEl.innerHTML = typeLabels.map((label, i) => {
        const pct = Math.round(typeValues[i] / data.length * 100);
        return `<div class="flex items-center gap-2 min-w-0">
          <span class="w-2.5 h-2.5 rounded-sm shrink-0" style="background:${PALETTE[i % PALETTE.length]}"></span>
          <span class="text-[11px] text-slate-600 dark:text-slate-300 truncate flex-1" title="${label}">${label}</span>
          <span class="text-[11px] font-bold text-slate-700 dark:text-slate-200 shrink-0">${typeValues[i]}</span>
          <span class="text-[10px] text-slate-400 shrink-0 w-8 text-right">${pct}%</span>
        </div>`;
      }).join('');
    }

    // === Bar Chart: สถานะตามประเภท (แสดงเฉพาะ top 8 ประเภท) ===
    const barCtx = document.getElementById('dash-status-bar');
    if (barCtx) {
      if (_barChart) { _barChart.destroy(); _barChart = null; }
      const topTypes = typeLabels.slice(0, 8);
      _barChart = new Chart(barCtx, {
        type: 'bar',
        data: {
          labels: topTypes.map(t => t.length > 18 ? t.substring(0,17)+'…' : t),
          datasets: [
            { label: 'เกินกำหนด',          data: topTypes.map(t => typeStatus[t]?.overdue||0),  backgroundColor: '#ef4444', borderRadius: 4 },
            { label: 'ใกล้ครบ 60 วัน',     data: topTypes.map(t => typeStatus[t]?.near60||0),   backgroundColor: '#f59e0b', borderRadius: 4 },
            { label: 'ปกติ',               data: topTypes.map(t => typeStatus[t]?.normal||0),   backgroundColor: '#10b981', borderRadius: 4 },
            { label: 'ไม่สอบเทียบ/บำรุงรักษา', data: topTypes.map(t => typeStatus[t]?.noCal||0),    backgroundColor: '#94a3b8', borderRadius: 4 },
            { label: 'รอส่งสอบ',           data: topTypes.map(t => typeStatus[t]?.waiting||0),  backgroundColor: '#3b82f6', borderRadius: 4 },
            { label: 'อยู่ระหว่างสอบเทียบ', data: topTypes.map(t => typeStatus[t]?.sending||0),  backgroundColor: '#8b5cf6', borderRadius: 4 },
            { label: 'เสีย/ซ่อม/ระงับ',    data: topTypes.map(t => typeStatus[t]?.broken||0),   backgroundColor: '#78716c', borderRadius: 4 },
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: true,
          scales: {
            x: {
              stacked: true,
              grid: { display: false },
              ticks: { font: { size: 10 }, color: '#94a3b8' }
            },
            y: {
              stacked: true,
              beginAtZero: true,
              ticks: { precision: 0, font: { size: 10 }, color: '#94a3b8' },
              grid: { color: '#f1f5f9' }
            }
          },
          plugins: {
            legend: {
              position: 'bottom',
              labels: { boxWidth: 10, padding: 12, font: { size: 10 }, color: '#64748b' }
            },
            tooltip: { mode: 'index', intersect: false }
          },
          animation: { duration: 600 },
        }
      });
    }
  }

  
  function renderDashboardFinancials() {
    const select = document.getElementById('budget-year-select');
    if (!select) return;

    if (select.options.length === 0) {
      const currentYear = new Date().getFullYear();
      const currentBuddhistYear = currentYear + 543;
      let html = '';
      for (let i = -2; i <= 5; i++) {
        const bYear = currentBuddhistYear + i;
        const cYear = currentYear + i;
        const selected = (i === 0) ? 'selected' : ''; 
        html += `<option value="${cYear}" ${selected}>ปีงบฯ ${bYear}</option>`;
      }
      select.innerHTML = html;
    }

    const startYear = parseInt(select.value);
    const startBYear = startYear + 543;

    let totalValue = 0;
    let totalCalCost = 0;
    
    let yearlyDepreciation = { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0 };
    let yearlyRemaining = { 1: 0, 2: 0, 3: 0, 4: 0, 5: 0 };
    let labels = [];

    for (let i = 0; i < 5; i++) {
      labels.push(`ปีงบฯ ${startBYear + i}`);
    }

    cachedEquipments.forEach(item => {
      let cost = parseFloat(item['มูลค่าเครื่องมือ']) || 0;
      let calCost = parseFloat(item['ค่าสอบเทียบ']) || 0; 
      let life = parseInt(item['อายุการใช้งาน (ปี)']) || 1;
      
      let dateStr = item['วันที่รับเครื่อง'] || item['วันที่เริ่มใช้เครื่อง'];
      let purchaseYear = startYear; 
      
      if (dateStr) {
        const pd = new Date(dateStr);
        if (!isNaN(pd.getTime())) {
          let py = pd.getFullYear();
          if (pd.getMonth() >= 9) py += 1; 
          purchaseYear = py;
        }
      }

      totalValue += cost;
      totalCalCost += calCost; 

      let rate = 2 / life; 

      for (let offset = 0; offset < 5; offset++) {
        let targetYear = startYear + offset;
        let age = targetYear - purchaseYear + 1;
        let yearKey = offset + 1; 

        if (age < 1) {
          yearlyRemaining[yearKey] += cost; 
          yearlyDepreciation[yearKey] += 0;
        } else if (age > life) {
          yearlyRemaining[yearKey] += 0;
          yearlyDepreciation[yearKey] += 0;
        } else {
          let currentVal = cost;
          for (let i = 1; i < age; i++) {
            let d = currentVal * rate;
            if (i === life) d = currentVal;
            currentVal -= d;
          }
          let currentDep = currentVal * rate;
          if (age === life) currentDep = currentVal; 
          
          yearlyDepreciation[yearKey] += currentDep;
          yearlyRemaining[yearKey] += (currentVal - currentDep); 
        }
      }
    });

    document.getElementById('dash-total-value').innerText = `฿${totalValue.toLocaleString(undefined, {maximumFractionDigits: 0})}`;
    document.getElementById('dash-total-cal').innerText = `฿${totalCalCost.toLocaleString(undefined, {maximumFractionDigits: 0})}`;

    const tbody = document.getElementById('dash-financial-table');
    if (tbody) {
      tbody.innerHTML = '';
      for (let i = 1; i <= 5; i++) {
        const row = `
          <tr class="hover:bg-slate-50 dark:hover:bg-slate-700/30 transition text-slate-700 dark:text-slate-300">
            <td class="px-4 py-3 font-bold">${labels[i-1]}</td>
            <td class="px-4 py-3 text-right">฿${yearlyRemaining[i].toLocaleString(undefined, {maximumFractionDigits: 0})}</td>
            <td class="px-4 py-3 text-right font-bold text-red-500">- ฿${yearlyDepreciation[i].toLocaleString(undefined, {maximumFractionDigits: 0})}</td>
          </tr>`;
        tbody.insertAdjacentHTML('beforeend', row);
      }
    }

    updateFinancialChart(labels, yearlyRemaining, yearlyDepreciation);
  }

  function updateFinancialChart(labels, remaining, depreciation) {
    const canvas = document.getElementById('financialChart');
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    
    const dataRemaining = [remaining[1], remaining[2], remaining[3], remaining[4], remaining[5]];
    const dataDepreciation = [depreciation[1], depreciation[2], depreciation[3], depreciation[4], depreciation[5]];

    const isDark = document.documentElement.classList.contains('dark');
    const textColor = isDark ? '#cbd5e1' : '#475569';
    const gridColor = isDark ? '#334155' : '#f1f5f9';

    if (financialChartInstance) {
      financialChartInstance.destroy(); 
    }

    financialChartInstance = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: labels,
        datasets: [
          {
            label: 'มูลค่าทรัพย์สินคงเหลือ (แท่ง)',
            data: dataRemaining,
            backgroundColor: 'rgba(99, 102, 241, 0.8)', 
            borderRadius: 4,
            order: 2
          },
          {
            label: 'ค่าเสื่อมที่ถูกหัก (เส้น)',
            data: dataDepreciation,
            type: 'line',
            borderColor: '#ef4444', 
            backgroundColor: '#ef4444',
            borderWidth: 2,
            pointBackgroundColor: '#fff',
            pointBorderWidth: 2,
            pointRadius: 4,
            fill: false,
            order: 1
          }
        ]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: {
          legend: { position: 'bottom', labels: { color: textColor, font: { family: 'Sarabun' } } },
          tooltip: {
            callbacks: {
              label: function(context) {
                let label = context.dataset.label || '';
                if (label) label += ': ';
                if (context.parsed.y !== null) label += '฿' + context.parsed.y.toLocaleString(undefined, {maximumFractionDigits: 0});
                return label;
              }
            }
          }
        },
        scales: {
          x: { ticks: { color: textColor, font: { family: 'Sarabun' } }, grid: { display: false } },
          y: { ticks: { color: textColor, font: { family: 'Sarabun' }, callback: function(value) { return '฿' + value.toLocaleString(); } }, grid: { color: gridColor } }
        }
      }
    });
  }

  // ==========================================
  // Master Equipment Registry Form Operation
  // ==========================================
  function toggleCalibrationFields() {
  _relabelScheduleFields();
    const isCalibrated = document.getElementById('isCalibrated').value;
    const calContainer = document.getElementById('calDateFieldContainer');
    
    const lastDateInput = document.getElementById('lastCalDate');
    const intervalInput = document.getElementById('calInterval');
    const nextDateOutput = document.getElementById('nextCalDate');

    if (_needsSchedule(isCalibrated)) {
      if (calContainer) calContainer.classList.remove('hidden');
      if (lastDateInput) lastDateInput.required = true;
      if (intervalInput) intervalInput.required = true;
    } else {
      if (calContainer) calContainer.classList.add('hidden');
      if (lastDateInput) { lastDateInput.required = false; lastDateInput.value = ''; }
      if (intervalInput) { intervalInput.required = false; intervalInput.value = ''; }
      if (nextDateOutput) nextDateOutput.value = '';
    }
  }

  function calculateLifespan() {
    const receiveDateVal = document.getElementById('dateReceive').value;
    if (!receiveDateVal) return;
    const receiveDate = new Date(receiveDateVal);
    const today = new Date();
    let years = today.getFullYear() - receiveDate.getFullYear();
    const m = today.getMonth() - receiveDate.getMonth();
    if (m < 0 || (m === 0 && today.getDate() < receiveDate.getDate())) { years--; }
    if (years < 1) years = 1;
    const lifespanInput = document.getElementById('lifespan');
    if (lifespanInput) { lifespanInput.value = years; }
  }

  function calculateNextCalDate() {
  const lastCalDateVal = document.getElementById('lastCalDate').value;
  const calIntervalVal = document.getElementById('calInterval').value;
  const nextCalDateInput = document.getElementById('nextCalDate');

  if (!lastCalDateVal || !calIntervalVal) {
    nextCalDateInput.value = '';
    return;
  }

  const nextDate = new Date(lastCalDateVal);
  const monthsToAdd = parseInt(calIntervalVal, 10);

  // 1. บวกจำนวนเดือนตามรอบ
  nextDate.setMonth(nextDate.getMonth() + monthsToAdd);

  // 🌟 2. เพิ่มบรรทัดนี้: ลบออก 1 วัน
  nextDate.setDate(nextDate.getDate() - 1);

  // 3. แปลงฟอร์แมตกลับไปแสดงใน input date
  const yyyy = nextDate.getFullYear();
  const mm = String(nextDate.getMonth() + 1).padStart(2, '0');
  const dd = String(nextDate.getDate()).padStart(2, '0');

  nextCalDateInput.value = `${yyyy}-${mm}-${dd}`;
}

// ฟังก์ชันซ่อน/แสดง กล่องกรอกข้อมูลการสอบเทียบตามความจำเป็น
function toggleCalibrationFields() {
  _relabelScheduleFields();
  const isCalibrated = document.getElementById('isCalibrated').value;
  const container = document.getElementById('calDateFieldContainer');
  
  if (_needsSchedule(isCalibrated)) {
    container.classList.remove('hidden');
  } else {
    container.classList.add('hidden');
    // ล้างค่าหากเลือก "ไม่สอบเทียบ"
    document.getElementById('lastCalDate').value = '';
    document.getElementById('calInterval').value = '';
    document.getElementById('nextCalDate').value = '';
  }
}

  function showFormPreview() {
    const form = document.getElementById('equipment-form');
    if (!form.checkValidity()) { 
      form.reportValidity(); 
      return; 
    }

    function getElementText(id) {
      const el = document.getElementById(id);
      if (!el) return "-";
      if (el.tagName.toLowerCase() === 'select') {
        return el.selectedIndex >= 0 ? el.options[el.selectedIndex].text : "-";
      }
      return el.value || "-";
    }

    const previewHtml = `
      <div><span class="font-bold">ชื่อเครื่องมือ:</span> ${getElementText('eqName')}</div>
      <div><span class="font-bold">หมายเลขครุภัณฑ์:</span> ${getElementText('eqCode')}</div>
      <div><span class="font-bold">ประเภทเครื่องมือ:</span> ${getElementText('eqType')}</div>
      <div><span class="font-bold">สถานที่ตั้ง:</span> ${getElementText('location')}</div>
      <div><span class="font-bold">สถานะ:</span> ${getElementText('status')}</div>
      <div class="col-span-1 md:col-span-2 text-red-600 text-xs mt-2">* กรุณาตรวจสอบข้อมูลให้ถูกต้อง หากต้องการแก้ไขให้กดปุ่ม "กลับไปแก้ไข"</div>
    `;

    document.getElementById('preview-content').innerHTML = previewHtml;
    document.getElementById('form-actions').classList.add('hidden');
    document.getElementById('form-preview-section').classList.remove('hidden');

    const inputs = form.querySelectorAll('input, select, textarea');
    inputs.forEach(input => input.disabled = true);
  }

  function backToEdit() {
    document.getElementById('form-preview-section').classList.add('hidden');
    document.getElementById('form-actions').classList.remove('hidden');
    
    const form = document.getElementById('equipment-form');
    const inputs = form.querySelectorAll('input, select, textarea');
    inputs.forEach(input => input.disabled = false);
  }

  

  // ==========================================
  // Calibration Management Table (Multi-Row Inputs)
  // ==========================================
  function loadTableData() {
    showLoader();
    const dept = currentUser ? currentUser.dept : "";
    const role = currentUser ? currentUser.role : "";

    google.script.run.withSuccessHandler(function(data) {
      hideLoader();
      cachedEquipments = data;
      cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });

      // คำนวณ diff สำหรับแต่ละรายการ
      var today = new Date(); today.setHours(0,0,0,0);

      function _getDiff(item) {
        var lastCal = item['วันที่ทำการสอบเทียบ (Cal-Date)'] || item['วันที่สอบเทียบล่าสุด'] || '';
        var intv    = item['ระยะเวลาการสอบเทียบ'] || '';
        var isCal   = _needsSchedule(item['การสอบเทียบหรือไม่'] || item['สถานะการสอบเทียบ'] || '', item);
        if (!isCal || !lastCal || !intv) return null;
        var d = new Date(lastCal);
        if (isNaN(d)) return null;
        d.setMonth(d.getMonth() + (parseInt(intv) || 12));
        d.setDate(d.getDate() - 1);
        return Math.ceil((d - today) / (1000*60*60*24));
      }

      var near30    = data.filter(function(item) { var diff = _getDiff(item); return diff !== null && diff <= 30 && diff >= 0; });
      var near60only = data.filter(function(item) { var diff = _getDiff(item); return diff !== null && diff > 30 && diff <= 60; });
      var near60all  = data.filter(function(item) { var diff = _getDiff(item); return diff !== null && diff <= 60 && diff >= 0; });
      var overdue    = data.filter(function(item) { var diff = _getDiff(item); return diff !== null && diff < 0; });
      var failed     = data.filter(function(item) {
        var c = item['สถานะเครื่องมือ'] || '';
        return c === 'สอบเทียบไม่ผ่านเกณฑ์';
      });
      var broken     = data.filter(function(item) {
        var c = item['สถานะเครื่องมือ'] || '';
        return c === 'เครื่องมือเสีย' || c === 'ส่งซ่อม' || c === 'ระงับการใช้งาน';
      });

      // default: แสดงเฉพาะที่ต้องดูแล (ใกล้ + เกิน + ไม่ผ่าน + เสีย)
      var defaultView = data.filter(function(item) {
        var diff = _getDiff(item);
        var c    = item['สถานะเครื่องมือ'] || '';
        var isNearOrOverdue = diff !== null && diff <= 60;
        var isProblematic   = c === 'สอบเทียบไม่ผ่านเกณฑ์' || c === 'เครื่องมือเสีย' || c === 'ส่งซ่อม' || c === 'ระงับการใช้งาน';
        return isNearOrOverdue || isProblematic;
      });

      renderTableBody(defaultView.length > 0 ? defaultView : data);

      var countEl = document.getElementById('table-row-count');
      if (countEl) {
        var alertCount = overdue.length + failed.length + broken.length;
        if (alertCount > 0 || near60all.length > 0) {
          countEl.innerHTML = '<span class="text-amber-600 font-bold">⚠️ ต้องดูแล: ' + defaultView.length + ' รายการ</span>'
            + ' <span class="text-slate-400 text-xs ml-1">(ทั้งหมด ' + data.length + ' รายการ)</span>';
        } else {
          countEl.innerHTML = '<span class="text-emerald-600 font-bold">✅ ทุกรายการปกติ</span>'
            + ' <span class="text-slate-400 text-xs ml-1">(ทั้งหมด ' + data.length + ' รายการ)</span>';
        }
      }

      // filter bar: ปุ่มครบทุกสถานะ
      var filterBar = document.getElementById('table-filter-bar');
      if (filterBar) {
        filterBar.innerHTML = [
          { fn: 'null',      label: 'ทั้งหมด',              cnt: data.length,       cls: 'bg-slate-100 text-slate-600 border-slate-200 hover:bg-slate-200' },
          { fn: "'overdue'", label: '🔴 เกินกำหนด',         cnt: overdue.length,    cls: 'bg-red-100 text-red-700 border-red-200 hover:bg-red-200' },
          { fn: '30',        label: '🔴 ≤30 วัน',           cnt: near30.length,     cls: 'bg-orange-100 text-orange-700 border-orange-200 hover:bg-orange-200' },
          { fn: '60',        label: '⚠️ 31–60 วัน',         cnt: near60only.length, cls: 'bg-amber-100 text-amber-700 border-amber-200 hover:bg-amber-200' },
          { fn: "'failed'",  label: '❌ ไม่ผ่านเกณฑ์',      cnt: failed.length,     cls: 'bg-red-100 text-red-800 border-red-300 hover:bg-red-200' },
          { fn: "'broken'",  label: '🔧 ส่งซ่อม/เสีย/ระงับ', cnt: broken.length,    cls: 'bg-stone-100 text-stone-700 border-stone-200 hover:bg-stone-200' },
        ].map(function(b) {
          return '<button onclick="filterTableByDiff(' + b.fn + ')" class="px-3 py-1.5 rounded-full border text-xs font-bold transition ' + b.cls + '">'
            + b.label + ' <span class="inline-block bg-white/60 rounded-full px-1.5 ml-0.5">' + b.cnt + '</span></button>';
        }).join(' ');
      }
    }).withFailureHandler(function(err){
      hideLoader();
      alert('โหลดข้อมูลตารางล้มเหลว: ' + err);
    }).getEquipments(dept, role);
  }

  // ===== config: parameters per equipment type =====
  var _calParamConfig = {
    thermo:    [{ key:'temp',     label:'Temperature', unit:'°C',      ph:'เช่น 2-8 หรือ 35-37', ptp:'เช่น 5, 37' }],
    thermohyg: [{ key:'temp',     label:'Temperature', unit:'°C',      ph:'เช่น 20-25',          ptp:'เช่น 22' },
                { key:'humidity', label:'Humidity',    unit:'%RH',     ph:'เช่น 40-60',          ptp:'เช่น 50' }],
    centri:    [{ key:'speed',    label:'Speed',       unit:'rcf/rpm', ph:'เช่น 1620-1980',      ptp:'เช่น 1500, 1700' },
                { key:'time',     label:'Time',        unit:'วินาที',  ph:'เช่น 540-660',        ptp:'เช่น 600' }],
    pipette:   [{ key:'volume',   label:'Volume',      unit:'µl',      ph:'เช่น 90-110%',        ptp:'เช่น 100, 200' }],
    balance:   [{ key:'mass',     label:'Mass',        unit:'g',       ph:'เช่น 99-101%',        ptp:'เช่น 1, 10, 100' }],
    ph:        [{ key:'ph',       label:'pH',          unit:'pH',      ph:'เช่น 6.86-7.14',      ptp:'เช่น 4.00, 7.00, 10.00' }],
    oven:      [{ key:'temp',     label:'Temperature', unit:'°C',      ph:'เช่น 58-62',          ptp:'เช่น 60' }],
    ruler:     [{ key:'length',   label:'Length',      unit:'mm',      ph:'เช่น ±0.05',          ptp:'เช่น 10, 50, 100' }],
    bsc:       [{ key:'inflow',   label:'Inflow',      unit:'m/s',     ph:'เช่น 0.38-0.50',      ptp:'เช่น 0.45' },
                { key:'downflow', label:'Downflow',    unit:'m/s',     ph:'เช่น 0.25-0.40',      ptp:'เช่น 0.30' }],
    pcr:       [{ key:'temp',     label:'Temperature', unit:'°C',      ph:'เช่น ±0.5',           ptp:'เช่น 95, 55, 72' }],
    default:   [{ key:'usage',    label:'ช่วงใช้งาน', unit:'',        ph:'ระบุช่วง',             ptp:'ระบุจุดทดสอบ' }]
  };

  function _getParamType(typeStr) {
    var t = (typeStr||'').toLowerCase();
    if (t.includes('hygro') || t.includes('ความชื้น'))                        return 'thermohyg';
    if (t.includes('thermo') || t.includes('thermometer') || t.includes('อุณหภูมิ')) return 'thermo';
    if (t.includes('centri') || t.includes('ปั่น'))                           return 'centri';
    if (t.includes('pipette') || t.includes('ปิเปต'))                         return 'pipette';
    if (t.includes('balance') || t.includes('เครื่องชั่ง'))                   return 'balance';
    // ✅ เพิ่ม Autoclave / เครื่องนึ่ง / ตู้บ่ม (เดิมไม่เข้ากลุ่ม จึงไม่ขึ้นในแบบฟอร์ม Autoclave / Oven / Incubator)
    if (t.includes('oven') || t.includes('incubat') || t.includes('ตู้อบ') || t.includes('ตู้บ่ม') ||
        t.includes('autoclave') || t.includes('นึ่ง') || t.includes('steriliz') || t.includes('hot air')) return 'oven';
    if (t.includes('ph') || t.includes('พีเอช'))                              return 'ph';
    if (t.includes('ruler') || t.includes('ไม้บรรทัด') || t.includes('vernier')) return 'ruler';
    if (t.includes('bsc') || t.includes('fume') || t.includes('ตู้ดูด'))      return 'bsc';
    if (t.includes('pcr') || t.includes('thermal cycler'))                    return 'pcr';
    return 'default';
  }

  function _buildExpandPanel(item) {
    var id     = item['ID'];
    var ptype  = _getParamType(item['ประเภทเครื่องมือ']);
    var params = _calParamConfig[ptype] || _calParamConfig['default'];

    // โหลดค่าเดิม (เก็บใน JSON string ในช่องเดิม)
    var savedRaw = item['ช่วงใช้งาน'] || '';
    var savedObj = {};
    try { savedObj = JSON.parse(savedRaw); } catch(e) {
      // fallback: ค่าเดิมเป็น text เดียว → ใส่ใน param แรก
      if (savedRaw && params[0]) savedObj[params[0].key + '_limit'] = savedRaw;
    }
    var savedTP = item['จุดทดสอบ'] || '';
    var savedTPObj = {};
    try { savedTPObj = JSON.parse(savedTP); } catch(e) {
      if (savedTP && params[0]) savedTPObj[params[0].key + '_points'] = savedTP;
    }

    var paramRows = params.map(function(p) {
      var limitVal  = savedObj[p.key + '_limit']  || '';
      var pointsVal = savedTPObj[p.key + '_points'] || '';
      return '<tr>'
        + '<td style="padding:8px 12px 8px 0;white-space:nowrap;vertical-align:middle;">'
        +   '<span style="display:inline-flex;align-items:center;gap:4px;font-size:11px;font-weight:700;color:#1d4ed8;background:#eff6ff;border-radius:6px;padding:3px 10px;">'
        +     p.label + ' <span style="font-weight:400;color:#93c5fd;font-size:10px;">(' + p.unit + ')</span>'
        +   '</span>'
        + '</td>'
        + '<td style="padding:6px 8px 6px 0;vertical-align:middle;">'
        +   '<input type="text" id="ep-limit-' + id + '-' + p.key + '" value="' + limitVal + '" placeholder="' + p.ph + '" '
        +   'style="width:100%;padding:7px 10px;border:1px solid #e2e8f0;border-radius:8px;font-size:12px;outline:none;background:#fff;box-sizing:border-box;">'
        + '</td>'
        + '<td style="padding:6px 0;vertical-align:middle;">'
        +   '<input type="text" id="ep-points-' + id + '-' + p.key + '" value="' + pointsVal + '" placeholder="' + p.ptp + '" '
        +   'style="width:100%;padding:7px 10px;border:1px solid #e2e8f0;border-radius:8px;font-size:12px;outline:none;background:#fff;box-sizing:border-box;">'
        + '</td>'
        + '</tr>';
    }).join('');

    return '<tr id="ep-row-' + id + '" class="hidden bg-primary-50/60 dark:bg-primary-900/10 border-b border-primary-100 dark:border-primary-800">'
      + '<td colspan="6" class="p-0">'
      + '<div style="padding:12px 20px 4px;">'
      +   '<span class="text-xs font-bold text-primary-700 dark:text-primary-300 flex items-center gap-1.5">'
      +     '<span class="material-symbols-outlined text-base" style="font-size:16px;">edit_note</span>'
      +     'กรอกช่วงใช้งาน / จุดทดสอบ &mdash; <em style="font-weight:400;color:#64748b;">' + (item['ชื่อเครื่องมือ']||'') + '</em>'
      +   '</span>'
      + '</div>'
      + '<div style="overflow-x:auto;padding:0 20px;">'
      + '<table style="width:100%;border-collapse:collapse;font-size:12px;"><thead>'
      + '<tr style="color:#94a3b8;font-size:10px;font-weight:600;text-transform:uppercase;">'
      + '<th style="padding-bottom:6px;padding-right:12px;text-align:left;width:160px;">Parameter</th>'
      + '<th style="padding-bottom:6px;padding-right:8px;text-align:left;min-width:200px;">ช่วงใช้งาน (Control limit)</th>'
      + '<th style="padding-bottom:6px;text-align:left;min-width:200px;">จุดทดสอบ (Testing points) — คั่นด้วย ,</th>'
      + '</tr></thead><tbody>' + paramRows + '</tbody></table>'
      + '</div>'
      + '<div style="padding:10px 20px 14px;display:flex;align-items:center;gap:8px;flex-wrap:wrap;">'
      +   '<button data-save-id="' + id + '" class="ep-save-btn" style="display:inline-flex;align-items:center;gap:6px;padding:8px 18px;background:#1d4ed8;color:#fff;border:none;border-radius:8px;font-size:13px;font-weight:700;cursor:pointer;">'
      +     '<span class="material-symbols-outlined" style="font-size:16px;">save</span> บันทึก'
      +   '</button>'
      +   '<button data-clear-id="' + id + '" class="ep-clear-btn" style="display:inline-flex;align-items:center;gap:6px;padding:8px 14px;background:#fff;color:#dc2626;border:1px solid #fca5a5;border-radius:8px;font-size:13px;cursor:pointer;" title="ล้างข้อมูลช่วงใช้งานและจุดทดสอบทั้งหมด">'
      +     '<span class="material-symbols-outlined" style="font-size:16px;">delete_sweep</span> ล้างข้อมูล'
      +   '</button>'
      +   '<button data-close-id="' + id + '" class="ep-close-btn" style="display:inline-flex;align-items:center;gap:6px;padding:8px 14px;background:#fff;color:#64748b;border:1px solid #e2e8f0;border-radius:8px;font-size:13px;cursor:pointer;">'
      +     '<span class="material-symbols-outlined" style="font-size:16px;">close</span> ปิด'
      +   '</button>'
      + '</div>'
      + '</td>'
      + '</tr>';
  }

  function clearParamRow(id) {
    if (!confirm('ต้องการล้างข้อมูลช่วงใช้งานและจุดทดสอบของรายการนี้ใช่ไหม?')) return;

    var item = cachedEquipments ? cachedEquipments.find(function(e){ return e['ID'] === id; }) : null;
    if (!item) { showToast('ไม่พบข้อมูลเครื่องมือ', 'danger'); return; }

    var ptype  = _getParamType(item['ประเภทเครื่องมือ']);
    var params = _calParamConfig[ptype] || _calParamConfig['default'];

    // ล้าง input fields
    params.forEach(function(p) {
      var limEl = document.getElementById('ep-limit-'  + id + '-' + p.key);
      var ptEl  = document.getElementById('ep-points-' + id + '-' + p.key);
      if (limEl) limEl.value = '';
      if (ptEl)  ptEl.value  = '';
    });

    // บันทึก empty ลง GAS
    var clearBtn = document.querySelector('[data-clear-id="' + id + '"]');
    var origHTML = clearBtn ? clearBtn.innerHTML : '';
    if (clearBtn) {
      clearBtn.disabled = true;
      clearBtn.innerHTML = '<span class="material-symbols-outlined" style="font-size:16px;">hourglass_top</span> กำลังล้าง...';
    }

    google.script.run
      .withSuccessHandler(function(res) {
        if (clearBtn) { clearBtn.disabled = false; clearBtn.innerHTML = origHTML; }
        if (res && res.success) {
          item['ช่วงใช้งาน'] = '';
          item['จุดทดสอบ']   = '';
          _updateRowSummary(id, params, {}, {});
          toggleExpand(id);
          showToast('ล้างข้อมูลเรียบร้อยแล้ว', 'info');
        } else {
          showToast('เกิดข้อผิดพลาด', 'danger');
        }
      })
      .withFailureHandler(function(err) {
        if (clearBtn) { clearBtn.disabled = false; clearBtn.innerHTML = origHTML; }
        showToast('เซิร์ฟเวอร์ขัดข้อง: ' + err, 'danger');
      })
      .saveAllCalibration([{
        eqId:      id,
        usage:     '',
        testPoint: '',
        addedBy:   currentUser ? _getDisplayName() : 'System'
      }]);
  }

  function toggleExpand(id) {
    var row = document.getElementById('ep-row-' + id);
    if (!row) return;
    var isHidden = row.classList.contains('hidden');

    // ถ้าสถานะ "รับเครื่องส่งสอบเทียบ" → ล็อกไม่ให้เปิด
    if (isHidden) {
      var item = cachedEquipments ? cachedEquipments.find(function(e){ return e['ID'] === id; }) : null;
      if (item && item['สถานะการส่งสอบ'] === 'รับเครื่องส่งสอบเทียบ') {
        showToast('🔒 ไม่สามารถแก้ไขได้ — เครื่องถูกรับส่งสอบเทียบแล้ว', 'warning');
        return;
      }
    }

    document.querySelectorAll('[id^="ep-row-"]').forEach(function(r) { r.classList.add('hidden'); });
    if (isHidden) {
      row.classList.remove('hidden');
      row.scrollIntoView({ behavior: 'smooth', block: 'nearest' });
    }
  }

  function saveParamRow(id) {
    var item = cachedEquipments ? cachedEquipments.find(function(e){ return e['ID'] === id; }) : null;
    if (!item) { alert('ไม่พบข้อมูลเครื่องมือ'); return; }

    var ptype  = _getParamType(item['ประเภทเครื่องมือ']);
    var params = _calParamConfig[ptype] || _calParamConfig['default'];

    var limitObj  = {};
    var pointsObj = {};
    params.forEach(function(p) {
      var limEl = document.getElementById('ep-limit-'  + id + '-' + p.key);
      var ptEl  = document.getElementById('ep-points-' + id + '-' + p.key);
      if (limEl && limEl.value.trim())  limitObj[p.key + '_limit']  = limEl.value.trim();
      if (ptEl  && ptEl.value.trim())   pointsObj[p.key + '_points'] = ptEl.value.trim();
    });

    if (Object.keys(limitObj).length === 0 && Object.keys(pointsObj).length === 0) {
      showToast('กรุณากรอกข้อมูลอย่างน้อย 1 ช่อง', 'warning'); return;
    }

    // visual feedback บนปุ่ม
    var saveBtn = document.querySelector('[data-save-id="' + id + '"]');
    var origHTML = '';
    if (saveBtn) {
      origHTML = saveBtn.innerHTML;
      saveBtn.disabled = true;
      saveBtn.innerHTML = '<span class="material-symbols-outlined text-sm animate-spin">progress_activity</span> กำลังบันทึก...';
      saveBtn.style.opacity = '0.7';
    }

    function restoreBtn() {
      if (saveBtn) { saveBtn.disabled = false; saveBtn.innerHTML = origHTML; saveBtn.style.opacity = ''; }
    }

    var payload = [{
      eqId:      id,
      usage:     JSON.stringify(limitObj),
      testPoint: JSON.stringify(pointsObj),
      addedBy:   currentUser ? _getDisplayName() : 'System'
    }];

    google.script.run
      .withSuccessHandler(function(res) {
        restoreBtn();
        if (res.success) {
          item['ช่วงใช้งาน'] = JSON.stringify(limitObj);
          item['จุดทดสอบ']   = JSON.stringify(pointsObj);
          _updateRowSummary(id, params, limitObj, pointsObj);
          toggleExpand(id);
          showToast('✅ บันทึกข้อมูลเรียบร้อยแล้ว', 'success');

          // ตั้งสถานะ "รอส่งการสอบเทียบ" อัตโนมัติ (ถ้ายังไม่มีสถานะ หรือไม่ใช่ "รับเครื่องแล้ว")
          var curStatus = item['สถานะการส่งสอบ'] || '';
          if (curStatus !== 'รับเครื่องส่งสอบเทียบ') {
            google.script.run
              .withSuccessHandler(function(r) {
                if (r && r.success) {
                  item['สถานะการส่งสอบ'] = 'รอส่งการสอบเทียบ';
                  _refreshSendStatusBadge(id, 'รอส่งการสอบเทียบ');
                }
              })
              .withFailureHandler(function(){})
              .updateCalSendStatus({
                eqId:      id,
                rowIndex:  item._rowIndex || (item._index !== undefined ? item._index + 2 : null),
                status:    'รอส่งการสอบเทียบ',
                updatedBy: currentUser ? _getDisplayName() : 'System'
              });
          }
        } else {
          showToast('เกิดข้อผิดพลาด: ' + (res.message||''), 'danger');
        }
      })
      .withFailureHandler(function(err) {
        restoreBtn();
        showToast('เซิร์ฟเวอร์ขัดข้อง: ' + err, 'danger');
      })
      .saveAllCalibration(payload);
  }

  function _refreshSendStatusBadge(eqId, newStatus) {
    // อัปเดต badge ในคอลัมน์สถานะส่งสอบของแถวที่มี ep-summary-{id}
    var btn = document.querySelector('[data-send-eqid="' + eqId + '"]');
    if (!btn) return;
    var td = btn.parentElement;
    if (!td) return;
    var badgeHtml = '';
    if (newStatus === 'รับเครื่องส่งสอบเทียบ') {
      badgeHtml = '<span style="display:inline-flex;align-items:center;gap:4px;padding:3px 8px;border-radius:20px;font-size:10px;font-weight:700;background:#ede9fe;color:#7c3aed;">📦 รับเครื่องแล้ว</span>';
    } else if (newStatus === 'รอส่งการสอบเทียบ') {
      badgeHtml = '<span style="display:inline-flex;align-items:center;gap:4px;padding:3px 8px;border-radius:20px;font-size:10px;font-weight:700;background:#dbeafe;color:#1d4ed8;">⏳ รอส่งสอบ</span>';
    } else {
      badgeHtml = '<span style="font-size:10px;color:#cbd5e1;">—</span>';
    }
    // แทนเฉพาะ badge (เก็บปุ่มแก้ไขไว้)
    td.innerHTML = badgeHtml + ' <button data-send-eqid="' + eqId + '" class="ep-send-btn" style="margin-left:4px;padding:2px 6px;border:1px solid #e2e8f0;border-radius:6px;font-size:10px;color:#64748b;background:#fff;cursor:pointer;vertical-align:middle;">แก้ไข</button>';
  }

  function _updateRowSummary(id, params, limitObj, pointsObj) {
    var summaryEl = document.getElementById('ep-summary-' + id);
    if (!summaryEl) return;
    var badges = params.map(function(p) {
      var lv = limitObj[p.key + '_limit']  || '';
      var pv = pointsObj[p.key + '_points'] || '';
      if (!lv && !pv) return '';
      return '<span class="inline-flex items-center gap-1 mr-1 mb-0.5">'
        + '<span class="text-[10px] font-bold text-primary-600 dark:text-primary-400 bg-primary-50 dark:bg-primary-900/30 rounded px-1.5 py-0.5">' + p.label + '</span>'
        + (lv ? '<span class="text-[10px] text-slate-600 dark:text-slate-300 font-mono">' + lv + '</span>' : '')
        + (pv ? '<span class="text-[9px] text-slate-400">▸ ' + pv + '</span>' : '')
        + '</span>';
    }).join('');

    if (badges.trim()) {
      summaryEl.innerHTML = '<div class="flex flex-wrap items-center gap-1">'
        + badges
        + '<button data-view-id="' + id + '" class="ep-view-btn ml-1 inline-flex items-center gap-0.5 px-2 py-0.5 rounded text-[10px] font-medium border border-slate-200 dark:border-slate-600 text-slate-500 hover:bg-slate-100 dark:hover:bg-slate-700 transition">'
        +   '<span class="material-symbols-outlined text-[13px]">visibility</span> ดู'
        + '</button>'
        + '</div>';
    } else {
      summaryEl.innerHTML = '<span class="text-slate-300 dark:text-slate-600 text-[10px] italic">ยังไม่มีข้อมูล</span>';
    }
  }

  function renderTableBody(data) {
    var tbody = document.getElementById('equipment-table-body');
    var countEl = document.getElementById('table-row-count');
    if (!tbody) return;
    tbody.innerHTML = '';

    if (!data || data.length === 0) {
      tbody.innerHTML = '<tr><td colspan="7" class="px-4 py-10 text-center text-slate-400"><span class="material-symbols-outlined text-3xl block mb-2 text-slate-300">inventory_2</span>ยังไม่มีข้อมูลในระบบ</td></tr>';
      if (countEl) countEl.textContent = 'ไม่พบข้อมูล';
      return;
    }

    var today = new Date(); today.setHours(0,0,0,0);

    data.forEach(function(item, index) {
      var lastCalStr = item['วันที่ทำการสอบเทียบ (Cal-Date)'] || item['วันที่สอบเทียบล่าสุด'] || '';
      var intv       = item['ระยะเวลาการสอบเทียบ'] || '';
      var isCal      = _needsSchedule(item['การสอบเทียบหรือไม่'] || item['สถานะการสอบเทียบ'] || '', item);
      var dueDateIso = '', dueDateText = '-', dueDateClass = 'text-slate-500 dark:text-slate-400';

      if (isCal && lastCalStr && intv) {
        var d = new Date(lastCalStr);
        if (!isNaN(d)) {
          d.setMonth(d.getMonth() + (parseInt(intv) || 12));
          d.setDate(d.getDate() - 1);
          dueDateIso  = d.toISOString().slice(0, 10);
          dueDateText = d.toLocaleDateString('th-TH', { year:'numeric', month:'short', day:'numeric' });
          var diff = Math.ceil((d - today) / (1000*60*60*24));
          if (diff < 0)       dueDateClass = 'text-red-600 font-bold';
          else if (diff <= 30) dueDateClass = 'text-orange-600 font-bold';
          else if (diff <= 60) dueDateClass = 'text-amber-600 font-bold';
          else                 dueDateClass = 'text-emerald-600 font-bold';
        }
      } else {
        dueDateText = isCal ? 'ข้อมูลไม่ครบ' : _noCalLabel(item);
      }

      // สร้าง summary badge จากข้อมูลที่บันทึกไว้
      var ptype  = _getParamType(item['ประเภทเครื่องมือ']);
      var params = _calParamConfig[ptype] || _calParamConfig['default'];
      var savedRaw = item['ช่วงใช้งาน'] || '';
      var savedTP  = item['จุดทดสอบ']   || '';
      var savedObj = {}, savedTPObj = {};
      try { savedObj   = JSON.parse(savedRaw); } catch(e) { if (savedRaw && params[0]) savedObj[params[0].key + '_limit']   = savedRaw; }
      try { savedTPObj = JSON.parse(savedTP);  } catch(e) { if (savedTP  && params[0]) savedTPObj[params[0].key + '_points'] = savedTP;  }

      var hasSaved = Object.keys(savedObj).length > 0 || Object.keys(savedTPObj).length > 0;
      var savedBadges = hasSaved
        ? params.map(function(p) {
            var lv = savedObj[p.key + '_limit'] || '';
            var pv = savedTPObj[p.key + '_points'] || '';
            if (!lv && !pv) return '';
            return '<span class="inline-flex items-center gap-1 mr-1">'
              + '<span class="text-[10px] font-bold text-primary-600 dark:text-primary-400 bg-primary-50 dark:bg-primary-900/30 rounded px-1.5 py-0.5">' + p.label + '</span>'
              + (lv ? '<span class="text-[10px] text-slate-600 dark:text-slate-300 font-mono">' + lv + '</span>' : '')
              + (pv ? '<span class="text-[9px] text-slate-400">▸ ' + pv + '</span>' : '')
              + '</span>';
          }).join('')
        : '';
      var summaryHtml = savedBadges
        ? '<div class="flex flex-wrap items-center gap-1">' + savedBadges
          + '<button data-view-id="' + item['ID'] + '" class="ep-view-btn ml-1 inline-flex items-center gap-0.5 px-2 py-0.5 rounded text-[10px] font-medium border border-slate-200 dark:border-slate-600 text-slate-500 hover:bg-slate-100 dark:hover:bg-slate-700 transition"><span class="material-symbols-outlined text-[13px]">visibility</span> ดู</button>'
          + '</div>'
        : '<span class="text-[10px] text-slate-300 dark:text-slate-600 italic">ยังไม่มีข้อมูล</span>';

      var row = '<tr class="hover:bg-slate-50/60 dark:hover:bg-slate-700/30 transition" '
        + 'data-name="' + (item['ชื่อเครื่องมือ']||'').toLowerCase() + '" '
        + 'data-code="' + (item['รหัสครุภัณฑ์']||'').toLowerCase() + '" '
        + 'data-due="' + dueDateIso + '">'
        + '<td class="px-4 py-3 text-center text-slate-400 text-xs">' + (index+1) + '</td>'
        + '<td class="px-4 py-3 font-bold text-primary-600 dark:text-primary-400 whitespace-nowrap text-sm">' + (item['รหัสครุภัณฑ์']||'-') + '</td>'
        + '<td class="px-4 py-3">'
        +   '<div class="font-semibold text-slate-800 dark:text-slate-100 text-sm">' + (item['ชื่อเครื่องมือ']||'-') + '</div>'
        +   '<div class="text-[11px] text-slate-400 mt-0.5">' + (item['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง']||'') + '</div>'
        + '</td>'
        + '<td class="px-4 py-3 text-xs text-slate-500 whitespace-nowrap">' + (item['สถานที่ติดตั้ง']||'-') + '</td>'
        + '<td class="px-4 py-3 text-center text-xs ' + dueDateClass + ' whitespace-nowrap">' + dueDateText + '</td>'
        + (function(){
            var ss  = item['สถานะการส่งสอบ'] || '';
            var ec  = item['สถานะเครื่องมือ'] || '';
            var badge = '', ecBadge = '', btn = '';

            // badge สถานะส่งสอบ
            if (ss === 'รับเครื่องส่งสอบเทียบ') {
              badge = '<span style="display:inline-flex;align-items:center;gap:4px;padding:3px 8px;border-radius:20px;font-size:10px;font-weight:700;background:#ede9fe;color:#7c3aed;">📦 รับเครื่องแล้ว</span>';
            } else if (ss === 'รอส่งการสอบเทียบ') {
              badge = '<span style="display:inline-flex;align-items:center;gap:4px;padding:3px 8px;border-radius:20px;font-size:10px;font-weight:700;background:#dbeafe;color:#1d4ed8;">⏳ รอส่งสอบ</span>';
            } else {
              badge = '<span style="font-size:10px;color:#cbd5e1;">—</span>';
            }

            // badge สถานะเครื่องมือ (แสดงแยกถ้ามีปัญหา)
            if (ec === 'สอบเทียบไม่ผ่านเกณฑ์') {
              ecBadge = '<span style="display:block;margin-top:3px;padding:2px 7px;border-radius:20px;font-size:9px;font-weight:700;background:#fee2e2;color:#b91c1c;">❌ ไม่ผ่านเกณฑ์</span>';
            } else if (ec === 'เครื่องมือเสีย') {
              ecBadge = '<span style="display:block;margin-top:3px;padding:2px 7px;border-radius:20px;font-size:9px;font-weight:700;background:#f5f5f4;color:#57534e;">🔧 เครื่องเสีย</span>';
            } else if (ec === 'ส่งซ่อม') {
              ecBadge = '<span style="display:block;margin-top:3px;padding:2px 7px;border-radius:20px;font-size:9px;font-weight:700;background:#ffedd5;color:#c2410c;">🛠️ ส่งซ่อม</span>';
            } else if (ec === 'ระงับการใช้งาน') {
              ecBadge = '<span style="display:block;margin-top:3px;padding:2px 7px;border-radius:20px;font-size:9px;font-weight:700;background:#ede9fe;color:#6d28d9;">🚫 ระงับการใช้</span>';
            } else if (ec === 'จำหน่ายออก') {
              ecBadge = '<span style="display:block;margin-top:3px;padding:2px 7px;border-radius:20px;font-size:9px;font-weight:700;background:#f1f5f9;color:#475569;">📦 จำหน่ายออก</span>';
            } else if (ec === 'รอสอบเทียบใหม่') {
              ecBadge = '<span style="display:block;margin-top:3px;padding:2px 7px;border-radius:20px;font-size:9px;font-weight:700;background:#fef9c3;color:#a16207;">🔄 รอสอบใหม่</span>';
            }

            if (isCal) btn = '<button data-send-eqid="' + (item['ID']||'') + '" class="ep-send-btn" style="margin-left:4px;padding:2px 6px;border:1px solid #e2e8f0;border-radius:6px;font-size:10px;color:#64748b;background:#fff;cursor:pointer;vertical-align:middle;">แก้ไข</button>';
            return '<td class="px-3 py-2.5 text-center whitespace-nowrap">'
              + badge + btn + ecBadge
              + '</td>';
          })()
        + '<td class="px-3 py-2.5">'
        +   '<div class="flex items-start gap-2">'
        +     '<div id="ep-summary-' + item['ID'] + '" class="flex-1 flex flex-wrap items-center min-h-[24px]">' + summaryHtml + '</div>'
        +     (item['สถานะการส่งสอบ'] === 'รับเครื่องส่งสอบเทียบ'
              ? '<span style="display:inline-flex;align-items:center;gap:4px;padding:4px 8px;border-radius:8px;background:#f3f4f6;color:#9ca3af;font-size:11px;" title="ล็อกระหว่างรับเครื่องส่งสอบ"><span class="material-symbols-outlined text-[13px]">lock</span> ล็อก</span>'
              : '<button data-edit-id="' + item['ID'] + '" class="ep-edit-btn shrink-0 flex items-center gap-1 px-2.5 py-1.5 rounded-lg border border-slate-200 dark:border-slate-600 text-slate-500 dark:text-slate-400 hover:border-primary-400 hover:text-primary-600 hover:bg-primary-50 dark:hover:bg-primary-900/20 text-[11px] font-medium transition">'
              + '<span class="material-symbols-outlined text-[14px]">edit_note</span> กรอก'
              + '</button>')
        +   '</div>'
        + '</td>'
        + '</tr>';

      tbody.insertAdjacentHTML('beforeend', row);
      // เพิ่ม expand panel ต่อจากแถวหลัก
      tbody.insertAdjacentHTML('beforeend', _buildExpandPanel(item));
    });

    if (countEl) countEl.textContent = 'แสดง ' + data.length + ' รายการ';
  }

  function filterTableSearch(query) {
    var q = (query||'').toLowerCase().trim();
    var rows = document.querySelectorAll('#equipment-table-body tr[data-name]');
    var visible = 0;
    rows.forEach(function(row) {
      var show = !q || (row.getAttribute('data-name')||'').includes(q) || (row.getAttribute('data-code')||'').includes(q);
      row.style.display = show ? '' : 'none';
      if (show) visible++;
    });
    var countEl = document.getElementById('table-row-count');
    if (countEl && q) countEl.innerHTML = 'แสดง ' + visible + ' จาก ' + rows.length + ' รายการ';
  }

  // ==========================================
  // ข้อ 4: พิมพ์ภาพรวม PDF
  // ==========================================
  function printDashboardPdf() {
    if (!cachedEquipments || cachedEquipments.length === 0) {
      alert('กรุณาโหลดข้อมูลก่อนพิมพ์'); return;
    }
    var dept  = currentUser ? currentUser.dept : '';
    var role  = currentUser ? currentUser.role : '';
    var today = new Date(); today.setHours(0,0,0,0);
    var todayStr = new Date().toLocaleDateString('th-TH',{year:'numeric',month:'long',day:'numeric'});
    var scopeLabel = role === 'Admin' ? 'ทุกห้องปฏิบัติการ' : 'ห้องปฏิบัติการ: ' + dept;

    var tableRows = cachedEquipments.map(function(item, i) {
      var eqCode  = item['รหัสครุภัณฑ์'] || '-';
      var eqName  = item['ชื่อเครื่องมือ'] || '-';
      var eqType  = item['ประเภทเครื่องมือ'] || '-';
      var loc     = item['สถานที่ติดตั้ง'] || '-';
      var lastCal = item['วันที่ทำการสอบเทียบ (Cal-Date)'] || item['วันที่สอบเทียบล่าสุด'] || '-';
      var intv    = item['ระยะเวลาการสอบเทียบ'] || '-';
      var isCal   = _needsSchedule(item['การสอบเทียบหรือไม่'] || item['สถานะการสอบเทียบ'] || '', item);
      var dueText = '-';
      var statusText = isCal ? 'ไม่สอบเทียบ' : _noCalLabel(item);
      var rowBg = '';

      if (isCal && lastCal !== '-' && intv !== '-') {
        var d = new Date(lastCal);
        if (!isNaN(d)) {
          d.setMonth(d.getMonth() + (parseInt(intv) || 12));
          d.setDate(d.getDate()-1);
          dueText = d.toLocaleDateString('th-TH',{year:'numeric',month:'short',day:'numeric'});
          var diff = Math.ceil((d - today)/(1000*60*60*24));
          if (diff < 0)      { statusText = '🔴 เกินกำหนด';       rowBg = '#fef2f2'; }
          else if (diff<=30) { statusText = '🟠 ใกล้ครบ 30 วัน';  rowBg = '#fff7ed'; }
          else if (diff<=60) { statusText = '🟡 ใกล้ครบ 60 วัน';  rowBg = '#fffbeb'; }
          else               { statusText = '🟢 ปกติ';             rowBg = ''; }
        }
      }

      var lastCalFmt = lastCal !== '-' && !isNaN(new Date(lastCal))
        ? new Date(lastCal).toLocaleDateString('th-TH',{year:'numeric',month:'short',day:'numeric'}) : '-';

      return '<tr style="background:' + rowBg + '">'
        + '<td style="border:1px solid #e2e8f0;padding:5px 7px;text-align:center">' + (i+1) + '</td>'
        + '<td style="border:1px solid #e2e8f0;padding:5px 7px;font-weight:700;color:#1e40af">' + eqCode + '</td>'
        + '<td style="border:1px solid #e2e8f0;padding:5px 7px;font-weight:600">' + eqName + '</td>'
        + '<td style="border:1px solid #e2e8f0;padding:5px 7px">' + eqType + '</td>'
        + '<td style="border:1px solid #e2e8f0;padding:5px 7px">' + loc + '</td>'
        + '<td style="border:1px solid #e2e8f0;padding:5px 7px;text-align:center">' + lastCalFmt + '</td>'
        + '<td style="border:1px solid #e2e8f0;padding:5px 7px;text-align:center">' + intv + '</td>'
        + '<td style="border:1px solid #e2e8f0;padding:5px 7px;text-align:center;font-weight:700">' + dueText + '</td>'
        + '<td style="border:1px solid #e2e8f0;padding:5px 7px;text-align:center;font-weight:700">' + statusText + '</td>'
        + '</tr>';
    }).join('');

    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="UTF-8"><title>ภาพรวมเครื่องมือ</title><style>'
      + '@page{size:A4 landscape;margin:12mm}'
      + 'body{font-family:"Sarabun","Tahoma",Arial,sans-serif;font-size:9.5pt;color:#1e293b;margin:0}'
      + 'h1{font-size:13pt;font-weight:700;color:#1e40af;margin:0 0 2px}'
      + 'p{font-size:9pt;color:#64748b;margin:0 0 8px}'
      + 'table{width:100%;border-collapse:collapse;font-size:9pt}'
      + 'th{background:#1e40af;color:white;padding:6px 7px;text-align:center;border:1px solid #1e3a8a;white-space:nowrap}'
      + '.leg{display:inline-flex;align-items:center;gap:4px;font-size:8pt;margin-right:12px}'
      + '.dot{width:10px;height:10px;border-radius:2px;border:1px solid #cbd5e1}'
      + '@media print{body{-webkit-print-color-adjust:exact;print-color-adjust:exact}}'
      + '</style></head><body>'
      + '<h1>รายงานภาพรวมเครื่องมือ — ' + scopeLabel + '</h1>'
      + '<p>วันที่พิมพ์: ' + todayStr + ' &nbsp;|&nbsp; จำนวน: ' + cachedEquipments.length + ' รายการ</p>'
      + '<div style="margin-bottom:8px">'
      + '<span class="leg"><span class="dot" style="background:#fef2f2"></span>เกินกำหนด</span>'
      + '<span class="leg"><span class="dot" style="background:#fff7ed"></span>ใกล้ครบ 30 วัน</span>'
      + '<span class="leg"><span class="dot" style="background:#fffbeb"></span>ใกล้ครบ 60 วัน</span>'
      + '<span class="leg"><span class="dot" style="background:#f0fdf4"></span>ปกติ</span>'
      + '</div>'
      + '<table><thead><tr>'
      + '<th>#</th><th>รหัสครุภัณฑ์</th><th>ชื่อเครื่องมือ</th><th>ประเภท</th><th>สถานที่</th>'
      + '<th>สอบเทียบล่าสุด</th><th>รอบ</th><th>Due Date</th><th>สถานะ</th>'
      + '</tr></thead><tbody>' + tableRows + '</tbody></table>'

      + '</body></html>';

    var win = window.open('','_blank','width=1100,height=800');
    if (!win) { alert('กรุณาอนุญาต Popup ก่อนครับ'); return; }
    win.document.write(html);
    win.document.close();
    win.focus();
    setTimeout(function(){ win.print(); }, 600);
  }

  function filterTableNear60(onlyNear) {
    filterTableByDiff(onlyNear ? 60 : null);
  }

  function filterTableByDiff(maxDays) {
    var today = new Date(); today.setHours(0,0,0,0);
    var all   = cachedEquipments || [];

    function getDiff(item) {
      var lastCal = item['วันที่ทำการสอบเทียบ (Cal-Date)'] || item['วันที่สอบเทียบล่าสุด'] || '';
      var intv    = item['ระยะเวลาการสอบเทียบ'] || '';
      var isCal   = _needsSchedule(item['การสอบเทียบหรือไม่'] || item['สถานะการสอบเทียบ'] || '', item);
      if (!isCal || !lastCal || !intv) return null;
      var d = new Date(lastCal);
      if (isNaN(d)) return null;
      d.setMonth(d.getMonth() + (parseInt(intv) || 12));
      d.setDate(d.getDate() - 1);
      return Math.ceil((d - today) / (1000*60*60*24));
    }

    var filtered, labelHtml;

    if (maxDays === 'overdue') {
      filtered = all.filter(function(item) { var d = getDiff(item); return d !== null && d < 0; });
      labelHtml = '<span class="text-red-600 font-bold">🔴 เกินกำหนดสอบเทียบ: ' + filtered.length + ' รายการ</span>';
    } else if (maxDays === 'failed') {
      filtered = all.filter(function(item) { return (item['สถานะเครื่องมือ']||'') === 'สอบเทียบไม่ผ่านเกณฑ์'; });
      labelHtml = '<span class="text-red-700 font-bold">❌ ไม่ผ่านเกณฑ์การสอบเทียบ: ' + filtered.length + ' รายการ</span>';
    } else if (maxDays === 'broken') {
      filtered = all.filter(function(item) {
        var c = item['สถานะเครื่องมือ'] || '';
        return c === 'เครื่องมือเสีย' || c === 'ส่งซ่อม' || c === 'ระงับการใช้งาน';
      });
      labelHtml = '<span class="text-stone-700 font-bold">🔧 ส่งซ่อม / เสีย / ระงับ: ' + filtered.length + ' รายการ</span>';
    } else if (maxDays === null) {
      filtered = all;
      labelHtml = 'แสดงทั้งหมด <strong>' + all.length + '</strong> รายการ';
    } else if (maxDays === 30) {
      filtered = all.filter(function(item) { var d = getDiff(item); return d !== null && d >= 0 && d <= 30; });
      labelHtml = '<span class="text-red-600 font-bold">🔴 ใกล้ครบกำหนด (≤30 วัน): ' + filtered.length + ' รายการ</span>';
    } else {
      // 31-60
      filtered = all.filter(function(item) { var d = getDiff(item); return d !== null && d > 30 && d <= 60; });
      labelHtml = '<span class="text-amber-600 font-bold">⚠️ ใกล้ครบกำหนด (31–60 วัน): ' + filtered.length + ' รายการ</span>';
    }

    renderTableBody(filtered);

    var countEl = document.getElementById('table-row-count');
    if (countEl) {
      countEl.innerHTML = labelHtml + ' <span class="text-slate-400 text-xs ml-1">(ทั้งหมด ' + all.length + ' รายการ)</span>';
    }

    // อัปเดตปุ่ม active
    var filterBar = document.getElementById('table-filter-bar');
    if (filterBar) {
      var btnDefs = [null, 'overdue', 30, 60, 'failed', 'broken'];
      filterBar.querySelectorAll('button').forEach(function(btn, i) {
        var isActive = String(btnDefs[i]) === String(maxDays);
        btn.classList.toggle('ring-2', isActive);
        btn.classList.toggle('ring-offset-1', isActive);
        btn.classList.toggle('ring-primary-500', isActive);
        btn.style.fontWeight = isActive ? '900' : '';
        btn.style.opacity    = isActive ? '1' : '0.75';
      });
    }
  }

  function saveAllTableData() {
    // บันทึกทีละแถวผ่านปุ่ม "บันทึก" ใน expand panel แล้ว
    // ฟังก์ชันนี้คงไว้เพื่อ backward compat
    showToast('กรุณากดปุ่ม "บันทึก" ในแต่ละรายการที่ต้องการบันทึก', 'info');
  }

  // ==========================================
  // Individual Calibration Modals Operation
  // ==========================================
  function openCalModal(id, code, name) {
    document.getElementById('cal-eqId').value = id; 
    document.getElementById('cal-eqCode').value = code;
    document.getElementById('cal-eqName').value = name; 
    
    const container = document.getElementById('test-points-container');
    if (container) {
      const rows = container.querySelectorAll('.test-point-row');
      for(let i = 1; i < rows.length; i++) { rows[i].remove(); }
      const firstRow = container.querySelector('.test-point-row');
      if (firstRow) {
        firstRow.querySelector('.usage-field').value = '';
        firstRow.querySelector('.test-field').value = '';
      }
    }

    const eq = cachedEquipments.find(item => item['ID'] === id);
    let dueDate = '';

    if (eq) {
      const serialNo = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || eq['Serial No.'] || '-';
      const serialInput = document.getElementById('cal-serialNo');
      if (serialInput) serialInput.value = serialNo;

      dueDate = eq['วันที่สอบเทียบรอบถัดไป'] || eq['Due Date'] || '';
      if (!dueDate && eq['วันที่สอบเทียบล่าสุด'] && eq['ระยะเวลาการสอบเทียบ']) {
        let lastDate = new Date(eq['วันที่สอบเทียบล่าสุด']);
        let interval = String(eq['ระยะเวลาการสอบเทียบ']).trim();
        if (!isNaN(lastDate.getTime())) {
          if (interval === "6" || interval === "6 เดือน") lastDate.setMonth(lastDate.getMonth() + 6);
          else if (interval === "12" || interval === "1 ปี") lastDate.setFullYear(lastDate.getFullYear() + 1);
          else if (interval === "24" || interval === "2 ปี") lastDate.setFullYear(lastDate.getFullYear() + 2);
          dueDate = lastDate;
        }
      } else if (dueDate) {
        dueDate = new Date(dueDate);
      }

      if (dueDate instanceof Date && !isNaN(dueDate.getTime())) {
        const y = dueDate.getFullYear();
        const m = String(dueDate.getMonth() + 1).padStart(2, '0');
        const d = String(dueDate.getDate()).padStart(2, '0');
        dueDate = `${y}-${m}-${d}`;
      } else {
        dueDate = '';
      }
    }

    document.getElementById('cal-dueDate').value = dueDate;

    const addedByInput = document.getElementById('cal-addedBy');
    if (addedByInput) {
      addedByInput.value = (currentUser) ? currentUser.username : "System";
    }

    const addedDateInput = document.getElementById('cal-addedDate');
    if (addedDateInput) {
      const today = new Date();
      const thMonths = ["ม.ค.", "ก.พ.", "มี.ค.", "เม.ย.", "พ.ค.", "มิ.ย.", "ก.ค.", "ส.ค.", "ก.ย.", "ต.ค.", "พ.ย.", "ธ.ค."];
      addedDateInput.value = `${today.getDate()} ${thMonths[today.getMonth()]} ${today.getFullYear() + 543}`;
    }

    document.getElementById('calibration-modal').classList.remove('hidden');
  }

  function closeCalModal() { document.getElementById('calibration-modal').classList.add('hidden'); }

  document.addEventListener('DOMContentLoaded', function() {
    const addRowBtn = document.getElementById('add-test-row-btn');
    if (addRowBtn) {
      addRowBtn.addEventListener('click', addTestRow);
    }
  });

  function addTestRow() {
    const container = document.getElementById('test-points-container');
    if (!container) return;
    const div = document.createElement('div');
    div.className = 'test-point-row grid grid-cols-2 gap-2 border-b pb-2 animate-fade-in';
    div.innerHTML = `
      <input type="text" placeholder="ช่วงใช้งาน" required class="p-2 border rounded text-sm usage-field">
      <input type="text" placeholder="จุดทดสอบ" required class="p-2 border rounded text-sm test-field">
      <button type="button" onclick="this.parentElement.remove()" class="col-span-2 text-[10px] text-red-500 text-right">ลบแถวนี้</button>
    `;
    container.appendChild(div);
  }

  function submitMultipleCalibrations() {
    const rows = document.querySelectorAll('.test-point-row');
    const details = [];
    
    rows.forEach(row => {
      details.push({
        usage: row.querySelector('.usage-field').value,
        testPoint: row.querySelector('.test-field').value
      });
    });

    const payload = {
      eqId: document.getElementById('cal-eqId').value,
      eqCode: document.getElementById('cal-eqCode').value,
      eqName: document.getElementById('cal-eqName').value,
      serialNo: document.getElementById('cal-serialNo').value,
      dueDate: document.getElementById('cal-dueDate').value,
      details: details,
      addedBy: document.getElementById('cal-addedBy').value
    };

    showLoader();
    google.script.run.withSuccessHandler((res) => {
      hideLoader();
      if(res && res.success) {
        alert("บันทึกข้อมูลเรียบร้อย");
        document.getElementById('calibration-form').reset();
        closeCalModal();
      } else {
        alert("เกิดข้อผิดพลาด: " + (res ? res.message : "ไม่ทราบสาเหตุ"));
      }
    }).withFailureHandler(() => { hideLoader(); alert("เซิร์ฟเวอร์ขัดข้อง"); }).saveMultipleCalibration(payload);
  }

  function openExpenseModal(id, code, name) {
    document.getElementById('exp-eqId').value = id; document.getElementById('exp-eqCode').value = code;
    document.getElementById('exp-eqName').value = name; document.getElementById('exp-amount').value = '';
    document.getElementById('exp-date').value = new Date().toISOString().split('T')[0];
    document.getElementById('expense-modal').classList.remove('hidden');
  }
  function closeExpenseModal() { document.getElementById('expense-modal').classList.add('hidden'); }
  
  function submitExpense() {
    showLoader();
    const payload = {
      eqId: document.getElementById('exp-eqId').value, eqCode: document.getElementById('exp-eqCode').value,
      amount: document.getElementById('exp-amount').value, date: document.getElementById('exp-date').value,
      addedBy: currentUser ? _getDisplayName() : 'System'
    };
    google.script.run.withSuccessHandler(function(res) {
      hideLoader();
      if(res.success) { alert('บันทึกค่าใช้จ่ายเรียบร้อยแล้ว'); document.getElementById('expense-form').reset(); closeExpenseModal(); }
      else { alert('เกิดข้อผิดพลาด: ' + res.message); }
    }).withFailureHandler(() => { hideLoader(); alert("เซิร์ฟเวอร์ขัดข้อง"); }).saveExpense(payload);
  }

  // ==========================================
  // Verification Modules for Lab Equipments
  // ==========================================

  // ==========================================
  // AUTOMATIC PIPETTE MODULE
  // ==========================================
  function loadEquipmentsForPipette() {
    const dept = currentUser ? currentUser.dept : "";
    const role = currentUser ? currentUser.role : "";

    google.script.run.withSuccessHandler(function(data) {
      cachedEquipments = data; 
      cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
      const select = document.getElementById('pi-eqSelect');
      if (!select) return;
      select.innerHTML = '<option value="">-- เลือกรายการเครื่องมือ --</option>';
      // กรองเฉพาะประเภทที่เกี่ยวข้อง
      // ใช้ _getParamType เหมือนกับระบบอื่น
      var filtered = data.filter(function(item) {
        return _getParamType(item['ประเภทเครื่องมือ'] || '') === 'pipette';
      });
      filtered.forEach(function(item) {
        select.innerHTML += '<option value="' + item['ID'] + '">' + (item['รหัสครุภัณฑ์']||'-') + ' | ' + (item['ชื่อเครื่องมือ']||'-') + '</option>';
      });
      
      document.getElementById('pipette-rows-container').innerHTML = '';
      pipetteRowCount = 0;
      addPipetteRow();
      addPipetteRow();
      addPipetteRow();
    }).getEquipments(dept, role);
  }

  function autoFillPipetteInfo() {
    const eq = cachedEquipments.find(item => item['ID'] === document.getElementById('pi-eqSelect').value);
    if(eq) {
      document.getElementById('pi-serialNo').value = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || "ไม่ระบุ";
      document.getElementById('pi-assetBrand').value = (eq['รหัสครุภัณฑ์'] || "") + " / " + (eq['ชื่อเครื่องมือ'] || "");
    }
    if (currentUser) {
      const rName = document.getElementById('print-recorder-name');
      if (rName) rName.innerText = (_getDisplayName() || currentUser.username);
    }
    const today = new Date();
    const rDate = document.getElementById('print-recorder-date');
    if (rDate) rDate.innerText = today.toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
  }

  function addPipetteRow() {
    pipetteRowCount++;
    const id = pipetteRowCount;
    const container = document.getElementById('pipette-rows-container');
    if (!container) return;
    
    const rowHtml = `
      <tr id="pi-row-${id}" class="hover:bg-slate-50 dark:hover:bg-slate-800/40 transition pi-row-item">
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="pi-testVol-${id}" required class="w-full p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center font-bold print:border-none print:bg-transparent">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="pi-meanVol-${id}" oninput="calcPipetteRow(${id})" required class="w-full p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center print:border-none print:bg-transparent">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="pi-uncertainty-${id}" oninput="calcPipetteRow(${id})" required class="w-full p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center print:border-none print:bg-transparent">
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center bg-slate-50 dark:bg-slate-900/20 font-bold text-slate-700 dark:text-slate-300 print:bg-transparent">
          <span id="pi-range-text-${id}">-</span>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center">
          <div class="flex items-center justify-center gap-1">
            <input type="number" step="any" id="pi-ctrlMin-${id}" oninput="calcPipetteRow(${id})" placeholder="Min" required class="w-16 p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center text-blue-600 font-bold print:border-none print:bg-transparent print:w-auto">
            <span class="text-slate-400">-</span>
            <input type="number" step="any" id="pi-ctrlMax-${id}" oninput="calcPipetteRow(${id})" placeholder="Max" required class="w-16 p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center text-blue-600 font-bold print:border-none print:bg-transparent print:w-auto">
          </div>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center font-bold">
          <span id="pi-status-badge-${id}" data-status="WAITING" class="px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black">รอคำนวณ</span>
        </td>
        <td class="p-2 text-center print:hidden">
          <button type="button" onclick="removePipetteRow(${id})" class="text-red-500 hover:text-red-700 transition-colors">
            <span class="material-symbols-outlined text-lg">delete</span>
          </button>
        </td>
      </tr>`;
    container.insertAdjacentHTML('beforeend', rowHtml);
    checkPipetteOverallStatus();
  }

  function removePipetteRow(id) {
    const row = document.getElementById(`pi-row-${id}`);
    if(row) { row.remove(); checkPipetteOverallStatus(); }
  }

  function calcPipetteRow(id) {
    const meanVol = parseFloat(document.getElementById(`pi-meanVol-${id}`).value);
    const uncertainty = parseFloat(document.getElementById(`pi-uncertainty-${id}`).value);
    const ctrlMin = parseFloat(document.getElementById(`pi-ctrlMin-${id}`).value);
    const ctrlMax = parseFloat(document.getElementById(`pi-ctrlMax-${id}`).value);
    
    const rangeSpan = document.getElementById(`pi-range-text-${id}`);
    const badge = document.getElementById(`pi-status-badge-${id}`);
    
    if (isNaN(meanVol) || isNaN(uncertainty)) {
      rangeSpan.innerText = "-";
      badge.className = "px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอคำนวณ";
      badge.setAttribute('data-status', 'WAITING');
      checkPipetteOverallStatus();
      return;
    }
    
    const minCalculated = parseFloat((meanVol - Math.abs(uncertainty)).toFixed(2));
    const maxCalculated = parseFloat((meanVol + Math.abs(uncertainty)).toFixed(2));
    rangeSpan.innerText = `${minCalculated.toFixed(2)}  -  ${maxCalculated.toFixed(2)}`;
    
    if (isNaN(ctrlMin) || isNaN(ctrlMax)) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-amber-50 text-amber-600 border border-amber-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอช่วงควบคุม";
      badge.setAttribute('data-status', 'WAITING');
      checkPipetteOverallStatus();
      return;
    }
    
    if (minCalculated >= ctrlMin && maxCalculated <= ctrlMax) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-green-100 text-green-700 border border-green-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ยอมรับ";
      badge.setAttribute('data-status', 'PASS');
    } else {
      badge.className = "px-2 py-1 rounded text-[11px] bg-red-100 text-red-700 border border-red-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ปฏิเสธ";
      badge.setAttribute('data-status', 'FAIL');
    }
    checkPipetteOverallStatus();
  }

  function checkPipetteOverallStatus() {
    const container = document.getElementById('pipette-rows-container');
    if (!container) return;
    const rows = container.querySelectorAll('.pi-row-item');
    const overallStatusEl = document.getElementById('pi-overall-status');
    if (!overallStatusEl) return;
    
    if (rows.length === 0) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-slate-200 text-slate-700 font-bold text-sm shadow-sm print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "รอข้อมูลจุดทดสอบ";
      return;
    }
    
    let totalPass = true;
    let hasWaiting = false;
    
    rows.forEach(row => {
      const id = row.id.replace('pi-row-', '');
      const badge = document.getElementById(`pi-status-badge-${id}`);
      const status = badge.getAttribute('data-status');
      if (status === 'FAIL') totalPass = false;
      if (status === 'WAITING') hasWaiting = true;
    });
    
    if (hasWaiting) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-amber-100 text-amber-700 font-bold text-sm border border-amber-200 animate-pulse print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "กำลังรอข้อมูลให้ครบถ้วน...";
      return;
    }
    
    if (totalPass) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-green-600 text-white font-bold text-sm shadow border border-green-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ยอมรับและนำไปใช้งานได้";
    } else {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-red-600 text-white font-bold text-sm shadow border border-red-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ปฏิเสธและไม่นำไปใช้งาน";
    }
  }

  window.submitPipetteForm = function() {
    const form = document.getElementById('pipette-form');
    if(!form.checkValidity()) { form.reportValidity(); return; }
    
    const container = document.getElementById('pipette-rows-container');
    const rows = container.querySelectorAll('.pi-row-item');
    if(rows.length === 0) {
      alert('กรุณาเพิ่มรายการจุดทดสอบในตารางอย่างน้อย 1 รายการครับ');
      return;
    }
    
    const testVolArr = []; const meanVolArr = []; const uncertaintyArr = [];
    const meanUncertaintyArr = []; const controlRangeArr = []; const resultArr = [];
    let incomplete = false;
    
    rows.forEach(row => {
      const id = row.id.replace('pi-row-', '');
      const tv = document.getElementById(`pi-testVol-${id}`).value;
      const mv = document.getElementById(`pi-meanVol-${id}`).value;
      const uc = document.getElementById(`pi-uncertainty-${id}`).value;
      const ran = document.getElementById(`pi-range-text-${id}`).innerText;
      const cMin = document.getElementById(`pi-ctrlMin-${id}`).value;
      const cMax = document.getElementById(`pi-ctrlMax-${id}`).value;
      const res = document.getElementById(`pi-status-badge-${id}`).innerText;
      
      if(!tv || !mv || !uc || !cMin || !cMax) incomplete = true;
      
      testVolArr.push(tv);
      meanVolArr.push(mv);
      uncertaintyArr.push(uc);
      meanUncertaintyArr.push(ran);
      controlRangeArr.push(`${cMin} - ${cMax}`);
      resultArr.push(res);
    });
    
    if(incomplete) {
      alert('กรุณากรอกข้อมูลตัวเลขในตารางจุดทดสอบให้ครบถ้วนทุกช่องครับ');
      return;
    }
    
    const fileInput = document.getElementById('pi-pdfFile');
    const overallStatus = document.getElementById('pi-overall-status').innerText;
    
    const payload = {
      calDept: document.getElementById('pi-cal-dept') ? document.getElementById('pi-cal-dept').value : "",
      eqId: document.getElementById('pi-eqSelect').value,
      serialNo: document.getElementById('pi-serialNo').value,
      assetBrand: document.getElementById('pi-assetBrand').value,
      calDate: document.getElementById('pi-calDate').value,
      certNo: document.getElementById('pi-certNo').value,
      capacity: document.getElementById('pi-capacity').value,
      testVol: testVolArr.join('\n'),
      meanVol: meanVolArr.join('\n'),
      uncertainty: uncertaintyArr.join('\n'),
      meanUncertainty: meanUncertaintyArr.join('\n'),
      controlRange: controlRangeArr.join('\n'),
      result: resultArr.join('\n') + '\n\nสรุป: ' + overallStatus,
      addedBy: currentUser ? _getDisplayName() : 'System'
    };
    _calEditInject('pipette', payload); // ✅ โหมดแก้ไข: ส่งเลขแถวเดิมไปเขียนทับ
    
    if (fileInput && fileInput.files.length > 0) {
      showLoader();
      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        payload.pdfBase64 = e.target.result.split(',')[1];
        payload.pdfName = file.name;
        executeSubmitPipette(payload);
      };
      reader.readAsDataURL(file);
    } else {
      if(confirm("ยืนยันบันทึกผลการตรวจสอบ Pipette โดยไม่มีไฟล์เอกสารแนบใช่หรือไม่?")) {
        showLoader();
        executeSubmitPipette(payload);
      }
    }
  }
  
  function executeSubmitPipette(payload) {
    google.script.run.withSuccessHandler(function(res) {
      if(res.success) {
        google.script.run.withSuccessHandler(function() {
          hideLoader();
          alert('บันทึกผลตรวจรับรองและส่งอีเมลแจ้งเตือนหัวหน้าห้องปฏิบัติการสำเร็จ');
          document.getElementById('pipette-form').reset();
          document.getElementById('pipette-rows-container').innerHTML = '';
          _afterCalSave('pipette');
        }).withFailureHandler(function() {
          hideLoader();
          alert('บันทึกข้อมูลสำเร็จ แต่เกิดข้อผิดพลาดในการส่งอีเมลแจ้งเตือน');
        }).sendApprovalEmail(payload);
      } else {
        hideLoader();
        alert('เกิดข้อผิดพลาดจากเซิร์ฟเวอร์: ' + res.message);
      }
    }).withFailureHandler(function(err) {
      hideLoader(); alert('เกิดข้อผิดพลาดในการเชื่อมต่อ: ' + err);
    }).savePipetteWithPdf(payload); 
  }

  function checkPipetteOverallStatus() {
    const container = document.getElementById('pipette-rows-container');
    if (!container) return;
    const rows = container.querySelectorAll('.pi-row-item');
    const overallStatusEl = document.getElementById('pi-overall-status');
    if (!overallStatusEl) return;
    
    if (rows.length === 0) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-slate-200 text-slate-700 font-bold text-sm shadow-sm print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "รอข้อมูลจุดทดสอบ";
      return;
    }
    
    let totalPass = true;
    let hasWaiting = false;
    
    rows.forEach(row => {
      const id = row.id.replace('pi-row-', '');
      const badge = document.getElementById(`pi-status-badge-${id}`);
      const status = badge.getAttribute('data-status');
      if (status === 'FAIL') totalPass = false;
      if (status === 'WAITING') hasWaiting = true;
    });
    
    if (hasWaiting) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-amber-100 text-amber-700 font-bold text-sm border border-amber-200 animate-pulse print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "กำลังรอข้อมูลให้ครบถ้วน...";
      return;
    }
    
    if (totalPass) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-green-600 text-white font-bold text-sm shadow border border-green-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ยอมรับและนำไปใช้งานได้";
    } else {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-red-600 text-white font-bold text-sm shadow border border-red-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ปฏิเสธและไม่นำไปใช้งาน";
    }
  }

  function submitPipetteForm() {
    const form = document.getElementById('pipette-form');
    if(!form.checkValidity()) { form.reportValidity(); return; }
    
    const container = document.getElementById('pipette-rows-container');
    const rows = container.querySelectorAll('.pi-row-item');
    if(rows.length === 0) {
      alert('กรุณาเพิ่มรายการจุดทดสอบในตารางอย่างน้อย 1 รายการครับ');
      return;
    }
    
    const testVolArr = []; const meanVolArr = []; const uncertaintyArr = [];
    const meanUncertaintyArr = []; const controlRangeArr = []; const resultArr = [];
    let incomplete = false;
    
    rows.forEach(row => {
      const id = row.id.replace('pi-row-', '');
      const tv = document.getElementById(`pi-testVol-${id}`).value;
      const mv = document.getElementById(`pi-meanVol-${id}`).value;
      const uc = document.getElementById(`pi-uncertainty-${id}`).value;
      const ran = document.getElementById(`pi-range-text-${id}`).innerText;
      const cMin = document.getElementById(`pi-ctrlMin-${id}`).value;
      const cMax = document.getElementById(`pi-ctrlMax-${id}`).value;
      const res = document.getElementById(`pi-status-badge-${id}`).innerText;
      
      if(!tv || !mv || !uc || !cMin || !cMax) incomplete = true;
      
      testVolArr.push(tv);
      meanVolArr.push(mv);
      uncertaintyArr.push(uc);
      meanUncertaintyArr.push(ran);
      controlRangeArr.push(`${cMin} - ${cMax}`);
      resultArr.push(res);
    });
    
    if(incomplete) {
      alert('กรุณากรอกข้อมูลตัวเลขในตารางจุดทดสอบให้ครบถ้วนทุกช่องครับ');
      return;
    }
    
    const fileInput = document.getElementById('pi-pdfFile');
    const overallStatus = document.getElementById('pi-overall-status').innerText;
    
    const payload = {
      calDept: document.getElementById('pi-cal-dept') ? document.getElementById('pi-cal-dept').value : "",
      eqId: document.getElementById('pi-eqSelect').value,
      serialNo: document.getElementById('pi-serialNo').value,
      assetBrand: document.getElementById('pi-assetBrand').value,
      calDate: document.getElementById('pi-calDate').value,
      certNo: document.getElementById('pi-certNo').value,
      capacity: document.getElementById('pi-capacity').value,
      testVol: testVolArr.join('\n'),
      meanVol: meanVolArr.join('\n'),
      uncertainty: uncertaintyArr.join('\n'),
      meanUncertainty: meanUncertaintyArr.join('\n'),
      controlRange: controlRangeArr.join('\n'),
      result: resultArr.join('\n') + '\n\nสรุป: ' + overallStatus,
      addedBy: currentUser ? _getDisplayName() : 'System'
    };
    _calEditInject('pipette', payload); // ✅ โหมดแก้ไข: ส่งเลขแถวเดิมไปเขียนทับ
    
    if (fileInput && fileInput.files.length > 0) {
      showLoader();
      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        payload.pdfBase64 = e.target.result.split(',')[1];
        payload.pdfName = file.name;
        executeSubmitPipette(payload);
      };
      reader.readAsDataURL(file);
    } else {
      if(confirm("ยืนยันบันทึกผลการตรวจสอบ Pipette โดยไม่มีไฟล์เอกสารแนบใช่หรือไม่?")) {
        showLoader();
        executeSubmitPipette(payload);
      }
    }
  }
  
  function executeSubmitPipette(payload) {
    google.script.run.withSuccessHandler(function(res) {
      if(res.success) {
        google.script.run.withSuccessHandler(function() {
          hideLoader();
          alert('บันทึกผลตรวจรับรองและส่งอีเมลแจ้งเตือนหัวหน้าห้องปฏิบัติการสำเร็จ');
          document.getElementById('pipette-form').reset();
          document.getElementById('pipette-rows-container').innerHTML = '';
          _afterCalSave('pipette');
        }).withFailureHandler(function() {
          hideLoader();
          alert('บันทึกข้อมูลสำเร็จ แต่เกิดข้อผิดพลาดในการส่งอีเมลแจ้งเตือน');
        }).sendApprovalEmail(payload);
      } else {
        hideLoader();
        alert('เกิดข้อผิดพลาดจากเซิร์ฟเวอร์: ' + res.message);
      }
    }).withFailureHandler(function(err) {
      hideLoader(); alert('เกิดข้อผิดพลาดในการเชื่อมต่อ: ' + err);
    }).savePipetteWithPdf(payload); 
  }

  // --- TEMPERATURE THERMOMETER MODULE ---
  function loadEquipmentsForThermo() {
    const dept = currentUser ? currentUser.dept : "";
    const role = currentUser ? currentUser.role : "";

    google.script.run.withSuccessHandler(function(data) {
      cachedEquipments = data; 
      cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
      const select = document.getElementById('th-eqSelect');
      if (!select) return;
      select.innerHTML = '<option value="">-- เลือกรายการเครื่องมือ --</option>';
      // กรองเฉพาะประเภทที่เกี่ยวข้อง
      // ใช้ _getParamType เหมือนกับระบบอื่น
      var filtered = data.filter(function(item) {
        return _getParamType(item['ประเภทเครื่องมือ'] || '') === 'thermo';
      });
      filtered.forEach(function(item) {
        select.innerHTML += '<option value="' + item['ID'] + '">' + (item['รหัสครุภัณฑ์']||'-') + ' | ' + (item['ชื่อเครื่องมือ']||'-') + '</option>';
      });
      
      document.getElementById('thermo-rows-container').innerHTML = '';
      thermoRowCount = 0;
      addThermoRow();
      addThermoRow();
    }).getEquipments(dept, role); // 🌟 ส่งพารามิเตอร์คัดกรองห้องปฏิบัติการ
  }

  function autoFillThermoInfo() {
    const eq = cachedEquipments.find(item => item['ID'] === document.getElementById('th-eqSelect').value);
    if(eq) {
      document.getElementById('th-serialNo').value = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || "ไม่ระบุ";
      document.getElementById('th-assetBrand').value = (eq['รหัสครุภัณฑ์'] || "") + " / " + (eq['ชื่อเครื่องมือ'] || "");
    }
    if (currentUser) {
      document.getElementById('th-print-recorder-name').innerText = (_getDisplayName() || currentUser.username);
    }
    document.getElementById('th-print-recorder-date').innerText = new Date().toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
  }

  function addThermoRow() {
    thermoRowCount++;
    const id = thermoRowCount;
    const container = document.getElementById('thermo-rows-container');
    if (!container) return;
    
    const rowHtml = `
      <tr id="th-row-${id}" class="hover:bg-slate-50 dark:hover:bg-slate-800/40 transition th-row-item">
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="th-testPoint-${id}" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center font-bold print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="th-stdTemp-${id}" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="th-reading-${id}" oninput="calcThermoRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="th-uncertainty-${id}" oninput="calcThermoRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center bg-slate-50 dark:bg-slate-900/20 font-bold text-slate-700 dark:text-slate-300 print:bg-transparent">
          <span id="th-range-text-${id}">-</span>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center">
          <div class="flex items-center justify-center gap-1">
            <input type="number" step="any" id="th-ctrlMin-${id}" oninput="calcThermoRow(${id})" placeholder="Min" required class="w-14 p-1 border rounded dark:bg-slate-800 text-center text-rose-600 font-bold print:border-none print:bg-transparent print:w-auto">
            <span class="text-slate-400">ถึง</span>
            <input type="number" step="any" id="th-ctrlMax-${id}" oninput="calcThermoRow(${id})" placeholder="Max" required class="w-14 p-1 border rounded dark:bg-slate-800 text-center text-rose-600 font-bold print:border-none print:bg-transparent print:w-auto">
          </div>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center font-bold">
          <span id="th-status-badge-${id}" data-status="WAITING" class="px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black">รอคำนวณ</span>
        </td>
        <td class="p-2 text-center print:hidden">
          <button type="button" onclick="removeThermoRow(${id})" class="text-red-500 hover:text-red-700 transition-colors">
            <span class="material-symbols-outlined text-lg">delete</span>
          </button>
        </td>
      </tr>`;
    container.insertAdjacentHTML('beforeend', rowHtml);
    checkThermoOverallStatus();
  }

  function removeThermoRow(id) {
    const row = document.getElementById(`th-row-${id}`);
    if(row) { row.remove(); checkThermoOverallStatus(); }
  }

  function calcThermoRow(id) {
    const reading = parseFloat(document.getElementById(`th-reading-${id}`).value);
    const uncertainty = parseFloat(document.getElementById(`th-uncertainty-${id}`).value);
    const ctrlMin = parseFloat(document.getElementById(`th-ctrlMin-${id}`).value);
    const ctrlMax = parseFloat(document.getElementById(`th-ctrlMax-${id}`).value);
    
    const rangeSpan = document.getElementById(`th-range-text-${id}`);
    const badge = document.getElementById(`th-status-badge-${id}`);
    
    if (isNaN(reading) || isNaN(uncertainty)) {
      rangeSpan.innerText = "-";
      badge.className = "px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอคำนวณ";
      badge.setAttribute('data-status', 'WAITING');
      checkThermoOverallStatus();
      return;
    }
    
    const minCalculated = parseFloat((reading - uncertainty).toFixed(2));
    const maxCalculated = parseFloat((reading + uncertainty).toFixed(2));
    rangeSpan.innerText = `(${minCalculated.toFixed(2)}) - (${maxCalculated.toFixed(2)})`;
    
    if (isNaN(ctrlMin) || isNaN(ctrlMax)) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-amber-50 text-amber-600 border border-amber-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอช่วงควบคุม";
      badge.setAttribute('data-status', 'WAITING');
      checkThermoOverallStatus();
      return;
    }
    
    if (minCalculated >= ctrlMin && maxCalculated <= ctrlMax) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-green-100 text-green-700 border border-green-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ยอมรับ";
      badge.setAttribute('data-status', 'PASS');
    } else {
      badge.className = "px-2 py-1 rounded text-[11px] bg-red-100 text-red-700 border border-red-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ปฏิเสธ";
      badge.setAttribute('data-status', 'FAIL');
    }
    checkThermoOverallStatus();
  }

  function checkThermoOverallStatus() {
    const container = document.getElementById('thermo-rows-container');
    if (!container) return;
    const rows = container.querySelectorAll('.th-row-item');
    const overallStatusEl = document.getElementById('th-overall-status');
    if (!overallStatusEl) return;
    
    if (rows.length === 0) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-slate-200 text-slate-700 font-bold text-sm shadow-sm print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "รอข้อมูลจุดทดสอบ";
      return;
    }
    
    let totalPass = true; let hasWaiting = false;
    rows.forEach(row => {
      const id = row.id.replace('th-row-', '');
      const badge = document.getElementById(`th-status-badge-${id}`);
      const status = badge.getAttribute('data-status');
      if (status === 'FAIL') totalPass = false;
      if (status === 'WAITING') hasWaiting = true;
    });
    
    if (hasWaiting) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-amber-100 text-amber-700 font-bold text-sm border border-amber-200 animate-pulse print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "กำลังรอข้อมูลให้ครบถ้วน...";
      return;
    }
    
    if (totalPass) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-green-600 text-white font-bold text-sm shadow border border-green-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ยอมรับและนำไปใช้งานได้";
    } else {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-red-600 text-white font-bold text-sm shadow border border-red-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ปฏิเสธและไม่นำไปใช้งาน";
    }
  }

  function submitThermoForm() {
    const form = document.getElementById('thermo-form');
    if(!form.checkValidity()) { form.reportValidity(); return; }
    
    const container = document.getElementById('thermo-rows-container');
    const rows = container.querySelectorAll('.th-row-item');
    if(rows.length === 0) {
      alert('กรุณาเพิ่มรายการจุดทดสอบในตารางอย่างน้อย 1 รายการครับ');
      return;
    }
    
    let thermoType = "";
    document.getElementsByName('thermoType').forEach(r => { if(r.checked) thermoType = r.value; });

    const testPointArr = []; const stdTempArr = []; const readingArr = [];
    const uncertaintyArr = []; const readingUncertaintyArr = []; const controlRangeArr = []; const resultArr = [];
    let incomplete = false;
    
    rows.forEach(row => {
      const id = row.id.replace('th-row-', '');
      const tp = document.getElementById(`th-testPoint-${id}`).value;
      const st = document.getElementById(`th-stdTemp-${id}`).value;
      const rd = document.getElementById(`th-reading-${id}`).value;
      const uc = document.getElementById(`th-uncertainty-${id}`).value;
      const ran = document.getElementById(`th-range-text-${id}`).innerText;
      const cMin = document.getElementById(`th-ctrlMin-${id}`).value;
      const cMax = document.getElementById(`th-ctrlMax-${id}`).value;
      const res = document.getElementById(`th-status-badge-${id}`).innerText;
      
      if(!tp || !st || !rd || !uc || !cMin || !cMax) incomplete = true;
      
      testPointArr.push(tp);
      stdTempArr.push(st);
      readingArr.push(rd);
      uncertaintyArr.push(uc);
      readingUncertaintyArr.push(ran);
      controlRangeArr.push(`(${cMin}) - (${cMax})`);
      resultArr.push(res);
    });
    
    if(incomplete) {
      alert('กรุณากรอกข้อมูลตัวเลขในตารางให้ครบถ้วนทุกช่องครับ');
      return;
    }
    
    const fileInput = document.getElementById('th-pdfFile');
    const overallStatus = document.getElementById('th-overall-status').innerText;
    
    const payload = {
      calDept: document.getElementById('th-cal-dept') ? document.getElementById('th-cal-dept').value : "",
      thermoType: thermoType,
      eqId: document.getElementById('th-eqSelect').value,
      serialNo: document.getElementById('th-serialNo').value,
      assetBrand: document.getElementById('th-assetBrand').value,
      calDate: document.getElementById('th-calDate').value,
      certNo: document.getElementById('th-certNo').value,
      testPoint: testPointArr.join('\n'),
      stdTemp: stdTempArr.join('\n'),
      reading: readingArr.join('\n'),
      uncertainty: uncertaintyArr.join('\n'),
      readingUncertainty: readingUncertaintyArr.join('\n'),
      controlRange: controlRangeArr.join('\n'),
      result: resultArr.join('\n') + '\n\nสรุป: ' + overallStatus,
      addedBy: currentUser ? _getDisplayName() : 'System'
    };
    _calEditInject('thermo', payload); // ✅ โหมดแก้ไข: ส่งเลขแถวเดิมไปเขียนทับ
    
    if (fileInput && fileInput.files.length > 0) {
      showLoader();
      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        payload.pdfBase64 = e.target.result.split(',')[1];
        payload.pdfName = file.name;
        executeSubmitThermo(payload);
      };
      reader.readAsDataURL(file);
    } else {
      if(confirm("ยืนยันบันทึกผลการตรวจสอบโดยไม่มีไฟล์เอกสารแนบใช่หรือไม่?")) {
        showLoader();
        executeSubmitThermo(payload);
      }
    }
  }

  function executeSubmitThermo(payload) {
    google.script.run.withSuccessHandler(function(res) {
      if(res.success) {
        google.script.run.withSuccessHandler(function() {
          hideLoader();
          alert('บันทึกผลและส่งอีเมลแจ้งเตือนสำเร็จ');
          document.getElementById('thermo-form').reset();
          document.getElementById('thermo-rows-container').innerHTML = '';
          _afterCalSave('thermo');
        }).withFailureHandler(function() {
          hideLoader(); alert('บันทึกสำเร็จ แต่ระบบส่งอีเมลล้มเหลว');
        }).sendApprovalEmail(payload); 
      } else {
        hideLoader(); alert('Error: ' + res.message);
      }
    }).withFailureHandler(function(err) {
      hideLoader(); alert('เซิร์ฟเวอร์ขัดข้อง: ' + err);
    }).saveThermometerWithPdf(payload); 
  }

  // --- CENTRIFUGE MODULE ---
  function loadEquipmentsForCentrifuge() {
    const dept = currentUser ? currentUser.dept : "";
    const role = currentUser ? currentUser.role : "";

    google.script.run.withSuccessHandler(function(data) {
      cachedEquipments = data; 
      cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
      const select = document.getElementById('ce-eqSelect');
      if (!select) return;
      select.innerHTML = '<option value="">-- เลือกรายการเครื่องมือ --</option>';
      // กรองเฉพาะประเภทที่เกี่ยวข้อง
      // ใช้ _getParamType เหมือนกับระบบอื่น
      var filtered = data.filter(function(item) {
        return _getParamType(item['ประเภทเครื่องมือ'] || '') === 'centri';
      });
      filtered.forEach(function(item) {
        select.innerHTML += '<option value="' + item['ID'] + '">' + (item['รหัสครุภัณฑ์']||'-') + ' | ' + (item['ชื่อเครื่องมือ']||'-') + '</option>';
      });
      
      document.getElementById('centrifuge-rows-container').innerHTML = '';
      centrifugeRowCount = 0;
      addCentrifugeRow('Speed'); 
      addCentrifugeRow('Time');  
    }).getEquipments(dept, role); // 🌟 ส่งพารามิเตอร์คัดกรองห้องปฏิบัติการ
  }

  function autoFillCentrifugeInfo() {
    const eq = cachedEquipments.find(item => item['ID'] === document.getElementById('ce-eqSelect').value);
    if(eq) {
      document.getElementById('ce-serialNo').value = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || "ไม่ระบุ";
      document.getElementById('ce-assetBrand').value = (eq['รหัสครุภัณฑ์'] || "") + " / " + (eq['ชื่อเครื่องมือ'] || "");
    }
    if (currentUser) {
      document.getElementById('ce-print-recorder-name').innerText = (_getDisplayName() || currentUser.username);
    }
    document.getElementById('ce-print-recorder-date').innerText = new Date().toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
  }

  function addCentrifugeRow(defaultFunc = "") {
    centrifugeRowCount++;
    const id = centrifugeRowCount;
    const container = document.getElementById('centrifuge-rows-container');
    if (!container) return;
    
    const rowHtml = `
      <tr id="ce-row-${id}" class="hover:bg-slate-50 dark:hover:bg-slate-800/40 transition ce-row-item">
        <td class="p-2 border-r dark:border-slate-700">
          <input type="text" id="ce-func-${id}" value="${defaultFunc}" list="ce-functions" placeholder="เช่น Speed" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center font-bold print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ce-setting-${id}" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ce-std-${id}" oninput="calcCentrifugeRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ce-unc-${id}" oninput="calcCentrifugeRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center bg-slate-50 dark:bg-slate-900/20 font-bold text-slate-700 dark:text-slate-300 print:bg-transparent">
          <span id="ce-range-text-${id}">-</span>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center">
          <div class="flex items-center justify-center gap-1">
            <input type="number" step="any" id="ce-ctrlMin-${id}" oninput="calcCentrifugeRow(${id})" placeholder="Min" required class="w-16 p-1 border rounded dark:bg-slate-800 text-center text-purple-600 font-bold print:border-none print:bg-transparent print:w-auto">
            <span class="text-slate-400">-</span>
            <input type="number" step="any" id="ce-ctrlMax-${id}" oninput="calcCentrifugeRow(${id})" placeholder="Max" required class="w-16 p-1 border rounded dark:bg-slate-800 text-center text-purple-600 font-bold print:border-none print:bg-transparent print:w-auto">
          </div>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center font-bold">
          <span id="ce-status-badge-${id}" data-status="WAITING" class="px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black">รอคำนวณ</span>
        </td>
        <td class="p-2 text-center print:hidden">
          <button type="button" onclick="removeCentrifugeRow(${id})" class="text-red-500 hover:text-red-700 transition-colors">
            <span class="material-symbols-outlined text-lg">delete</span>
          </button>
        </td>
      </tr>`;
    container.insertAdjacentHTML('beforeend', rowHtml);
    checkCentrifugeOverallStatus();
  }

  function removeCentrifugeRow(id) {
    const row = document.getElementById(`ce-row-${id}`);
    if(row) { row.remove(); checkCentrifugeOverallStatus(); }
  }

  function calcCentrifugeRow(id) {
    const stdRead = parseFloat(document.getElementById(`ce-std-${id}`).value);
    const unc = parseFloat(document.getElementById(`ce-unc-${id}`).value);
    const ctrlMin = parseFloat(document.getElementById(`ce-ctrlMin-${id}`).value);
    const ctrlMax = parseFloat(document.getElementById(`ce-ctrlMax-${id}`).value);
    
    const rangeSpan = document.getElementById(`ce-range-text-${id}`);
    const badge = document.getElementById(`ce-status-badge-${id}`);
    
    if (isNaN(stdRead) || isNaN(unc)) {
      rangeSpan.innerText = "-";
      badge.className = "px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอคำนวณ";
      badge.setAttribute('data-status', 'WAITING');
      checkCentrifugeOverallStatus();
      return;
    }
    
    const minCalculated = parseFloat((stdRead - unc).toFixed(2));
    const maxCalculated = parseFloat((stdRead + unc).toFixed(2));
    rangeSpan.innerText = `${minCalculated.toFixed(2)} - ${maxCalculated.toFixed(2)}`;
    
    if (isNaN(ctrlMin) || isNaN(ctrlMax)) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-amber-50 text-amber-600 border border-amber-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอช่วงควบคุม";
      badge.setAttribute('data-status', 'WAITING');
      checkCentrifugeOverallStatus();
      return;
    }
    
    if (minCalculated >= ctrlMin && maxCalculated <= ctrlMax) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-green-100 text-green-700 border border-green-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ยอมรับ";
      badge.setAttribute('data-status', 'PASS');
    } else {
      badge.className = "px-2 py-1 rounded text-[11px] bg-red-100 text-red-700 border border-red-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ปฏิเสธ";
      badge.setAttribute('data-status', 'FAIL');
    }
    checkCentrifugeOverallStatus();
  }

  function checkCentrifugeOverallStatus() {
    const container = document.getElementById('centrifuge-rows-container');
    if (!container) return;
    const rows = container.querySelectorAll('.ce-row-item');
    const overallStatusEl = document.getElementById('ce-overall-status');
    if (!overallStatusEl) return;
    
    if (rows.length === 0) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-slate-200 text-slate-700 font-bold text-sm shadow-sm print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "รอข้อมูลจุดทดสอบ";
      return;
    }
    
    let totalPass = true; let hasWaiting = false;
    rows.forEach(row => {
      const id = row.id.replace('ce-row-', '');
      const badge = document.getElementById(`ce-status-badge-${id}`);
      const status = badge.getAttribute('data-status');
      if (status === 'FAIL') totalPass = false;
      if (status === 'WAITING') hasWaiting = true;
    });
    
    if (hasWaiting) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-amber-100 text-amber-700 font-bold text-sm border border-amber-200 animate-pulse print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "กำลังรอข้อมูลให้ครบถ้วน...";
      return;
    }
    
    if (totalPass) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-green-600 text-white font-bold text-sm shadow border border-green-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ยอมรับและนำไปใช้งานได้";
    } else {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-red-600 text-white font-bold text-sm shadow border border-red-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ปฏิเสธและไม่นำไปใช้งาน";
    }
  }

  function submitCentrifugeForm() {
    const form = document.getElementById('centrifuge-form');
    if(!form.checkValidity()) { form.reportValidity(); return; }
    
    const container = document.getElementById('centrifuge-rows-container');
    const rows = container.querySelectorAll('.ce-row-item');
    if(rows.length === 0) {
      alert('กรุณาเพิ่มรายการจุดทดสอบอย่างน้อย 1 รายการครับ');
      return;
    }

    const funcArr = []; const settingArr = []; const stdReadArr = [];
    const uncArr = []; const calcRangeArr = []; const ctrlRangeArr = []; const resultArr = [];
    let incomplete = false;
    
    rows.forEach(row => {
      const id = row.id.replace('ce-row-', '');
      const fn = document.getElementById(`ce-func-${id}`).value;
      const st = document.getElementById(`ce-setting-${id}`).value;
      const sr = document.getElementById(`ce-std-${id}`).value;
      const uc = document.getElementById(`ce-unc-${id}`).value;
      const ran = document.getElementById(`ce-range-text-${id}`).innerText;
      const cMin = document.getElementById(`ce-ctrlMin-${id}`).value;
      const cMax = document.getElementById(`ce-ctrlMax-${id}`).value;
      const res = document.getElementById(`ce-status-badge-${id}`).innerText;
      
      if(!fn || !st || !sr || !uc || !cMin || !cMax) incomplete = true;
      
      funcArr.push(fn);
      settingArr.push(st);
      stdReadArr.push(sr);
      uncArr.push(uc);
      calcRangeArr.push(ran);
      ctrlRangeArr.push(`${cMin} - ${cMax}`);
      resultArr.push(res);
    });
    
    if(incomplete) {
      alert('กรุณากรอกข้อมูลในตารางให้ครบถ้วนทุกช่องครับ');
      return;
    }
    
    const fileInput = document.getElementById('ce-pdfFile');
    const overallStatus = document.getElementById('ce-overall-status').innerText;
    
    const payload = {
      calDept: document.getElementById('ce-cal-dept') ? document.getElementById('ce-cal-dept').value : "",
      eqId: document.getElementById('ce-eqSelect').value,
      serialNo: document.getElementById('ce-serialNo').value,
      assetBrand: document.getElementById('ce-assetBrand').value,
      calDate: document.getElementById('ce-calDate').value,
      certNo: document.getElementById('ce-certNo').value,
      ceFunction: funcArr.join('\n'),
      setting: settingArr.join('\n'),
      stdReading: stdReadArr.join('\n'),
      uncertainty: uncArr.join('\n'),
      calcRange: calcRangeArr.join('\n'),
      controlRange: ctrlRangeArr.join('\n'),
      result: resultArr.join('\n') + '\n\nสรุป: ' + overallStatus,
      addedBy: currentUser ? _getDisplayName() : 'System'
    };
    _calEditInject('centrifuge', payload); // ✅ โหมดแก้ไข: ส่งเลขแถวเดิมไปเขียนทับ
    
    if (fileInput && fileInput.files.length > 0) {
      showLoader();
      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        payload.pdfBase64 = e.target.result.split(',')[1];
        payload.pdfName = file.name;
        executeSubmitCentrifuge(payload);
      };
      reader.readAsDataURL(file);
    } else {
      if(confirm("ยืนยันบันทึกผลการตรวจสอบโดยไม่มีไฟล์เอกสารแนบใช่หรือไม่?")) {
        showLoader();
        executeSubmitCentrifuge(payload);
      }
    }
  }

  function executeSubmitCentrifuge(payload) {
    google.script.run.withSuccessHandler(function(res) {
      if(res.success) {
        google.script.run.withSuccessHandler(function() {
          hideLoader();
          alert('บันทึกผลและส่งอีเมลแจ้งเตือนสำเร็จ');
          document.getElementById('centrifuge-form').reset();
          document.getElementById('centrifuge-rows-container').innerHTML = '';
          _afterCalSave('centrifuge');
        }).withFailureHandler(function() {
          hideLoader(); alert('บันทึกสำเร็จ แต่ระบบส่งอีเมลล้มเหลว');
        }).sendApprovalEmail(payload); 
      } else {
        hideLoader(); alert('Error: ' + res.message);
      }
    }).withFailureHandler(function(err) {
      hideLoader(); alert('เซิร์ฟเวอร์ขัดข้อง: ' + err);
    }).saveCentrifugeWithPdf(payload); 
  }

  // --- ANALYTICAL BALANCE MODULE ---
  function loadEquipmentsForBalance() {
    const dept = currentUser ? currentUser.dept : "";
    const role = currentUser ? currentUser.role : "";

    google.script.run.withSuccessHandler(function(data) {
      cachedEquipments = data; 
      cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
      const select = document.getElementById('ba-eqSelect');
      if (!select) return;
      select.innerHTML = '<option value="">-- เลือกรายการเครื่องมือ --</option>';
      // กรองเฉพาะประเภทที่เกี่ยวข้อง
      // ใช้ _getParamType เหมือนกับระบบอื่น
      var filtered = data.filter(function(item) {
        return _getParamType(item['ประเภทเครื่องมือ'] || '') === 'balance';
      });
      filtered.forEach(function(item) {
        select.innerHTML += '<option value="' + item['ID'] + '">' + (item['รหัสครุภัณฑ์']||'-') + ' | ' + (item['ชื่อเครื่องมือ']||'-') + '</option>';
      });
      
      document.getElementById('balance-rows-container').innerHTML = '';
      balanceRowCount = 0;
      addBalanceRow();
      addBalanceRow();
      addBalanceRow();
    }).getEquipments(dept, role); // 🌟 ส่งพารามิเตอร์คัดกรองห้องปฏิบัติการ
  }

  function autoFillBalanceInfo() {
    const eq = cachedEquipments.find(item => item['ID'] === document.getElementById('ba-eqSelect').value);
    if(eq) {
      document.getElementById('ba-serialNo').value = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || "ไม่ระบุ";
      document.getElementById('ba-assetBrand').value = (eq['รหัสครุภัณฑ์'] || "") + " / " + (eq['ชื่อเครื่องมือ'] || "");
    }
    if (currentUser) {
      document.getElementById('ba-print-recorder-name').innerText = (_getDisplayName() || currentUser.username);
    }
    document.getElementById('ba-print-recorder-date').innerText = new Date().toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
  }

  function addBalanceRow() {
    balanceRowCount++;
    const id = balanceRowCount;
    const container = document.getElementById('balance-rows-container');
    if (!container) return;
    
    const rowHtml = `
      <tr id="ba-row-${id}" class="hover:bg-slate-50 dark:hover:bg-slate-800/40 transition ba-row-item">
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ba-nom-${id}" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center font-bold print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ba-disp-${id}" oninput="calcBalanceRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ba-unc-${id}" oninput="calcBalanceRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center bg-slate-50 dark:bg-slate-900/20 font-bold text-slate-700 dark:text-slate-300 print:bg-transparent">
          <span id="ba-range-text-${id}">-</span>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center">
          <div class="flex items-center justify-center gap-1">
            <input type="number" step="any" id="ba-ctrlMin-${id}" oninput="calcBalanceRow(${id})" placeholder="Min" required class="w-16 p-1 border rounded dark:bg-slate-800 text-center text-amber-600 font-bold print:border-none print:bg-transparent print:w-auto">
            <span class="text-slate-400">ถึง</span>
            <input type="number" step="any" id="ba-ctrlMax-${id}" oninput="calcBalanceRow(${id})" placeholder="Max" required class="w-16 p-1 border rounded dark:bg-slate-800 text-center text-amber-600 font-bold print:border-none print:bg-transparent print:w-auto">
          </div>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center font-bold">
          <span id="ba-status-badge-${id}" data-status="WAITING" class="px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black">รอคำนวณ</span>
        </td>
        <td class="p-2 text-center print:hidden">
          <button type="button" onclick="removeBalanceRow(${id})" class="text-red-500 hover:text-red-700 transition-colors">
            <span class="material-symbols-outlined text-lg">delete</span>
          </button>
        </td>
      </tr>`;
    container.insertAdjacentHTML('beforeend', rowHtml);
    checkBalanceOverallStatus();
  }

  function removeBalanceRow(id) {
    const row = document.getElementById(`ba-row-${id}`);
    if(row) { row.remove(); checkBalanceOverallStatus(); }
  }

  function calcBalanceRow(id) {
    const dispValue = parseFloat(document.getElementById(`ba-disp-${id}`).value);
    const uncertainty = parseFloat(document.getElementById(`ba-unc-${id}`).value);
    const ctrlMin = parseFloat(document.getElementById(`ba-ctrlMin-${id}`).value);
    const ctrlMax = parseFloat(document.getElementById(`ba-ctrlMax-${id}`).value);
    
    const rangeSpan = document.getElementById(`ba-range-text-${id}`);
    const badge = document.getElementById(`ba-status-badge-${id}`);
    
    if (isNaN(dispValue) || isNaN(uncertainty)) {
      rangeSpan.innerText = "-";
      badge.className = "px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอคำนวณ";
      badge.setAttribute('data-status', 'WAITING');
      checkBalanceOverallStatus();
      return;
    }
    
    const minCalculated = parseFloat((dispValue - Math.abs(uncertainty)).toFixed(2));
    const maxCalculated = parseFloat((dispValue + Math.abs(uncertainty)).toFixed(2));
    rangeSpan.innerText = `(${minCalculated.toFixed(2)}) - (${maxCalculated.toFixed(2)})`;
    
    if (isNaN(ctrlMin) || isNaN(ctrlMax)) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-amber-50 text-amber-600 border border-amber-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอช่วงควบคุม";
      badge.setAttribute('data-status', 'WAITING');
      checkBalanceOverallStatus();
      return;
    }
    
    if (minCalculated >= ctrlMin && maxCalculated <= ctrlMax) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-green-100 text-green-700 border border-green-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ยอมรับ";
      badge.setAttribute('data-status', 'PASS');
    } else {
      badge.className = "px-2 py-1 rounded text-[11px] bg-red-100 text-red-700 border border-red-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ปฏิเสธ";
      badge.setAttribute('data-status', 'FAIL');
    }
    checkBalanceOverallStatus();
  }

  function checkBalanceOverallStatus() {
    const container = document.getElementById('balance-rows-container');
    if (!container) return;
    const rows = container.querySelectorAll('.ba-row-item');
    const overallStatusEl = document.getElementById('ba-overall-status');
    if (!overallStatusEl) return;
    
    if (rows.length === 0) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-slate-200 text-slate-700 font-bold text-sm shadow-sm print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "รอข้อมูลจุดทดสอบ";
      return;
    }
    
    let totalPass = true; let hasWaiting = false;
    rows.forEach(row => {
      const id = row.id.replace('ba-row-', '');
      const badge = document.getElementById(`ba-status-badge-${id}`);
      const status = badge.getAttribute('data-status');
      if (status === 'FAIL') totalPass = false;
      if (status === 'WAITING') hasWaiting = true;
    });
    
    if (hasWaiting) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-amber-100 text-amber-700 font-bold text-sm border border-amber-200 animate-pulse print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "กำลังรอข้อมูลให้ครบถ้วน...";
      return;
    }
    
    if (totalPass) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-green-600 text-white font-bold text-sm shadow border border-green-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ยอมรับและนำไปใช้งานได้";
    } else {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-red-600 text-white font-bold text-sm shadow border border-red-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ปฏิเสธและไม่นำไปใช้งาน";
    }
  }

  function submitBalanceForm() {
    const form = document.getElementById('balance-form');
    if(!form.checkValidity()) { form.reportValidity(); return; }
    
    const container = document.getElementById('balance-rows-container');
    const rows = container.querySelectorAll('.ba-row-item');
    if(rows.length === 0) {
      alert('กรุณาเพิ่มรายการจุดทดสอบอย่างน้อย 1 รายการครับ');
      return;
    }

    const nomArr = []; const dispArr = []; const uncArr = [];
    const calcRangeArr = []; const ctrlRangeArr = []; const resultArr = [];
    let incomplete = false;
    
    rows.forEach(row => {
      const id = row.id.replace('ba-row-', '');
      const nm = document.getElementById(`ba-nom-${id}`).value;
      const dp = document.getElementById(`ba-disp-${id}`).value;
      const uc = document.getElementById(`ba-unc-${id}`).value;
      const ran = document.getElementById(`ba-range-text-${id}`).innerText;
      const cMin = document.getElementById(`ba-ctrlMin-${id}`).value;
      const cMax = document.getElementById(`ba-ctrlMax-${id}`).value;
      const res = document.getElementById(`ba-status-badge-${id}`).innerText;
      
      if(!nm || !dp || !uc || !cMin || !cMax) incomplete = true;
      
      nomArr.push(nm);
      dispArr.push(dp);
      uncArr.push(uc);
      calcRangeArr.push(ran);
      ctrlRangeArr.push(`(${cMin}) - (${cMax})`);
      resultArr.push(res);
    });
    
    if(incomplete) {
      alert('กรุณากรอกข้อมูลตัวเลขในตารางให้ครบถ้วนทุกช่องครับ');
      return;
    }
    
    const fileInput = document.getElementById('ba-pdfFile');
    const overallStatus = document.getElementById('ba-overall-status').innerText;
    
    const payload = {
      calDept: document.getElementById('ba-cal-dept') ? document.getElementById('ba-cal-dept').value : "",
      eqId: document.getElementById('ba-eqSelect').value,
      serialNo: document.getElementById('ba-serialNo').value,
      assetBrand: document.getElementById('ba-assetBrand').value,
      calDate: document.getElementById('ba-calDate').value,
      certNo: document.getElementById('ba-certNo').value,
      resolution: document.getElementById('ba-resolution')?.value || "", 
      nomValue: nomArr.join('\n'),
      dispValue: dispArr.join('\n'),
      uncertainty: uncArr.join('\n'),
      calcRange: calcRangeArr.join('\n'),
      controlRange: ctrlRangeArr.join('\n'),
      result: resultArr.join('\n') + '\n\nสรุป: ' + overallStatus,
      addedBy: currentUser ? _getDisplayName() : 'System'
    };
    _calEditInject('balance', payload); // ✅ โหมดแก้ไข: ส่งเลขแถวเดิมไปเขียนทับ
    
    if (fileInput && fileInput.files.length > 0) {
      showLoader();
      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        payload.pdfBase64 = e.target.result.split(',')[1];
        payload.pdfName = file.name;
        executeSubmitBalance(payload);
      };
      reader.readAsDataURL(file);
    } else {
      if(confirm("ยืนยันบันทึกผลการตรวจสอบโดยไม่มีไฟล์เอกสารแนบใช่หรือไม่?")) {
        showLoader();
        executeSubmitBalance(payload);
      }
    }
  }

  function executeSubmitBalance(payload) {
    google.script.run.withSuccessHandler(function(res) {
      if(res.success) {
        google.script.run.withSuccessHandler(function() {
          hideLoader();
          alert('บันทึกผลและส่งอีเมลแจ้งเตือนสำเร็จ');
          document.getElementById('balance-form').reset();
          document.getElementById('balance-rows-container').innerHTML = '';
          _afterCalSave('balance');
        }).withFailureHandler(function() {
          hideLoader(); alert('บันทึกสำเร็จ แต่ระบบส่งอีเมลล้มเหลว');
        }).sendApprovalEmail(payload); 
      } else {
        hideLoader(); alert('Error: ' + res.message);
      }
    }).withFailureHandler(function(err) {
      hideLoader(); alert('เซิร์ฟเวอร์ขัดข้อง: ' + err);
    }).saveBalanceWithPdf(payload); 
  }

  // --- OVEN MODULE ---
  function loadEquipmentsForOven() {
    const dept = currentUser ? currentUser.dept : "";
    const role = currentUser ? currentUser.role : "";

    google.script.run.withSuccessHandler(function(data) {
      cachedEquipments = data; 
      cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
      const select = document.getElementById('ov-eqSelect');
      if (!select) return;
      select.innerHTML = '<option value="">-- เลือกรายการเครื่องมือ --</option>';
      // กรองเฉพาะประเภทที่เกี่ยวข้อง
      // ใช้ _getParamType เหมือนกับระบบอื่น
      var filtered = data.filter(function(item) {
        return _getParamType(item['ประเภทเครื่องมือ'] || '') === 'oven' || _getParamType(item['ชื่อเครื่องมือ'] || '') === 'oven';
      });
      filtered.forEach(function(item) {
        select.innerHTML += '<option value="' + item['ID'] + '">' + (item['รหัสครุภัณฑ์']||'-') + ' | ' + (item['ชื่อเครื่องมือ']||'-') + '</option>';
      });
      
      document.getElementById('oven-rows-container').innerHTML = '';
      ovenRowCount = 0;
      addOvenRow();
      addOvenRow();
      addOvenRow();
    }).getEquipments(dept, role); // 🌟 ส่งพารามิเตอร์คัดกรองห้องปฏิบัติการ
  }

  function autoFillOvenInfo() {
    const eq = cachedEquipments.find(item => item['ID'] === document.getElementById('ov-eqSelect').value);
    if (eq) {   // ✅ เลือกประเภท Autoclave / Oven / Incubator ให้อัตโนมัติจากประเภท/ชื่อเครื่อง
      const t = ((eq['ประเภทเครื่องมือ'] || '') + ' ' + (eq['ชื่อเครื่องมือ'] || '')).toLowerCase();
      const kind = (t.includes('autoclave') || t.includes('นึ่ง') || t.includes('steriliz')) ? 'Autoclave'
                 : (t.includes('incubat') || t.includes('ตู้บ่ม')) ? 'Incubator'
                 : (t.includes('oven') || t.includes('ตู้อบ') || t.includes('hot air')) ? 'Oven' : '';
      const r = kind && document.querySelector('input[name="ovenType"][value="' + kind + '"]');
      if (r && !r.checked) { r.checked = true; if (typeof window.toggleOvenType === 'function') window.toggleOvenType(); }
    }
    if(eq) {
      document.getElementById('ov-serialNo').value = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || "ไม่ระบุ";
      document.getElementById('ov-assetBrand').value = (eq['รหัสครุภัณฑ์'] || "") + " / " + (eq['ชื่อเครื่องมือ'] || "");
    }
    if (currentUser) {
      document.getElementById('ov-print-recorder-name').innerText = (_getDisplayName() || currentUser.username);
    }
    document.getElementById('ov-print-recorder-date').innerText = new Date().toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
  }

  function toggleOvenType() {
    const otherRadio = document.getElementById('ov-type-other');
    const otherInput = document.getElementById('ov-type-other-text');
    if(otherRadio && otherRadio.checked) {
      if (otherInput) {
        otherInput.classList.remove('hidden');
        otherInput.setAttribute('required', 'true');
      }
    } else {
      if (otherInput) {
        otherInput.classList.add('hidden');
        otherInput.removeAttribute('required');
        otherInput.value = '';
      }
    }
  }

  function addOvenRow() {
    ovenRowCount++;
    const id = ovenRowCount;
    const container = document.getElementById('oven-rows-container');
    if (!container) return;
    
    const rowHtml = `
      <tr id="ov-row-${id}" class="hover:bg-slate-50 dark:hover:bg-slate-800/40 transition ov-row-item">
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ov-read-${id}" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ov-std-${id}" oninput="calcOvenRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center font-bold print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ov-unc-${id}" oninput="calcOvenRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center bg-orange-50 dark:bg-orange-900/20 font-bold text-orange-800 dark:text-orange-300 print:bg-transparent">
          <span id="ov-range-text-${id}">-</span>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center">
          <div class="flex items-center justify-center gap-1">
            <input type="number" step="any" id="ov-ctrlMin-${id}" oninput="calcOvenRow(${id})" placeholder="Min" required class="w-16 p-1 border rounded dark:bg-slate-800 text-center text-orange-600 font-bold print:border-none print:bg-transparent print:w-auto">
            <span class="text-slate-400">ถึง</span>
            <input type="number" step="any" id="ov-ctrlMax-${id}" oninput="calcOvenRow(${id})" placeholder="Max" required class="w-16 p-1 border rounded dark:bg-slate-800 text-center text-orange-600 font-bold print:border-none print:bg-transparent print:w-auto">
          </div>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center font-bold">
          <span id="ov-status-badge-${id}" data-status="WAITING" class="px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black">รอคำนวณ</span>
        </td>
        <td class="p-2 text-center print:hidden">
          <button type="button" onclick="removeOvenRow(${id})" class="text-red-500 hover:text-red-700 transition-colors">
            <span class="material-symbols-outlined text-lg">delete</span>
          </button>
        </td>
      </tr>`;
    container.insertAdjacentHTML('beforeend', rowHtml);
    checkOvenOverallStatus();
  }

  function removeOvenRow(id) {
    const row = document.getElementById(`ov-row-${id}`);
    if(row) { row.remove(); checkOvenOverallStatus(); }
  }

  function calcOvenRow(id) {
    const stdValue = parseFloat(document.getElementById(`ov-std-${id}`).value);
    const uncertainty = parseFloat(document.getElementById(`ov-unc-${id}`).value);
    const ctrlMin = parseFloat(document.getElementById(`ov-ctrlMin-${id}`).value);
    const ctrlMax = parseFloat(document.getElementById(`ov-ctrlMax-${id}`).value);
    
    const rangeSpan = document.getElementById(`ov-range-text-${id}`);
    const badge = document.getElementById(`ov-status-badge-${id}`);
    
    if (isNaN(stdValue) || isNaN(uncertainty)) {
      rangeSpan.innerText = "-";
      badge.className = "px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอคำนวณ";
      badge.setAttribute('data-status', 'WAITING');
      checkOvenOverallStatus();
      return;
    }
    
    const minCalculated = parseFloat((stdValue - Math.abs(uncertainty)).toFixed(2));
    const maxCalculated = parseFloat((stdValue + Math.abs(uncertainty)).toFixed(2));
    rangeSpan.innerText = `${minCalculated.toFixed(2)} - ${maxCalculated.toFixed(2)}`;
    
    if (isNaN(ctrlMin) || isNaN(ctrlMax)) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-amber-50 text-amber-600 border border-amber-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอช่วงควบคุม";
      badge.setAttribute('data-status', 'WAITING');
      checkOvenOverallStatus();
      return;
    }
    
    if (minCalculated >= ctrlMin && maxCalculated <= ctrlMax) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-green-100 text-green-700 border border-green-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ยอมรับ";
      badge.setAttribute('data-status', 'PASS');
    } else {
      badge.className = "px-2 py-1 rounded text-[11px] bg-red-100 text-red-700 border border-red-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ปฏิเสธ";
      badge.setAttribute('data-status', 'FAIL');
    }
    checkOvenOverallStatus();
  }

  function checkOvenOverallStatus() {
    const container = document.getElementById('oven-rows-container');
    if (!container) return;
    const rows = container.querySelectorAll('.ov-row-item');
    const overallStatusEl = document.getElementById('ov-overall-status');
    if (!overallStatusEl) return;
    
    if (rows.length === 0) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-slate-200 text-slate-700 font-bold text-sm shadow-sm print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "รอข้อมูลจุดทดสอบ";
      return;
    }
    
    let totalPass = true; let hasWaiting = false;
    rows.forEach(row => {
      const id = row.id.replace('ov-row-', '');
      const badge = document.getElementById(`ov-status-badge-${id}`);
      const status = badge.getAttribute('data-status');
      if (status === 'FAIL') totalPass = false;
      if (status === 'WAITING') hasWaiting = true;
    });
    
    if (hasWaiting) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-amber-100 text-amber-700 font-bold text-sm border border-amber-200 animate-pulse print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "กำลังรอข้อมูลให้ครบถ้วน...";
      return;
    }
    
    if (totalPass) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-green-600 text-white font-bold text-sm shadow border border-green-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ยอมรับและนำไปใช้งานได้";
    } else {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-red-600 text-white font-bold text-sm shadow border border-red-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ปฏิเสธและไม่นำไปใช้งาน";
    }
  }

  function submitOvenForm() {
    const form = document.getElementById('oven-form');
    if(!form.checkValidity()) { form.reportValidity(); return; }
    
    const container = document.getElementById('oven-rows-container');
    const rows = container.querySelectorAll('.ov-row-item');
    if(rows.length === 0) {
      alert('กรุณาเพิ่มรายการจุดทดสอบอย่างน้อย 1 รายการครับ');
      return;
    }

    let finalType = "";
    const selectedRadio = document.querySelector('input[name="ovenType"]:checked');
    if(selectedRadio) {
      if(selectedRadio.value === "Other") {
        finalType = document.getElementById('ov-type-other-text').value;
      } else {
        finalType = selectedRadio.value;
      }
    }

    const readingArr = []; const stdArr = []; const uncArr = [];
    const calcRangeArr = []; const ctrlRangeArr = []; const resultArr = [];
    let incomplete = false;
    
    rows.forEach(row => {
      const id = row.id.replace('ov-row-', '');
      const rd = document.getElementById(`ov-read-${id}`).value;
      const st = document.getElementById(`ov-std-${id}`).value;
      const uc = document.getElementById(`ov-unc-${id}`).value;
      const ran = document.getElementById(`ov-range-text-${id}`).innerText;
      const cMin = document.getElementById(`ov-ctrlMin-${id}`).value;
      const cMax = document.getElementById(`ov-ctrlMax-${id}`).value;
      const res = document.getElementById(`ov-status-badge-${id}`).innerText;
      
      if(!rd || !st || !uc || !cMin || !cMax) incomplete = true;
      
      readingArr.push(rd);
      stdArr.push(st);
      uncArr.push(uc);
      calcRangeArr.push(ran);
      ctrlRangeArr.push(`${cMin} - ${cMax}`);
      resultArr.push(res);
    });
    
    if(incomplete) {
      alert('กรุณากรอกข้อมูลตัวเลขในตารางให้ครบถ้วนทุกช่องครับ');
      return;
    }
    
    const fileInput = document.getElementById('ov-pdfFile');
    const overallStatus = document.getElementById('ov-overall-status').innerText;
    
    const payload = {
      ovenType: finalType,
      calDept: document.getElementById('ov-cal-dept') ? document.getElementById('ov-cal-dept').value : "",
      eqId: document.getElementById('ov-eqSelect').value,
      serialNo: document.getElementById('ov-serialNo').value,
      assetBrand: document.getElementById('ov-assetBrand').value,
      calDate: document.getElementById('ov-calDate').value,
      certNo: document.getElementById('ov-certNo').value,
      reading: readingArr.join('\n'),
      stdTemp: stdArr.join('\n'),
      uncertainty: uncArr.join('\n'),
      readingUncertainty: calcRangeArr.join('\n'),
      controlRange: ctrlRangeArr.join('\n'),
      result: resultArr.join('\n') + '\n\nสรุป: ' + overallStatus,
      addedBy: currentUser ? _getDisplayName() : 'System'
    };
    _calEditInject('oven', payload); // ✅ โหมดแก้ไข: ส่งเลขแถวเดิมไปเขียนทับ
    
    if (fileInput && fileInput.files.length > 0) {
      showLoader();
      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        payload.pdfBase64 = e.target.result.split(',')[1];
        payload.pdfName = file.name;
        executeSubmitOven(payload);
      };
      reader.readAsDataURL(file);
    } else {
      if(confirm("ยืนยันบันทึกผลการตรวจสอบโดยไม่มีไฟล์เอกสารแนบใช่หรือไม่?")) {
        showLoader();
        executeSubmitOven(payload);
      }
    }
  }

  function executeSubmitOven(payload) {
    google.script.run.withSuccessHandler(function(res) {
      if(res.success) {
        google.script.run.withSuccessHandler(function() {
          hideLoader();
          alert('บันทึกผลและส่งอีเมลแจ้งเตือนสำเร็จ');
          document.getElementById('oven-form').reset();
          document.getElementById('oven-rows-container').innerHTML = '';
          const otherTxt = document.getElementById('ov-type-other-text');
          if (otherTxt) otherTxt.classList.add('hidden'); 
          _afterCalSave('oven');
        }).withFailureHandler(function() {
          hideLoader(); alert('บันทึกสำเร็จ แต่ระบบส่งอีเมลล้มเหลว');
        }).sendApprovalEmail(payload); 
      } else {
        hideLoader(); alert('Error: ' + res.message);
      }
    }).withFailureHandler(function(err) {
      hideLoader(); alert('เซิร์ฟเวอร์ขัดข้อง: ' + err);
    }).saveOvenWithPdf(payload); 
  }

  // --- THERMO-HYGROMETER MODULE ---
  function loadEquipmentsForThermoHygro() {
    const dept = currentUser ? currentUser.dept : "";
    const role = currentUser ? currentUser.role : "";

    google.script.run.withSuccessHandler(function(data) {
      cachedEquipments = data; 
      cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
      const select = document.getElementById('thh-eqSelect');
      if (!select) return;
      select.innerHTML = '<option value="">-- เลือกรายการเครื่องมือ --</option>';
      // กรองเฉพาะประเภทที่เกี่ยวข้อง
      // ใช้ _getParamType เหมือนกับระบบอื่น
      var filtered = data.filter(function(item) {
        return _getParamType(item['ประเภทเครื่องมือ'] || '') === 'thermohyg';
      });
      filtered.forEach(function(item) {
        select.innerHTML += '<option value="' + item['ID'] + '">' + (item['รหัสครุภัณฑ์']||'-') + ' | ' + (item['ชื่อเครื่องมือ']||'-') + '</option>';
      });
      
      document.getElementById('thh-rows-container').innerHTML = '';
      thermoHygroRowCount = 0;
      addThermoHygroRow('Temperature');
      addThermoHygroRow('Temperature');
      addThermoHygroRow('Humidity');
      addThermoHygroRow('Humidity');
    }).getEquipments(dept, role); // 🌟 ส่งพารามิเตอร์คัดกรองห้องปฏิบัติการ
  }

  function autoFillThermoHygroInfo() {
    const eq = cachedEquipments.find(item => item['ID'] === document.getElementById('thh-eqSelect').value);
    if(eq) {
      document.getElementById('thh-serialNo').value = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || "ไม่ระบุ";
      document.getElementById('thh-assetBrand').value = (eq['รหัสครุภัณฑ์'] || "") + " / " + (eq['ชื่อเครื่องมือ'] || "");
    }
    if (currentUser) {
      document.getElementById('thh-print-recorder-name').innerText = (_getDisplayName() || currentUser.username);
    }
    document.getElementById('thh-print-recorder-date').innerText = new Date().toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
  }

  function addThermoHygroRow(defaultFunc = "") {
    thermoHygroRowCount++;
    const id = thermoHygroRowCount;
    const container = document.getElementById('thh-rows-container');
    if (!container) return;
    
    const rowHtml = `
      <tr id="thh-row-${id}" class="hover:bg-slate-50 dark:hover:bg-slate-800/40 transition thh-row-item">
        <td class="p-2 border-r dark:border-slate-700">
          <input type="text" id="thh-func-${id}" value="${defaultFunc}" list="thh-functions" placeholder="เช่น Temperature" required class="w-full mx-auto p-1 border rounded dark:bg-slate-800 text-center font-bold print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="thh-std-${id}" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="thh-inst-${id}" oninput="calcThermoHygroRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center font-bold print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="thh-unc-${id}" oninput="calcThermoHygroRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center bg-teal-50 dark:bg-teal-900/20 font-bold text-teal-800 dark:text-teal-300 print:bg-transparent">
          <span id="thh-range-text-${id}">-</span>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center">
          <div class="flex items-center justify-center gap-1">
            <input type="number" step="any" id="thh-ctrlMin-${id}" oninput="calcThermoHygroRow(${id})" placeholder="Min" required class="w-16 p-1 border rounded dark:bg-slate-800 text-center text-teal-600 font-bold print:border-none print:bg-transparent print:w-auto">
            <span class="text-slate-400">ถึง</span>
            <input type="number" step="any" id="thh-ctrlMax-${id}" oninput="calcThermoHygroRow(${id})" placeholder="Max" required class="w-16 p-1 border rounded dark:bg-slate-800 text-center text-teal-600 font-bold print:border-none print:bg-transparent print:w-auto">
          </div>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center font-bold">
          <span id="thh-status-badge-${id}" data-status="WAITING" class="px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black">รอคำนวณ</span>
        </td>
        <td class="p-2 text-center print:hidden">
          <button type="button" onclick="removeThermoHygroRow(${id})" class="text-red-500 hover:text-red-700 transition-colors">
            <span class="material-symbols-outlined text-lg">delete</span>
          </button>
        </td>
      </tr>`;
    container.insertAdjacentHTML('beforeend', rowHtml);
    checkThermoHygroOverallStatus();
  }

  function removeThermoHygroRow(id) {
    const row = document.getElementById(`thh-row-${id}`);
    if(row) { row.remove(); checkThermoHygroOverallStatus(); }
  }

  function calcThermoHygroRow(id) {
    const instRead = parseFloat(document.getElementById(`thh-inst-${id}`).value);
    const uncertainty = parseFloat(document.getElementById(`thh-unc-${id}`).value);
    const ctrlMin = parseFloat(document.getElementById(`thh-ctrlMin-${id}`).value);
    const ctrlMax = parseFloat(document.getElementById(`thh-ctrlMax-${id}`).value);
    
    const rangeSpan = document.getElementById(`thh-range-text-${id}`);
    const badge = document.getElementById(`thh-status-badge-${id}`);
    
    if (isNaN(instRead) || isNaN(uncertainty)) {
      rangeSpan.innerText = "-";
      badge.className = "px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอคำนวณ";
      badge.setAttribute('data-status', 'WAITING');
      checkThermoHygroOverallStatus();
      return;
    }
    
    const minCalculated = parseFloat((instRead - Math.abs(uncertainty)).toFixed(2));
    const maxCalculated = parseFloat((instRead + Math.abs(uncertainty)).toFixed(2));
    rangeSpan.innerText = `${minCalculated.toFixed(2)} - ${maxCalculated.toFixed(2)}`;
    
    if (isNaN(ctrlMin) || isNaN(ctrlMax)) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-amber-50 text-amber-600 border border-amber-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอช่วงควบคุม";
      badge.setAttribute('data-status', 'WAITING');
      checkThermoHygroOverallStatus();
      return;
    }
    
    if (minCalculated >= ctrlMin && maxCalculated <= ctrlMax) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-green-100 text-green-700 border border-green-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ยอมรับ";
      badge.setAttribute('data-status', 'PASS');
    } else {
      badge.className = "px-2 py-1 rounded text-[11px] bg-red-100 text-red-700 border border-red-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ปฏิเสธ";
      badge.setAttribute('data-status', 'FAIL');
    }
    checkThermoHygroOverallStatus();
  }

  function checkThermoHygroOverallStatus() {
    const container = document.getElementById('thh-rows-container');
    if (!container) return;
    const rows = container.querySelectorAll('.thh-row-item');
    const overallStatusEl = document.getElementById('thh-overall-status');
    if (!overallStatusEl) return;
    
    if (rows.length === 0) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-slate-200 text-slate-700 font-bold text-sm shadow-sm print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "รอข้อมูลจุดทดสอบ";
      return;
    }
    
    let totalPass = true; let hasWaiting = false;
    rows.forEach(row => {
      const id = row.id.replace('thh-row-', '');
      const badge = document.getElementById(`thh-status-badge-${id}`);
      const status = badge.getAttribute('data-status');
      if (status === 'FAIL') totalPass = false;
      if (status === 'WAITING') hasWaiting = true;
    });
    
    if (hasWaiting) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-amber-100 text-amber-700 font-bold text-sm border border-amber-200 animate-pulse print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "กำลังรอข้อมูลให้ครบถ้วน...";
      return;
    }
    
    if (totalPass) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-green-600 text-white font-bold text-sm shadow border border-green-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ยอมรับและนำไปใช้งานได้";
    } else {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-red-600 text-white font-bold text-sm shadow border border-red-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ปฏิเสธและไม่นำไปใช้งาน";
    }
  }

  function submitThermoHygroForm() {
    const form = document.getElementById('thermohygro-form');
    if(!form.checkValidity()) { form.reportValidity(); return; }
    
    const container = document.getElementById('thh-rows-container');
    const rows = container.querySelectorAll('.thh-row-item');
    if(rows.length === 0) {
      alert('กรุณาเพิ่มรายการจุดทดสอบอย่างน้อย 1 รายการครับ');
      return;
    }

    const funcArr = []; const stdArr = []; const instArr = [];
    const uncArr = []; const calcRangeArr = []; const ctrlRangeArr = []; const resultArr = [];
    let incomplete = false;
    
    rows.forEach(row => {
      const id = row.id.replace('thh-row-', '');
      const fn = document.getElementById(`thh-func-${id}`).value;
      const st = document.getElementById(`thh-std-${id}`).value;
      const ir = document.getElementById(`thh-inst-${id}`).value;
      const uc = document.getElementById(`thh-unc-${id}`).value;
      const ran = document.getElementById(`thh-range-text-${id}`).innerText;
      const cMin = document.getElementById(`thh-ctrlMin-${id}`).value;
      const cMax = document.getElementById(`thh-ctrlMax-${id}`).value;
      const res = document.getElementById(`thh-status-badge-${id}`).innerText;
      
      if(!fn || !st || !ir || !uc || !cMin || !cMax) incomplete = true;
      
      funcArr.push(fn);
      stdArr.push(st);
      instArr.push(ir);
      uncArr.push(uc);
      calcRangeArr.push(ran);
      ctrlRangeArr.push(`${cMin} - ${cMax}`);
      resultArr.push(res);
    });
    
    if(incomplete) {
      alert('กรุณากรอกข้อมูลตัวเลขในตารางให้ครบถ้วนทุกช่องครับ');
      return;
    }
    
    const fileInput = document.getElementById('thh-pdfFile');
    const overallStatus = document.getElementById('thh-overall-status').innerText;
    
    const payload = {
      calDept: document.getElementById('thh-cal-dept') ? document.getElementById('thh-cal-dept').value : "",
      eqId: document.getElementById('thh-eqSelect').value,
      serialNo: document.getElementById('thh-serialNo').value,
      assetBrand: document.getElementById('thh-assetBrand').value,
      calDate: document.getElementById('thh-calDate').value,
      certNo: document.getElementById('thh-certNo').value,
      func: funcArr.join('\n'),
      stdReading: stdArr.join('\n'),
      instReading: instArr.join('\n'),
      uncertainty: uncArr.join('\n'),
      instUncertainty: calcRangeArr.join('\n'),
      controlRange: ctrlRangeArr.join('\n'),
      result: resultArr.join('\n') + '\n\nสรุป: ' + overallStatus,
      addedBy: currentUser ? _getDisplayName() : 'System'
    };
    _calEditInject('thermohygro', payload); // ✅ โหมดแก้ไข: ส่งเลขแถวเดิมไปเขียนทับ
    
    if (fileInput && fileInput.files.length > 0) {
      showLoader();
      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        payload.pdfBase64 = e.target.result.split(',')[1];
        payload.pdfName = file.name;
        executeSubmitThermoHygro(payload);
      };
      reader.readAsDataURL(file);
    } else {
      if(confirm("ยืนยันบันทึกผลการตรวจสอบโดยไม่มีไฟล์เอกสารแนบใช่หรือไม่?")) {
        showLoader();
        executeSubmitThermoHygro(payload);
      }
    }
  }

  function executeSubmitThermoHygro(payload) {
    google.script.run.withSuccessHandler(function(res) {
      if(res.success) {
        google.script.run.withSuccessHandler(function() {
          hideLoader();
          alert('บันทึกผลและส่งอีเมลแจ้งเตือนสำเร็จ');
          document.getElementById('thermohygro-form').reset();
          document.getElementById('thh-rows-container').innerHTML = '';
          _afterCalSave('thermohygro');
        }).withFailureHandler(function() {
          hideLoader(); alert('บันทึกสำเร็จ แต่ระบบส่งอีเมลล้มเหลว');
        }).sendApprovalEmail(payload); 
      } else {
        hideLoader(); alert('Error: ' + res.message);
      }
    }).withFailureHandler(function(err) {
      hideLoader(); alert('เซิร์ฟเวอร์ขัดข้อง: ' + err);
    }).saveThermoHygroWithPdf(payload); 
  }

  // --- THERMAL CYCLER PCR MODULE ---
  function loadEquipmentsForPCR() {
    const dept = currentUser ? currentUser.dept : "";
    const role = currentUser ? currentUser.role : "";

    google.script.run.withSuccessHandler(function(data) {
      cachedEquipments = data; 
      cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
      const select = document.getElementById('pcr-eqSelect');
      if (!select) return;
      select.innerHTML = '<option value="">-- เลือกรายการเครื่องมือ --</option>';
      // กรองเฉพาะประเภทที่เกี่ยวข้อง
      // ใช้ _getParamType เหมือนกับระบบอื่น
      var filtered = data.filter(function(item) {
        return _getParamType(item['ประเภทเครื่องมือ'] || '') === 'pcr';
      });
      filtered.forEach(function(item) {
        select.innerHTML += '<option value="' + item['ID'] + '">' + (item['รหัสครุภัณฑ์']||'-') + ' | ' + (item['ชื่อเครื่องมือ']||'-') + '</option>';
      });
      
      document.getElementById('pcr-rows-container').innerHTML = '';
      pcrRowCount = 0;
      addPcrRow(96.000);
      addPcrRow(60.000);
      addPcrRow(72.000);
      addPcrRow(4.000);
    }).getEquipments(dept, role); // 🌟 ส่งพารามิเตอร์คัดกรองห้องปฏิบัติการ
  }

  function autoFillPcrInfo() {
    const eq = cachedEquipments.find(item => item['ID'] === document.getElementById('pcr-eqSelect').value);
    if(eq) {
      document.getElementById('pcr-serialNo').value = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || 'ไม่ระบุ';
      document.getElementById('pcr-assetBrand').value = (eq['รหัสครุภัณฑ์'] || '') + ' / ' + (eq['ชื่อเครื่องมือ'] || '');
      // sync print elements
      var ps = document.getElementById('pcr-print-serial');
      var pa = document.getElementById('pcr-print-asset');
      var pn = document.getElementById('pcr-print-eqname');
      if (ps) ps.textContent = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || '-';
      if (pa) pa.textContent = (eq['รหัสครุภัณฑ์'] || '') + ' / ' + (eq['ชื่อเครื่องมือ'] || '');
      if (pn) pn.textContent = eq['ชื่อเครื่องมือ'] || '-';
    }
    if (currentUser) {
      document.getElementById('pcr-print-recorder-name').innerText = (_getDisplayName() || currentUser.username);
      // sync dept
      var pd = document.getElementById('pcr-print-dept');
      if (pd) pd.textContent = currentUser.dept || '';
    }
    document.getElementById('pcr-print-recorder-date').innerText = new Date().toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
    // sync cal dept
    var calDeptEl = document.getElementById('pcr-cal-dept');
    var pcd = document.getElementById('pcr-print-caldept');
    if (calDeptEl && pcd) pcd.textContent = calDeptEl.options[calDeptEl.selectedIndex]?.text || '';
  }

  // sync print fields on date/certNo change
  function _syncPcrPrintFields() {
    var cd = document.getElementById('pcr-calDate');
    var cn = document.getElementById('pcr-certNo');
    var pcd2 = document.getElementById('pcr-print-caldate');
    var pcn = document.getElementById('pcr-print-certno');
    if (cd && pcd2) pcd2.textContent = cd.value ? new Date(cd.value).toLocaleDateString('th-TH', {year:'numeric',month:'short',day:'numeric'}) : '';
    if (cn && pcn) pcn.textContent = cn.value || '';
    // overall status
    var os = document.getElementById('pcr-overall-status');
    var po = document.getElementById('pcr-print-overall');
    if (os && po) po.textContent = '= ' + (os.innerText || '');
  }

  function addPcrRow(defaultTp = "") {
    pcrRowCount++;
    const id = pcrRowCount;
    const container = document.getElementById('pcr-rows-container');
    if (!container) return;
    
    const rowHtml = `
      <tr id="pcr-row-${id}" class="hover:bg-slate-50 dark:hover:bg-slate-800/40 transition pcr-row-item">
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="pcr-tp-${id}" value="${defaultTp}" oninput="calcPcrRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center font-bold print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="pcr-mean-${id}" oninput="calcPcrRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="pcr-min-${id}" oninput="calcPcrRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="pcr-max-${id}" oninput="calcPcrRow(${id})" required class="w-20 mx-auto p-1 border rounded dark:bg-slate-800 text-center print:border-none print:bg-transparent print:w-auto">
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center bg-indigo-50 dark:bg-indigo-900/20 font-bold text-indigo-800 dark:text-indigo-300 print:bg-transparent">
          <span id="pcr-uni-text-${id}">-</span>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center bg-indigo-50 dark:bg-indigo-900/20 font-bold text-indigo-800 dark:text-indigo-300 print:bg-transparent">
          <span id="pcr-acc-text-${id}">-</span>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center font-bold">
          <span id="pcr-status-badge-${id}" data-status="WAITING" class="px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black">รอคำนวณ</span>
        </td>
        <td class="p-2 text-center print:hidden">
          <button type="button" onclick="removePcrRow(${id})" class="text-red-500 hover:text-red-700 transition-colors">
            <span class="material-symbols-outlined text-lg">delete</span>
          </button>
        </td>
      </tr>`;
    container.insertAdjacentHTML('beforeend', rowHtml);
    checkPcrOverallStatus();
  }

  function removePcrRow(id) {
    const row = document.getElementById(`pcr-row-${id}`);
    if(row) { row.remove(); checkPcrOverallStatus(); }
  }

  function calcPcrRow(id) {
    const tpVal = parseFloat(document.getElementById(`pcr-tp-${id}`).value);
    const meanVal = parseFloat(document.getElementById(`pcr-mean-${id}`).value);
    const minVal = parseFloat(document.getElementById(`pcr-min-${id}`).value);
    const maxVal = parseFloat(document.getElementById(`pcr-max-${id}`).value);
    
    const uniSpan = document.getElementById(`pcr-uni-text-${id}`);
    const accSpan = document.getElementById(`pcr-acc-text-${id}`);
    const badge = document.getElementById(`pcr-status-badge-${id}`);
    
    if (isNaN(tpVal) || isNaN(meanVal) || isNaN(minVal) || isNaN(maxVal)) {
      uniSpan.innerText = "-"; accSpan.innerText = "-";
      badge.className = "px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอคำนวณ";
      badge.setAttribute('data-status', 'WAITING');
      checkPcrOverallStatus();
      return;
    }
    
    const uniformity = parseFloat((maxVal - minVal).toFixed(2));
    uniSpan.innerText = uniformity.toFixed(2);
    
    const accuracy = parseFloat((meanVal - tpVal).toFixed(2));
    accSpan.innerText = accuracy.toFixed(2);
    
    // ✅ เกณฑ์การยอมรับ PCR / Thermal cycler (แก้ที่ PCR_CRITERIA จุดเดียว)
    const isUniformityPass = uniformity <= PCR_CRITERIA.uniformityMax;
    const isAccuracyPass = Math.abs(accuracy) <= PCR_CRITERIA.accuracyAbs;
    
    if (isUniformityPass && isAccuracyPass) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-green-100 text-green-700 border border-green-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ยอมรับ";
      badge.setAttribute('data-status', 'PASS');
    } else {
      badge.className = "px-2 py-1 rounded text-[11px] bg-red-100 text-red-700 border border-red-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "ปฏิเสธ";
      badge.setAttribute('data-status', 'FAIL');
    }
    checkPcrOverallStatus();
  }

  // เกณฑ์การยอมรับ: Uniformity ≤ 1.0 และ Accuracy อยู่ในช่วง ± 1.5
  var PCR_CRITERIA = { uniformityMax: 1.0, accuracyAbs: 1.5 };

  function checkPcrOverallStatus() {
    const container = document.getElementById('pcr-rows-container');
    if (!container) return;
    const rows = container.querySelectorAll('.pcr-row-item');
    const overallStatusEl = document.getElementById('pcr-overall-status');
    if (!overallStatusEl) return;
    
    if (rows.length === 0) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-slate-200 text-slate-700 font-bold text-sm shadow-sm print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "รอข้อมูลจุดทดสอบ";
      return;
    }
    
    let totalPass = true; let hasWaiting = false;
    rows.forEach(row => {
      const id = row.id.replace('pcr-row-', '');
      const badge = document.getElementById(`pcr-status-badge-${id}`);
      const status = badge.getAttribute('data-status');
      if (status === 'FAIL') totalPass = false;
      if (status === 'WAITING') hasWaiting = true;
    });
    
    if (hasWaiting) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-amber-100 text-amber-700 font-bold text-sm border border-amber-200 animate-pulse print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "กำลังรอข้อมูลให้ครบถ้วน...";
      return;
    }
    
    if (totalPass) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-green-600 text-white font-bold text-sm shadow border border-green-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ยอมรับและนำไปใช้งานได้";
    } else {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-red-600 text-white font-bold text-sm shadow border border-red-700 print:bg-transparent print:border-none print:text-black print:p-0";
      overallStatusEl.innerText = "ปฏิเสธและไม่นำไปใช้งาน";
    }
  }

  function submitPcrForm() {
    const form = document.getElementById('pcr-form');
    if(!form.checkValidity()) { form.reportValidity(); return; }
    
    const container = document.getElementById('pcr-rows-container');
    const rows = container.querySelectorAll('.pcr-row-item');
    if(rows.length === 0) {
      alert('กรุณาเพิ่มรายการจุดทดสอบอย่างน้อย 1 รายการครับ');
      return;
    }

    const tpArr = []; const meanArr = []; const minArr = []; const maxArr = [];
    const uniArr = []; const accArr = []; const resultArr = [];
    let incomplete = false;
    
    rows.forEach(row => {
      const id = row.id.replace('pcr-row-', '');
      const tp = document.getElementById(`pcr-tp-${id}`).value;
      const mean = document.getElementById(`pcr-mean-${id}`).value;
      const min = document.getElementById(`pcr-min-${id}`).value;
      const max = document.getElementById(`pcr-max-${id}`).value;
      const uni = document.getElementById(`pcr-uni-text-${id}`).innerText;
      const acc = document.getElementById(`pcr-acc-text-${id}`).innerText;
      const res = document.getElementById(`pcr-status-badge-${id}`).innerText;
      
      if(!tp || !mean || !min || !max) incomplete = true;
      
      tpArr.push(tp);
      meanArr.push(mean);
      minArr.push(min);
      maxArr.push(max);
      uniArr.push(uni);
      accArr.push(acc);
      resultArr.push(res);
    });
    
    if(incomplete) {
      alert('กรุณากรอกข้อมูลตัวเลขในตารางให้ครบถ้วนทุกช่องครับ');
      return;
    }
    
    const fileInput = document.getElementById('pcr-pdfFile');
    const overallStatus = document.getElementById('pcr-overall-status').innerText;
    
    const payload = {
      calDept: document.getElementById('pcr-cal-dept') ? document.getElementById('pcr-cal-dept').value : "",
      eqId: document.getElementById('pcr-eqSelect').value,
      serialNo: document.getElementById('pcr-serialNo').value,
      assetBrand: document.getElementById('pcr-assetBrand').value,
      calDate: document.getElementById('pcr-calDate').value,
      certNo: document.getElementById('pcr-certNo').value,
      testingPoint: tpArr.join('\n'),
      meanReading: meanArr.join('\n'),
      minReading: minArr.join('\n'),
      maxReading: maxArr.join('\n'),
      uniformity: uniArr.join('\n'),
      accuracy: accArr.join('\n'),
      result: resultArr.join('\n') + '\n\nสรุป: ' + overallStatus,
      addedBy: currentUser ? _getDisplayName() : 'System'
    };
    _calEditInject('pcr', payload); // ✅ โหมดแก้ไข: ส่งเลขแถวเดิมไปเขียนทับ
    
    if (fileInput && fileInput.files.length > 0) {
      showLoader();
      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        payload.pdfBase64 = e.target.result.split(',')[1];
        payload.pdfName = file.name;
        executeSubmitPcr(payload);
      };
      reader.readAsDataURL(file);
    } else {
      if(confirm("ยืนยันบันทึกผลการตรวจสอบโดยไม่มีไฟล์เอกสารแนบใช่หรือไม่?")) {
        showLoader();
        executeSubmitPcr(payload);
      }
    }
  }

  function executeSubmitPcr(payload) {
    google.script.run.withSuccessHandler(function(res) {
      if(res.success) {
        google.script.run.withSuccessHandler(function() {
          hideLoader();
          alert('บันทึกผลและส่งอีเมลแจ้งเตือนสำเร็จ');
          document.getElementById('pcr-form').reset();
          document.getElementById('pcr-rows-container').innerHTML = '';
          _afterCalSave('pcr');
        }).withFailureHandler(function() {
          hideLoader(); alert('บันทึกสำเร็จ แต่ระบบส่งอีเมลล้มเหลว');
        }).sendApprovalEmail(payload); 
      } else {
        hideLoader(); alert('Error: ' + res.message);
      }
    }).withFailureHandler(function(err) {
      hideLoader(); alert('เซิร์ฟเวอร์ขัดข้อง: ' + err);
    }).savePCRWithPdf(payload); 
  }

  // ==========================================
  // Calibration Plan & Executive Summary Report
  // ==========================================
  // ตัวแปรเก็บ calibration summary (eqId -> { certNo, result, calDate, approvalStatus })
  var calSummaryData = {};

  function loadCalibrationReport() {
    const fySelect = document.getElementById('report-fy-filter');
    if (!fySelect) return;
    if (fySelect.options.length === 0) {
      const currentYear = new Date().getFullYear();
      const currentBYear = currentYear + 543;
      fySelect.innerHTML = '<option value="all">-- รวมทุกปีงบประมาณ --</option>';
      for (let i = -1; i <= 3; i++) {
        let y = currentYear + i; let by = currentBYear + i;
        let sel = (i === 0) ? 'selected' : '';
        fySelect.innerHTML += `<option value="${y}" ${sel}>ปีงบประมาณ พ.ศ. ${by}</option>`;
      }
    }

    const typeSelect = document.getElementById('report-type-filter');
    if (typeSelect) {
      const types = [...new Set(cachedEquipments.map(item => item['ประเภทเครื่องมือ']).filter(Boolean))];
      typeSelect.innerHTML = '<option value="all">-- รวมทุกประเภทเครื่องมือ --</option>';
      types.sort().forEach(t => {
        typeSelect.innerHTML += `<option value="${t}">${t}</option>`;
      });
    }

    // เติม dropdown หน่วยงาน
    const deptSelect = document.getElementById('report-dept-filter');
    if (deptSelect) {
      const depts = [...new Set(cachedEquipments.map(item => item['สถานที่ติดตั้ง']).filter(Boolean))];
      deptSelect.innerHTML = '<option value="all">— ทุกหน่วยงาน —</option>';
      depts.sort().forEach(d => {
        deptSelect.innerHTML += `<option value="${d}">${d}</option>`;
      });
    }

    // ดึงข้อมูลสรุปผลการสอบเทียบจากทุกชีต
    const tbody = document.getElementById('cal-report-table-body');
    if (tbody) tbody.innerHTML = '<tr><td colspan="5" class="text-center py-8 text-slate-400">กำลังโหลดข้อมูล...</td></tr>';

    google.script.run
      .withSuccessHandler(function(res) {
        if (res && res.success) {
          calSummaryData = res.data || {};
        } else {
          calSummaryData = {};
        }
        renderReportTable();
      })
      .withFailureHandler(function() {
        calSummaryData = {};
        renderReportTable();
      })
      .getCalibrationSummary();
  }

 // ==========================================
  // pH METER CALIBRATION MODULE
  // ==========================================
  let phRowCount = 0;

  // 🌟 เปลี่ยนจาก window.addPHRow = function() เป็นรูปแบบนี้
  function addPHRow() {
    phRowCount++;
    const id = phRowCount;
    const container = document.getElementById('ph-rows-container');
    if (!container) return;

    const rowHtml = `
      <tr id="ph-row-${id}" class="hover:bg-slate-50 dark:hover:bg-slate-800/40 transition ph-row-item">
        <td class="p-1 border-r dark:border-slate-700">
          <input type="text" id="ph-function-${id}" placeholder="pH4" required class="w-full p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center font-bold print:border-none print:bg-transparent">
        </td>
        <td class="p-1 border-r dark:border-slate-700">
          <input type="number" step="any" id="ph-stdRead-${id}" required class="w-full p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center print:border-none print:bg-transparent">
        </td>
        <td class="p-1 border-r dark:border-slate-700">
          <input type="number" step="any" id="ph-instRead-${id}" oninput="calcPHRow(${id})" required class="w-full p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center print:border-none print:bg-transparent">
        </td>
        <td class="p-1 border-r dark:border-slate-700">
          <input type="number" step="any" id="ph-uncertainty-${id}" oninput="calcPHRow(${id})" required class="w-full p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center print:border-none print:bg-transparent">
        </td>
        <td class="p-1 border-r dark:border-slate-700 text-center bg-slate-50 dark:bg-slate-900/20 font-bold text-slate-700 dark:text-slate-300 print:bg-transparent">
          <span id="ph-range-text-${id}">-</span>
        </td>
        <td class="p-1 border-r dark:border-slate-700 text-center">
          <div class="flex items-center justify-center gap-1">
            <input type="number" step="any" id="ph-ctrlMin-${id}" oninput="calcPHRow(${id})" placeholder="Min" required class="w-14 p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center text-blue-600 font-bold print:border-none print:bg-transparent print:w-auto">
            <span class="text-slate-400">-</span>
            <input type="number" step="any" id="ph-ctrlMax-${id}" oninput="calcPHRow(${id})" placeholder="Max" required class="w-14 p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center text-blue-600 font-bold print:border-none print:bg-transparent print:w-auto">
          </div>
        </td>
        <td class="p-1 border-r dark:border-slate-700 text-center font-bold">
          <span id="ph-status-badge-${id}" data-status="WAITING" class="px-2 py-1 rounded text-[10px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black">รอคำนวณ</span>
        </td>
        <td class="p-1 text-center print:hidden">
          <button type="button" onclick="removePHRow(${id})" class="text-red-500 hover:text-red-700 transition-colors">
            <span class="material-symbols-outlined text-lg">delete</span>
          </button>
        </td>
      </tr>`;
    
    container.insertAdjacentHTML('beforeend', rowHtml);
    checkPHOverallStatus();
  }

  // 🌟 เปลี่ยนเป็นฟังก์ชันปกติ
  function removePHRow(id) {
    const row = document.getElementById(`ph-row-${id}`);
    if (row) { row.remove(); checkPHOverallStatus(); }
  }

  // 🌟 เปลี่ยนเป็นฟังก์ชันปกติ
  function calcPHRow(id) {
    const instRead = parseFloat(document.getElementById(`ph-instRead-${id}`).value);
    const uncertainty = parseFloat(document.getElementById(`ph-uncertainty-${id}`).value);
    const ctrlMin = parseFloat(document.getElementById(`ph-ctrlMin-${id}`).value);
    const ctrlMax = parseFloat(document.getElementById(`ph-ctrlMax-${id}`).value);
    
    const rangeSpan = document.getElementById(`ph-range-text-${id}`);
    const badge = document.getElementById(`ph-status-badge-${id}`);
    
    if (isNaN(instRead) || isNaN(uncertainty)) {
      rangeSpan.innerText = "-";
      badge.className = "px-2 py-1 rounded text-[10px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอคำนวณ";
      badge.setAttribute('data-status', 'WAITING');
      checkPHOverallStatus();
      return;
    }
    
    const minCalculated = parseFloat((instRead - Math.abs(uncertainty)).toFixed(2));
    const maxCalculated = parseFloat((instRead + Math.abs(uncertainty)).toFixed(2));
    rangeSpan.innerText = `${minCalculated.toFixed(2)}  -  ${maxCalculated.toFixed(2)}`;
    
    if (isNaN(ctrlMin) || isNaN(ctrlMax)) {
      badge.className = "px-2 py-1 rounded text-[10px] bg-amber-50 text-amber-600 border border-amber-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอช่วงควบคุม";
      badge.setAttribute('data-status', 'WAITING');
      checkPHOverallStatus();
      return;
    }
    
    if (minCalculated >= ctrlMin && maxCalculated <= ctrlMax) {
      badge.className = "px-2 py-1 rounded text-[10px] bg-green-100 text-green-700 border border-green-200 block w-full print:bg-transparent print:border-none print:text-green-700";
      badge.innerText = "ยอมรับ";
      badge.setAttribute('data-status', 'PASS');
    } else {
      badge.className = "px-2 py-1 rounded text-[10px] bg-red-100 text-red-700 border border-red-200 block w-full print:bg-transparent print:border-none print:text-red-700 font-bold";
      badge.innerText = "ปฏิเสธ";
      badge.setAttribute('data-status', 'FAIL');
    }
    checkPHOverallStatus();
  }

  function checkPHOverallStatus() {
    const container = document.getElementById('ph-rows-container');
    if (!container) return;
    const rows = container.querySelectorAll('.ph-row-item');
    const overallStatusEl = document.getElementById('ph-overall-status');
    if (!overallStatusEl) return;
    
    if (rows.length === 0) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-slate-200 text-slate-700 font-bold text-sm print:p-0";
      overallStatusEl.innerText = "รอข้อมูลจุดทดสอบ";
      return;
    }
    
    let totalPass = true;
    let hasWaiting = false;
    
    rows.forEach(row => {
      const id = row.id.replace('ph-row-', '');
      const badge = document.getElementById(`ph-status-badge-${id}`);
      const status = badge ? badge.getAttribute('data-status') : 'WAITING';
      if (status === 'FAIL') totalPass = false;
      if (status === 'WAITING') hasWaiting = true;
    });
    
    if (hasWaiting) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-amber-100 text-amber-700 font-bold text-sm border border-amber-200 animate-pulse print:p-0";
      overallStatusEl.innerText = "กำลังรอข้อมูลให้ครบถ้วน...";
      return;
    }
    
    if (totalPass) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-green-600 text-white font-bold text-sm shadow border border-green-700 print:bg-transparent print:text-green-700 print:p-0 print:border-none";
      overallStatusEl.innerText = "ยอมรับและนำไปใช้งานได้";
    } else {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-red-600 text-white font-bold text-sm shadow border border-red-700 print:bg-transparent print:text-red-700 print:p-0 print:border-none";
      overallStatusEl.innerText = "ปฏิเสธและไม่นำไปใช้งาน";
    }
  }

  // 🌟 เปลี่ยนเป็นฟังก์ชันปกติ
  function submitPHMeterForm() {
    const form = document.getElementById('phmeter-form');
    if (!form.checkValidity()) { form.reportValidity(); return; }
    
    const container = document.getElementById('ph-rows-container');
    const rows = container.querySelectorAll('.ph-row-item');
    if (rows.length === 0) {
      alert('กรุณาเพิ่มรายการจุดทดสอบในตารางอย่างน้อย 1 รายการครับ');
      return;
    }
    
    const funcArr = []; const stdArr = []; const instArr = []; 
    const uncArr = []; const rangeArr = []; const ctrlArr = []; const resultArr = [];
    let incomplete = false;
    
    rows.forEach(row => {
      const id = row.id.replace('ph-row-', '');
      const fn = document.getElementById(`ph-function-${id}`).value;
      const sr = document.getElementById(`ph-stdRead-${id}`).value;
      const ir = document.getElementById(`ph-instRead-${id}`).value;
      const uc = document.getElementById(`ph-uncertainty-${id}`).value;
      const ran = document.getElementById(`ph-range-text-${id}`).innerText;
      const cMin = document.getElementById(`ph-ctrlMin-${id}`).value;
      const cMax = document.getElementById(`ph-ctrlMax-${id}`).value;
      const res = document.getElementById(`ph-status-badge-${id}`).innerText;
      
      if (!fn || !sr || !ir || !uc || !cMin || !cMax) incomplete = true;
      
      funcArr.push(fn);
      stdArr.push(sr);
      instArr.push(ir);
      uncArr.push(uc);
      rangeArr.push(ran);
      ctrlArr.push(`${cMin} - ${cMax}`);
      resultArr.push(res);
    });
    
    if (incomplete) {
      alert('กรุณากรอกข้อมูลในตารางจุดทดสอบให้ครบถ้วนทุกช่องครับ');
      return;
    }
    
    const fileInput = document.getElementById('ph-pdfFile');
    const overallStatus = document.getElementById('ph-overall-status').innerText;
    
    const payload = {
      calDept: document.getElementById('ph-cal-dept') ? document.getElementById('ph-cal-dept').value : "",
      eqId: document.getElementById('ph-eqSelect').value,
      serialNo: document.getElementById('ph-serialNo').value,
      assetBrand: document.getElementById('ph-assetBrand').value,
      calDate: document.getElementById('ph-calDate').value,
      certNo: document.getElementById('ph-certNo').value,
      
      phFunction: funcArr.join('\n'),
      stdReading: stdArr.join('\n'),
      instReading: instArr.join('\n'),
      uncertainty: uncArr.join('\n'),
      calcRange: rangeArr.join('\n'),
      controlRange: ctrlArr.join('\n'),
      result: resultArr.join('\n') + '\n\nสรุป: ' + overallStatus,
      addedBy: currentUser ? _getDisplayName() : 'System'
    };
    _calEditInject('phmeter', payload); // ✅ โหมดแก้ไข: ส่งเลขแถวเดิมไปเขียนทับ
    
    if (fileInput && fileInput.files.length > 0) {
      showLoader();
      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        payload.pdfBase64 = e.target.result.split(',')[1];
        payload.pdfName = file.name;
        executeSubmitPHMeter(payload);
      };
      reader.readAsDataURL(file);
    } else {
      if (confirm("ยืนยันบันทึกผลการตรวจสอบ pH Meter โดยไม่มีไฟล์เอกสารแนบใช่หรือไม่?")) {
        showLoader();
        executeSubmitPHMeter(payload);
      }
    }
  }

  function executeSubmitPHMeter(payload) {
    google.script.run.withSuccessHandler(function(res) {
      if (res.success) {
        google.script.run.withSuccessHandler(function() {
          hideLoader();
          alert('บันทึกผลตรวจรับรองและส่งอีเมลแจ้งเตือนสำเร็จ');
          document.getElementById('phmeter-form').reset();
          document.getElementById('ph-rows-container').innerHTML = '';
          _afterCalSave('phmeter');
        }).withFailureHandler(function() {
          hideLoader();
          alert('บันทึกข้อมูลสำเร็จ แต่เกิดข้อผิดพลาดในการส่งอีเมลแจ้งเตือน');
          switchPage('dashboard');
        }).sendApprovalEmail(payload);
      } else {
        hideLoader();
        alert('เกิดข้อผิดพลาดจากเซิร์ฟเวอร์: ' + res.message);
      }
    }).withFailureHandler(function(err) {
      hideLoader(); alert('เกิดข้อผิดพลาดในการเชื่อมต่อ: ' + err);
    }).savePHMeterWithPdf(payload); 
  }

  // 🌟 เปลี่ยนเป็นฟังก์ชันปกติ
  function autoFillPHInfo() {
    const eq = cachedEquipments.find(item => item['ID'] === document.getElementById('ph-eqSelect').value);
    if(eq) {
      document.getElementById('ph-serialNo').value = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || "ไม่ระบุ";
      document.getElementById('ph-assetBrand').value = (eq['รหัสครุภัณฑ์'] || "") + " / " + (eq['ชื่อเครื่องมือ'] || "");
    }
    if (currentUser) {
      const rName = document.getElementById('ph-print-recorder-name');
      if (rName) rName.innerText = (_getDisplayName() || currentUser.username);
    }
    const today = new Date();
    const rDate = document.getElementById('ph-print-recorder-date');
    if (rDate) rDate.innerText = today.toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
    
    document.getElementById('ph-rows-container').innerHTML = '';
    phRowCount = 0;
    addPHRow();
    addPHRow();
  }

  function renderReportTable() {
    const tbody = document.getElementById('cal-report-table-body');
    if (!tbody) return;
    const fyFilter     = document.getElementById('report-fy-filter').value;
    const typeFilter   = document.getElementById('report-type-filter').value;
    const statusFilter = (document.getElementById('report-status-filter') || {}).value || 'all';
    const deptFilter   = (document.getElementById('report-dept-filter') || {}).value || 'all';

    const today = new Date(); today.setHours(0,0,0,0);

    let html = ''; let count = 1;
    // counters for quick stats
    var stPass=0, stFail=0, stPending=0, stOverdue=0, stNear60=0, stTotal=0;

    if (!cachedEquipments || cachedEquipments.length === 0) {
      tbody.innerHTML = '<tr><td colspan="7" class="text-center py-8 text-slate-500">ยังไม่มีข้อมูลเครื่องมือในระบบ</td></tr>';
      return;
    }

    cachedEquipments.forEach(item => {
      if (typeFilter !== 'all' && item['ประเภทเครื่องมือ'] !== typeFilter) return;
      if (deptFilter !== 'all' && (item['สถานที่ติดตั้ง']||'') !== deptFilter) return;

      const eqId = item['ID'] || '';
      const calRec = calSummaryData[eqId] || {};
      const certNo = calRec.certNo || '-';
      const calDate = calRec.calDate || '-';
      const result = calRec.result || 'รอผล';
      const approvalStatus = calRec.approvalStatus || '-';
      const eqCondition = item['สถานะเครื่องมือ'] || '';

      // แปลง calDate เป็นภาษาไทย
      let calDateDisplay = '-';
      if (calDate && calDate !== '-') {
        const d = new Date(calDate);
        if (!isNaN(d.getTime())) calDateDisplay = d.toLocaleDateString('th-TH', { year:'numeric', month:'short', day:'numeric' });
        else calDateDisplay = calDate;
      }

      // คำนวณ Due Date
      const isCal = _needsSchedule(item['สถานะการสอบเทียบ'], item);
      const dueDateRaw = item['สอบเทียบครั้งถัดไป'] || '';
      const intervalRaw = String(item['ระยะเวลาการสอบเทียบ'] || '').trim();
      const thaiMonths = ["ม.ค.","ก.พ.","มี.ค.","เม.ย.","พ.ค.","มิ.ย.","ก.ค.","ส.ค.","ก.ย.","ต.ค.","พ.ย.","ธ.ค."];
      function _addInterval(d, iv) {
        if (iv==="6 เดือน"||iv==="6") d.setMonth(d.getMonth()+6);
        else if (iv==="1 ปี"||iv==="12") d.setFullYear(d.getFullYear()+1);
        else if (iv==="2 ปี"||iv==="24") d.setFullYear(d.getFullYear()+2);
        else { var m=parseInt(iv); if(!isNaN(m)) d.setMonth(d.getMonth()+m); }
        return d;
      }

      let planText="-", eqFy="all", dueDate=null, dueDiff=null, dueDateFmt='-';
      if (!isCal) {
        planText="ไม่ระบุแผน";
      } else {
        if (dueDateRaw) { var t=new Date(dueDateRaw); if(!isNaN(t.getTime())) dueDate=t; }
        if (!dueDate && calRec.calDate && calRec.calDate!=='-' && intervalRaw) {
          var t2=new Date(calRec.calDate); if(!isNaN(t2.getTime())) dueDate=_addInterval(t2,intervalRaw);
        }
        if (!dueDate && item['วันที่สอบเทียบล่าสุด'] && intervalRaw) {
          var t3=new Date(item['วันที่สอบเทียบล่าสุด']); if(!isNaN(t3.getTime())) dueDate=_addInterval(t3,intervalRaw);
        }
        if (dueDate) {
          let m=dueDate.getMonth(); let y=dueDate.getFullYear();
          let fy=y; if(m>=9) fy+=1;
          eqFy=fy.toString();
          planText=thaiMonths[m]+' '+(fy+543)+(_isMaint(item)?' (บำรุงรักษา)':'');
          dueDiff=Math.ceil((dueDate-today)/(1000*60*60*24));
          dueDateFmt=dueDate.toLocaleDateString('th-TH',{year:'numeric',month:'short',day:'numeric'});
        }
      }

      if (fyFilter!=='all' && eqFy!==fyFilter && planText!=="ไม่ระบุแผน") return;

      // คำนวณ rowStatus สำหรับ filter
      var isPass   = result.includes('ยอมรับ')||result.includes('ผ่าน')||result.includes('Pass');
      var isFail   = result.includes('ปฏิเสธ')||result.includes('ไม่ผ่าน')||result.includes('Fail');
      var isOverdue = dueDiff !== null && dueDiff < 0;
      var isNear60  = dueDiff !== null && dueDiff >= 0 && dueDiff <= 60;

      // นับ stats
      stTotal++;
      if (isPass) stPass++;
      else if (isFail) stFail++;
      else stPending++;
      if (isOverdue) stOverdue++;
      else if (isNear60) stNear60++;

      // apply status filter
      if (statusFilter === 'pass'    && !isPass)    return;
      if (statusFilter === 'fail'    && !isFail)    return;
      if (statusFilter === 'pending' && (isPass||isFail)) return;
      if (statusFilter === 'overdue' && !isOverdue) return;
      if (statusFilter === 'near60'  && !isNear60)  return;

      // badge ผลการตรวจรับรอง
      let resultBadge;
      if (isPass) {
        resultBadge = `<span class="px-2 py-1 bg-green-100 dark:bg-green-900/30 text-green-700 dark:text-green-400 border border-green-200 dark:border-green-800 rounded-md text-xs font-bold inline-block">✅ ผ่าน / ยอมรับ</span>`;
      } else if (isFail) {
        resultBadge = `<span class="px-2 py-1 bg-red-100 dark:bg-red-900/30 text-red-700 dark:text-red-400 border border-red-200 dark:border-red-800 rounded-md text-xs font-bold inline-block">❌ ไม่ผ่าน / ปฏิเสธ</span>`;
      } else {
        resultBadge = `<span class="px-2 py-1 bg-slate-100 text-slate-500 rounded-md text-xs inline-block">⏳ ${result}</span>`;
      }

      // badge สถานะอนุมัติ
      let approveBadge;
      if (approvalStatus==='อนุมัติแล้ว') {
        approveBadge = `<span class="px-2 py-1 bg-blue-100 text-blue-700 border border-blue-200 rounded-md text-xs font-bold inline-block">✔ อนุมัติแล้ว</span>`;
      } else if (approvalStatus && approvalStatus!=='-') {
        approveBadge = `<span class="text-amber-600 text-xs font-bold">${approvalStatus}</span>`;
      } else {
        approveBadge = `<span class="text-slate-400 text-xs">รออนุมัติ</span>`;
      }

      // badge สถานะเครื่องมือ + Due Date
      let eqStatusBadge;
      let dueBadge = '';
      if (eqCondition === 'สอบเทียบไม่ผ่านเกณฑ์') {
        eqStatusBadge = `<span class="px-2 py-1 bg-red-100 text-red-700 border border-red-200 rounded-md text-xs font-bold inline-block">⚠️ ไม่ผ่านเกณฑ์</span>`;
      } else if (eqCondition === 'เครื่องมือเสีย') {
        eqStatusBadge = `<span class="px-2 py-1 bg-stone-100 text-stone-700 rounded-md text-xs font-bold inline-block">🔧 เครื่องเสีย</span>`;
      } else if (eqCondition === 'ส่งซ่อม') {
        eqStatusBadge = `<span class="px-2 py-1 bg-orange-100 text-orange-700 rounded-md text-xs font-bold inline-block">🛠️ ส่งซ่อม</span>`;
      } else if (eqCondition === 'จำหน่ายออก') {
        eqStatusBadge = `<span class="px-2 py-1 bg-slate-100 text-slate-600 rounded-md text-xs font-bold inline-block">📦 จำหน่ายออก</span>`;
      } else if (eqCondition === 'ระงับการใช้งาน') {
        eqStatusBadge = `<span class="px-2 py-1 bg-violet-100 text-violet-700 rounded-md text-xs font-bold inline-block">🚫 ระงับการใช้</span>`;
      } else if (eqCondition === 'พร้อมใช้งาน') {
        eqStatusBadge = `<span class="px-2 py-1 bg-emerald-100 text-emerald-700 rounded-md text-xs font-bold inline-block">✅ พร้อมใช้งาน</span>`;
      } else if (eqCondition === 'รอสอบเทียบใหม่') {
        eqStatusBadge = `<span class="px-2 py-1 bg-amber-100 text-amber-700 rounded-md text-xs font-bold inline-block">🔄 รอสอบเทียบใหม่</span>`;
      } else {
        eqStatusBadge = `<span class="text-slate-400 text-xs">-</span>`;
      }

      // Due Date badge (แสดงใต้แผน)
      if (dueDiff !== null) {
        if (dueDiff < 0) {
          dueBadge = `<div class="text-[10px] mt-1 text-red-500 font-bold">🔴 เกิน ${Math.abs(dueDiff)} วัน (${dueDateFmt})</div>`;
        } else if (dueDiff <= 30) {
          dueBadge = `<div class="text-[10px] mt-1 text-orange-500 font-bold">🔴 อีก ${dueDiff} วัน (${dueDateFmt})</div>`;
        } else if (dueDiff <= 60) {
          dueBadge = `<div class="text-[10px] mt-1 text-amber-500 font-bold">⚠️ อีก ${dueDiff} วัน (${dueDateFmt})</div>`;
        } else {
          dueBadge = `<div class="text-[10px] mt-1 text-slate-400">${dueDateFmt}</div>`;
        }
      }

      // row highlight
      var rowClass = 'hover:bg-slate-50 dark:hover:bg-slate-700/30 transition';
      if (isOverdue) rowClass += ' bg-red-50/40 dark:bg-red-900/10';
      else if (isNear60) rowClass += ' bg-amber-50/40 dark:bg-amber-900/10';

      html += `
        <tr class="${rowClass}" data-report-status="${isPass?'pass':isFail?'fail':'pending'}">
          <td class="px-4 py-3 border-r dark:border-slate-700 text-center font-medium">${count++}</td>
          <td class="px-4 py-3 border-r dark:border-slate-700">
            <div class="font-bold text-slate-800 dark:text-slate-100">${item['ชื่อเครื่องมือ']||'-'}</div>
            <div class="text-[11px] text-slate-500 uppercase mt-0.5">${item['รหัสครุภัณฑ์']||'-'}</div>
            <div class="text-[11px] text-slate-400 mt-0.5">${item['ประเภทเครื่องมือ']||''}</div>
          </td>
          <td class="px-4 py-3 border-r dark:border-slate-700 text-center text-xs text-slate-600 whitespace-nowrap">${item['สถานที่ติดตั้ง']||'-'}</td>
          <td class="px-4 py-3 border-r dark:border-slate-700 text-center">
            <div class="font-bold text-indigo-600 dark:text-indigo-400">${planText}</div>
            ${dueBadge}
          </td>
          <td class="px-4 py-3 border-r dark:border-slate-700 text-center">
            <div class="text-sm font-mono font-bold text-slate-700 dark:text-slate-200">${certNo}</div>
            <div class="text-[11px] text-slate-400 mt-0.5">${calDateDisplay!=='-'?'สอบเทียบ: '+calDateDisplay:''}</div>
          </td>
          <td class="px-4 py-3 border-r dark:border-slate-700 text-center">${resultBadge}</td>
          <td class="px-4 py-3 border-r dark:border-slate-700 text-center">${approveBadge}</td>
          <td class="px-4 py-3 text-center">${eqStatusBadge}</td>
        </tr>`;
    });

    if (html==='') { html=`<tr><td colspan="8" class="text-center py-8 text-slate-500">ไม่พบเครื่องมือที่ตรงกับเงื่อนไข</td></tr>`; }
    tbody.innerHTML = html;

    // อัปเดต Quick Stats
    var statsEl = document.getElementById('report-quick-stats');
    if (statsEl) {
      var statsHtml = [
        { label:'ทั้งหมด',        val: stTotal,   cls: 'bg-slate-100 text-slate-700 border-slate-300', icon:'📋' },
        { label:'ผ่านการรับรอง',  val: stPass,    cls: 'bg-green-100 text-green-700 border-green-300', icon:'✅' },
        { label:'ไม่ผ่านเกณฑ์',  val: stFail,    cls: 'bg-red-100 text-red-700 border-red-300',       icon:'❌' },
        { label:'รอผลสอบเทียบ',  val: stPending, cls: 'bg-slate-100 text-slate-500 border-slate-200', icon:'⏳' },
        { label:'เกินกำหนด',     val: stOverdue, cls: 'bg-red-50 text-red-600 border-red-200',        icon:'🔴' },
        { label:'ใกล้ถึงกำหนด ≤60 วัน', val: stNear60, cls: 'bg-amber-50 text-amber-700 border-amber-200', icon:'⚠️' },
      ].map(function(s){
        var act = statusFilter === 'all' ? '' : (
          (statusFilter==='pass'&&s.label==='ผ่านการรับรอง')||
          (statusFilter==='fail'&&s.label==='ไม่ผ่านเกณฑ์')||
          (statusFilter==='pending'&&s.label==='รอผลสอบเทียบ')||
          (statusFilter==='overdue'&&s.label==='เกินกำหนด')||
          (statusFilter==='near60'&&s.label==='ใกล้ถึงกำหนด ≤60 วัน')
        ) ? ' ring-2 ring-offset-1 ring-primary-400' : '';
        return `<span class="inline-flex items-center gap-1.5 px-3 py-1.5 rounded-full border text-xs font-bold ${s.cls}${act}">${s.icon} ${s.label} <strong>(${s.val})</strong></span>`;
      }).join('');
      statsEl.innerHTML = statsHtml;
    }

    const subTitle = document.getElementById('print-subtitle');
    if (subTitle) {
      const selectedFyText   = document.getElementById('report-fy-filter').options[document.getElementById('report-fy-filter').selectedIndex].text;
      const selectedTypeText = document.getElementById('report-type-filter').options[document.getElementById('report-type-filter').selectedIndex].text;
      subTitle.innerText = `เงื่อนไข: ${selectedFyText} | ${selectedTypeText}`;
    }
  }

  function closeExecutiveReport() {
    const modal = document.getElementById('executive-report-modal');
    if (modal) modal.classList.add('hidden');
  }

 function loadDepartmentsToDropdown() {
  google.script.run
    .withSuccessHandler(function(departments) {
      const deptDropdowns = [
        'th-cal-dept', 'pi-cal-dept', 'ce-cal-dept', 'ba-cal-dept', 
        'thh-cal-dept', 'ph-cal-dept', 'aoi-cal-dept', 'ov-cal-dept', 
        'ru-cal-dept', 'pcr-cal-dept', 'bsc-cal-dept', 'bf-cal-dept'
      ];
      
      deptDropdowns.forEach(id => {
        const selectEl = document.getElementById(id);
        if (selectEl) {
          selectEl.innerHTML = '<option value="">-- เลือกห้องปฏิบัติการ --</option>';
          departments.forEach(dept => {
            selectEl.innerHTML += `<option value="${dept}">${dept}</option>`;
          });
        }
      });
    })
    .withFailureHandler(function(error) {
      console.error("ไม่สามารถโหลดรายชื่อบริษัทผู้ให้บริการได้:", error);
    })
    .getDepartmentList();
}

// นำฟังก์ชันนี้ไปผูกไว้กับตอนเริ่มต้นโหลดระบบ (สามารถรวมไว้ใน window.addEventListener('DOMContentLoaded', ... ) ที่คุณมีอยู่แล้วได้เลย)
document.addEventListener('DOMContentLoaded', function() {
  loadDepartmentsToDropdown();
});

function exportPdfByUnit() {
  if (!cachedEquipments || cachedEquipments.length === 0) {
    alert("ไม่มีข้อมูลสำหรับสร้างเอกสาร PDF ครับ");
    return;
  }

  // ดึงชื่อหน่วยงานของผู้ใช้งานปัจจุบัน
  const dept = currentUser ? currentUser.dept : "ทั้งหมด";
  
  // เปิดหน้าต่างใหม่เพื่อเตรียมเอกสาร PDF
  const printWindow = window.open('', '_blank');
  
  let html = `
  <!DOCTYPE html>
  <html>
  <head>
    <title>สรุปแผนการสอบเทียบเครื่องมือ_${dept}</title>
    <style>
      @import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@400;700&display=swap');
      body { font-family: 'Sarabun', sans-serif; color: #1e293b; padding: 20px; }
      h2 { text-align: center; margin-bottom: 5px; }
      p { text-align: center; margin-top: 0; color: #475569; margin-bottom: 20px; }
      table { width: 100%; border-collapse: collapse; margin-top: 20px; font-size: 13px; }
      th, td { border: 1px solid #cbd5e1; padding: 10px; text-align: center; }
      th { background-color: #f8fafc; font-weight: bold; }
      .text-left { text-align: left; }
      
      /* โครงสร้างตารางลายเซ็น (ซ่อนเส้นขอบ) */
      .sig-table { width: 100%; margin-top: 60px; text-align: center; border: none; page-break-inside: avoid; }
      .sig-table th, .sig-table td { border: none; padding: 5px; background-color: transparent; }
      
      @media print {
        @page { size: A4 landscape; margin: 15mm; }
        body { -webkit-print-color-adjust: exact; print-color-adjust: exact; }
      }
    </style>
  </head>
  <body>
    <h2>สรุปแผนและบันทึกข้อมูลการสอบเทียบเครื่องมือ</h2>
    <p>หน่วยงาน/ห้องปฏิบัติการ: <b>${dept}</b></p>
    
    <table>
      <thead>
        <tr>
          <th width="5%">ลำดับ</th>
          <th width="15%">รหัสครุภัณฑ์</th>
          <th width="20%">ชื่อเครื่องมือ</th>
          <th width="15%">ช่วงใช้งาน</th>
          <th width="15%">จุดทดสอบ</th>
          <th width="15%">สอบเทียบล่าสุด</th>
          <th width="15%">กำหนดสอบเทียบถัดไป</th>
        </tr>
      </thead>
      <tbody>
  `;

  // วนลูปนำข้อมูลเครื่องมือมาใส่ในตาราง
  cachedEquipments.forEach((item, index) => {
    let dueDateText = item['วันที่สอบเทียบรอบถัดไป'] || item['สอบเทียบครั้งถัดไป'] || item['Due Date'] || '-';
    let lastDateText = item['วันที่สอบเทียบล่าสุด'] || '-';
    
    // แปลงวันที่สอบเทียบถัดไปให้เป็น Format ภาษาไทย
    if (!dueDateText || dueDateText === '-') {
      if (item['วันที่สอบเทียบล่าสุด'] && item['ระยะเวลาการสอบเทียบ']) {
         let lastDate = new Date(item['วันที่สอบเทียบล่าสุด']);
         let interval = String(item['ระยะเวลาการสอบเทียบ']).trim();
         if (!isNaN(lastDate.getTime())) {
            if (interval === "6" || interval === "6 เดือน") lastDate.setMonth(lastDate.getMonth() + 6);
            else if (interval === "12" || interval === "1 ปี") lastDate.setFullYear(lastDate.getFullYear() + 1);
            else if (interval === "24" || interval === "2 ปี") lastDate.setFullYear(lastDate.getFullYear() + 2);
            
            const thMonths = ["ม.ค.", "ก.พ.", "มี.ค.", "เม.ย.", "พ.ค.", "มิ.ย.", "ก.ค.", "ส.ค.", "ก.ย.", "ต.ค.", "พ.ย.", "ธ.ค."];
            dueDateText = `${lastDate.getDate()} ${thMonths[lastDate.getMonth()]} ${lastDate.getFullYear() + 543}`;
         }
      }
    } else if (dueDateText.includes("-")) {
       let dParts = dueDateText.split("-");
       if(dParts.length === 3) {
           const thMonths = ["ม.ค.", "ก.พ.", "มี.ค.", "เม.ย.", "พ.ค.", "มิ.ย.", "ก.ค.", "ส.ค.", "ก.ย.", "ต.ค.", "พ.ย.", "ธ.ค."];
           dueDateText = `${parseInt(dParts[2])} ${thMonths[parseInt(dParts[1])-1]} ${parseInt(dParts[0]) + 543}`;
       }
    }

    // แปลงวันที่สอบเทียบล่าสุดเป็นภาษาไทย (ถ้าเป็น YYYY-MM-DD)
    if (lastDateText.includes("-") && lastDateText.split("-").length === 3) {
       let dParts = lastDateText.split("-");
       const thMonths = ["ม.ค.", "ก.พ.", "มี.ค.", "เม.ย.", "พ.ค.", "มิ.ย.", "ก.ค.", "ส.ค.", "ก.ย.", "ต.ค.", "พ.ย.", "ธ.ค."];
       lastDateText = `${parseInt(dParts[2])} ${thMonths[parseInt(dParts[1])-1]} ${parseInt(dParts[0]) + 543}`;
    }

    // ดึงค่าล่าสุดจากช่อง Input (กรณีผู้ใช้พิมพ์ไว้แต่ยังไม่ได้กดบันทึก)
    let usage = document.getElementById(`table-usage-${item['ID']}`) ? document.getElementById(`table-usage-${item['ID']}`).value : (item['ช่วงใช้งาน'] || "-");
    let testPoint = document.getElementById(`table-testpoint-${item['ID']}`) ? document.getElementById(`table-testpoint-${item['ID']}`).value : (item['จุดทดสอบ'] || "-");

    html += `
      <tr>
        <td>${index + 1}</td>
        <td class="text-left">${item['รหัสครุภัณฑ์'] || '-'}</td>
        <td class="text-left font-bold">${item['ชื่อเครื่องมือ'] || '-'}</td>
        <td>${usage || '-'}</td>
        <td>${testPoint || '-'}</td>
        <td>${lastDateText}</td>
        <td style="color: #b91c1c; font-weight: bold;">${dueDateText}</td>
      </tr>
    `;
  });

  html += `
      </tbody>
    </table>

    <table class="sig-table">
      <tr>
        <td style="padding-bottom: 30px;">ผู้รับรอง .....................................................</td>
        <td style="padding-bottom: 30px;">ผู้ตรวจสอบ .....................................................</td>
        <td style="padding-bottom: 30px;">ผู้อนุมัติ .....................................................</td>
      </tr>
      <tr>
        <td>(.................................................)</td>
        <td>(.................................................)</td>
        <td>(.................................................)</td>
      </tr>
      <tr>
        <td style="font-weight: bold; padding-top: 5px;">หัวหน้าหน่วย</td>
        <td style="font-weight: bold; padding-top: 5px;">ผู้จัดการด้านเทคนิควิชาการ</td>
        <td style="font-weight: bold; padding-top: 5px;">หัวหน้าสาขาวิชาพยาธิวิทยา</td>
      </tr>
      <tr>
        <td style="padding-top: 15px;">ว/ด/ป..........................</td>
        <td style="padding-top: 15px;">ว/ด/ป..........................</td>
        <td style="padding-top: 15px;">ว/ด/ป..........................</td>
      </tr>
    </table>
  </body>
  </html>
  `;

  printWindow.document.open();
  printWindow.document.write(html);
  printWindow.document.close();

  // หน่วงเวลาเล็กน้อยเพื่อให้เบราว์เซอร์โหลดฟอนต์ Sarabun เสร็จก่อนเรียกหน้าต่าง Save PDF
  setTimeout(() => {
    printWindow.print();
  }, 800);
}

function submitForm() {
    const form = document.getElementById('equipment-form');
    const inputs = form.querySelectorAll('input, select, textarea');
    inputs.forEach(input => input.disabled = false);

    showLoader();
    
    const brandVal = document.getElementById('brand')?.value.trim() || "";
    const modelVal = document.getElementById('model')?.value.trim() || "";
    const snVal = document.getElementById('serialNumber')?.value.trim() || "";
    const eqModelCombined = [brandVal, modelVal, snVal].filter(Boolean).join(" / ");

    // 🛠️ แก้ไข ID ให้ตรงกับ HTML ของคุณ
    const companyNameVal = document.getElementById('compName')?.value.trim() || "";
    const salesNameVal = document.getElementById('compRep')?.value.trim() || "";
    const companyPhoneVal = document.getElementById('compPhone')?.value.trim() || "";
    // ถ้าไม่มีช่อง companyAddress ใน HTML จะได้ค่าว่างไป
    const companyAddressVal = document.getElementById('companyAddress')?.value.trim() || "";

    const techN = document.getElementById('techName')?.value.trim() || "";
    const techP = document.getElementById('techPhone')?.value.trim() || "";
    const eqTechnicianCombined = [techN, techP].filter(Boolean).join(" / ");
    
    // ดึงค่าการสอบเทียบเพื่อใช้เป็นเงื่อนไข
    const isCalVal = document.getElementById('isCalibrated')?.value || "";

    const payload = {
      eqName: document.getElementById('eqName')?.value || "", 
      eqType: document.getElementById('eqType')?.value || "",
      eqCode: document.getElementById('eqCode')?.value || "", 
      eqModel: eqModelCombined,
      eqSpec: document.getElementById('eqSpec')?.value || "", 
      companyName: companyNameVal,
      salesName: salesNameVal,
      companyPhone: companyPhoneVal,
      companyAddress: companyAddressVal,
      eqTechnician: eqTechnicianCombined,
      dateReceive: document.getElementById('dateReceive')?.value || "", 
      dateStart: document.getElementById('dateStart')?.value || "",
      location: document.getElementById('location')?.value || "", 
      status: document.getElementById('status')?.value || "",
      manual: document.getElementById('manualFile')?.value || "", // แก้ ID เป็น manualFile ตาม HTML
      lifespan: document.getElementById('lifespan')?.value || "",
      value: document.getElementById('value')?.value || "", 
      
      // 🌟 ข้อมูลการสอบเทียบ (ถ้าเลือก "สอบเทียบ" ถึงจะส่งค่าไป ถ้าไม่ใช่ให้ส่ง "-")
      isCalibrated: isCalVal,
      lastCalDate: _needsSchedule(isCalVal) ? (document.getElementById('lastCalDate')?.value || "") : "-", 
      calInterval: _needsSchedule(isCalVal) ? (document.getElementById('calInterval')?.value || "") : "-",
      nextCalDate: _needsSchedule(isCalVal) ? (document.getElementById('nextCalDate')?.value || "") : "-",
      
      addedBy: (typeof currentUser !== 'undefined' && currentUser) ? currentUser.username : "System",
      rowIndex: parseInt(document.getElementById('eqRowIndex')?.value || '') || null
    };

    google.script.run.withSuccessHandler(function(res) {
      hideLoader();
      if(res.success) {
        var rowIdx = document.getElementById('eqRowIndex')?.value;
        var msg = (res.action === 'updated' || rowIdx)
          ? '✅ แก้ไขข้อมูลเครื่องมือสำเร็จ'
          : '✅ บันทึกและลงทะเบียนเครื่องมือสำเร็จ';
        alert(msg);
        form.reset();
        document.getElementById('calDateFieldContainer')?.classList.add('hidden');
        document.getElementById('eqRowIndex').value = '';
        // reset title กลับ
        var titleEl = document.getElementById('form-panel-title');
        var iconEl  = document.getElementById('form-panel-icon');
        if (titleEl) titleEl.textContent = 'ลงทะเบียนเครื่องมือใหม่';
        if (iconEl)  iconEl.textContent  = 'add_circle';
        // รีโหลดข้อมูลและกลับหน้า equipment-list
        if (typeof loadEquipments === 'function') loadEquipments();
        if (typeof switchPage === 'function') switchPage('equipment-list');
      } else { 
        alert('เกิดข้อผิดพลาด: ' + res.message); 
      }
    }).withFailureHandler(function(err) { 
      hideLoader(); 
      alert('เซิร์ฟเวอร์แจ้งเตือน Error: ' + err.message);
      console.error(err);
    }).saveEquipment(payload);
}

  // เปิด Modal และโหลดรายชื่อเครื่องมือใส่ Dropdown
  function openHistoryPrintModal() {
    const select = document.getElementById('hp-eq-select');
    if (select) {
      select.innerHTML = '<option value="">-- เลือกรายการเครื่องมือ --</option>';
      if (cachedEquipments && cachedEquipments.length > 0) {
        cachedEquipments.forEach(item => {
          const code = item['รหัสครุภัณฑ์'] || "-";
          const name = item['ชื่อเครื่องมือ'] || "-";
          select.innerHTML += `<option value="${item['ID']}">${code} | ${name}</option>`;
        });
      } else {
         select.innerHTML = '<option value="">ยังไม่มีข้อมูลเครื่องมือในระบบ</option>';
      }
    }
    document.getElementById('history-print-modal').classList.remove('hidden');
  }

  function closeHistoryPrintModal() {
    document.getElementById('history-print-modal').classList.add('hidden');
  }

  // สร้างไฟล์ PDF ตามเครื่องมือที่เลือก
  function executePrintHistoryPdf() {
    const eqId = document.getElementById('hp-eq-select').value;
    if (!eqId) {
      alert("กรุณาเลือกเครื่องมือที่ต้องการพิมพ์ประวัติครับ");
      return;
    }
    
    // ค้นหาข้อมูลเครื่องมือจาก ID
    const item = cachedEquipments.find(e => e['ID'] === eqId);
    if (!item) return;

    const revision = document.getElementById('hp-revision').value || "00";
    const effDate = document.getElementById('hp-eff-date').value || "15 ต.ค. 2567";

    // ดึงค่าอื่นๆ โดยพยายามเช็คชื่อ Column ใน Sheet (ปรับแก้ให้ตรงกับหัวตารางใน Sheet ของคุณได้)
    const brandModelSn = item['รายละเอียด/สเปค'] || item['ยี่ห้อ/รุ่น/Serial'] || "-";
    const companyName = item['ชื่อบริษัทที่จำหน่าย'] || item['บริษัท'] || item['Company Name'] || "-";
    const salesName = item['ชื่อเซลล์/ผู้ติดต่อ'] || item['เซลล์'] || "-";
    const companyPhone = item['เบอร์โทรศัพท์'] || item['เบอร์ติดต่อ'] || "-";

    const printWindow = window.open('', '_blank');
    const html = `
    <!DOCTYPE html>
    <html>
    <head>
      <title>แบบบันทึกประวัติเครื่องมือ_${item['รหัสครุภัณฑ์'] || 'Unknow'}</title>
      <style>
        @import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@400;700&display=swap');
        body { font-family: 'Sarabun', sans-serif; color: #000; padding: 20px; font-size: 14px; }
        table { width: 100%; border-collapse: collapse; margin-bottom: 20px; }
        th, td { border: 1px solid #000; padding: 8px 12px; }
        .text-center { text-align: center; }
        .font-bold { font-weight: bold; }
        .bg-gray { background-color: #f3f4f6; }
        
        .iso-header th, .iso-header td { padding: 5px 10px; font-size: 14px; }
        .iso-header .title { font-size: 18px; font-weight: bold; }
        .iso-header .sub-info { font-size: 12px; text-align: left; }
        
        @media print {
          @page { size: A4 portrait; margin: 15mm; }
          body { -webkit-print-color-adjust: exact; print-color-adjust: exact; }
        }
      </style>
    </head>
    <body>
      <table class="iso-header">
        <tr>
          <td rowspan="2" width="15%" class="text-center font-bold">PSU</td>
          <td rowspan="2" width="50%" class="text-center">
            <div class="title">แบบบันทึกประวัติเครื่องมือ</div>
            <div>SD-Fo-06-Patho-QMU-007</div>
          </td>
          <td width="35%" class="sub-info">สาขาวิชาพยาธิวิทยา</td>
        </tr>
        <tr>
          <td class="sub-info" style="line-height: 1.6;">
            แก้ไขครั้งที่ ${revision}<br>
            วันที่ประกาศใช้ ${effDate}<br>
            หน้า 1 จากทั้งหมด 2 หน้า
          </td>
        </tr>
      </table>

      <h3 style="margin-top: 30px; border-bottom: 1px solid #000; padding-bottom: 5px;">ข้อมูลรายละเอียดเครื่องมือ (Equipment Details)</h3>
      <table>
        <tr><td width="30%" class="bg-gray font-bold">ชื่อเครื่องมือ</td><td width="70%">${item['ชื่อเครื่องมือ'] || '-'}</td></tr>
        <tr><td class="bg-gray font-bold">รหัสครุภัณฑ์</td><td style="font-size: 16px; font-weight: bold;">${item['รหัสครุภัณฑ์'] || '-'}</td></tr>
        <tr><td class="bg-gray font-bold">ประเภทเครื่องมือ</td><td>${item['ประเภทเครื่องมือ'] || '-'}</td></tr>
        <tr><td class="bg-gray font-bold">สถานที่ติดตั้ง</td><td>${item['สถานที่ติดตั้ง'] || '-'}</td></tr>
        <tr><td class="bg-gray font-bold">รายละเอียด (ยี่ห้อ/รุ่น/SN)</td><td>${brandModelSn}</td></tr>
        <tr><td class="bg-gray font-bold">วันที่รับเครื่อง / เริ่มใช้งาน</td><td>รับเครื่อง: ${item['วันที่รับเครื่อง'] || '-'} &nbsp;&nbsp;|&nbsp;&nbsp; เริ่มใช้งาน: ${item['วันที่เริ่มใช้เครื่อง'] || '-'}</td></tr>
        <tr><td class="bg-gray font-bold">มูลค่าเครื่องมือ</td><td>${item['มูลค่าเครื่องมือ'] ? parseFloat(item['มูลค่าเครื่องมือ']).toLocaleString() + ' บาท' : '-'}</td></tr>
      </table>

      <h3 style="margin-top: 20px; border-bottom: 1px solid #000; padding-bottom: 5px;">ข้อมูลบริษัทตัวแทนจำหน่าย (Supplier Information)</h3>
      <table>
        <tr><td width="30%" class="bg-gray font-bold">ชื่อบริษัท / ตัวแทน</td><td width="70%">${companyName}</td></tr>
        <tr><td class="bg-gray font-bold">พนักงานขาย / ผู้ประสานงาน</td><td>${salesName}</td></tr>
        <tr><td class="bg-gray font-bold">เบอร์โทรศัพท์ติดต่อ</td><td>${companyPhone}</td></tr>
      </table>
    </body>
    </html>
    `;
    printWindow.document.open();
    printWindow.document.write(html);
    printWindow.document.close();
    
    // สั่งปิด Modal หลักและสั่ง Print
    closeHistoryPrintModal();
    setTimeout(() => { printWindow.print(); }, 800);
  }

// ==========================================
  // RULER CALIBRATION MODULE (MULTI-ROWS)
  // ==========================================
  let rulerRowCount = 0;

  // ฟังก์ชันเริ่มต้นเพิ่มบรรทัดให้อัตโนมัติ 3 แถวเมื่อเปิดหน้า
  function initRulerModule() {
    document.getElementById('ruler-rows-container').innerHTML = '';
    rulerRowCount = 0;
    addRulerRow();
    addRulerRow();
    addRulerRow();
  }

  // ฟังก์ชันเพิ่มแถวในตารางไม้บรรทัด
  window.addRulerRow = function() {
    rulerRowCount++;
    const id = rulerRowCount;
    const container = document.getElementById('ruler-rows-container');
    if (!container) return;

    const rowHtml = `
      <tr id="ru-row-${id}" class="hover:bg-slate-50 dark:hover:bg-slate-800/40 transition ru-row-item">
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ru-nominalVal-${id}" required class="w-full p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center font-bold print:border-none print:bg-transparent">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ru-displayVal-${id}" oninput="calcRulerRow(${id})" required class="w-full p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center print:border-none print:bg-transparent">
        </td>
        <td class="p-2 border-r dark:border-slate-700">
          <input type="number" step="any" id="ru-uncertainty-${id}" oninput="calcRulerRow(${id})" required class="w-full p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center print:border-none print:bg-transparent">
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center bg-slate-50 dark:bg-slate-900/20 font-bold text-slate-700 dark:text-slate-300 print:bg-transparent">
          <span id="ru-range-text-${id}">-</span>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center">
          <div class="flex items-center justify-center gap-1">
            <input type="number" step="any" id="ru-ctrlMin-${id}" oninput="calcRulerRow(${id})" placeholder="Min" required class="w-16 p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center text-blue-600 font-bold print:border-none print:bg-transparent print:w-auto">
            <span class="text-slate-400">-</span>
            <input type="number" step="any" id="ru-ctrlMax-${id}" oninput="calcRulerRow(${id})" placeholder="Max" required class="w-16 p-1 border rounded dark:bg-slate-800 dark:border-slate-600 text-center text-blue-600 font-bold print:border-none print:bg-transparent print:w-auto">
          </div>
        </td>
        <td class="p-2 border-r dark:border-slate-700 text-center font-bold">
          <span id="ru-status-badge-${id}" data-status="WAITING" class="px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black">รอคำนวณ</span>
        </td>
        <td class="p-2 text-center print:hidden">
          <button type="button" onclick="removeRulerRow(${id})" class="text-red-500 hover:text-red-700 transition-colors">
            <span class="material-symbols-outlined text-lg">delete</span>
          </button>
        </td>
      </tr>`;
    
    container.insertAdjacentHTML('beforeend', rowHtml);
    checkRulerOverallStatus();
  }

  window.removeRulerRow = function(id) {
    const row = document.getElementById(`ru-row-${id}`);
    if (row) {
      row.remove();
      checkRulerOverallStatus();
    }
  }

  // ฟังก์ชันคำนวณผลลัพธ์เปรียบเทียบกับช่วงควบคุม รายบรรทัด
  window.calcRulerRow = function(id) {
    const displayVal = parseFloat(document.getElementById(`ru-displayVal-${id}`).value);
    const uncertainty = parseFloat(document.getElementById(`ru-uncertainty-${id}`).value);
    const ctrlMin = parseFloat(document.getElementById(`ru-ctrlMin-${id}`).value);
    const ctrlMax = parseFloat(document.getElementById(`ru-ctrlMax-${id}`).value);
    
    const rangeSpan = document.getElementById(`ru-range-text-${id}`);
    const badge = document.getElementById(`ru-status-badge-${id}`);
    
    if (isNaN(displayVal) || isNaN(uncertainty)) {
      rangeSpan.innerText = "-";
      badge.className = "px-2 py-1 rounded text-[11px] bg-slate-100 text-slate-500 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอคำนวณ";
      badge.setAttribute('data-status', 'WAITING');
      checkRulerOverallStatus();
      return;
    }
    
    // คำนวณช่วงค่าความคลาดเคลื่อนจริง
    const minCalculated = parseFloat((displayVal - Math.abs(uncertainty)).toFixed(2));
    const maxCalculated = parseFloat((displayVal + Math.abs(uncertainty)).toFixed(2));
    rangeSpan.innerText = `${minCalculated.toFixed(2)}  -  ${maxCalculated.toFixed(2)}`;
    
    if (isNaN(ctrlMin) || isNaN(ctrlMax)) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-amber-50 text-amber-600 border border-amber-200 block w-full print:bg-transparent print:border-none print:text-black";
      badge.innerText = "รอช่วงควบคุม";
      badge.setAttribute('data-status', 'WAITING');
      checkRulerOverallStatus();
      return;
    }
    
    // 🌟 เกณฑ์เงื่อนไข: ช่วงคำนวณต้องตกอยู่ในขอบเขตช่วงควบคุมทั้งหมด
    if (minCalculated >= ctrlMin && maxCalculated <= ctrlMax) {
      badge.className = "px-2 py-1 rounded text-[11px] bg-green-100 text-green-700 border border-green-200 block w-full print:bg-transparent print:border-none print:text-green-700";
      badge.innerText = "ยอมรับ";
      badge.setAttribute('data-status', 'PASS');
    } else {
      badge.className = "px-2 py-1 rounded text-[11px] bg-red-100 text-red-700 border border-red-200 block w-full print:bg-transparent print:border-none print:text-red-700 font-bold";
      badge.innerText = "ปฏิเสธ";
      badge.setAttribute('data-status', 'FAIL');
    }
    checkRulerOverallStatus();
  }

  // ฟังก์ชันคำนวณสถานะรวมของเครื่องมือไม้บรรทัดชิ้นนี้
  function checkRulerOverallStatus() {
    const container = document.getElementById('ruler-rows-container');
    if (!container) return;
    const rows = container.querySelectorAll('.ru-row-item');
    const overallStatusEl = document.getElementById('ru-overall-status');
    if (!overallStatusEl) return;
    
    if (rows.length === 0) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-slate-200 text-slate-700 font-bold text-sm print:p-0";
      overallStatusEl.innerText = "รอข้อมูลจุดทดสอบ";
      return;
    }
    
    let totalPass = true;
    let hasWaiting = false;
    
    rows.forEach(row => {
      const id = row.id.replace('ru-row-', '');
      const badge = document.getElementById(`ru-status-badge-${id}`);
      const status = badge ? badge.getAttribute('data-status') : 'WAITING';
      if (status === 'FAIL') totalPass = false;
      if (status === 'WAITING') hasWaiting = true;
    });
    
    if (hasWaiting) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-amber-100 text-amber-700 font-bold text-sm border border-amber-200 animate-pulse print:p-0";
      overallStatusEl.innerText = "กำลังรอข้อมูลให้ครบถ้วน...";
      return;
    }
    
    if (totalPass) {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-green-600 text-white font-bold text-sm shadow border border-green-700 print:bg-transparent print:text-green-700 print:p-0 print:border-none";
      overallStatusEl.innerText = "ยอมรับและนำไปใช้งานได้";
    } else {
      overallStatusEl.className = "px-4 py-2 rounded-lg bg-red-600 text-white font-bold text-sm shadow border border-red-700 print:bg-transparent print:text-red-700 print:p-0 print:border-none";
      overallStatusEl.innerText = "ปฏิเสธและไม่นำไปใช้งาน";
    }
  }

  // ฟังก์ชันรวบรวมข้อมูลส่งไปบันทึกหลังบ้าน
  window.submitRulerForm = function() {
    const form = document.getElementById('ruler-form');
    if (!form.checkValidity()) { form.reportValidity(); return; }
    
    const container = document.getElementById('ruler-rows-container');
    const rows = container.querySelectorAll('.ru-row-item');
    if (rows.length === 0) {
      alert('กรุณาเพิ่มรายการจุดทดสอบในตารางอย่างน้อย 1 รายการครับ');
      return;
    }
    
    const nominalArr = []; const displayArr = []; const uncertaintyArr = [];
    const displayUncArr = []; const controlRangeArr = []; const resultArr = [];
    let incomplete = false;
    
    rows.forEach(row => {
      const id = row.id.replace('ru-row-', '');
      const nv = document.getElementById(`ru-nominalVal-${id}`).value;
      const dv = document.getElementById(`ru-displayVal-${id}`).value;
      const uc = document.getElementById(`ru-uncertainty-${id}`).value;
      const ran = document.getElementById(`ru-range-text-${id}`).innerText;
      const cMin = document.getElementById(`ru-ctrlMin-${id}`).value;
      const cMax = document.getElementById(`ru-ctrlMax-${id}`).value;
      const res = document.getElementById(`ru-status-badge-${id}`).innerText;
      
      if (!nv || !dv || !uc || !cMin || !cMax) incomplete = true;
      
      nominalArr.push(nv);
      displayArr.push(dv);
      uncertaintyArr.push(uc);
      displayUncArr.push(ran);
      controlRangeArr.push(`${cMin} - ${cMax}`);
      resultArr.push(res);
    });
    
    if (incomplete) {
      alert('กรุณากรอกข้อมูลตัวเลขในตารางจุดทดสอบให้ครบถ้วนทุกช่องครับ');
      return;
    }
    
    const fileInput = document.getElementById('ru-pdfFile');
    const overallStatus = document.getElementById('ru-overall-status').innerText;
    
    const payload = {
      calDept: document.getElementById('ru-cal-dept') ? document.getElementById('ru-cal-dept').value : "",
      eqId: document.getElementById('ru-eqSelect').value,
      serialNo: document.getElementById('ru-serialNo').value,
      assetBrand: document.getElementById('ru-assetBrand').value,
      calDate: document.getElementById('ru-calDate').value,
      certNo: document.getElementById('ru-certNo').value,
      
      // รวมอาเรย์แต่ละคอลัมน์เพื่อจัดลงบรรทัดใน Google Sheet ด้วย \n
      nominalVal: nominalArr.join('\n'),
      displayVal: displayArr.join('\n'),
      uncertainty: uncertaintyArr.join('\n'),
      displayUncertainty: displayUncArr.join('\n'),
      controlRange: controlRangeArr.join('\n'),
      result: resultArr.join('\n') + '\n\nสรุป: ' + overallStatus,
      addedBy: currentUser ? _getDisplayName() : 'System'
    };
    _calEditInject('ruler', payload); // ✅ โหมดแก้ไข: ส่งเลขแถวเดิมไปเขียนทับ
    
    if (fileInput && fileInput.files.length > 0) {
      showLoader();
      const file = fileInput.files[0];
      const reader = new FileReader();
      reader.onload = function(e) {
        payload.pdfBase64 = e.target.result.split(',')[1];
        payload.pdfName = file.name;
        executeSubmitRuler(payload);
      };
      reader.readAsDataURL(file);
    } else {
      if (confirm("ยืนยันบันทึกผลการตรวจสอบไม้บรรทัดโดยไม่มีไฟล์เอกสารแนบใช่หรือไม่?")) {
        showLoader();
        executeSubmitRuler(payload);
      }
    }
  }

  function executeSubmitRuler(payload) {
    google.script.run.withSuccessHandler(function(res) {
      if (res.success) {
        google.script.run.withSuccessHandler(function() {
          hideLoader();
          alert('บันทึกผลตรวจรับรองและส่งอีเมลแจ้งเตือนสำเร็จ');
          document.getElementById('ruler-form').reset();
          document.getElementById('ruler-rows-container').innerHTML = '';
          _afterCalSave('ruler');
        }).withFailureHandler(function() {
          hideLoader();
          alert('บันทึกข้อมูลสำเร็จ แต่เกิดข้อผิดพลาดในการส่งอีเมลแจ้งเตือน');
          switchPage('dashboard');
        }).sendApprovalEmail(payload);
      } else {
        hideLoader();
        alert('เกิดข้อผิดพลาดจากเซิร์ฟเวอร์: ' + res.message);
      }
    }).withFailureHandler(function(err) {
      hideLoader(); alert('เกิดข้อผิดพลาดในการเชื่อมต่อ: ' + err);
    }).saveRulerWithPdf(payload); 
  }

  // ผูกฟังก์ชันเรียกใช้การเติมลายเซ็นพิมพ์งานตามผู้ใช้ปัจจุบัน
  window.autoFillRulerInfo = function() {
    const eq = cachedEquipments.find(item => item['ID'] === document.getElementById('ru-eqSelect').value);
    if(eq) {
      document.getElementById('ru-serialNo').value = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || "ไม่ระบุ";
      document.getElementById('ru-assetBrand').value = (eq['รหัสครุภัณฑ์'] || "") + " / " + (eq['ชื่อเครื่องมือ'] || "");
    }
    if (currentUser) {
      const rName = document.getElementById('ru-print-recorder-name');
      if (rName) rName.innerText = (_getDisplayName() || currentUser.username);
    }
    const today = new Date();
    const rDate = document.getElementById('ru-print-recorder-date');
    if (rDate) rDate.innerText = today.toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
  }
  
  function loadAdminData() {
    showLoader();
    const listIds = ['admin-location-list','admin-department-list','admin-category-list'];
    listIds.forEach(function(id) {
      var el = document.getElementById(id);
      if (el) el.innerHTML = '<li class="text-center text-xs text-slate-400 py-6">กำลังโหลด...</li>';
    });
    var userTbody = document.getElementById('admin-user-list');
    if (userTbody) userTbody.innerHTML = '<tr><td colspan="4" class="text-center py-6 text-slate-400 text-sm">กำลังโหลด...</td></tr>';

    var done = false;
    var timeout = setTimeout(function() {
      if (done) return;
      hideLoader();
      listIds.forEach(function(id) {
        var el = document.getElementById(id);
        if (el) el.innerHTML = '<li class="text-center text-xs text-amber-500 py-4">⏱ GAS ตอบช้า — กรุณากด "โหลดข้อมูล" อีกครั้ง</li>';
      });
    }, 10000);

    google.script.run
      .withSuccessHandler(function(data) {
        done = true;
        clearTimeout(timeout);
        hideLoader();
        if (!data) {
          listIds.forEach(function(id) {
            var el = document.getElementById(id);
            if (el) el.innerHTML = '<li class="text-center text-xs text-red-400 py-4">ไม่ได้รับข้อมูล</li>';
          });
          return;
        }
        window._adminLocations = data.locations || [];
        renderAdminLocations(data.locations || []);
        renderAdminDepartments(data.departments || []);
        renderAdminCategories(data.categories || []);
        renderAdminUsers(data.users || [], data.locations || []);  // ใช้ locations แทน departments
      })
      .withFailureHandler(function(err) {
        done = true;
        clearTimeout(timeout);
        hideLoader();
        var msg = err.message || String(err);
        listIds.forEach(function(id) {
          var el = document.getElementById(id);
          if (el) el.innerHTML = '<li class="text-center text-xs text-red-400 py-4">❌ ' + msg + '</li>';
        });
        var tb = document.getElementById('admin-user-list');
        if (tb) tb.innerHTML = '<tr><td colspan="4" class="text-center py-4 text-red-400 text-xs">❌ ' + msg + '</td></tr>';
      })
      .getAdminSetupData();
  }

// ---------------- 1. จัดการสถานที่ตั้ง (Locations) ----------------
function renderAdminLocations(locations) {
  const list = document.getElementById('admin-location-list');
  if (!list) return;
  if (!locations || locations.length === 0) {
    list.innerHTML = '<li class="text-center text-xs text-slate-400 py-6">ยังไม่มีข้อมูลสถานที่ — กดปุ่ม + เพื่อเพิ่ม</li>';
    return;
  }
  list.innerHTML = locations.map(loc =>
    `<li class="flex justify-between items-center px-3 py-2 bg-slate-50 dark:bg-slate-700/50 rounded-lg border border-slate-100 dark:border-slate-700 hover:bg-slate-100 dark:hover:bg-slate-700 transition group">
      <span class="text-sm text-slate-700 dark:text-slate-200 truncate flex-1">${loc}</span>
      <button onclick="deleteLocation('${loc.replace(/'/g,"\\'")}') " class="opacity-0 group-hover:opacity-100 ml-2 p-1 text-slate-300 hover:text-red-500 transition rounded" title="ลบ">
        <span class="material-symbols-outlined text-base leading-none">delete</span>
      </button>
    </li>`
  ).join('');
}

function addLocation() {
  const newLoc = document.getElementById('admin-new-location').value.trim();
  if (!newLoc) return;
  showLoader();
  google.script.run
    .withSuccessHandler(res => { document.getElementById('admin-new-location').value = ''; loadAdminData(); })
    .withFailureHandler(err => { hideLoader(); alert(err.message); })
    .saveLocation(newLoc); 
}

function deleteLocation(locName) {
  if (!confirm(`ต้องการลบสถานที่ "${locName}" ใช่หรือไม่?`)) return;
  showLoader();
  google.script.run
    .withSuccessHandler(res => { loadAdminData(); })
    .withFailureHandler(err => { hideLoader(); alert(err.message); })
    .removeLocation(locName);
}

// ---------------- 2.ชื่อห้องปฏิบัติการที่ทำการสอบเทียบ (Departments) ----------------
function renderAdminDepartments(departments) {
  const list = document.getElementById('admin-department-list');
  if (!list) return;
  if (!departments || departments.length === 0) {
    list.innerHTML = '<li class="text-center text-xs text-slate-400 py-6">ยังไม่มีบริษัทผู้ให้บริการ — กดปุ่ม + เพื่อเพิ่ม</li>';
    return;
  }
  list.innerHTML = departments.map(dept =>
    `<li class="flex justify-between items-center px-3 py-2 bg-slate-50 dark:bg-slate-700/50 rounded-lg border border-slate-100 dark:border-slate-700 hover:bg-slate-100 dark:hover:bg-slate-700 transition group">
      <span class="text-sm text-slate-700 dark:text-slate-200 truncate flex-1">${dept}</span>
      <button onclick="deleteDepartment('${dept.replace(/'/g,"\\'")}') " class="opacity-0 group-hover:opacity-100 ml-2 p-1 text-slate-300 hover:text-red-500 transition rounded" title="ลบ">
        <span class="material-symbols-outlined text-base leading-none">delete</span>
      </button>
    </li>`
  ).join('');
}

function addDepartment() {
  const newDept = document.getElementById('admin-new-department').value.trim();
  if (!newDept) return;
  showLoader();
  google.script.run
    .withSuccessHandler(res => { document.getElementById('admin-new-department').value = ''; loadAdminData(); })
    .withFailureHandler(err => { hideLoader(); alert(err.message); })
    .saveDepartment(newDept); 
}

function deleteDepartment(deptName) {
  if (!confirm(`ต้องการลบบริษัทผู้ให้บริการ "${deptName}" ใช่หรือไม่?`)) return;
  showLoader();
  google.script.run
    .withSuccessHandler(res => { loadAdminData(); })
    .withFailureHandler(err => { hideLoader(); alert(err.message); })
    .removeDepartment(deptName);
}

// ---------------- 3. จัดการบัญชีผู้ใช้งาน (Users) ----------------
function renderAdminUsers(users, locations) {
  const tbody = document.getElementById('admin-user-list');
  tbody.innerHTML = '';
  
  // อัปเดต dropdown department ใน modal ให้ใช้ locations
  var deptSelect = document.getElementById('u-department');
  if (deptSelect) {
    deptSelect.innerHTML = '<option value="ทั้งหมด">ทั้งหมด (All)</option>';
    (locations || []).forEach(function(loc) {
      deptSelect.insertAdjacentHTML('beforeend', '<option value="' + loc + '">' + loc + '</option>');
    });
  }

  if (!users || users.length === 0) {
    tbody.innerHTML = '<tr><td colspan="7" class="px-4 py-6 text-center text-xs text-slate-400">ยังไม่มีผู้ใช้งาน</td></tr>';
    return;
  }

  // อัปเดต thead ให้มีคอลัมน์ครบ
  const thead = tbody.closest('table')?.querySelector('thead tr');
  if (thead) {
    thead.innerHTML = `
      <th class="px-5 py-3 text-xs font-bold text-slate-500 uppercase tracking-wide">Username</th>
      <th class="px-5 py-3 text-xs font-bold text-slate-500 uppercase tracking-wide">ชื่อ-นามสกุล</th>
      <th class="px-5 py-3 text-xs font-bold text-slate-500 uppercase tracking-wide">อีเมล</th>
      <th class="px-5 py-3 text-xs font-bold text-slate-500 uppercase tracking-wide">ตำแหน่ง</th>
      <th class="px-5 py-3 text-xs font-bold text-slate-500 uppercase tracking-wide">Role</th>
      <th class="px-5 py-3 text-xs font-bold text-slate-500 uppercase tracking-wide">หน่วยงาน</th>
      <th class="px-5 py-3 text-xs font-bold text-slate-500 uppercase tracking-wide text-center">จัดการ</th>
    `;
  }

  users.forEach(user => {
    const roleBadge = user.role === 'Admin'
      ? '<span class="bg-purple-100 text-purple-700 px-2 py-0.5 rounded text-[10px] font-bold">Admin</span>'
      : '<span class="bg-blue-100 text-blue-700 px-2 py-0.5 rounded text-[10px] font-bold">User</span>';
    const headBadge = (user.isHead === 'true' || user.isHead === true)
      ? '<span class="ml-1 bg-amber-100 text-amber-700 px-1.5 py-0.5 rounded text-[9px] font-bold">🔑 หัวหน้า</span>' : '';

    const tr = document.createElement('tr');
    tr.className = "hover:bg-slate-50 dark:hover:bg-slate-700/50 transition";
    tr.innerHTML = `
      <td class="px-4 py-2.5 font-mono text-sm font-bold text-slate-700 dark:text-slate-200">${user.username}</td>
      <td class="px-4 py-2.5 text-sm font-semibold text-slate-700 dark:text-slate-200">${user.fullname || user.fullName || '-'}</td>
      <td class="px-4 py-2.5 text-xs text-blue-500">${user.email || '-'}</td>
      <td class="px-4 py-2.5 text-xs text-slate-500">${user.position || '-'}</td>
      <td class="px-4 py-2.5">${roleBadge}${headBadge}</td>
      <td class="px-4 py-2.5 text-xs text-slate-500">${user.dept || '-'}</td>
      <td class="px-4 py-2.5 text-center whitespace-nowrap">
        <button onclick='openUserModal(${JSON.stringify(user)})' class="text-blue-500 hover:text-blue-700 mr-2" title="แก้ไข">
          <span class="material-symbols-outlined text-[18px]">edit</span>
        </button>
        <button onclick="deleteUser('${user.username}')" class="text-red-400 hover:text-red-600" title="ลบ">
          <span class="material-symbols-outlined text-[18px]">delete</span>
        </button>
      </td>
    `;
    tbody.appendChild(tr);
  });
}

function openUserModal(user = null) {
  document.getElementById('user-modal').classList.remove('hidden');
  const form = document.getElementById('user-form');
  form.reset();

  // toggle hint เมื่อ checkbox เปลี่ยน
  var cb   = document.getElementById('u-ishead');
  var hint = document.getElementById('u-ishead-hint');
  if (cb) cb.onchange = function() { if (hint) hint.classList.toggle('hidden', !cb.checked); };

  if (user) {
    document.getElementById('user-modal-title').innerText = "แก้ไขผู้ใช้งาน";
    document.getElementById('u-old-username').value = user.username;
    document.getElementById('u-username').value     = user.username;
    document.getElementById('u-fullname').value     = user.fullname || user.fullName || '';
    document.getElementById('u-email').value        = user.email    || '';
    document.getElementById('u-position').value     = user.position || '';
    document.getElementById('u-role').value         = user.role;
    document.getElementById('u-department').value   = user.dept || 'ทั้งหมด';
    document.getElementById('u-ishead').checked     = user.isHead === true || user.isHead === 'true' || user.isHead === '1';
    if (hint) hint.classList.toggle('hidden', !document.getElementById('u-ishead').checked);
    document.getElementById('u-password').required  = false;
  } else {
    document.getElementById('user-modal-title').innerText = "เพิ่มผู้ใช้งานใหม่";
    document.getElementById('u-old-username').value = "";
    document.getElementById('u-password').required  = true;
  }
}

function closeUserModal() {
  document.getElementById('user-modal').classList.add('hidden');
}

function saveUser() {
  const payload = {
    oldUsername: document.getElementById('u-old-username').value,
    username:    document.getElementById('u-username').value.trim(),
    password:    document.getElementById('u-password').value.trim(),
    fullname:    document.getElementById('u-fullname').value.trim(),
    email:       document.getElementById('u-email').value.trim(),
    position:    document.getElementById('u-position').value.trim(),
    role:        document.getElementById('u-role').value,
    dept:        document.getElementById('u-department').value,
    isHead:      document.getElementById('u-ishead').checked ? 'true' : 'false'
  };

  if (!payload.username) return alert("กรุณาระบุชื่อผู้ใช้งาน");
  if (!payload.fullname) return alert("กรุณาระบุชื่อ-นามสกุล");

  showLoader();
  google.script.run
    .withSuccessHandler(function(res) {
      if (res.success) {
        closeUserModal();
        loadAdminData();
        showToast("บันทึกข้อมูลผู้ใช้สำเร็จ", "success");
      } else {
        hideLoader();
        alert("เกิดข้อผิดพลาด: " + res.message);
      }
    })
    .withFailureHandler(err => { hideLoader(); alert(err.message); })
    .saveAdminUser(payload);
}

function deleteUser(username) {
  if (username === currentUser.username) return alert("ไม่สามารถลบบัญชีที่กำลังล็อกอินอยู่ได้");
  if (!confirm(`ยืนยันการลบผู้ใช้งาน "${username}"?`)) return;
  
  showLoader();
  google.script.run
    .withSuccessHandler(res => { loadAdminData(); })
    .withFailureHandler(err => { hideLoader(); alert(err.message); })
    .removeUser(username);
}
// ---------------- 3. จัดการประเภทเครื่องมือ (Categories) ----------------
function renderAdminCategories(categories) {
  const list = document.getElementById('admin-category-list');
  if (!list) return;
  if (!categories || categories.length === 0) {
    list.innerHTML = '<li class="text-center text-xs text-slate-400 py-6">ยังไม่มีข้อมูลประเภทเครื่องมือ — กดปุ่ม + เพื่อเพิ่ม</li>';
    return;
  }
  list.innerHTML = categories.map(cat =>
    `<li class="flex justify-between items-center px-3 py-2 bg-slate-50 dark:bg-slate-700/50 rounded-lg border border-slate-100 dark:border-slate-700 hover:bg-slate-100 dark:hover:bg-slate-700 transition group">
      <span class="text-sm text-slate-700 dark:text-slate-200 truncate flex-1">${cat}</span>
      <button onclick="deleteCategory('${cat.replace(/'/g,"\\'")}') " class="opacity-0 group-hover:opacity-100 ml-2 p-1 text-slate-300 hover:text-red-500 transition rounded" title="ลบ">
        <span class="material-symbols-outlined text-base leading-none">delete</span>
      </button>
    </li>`
  ).join('');
}

function addCategory() {
  const newCat = document.getElementById('admin-new-category').value.trim();
  if (!newCat) return;
  showLoader();
  google.script.run
    .withSuccessHandler(res => { 
      document.getElementById('admin-new-category').value = ''; 
      loadAdminData(); 
    })
    .withFailureHandler(err => { hideLoader(); alert(err.message); })
    .saveCategory(newCat); 
}

function deleteCategory(catName) {
  if (!confirm(`ต้องการลบประเภท "${catName}" ใช่หรือไม่?`)) return;
  showLoader();
  google.script.run
    .withSuccessHandler(res => { loadAdminData(); })
    .withFailureHandler(err => { hideLoader(); alert(err.message); })
    .removeCategory(catName);
}

  // ==========================================
  // Calibration Pages: Panel Open/Close + History + Search
  // ==========================================
  const CAL_PAGE_CONFIG = {
    thermo:      { prefix: 'th',  formId: 'thermo-form',     loadFn: 'loadEquipmentsForThermo',    title: 'Thermometer' },
    pipette:     { prefix: 'pi',  formId: 'pipette-form',    loadFn: 'loadEquipmentsForPipette',   title: 'Pipette' },
    centrifuge:  { prefix: 'ce',  formId: 'centrifuge-form', loadFn: 'loadEquipmentsForCentrifuge',title: 'Centrifuge' },
    balance:     { prefix: 'ba',  formId: 'balance-form',    loadFn: 'loadEquipmentsForBalance',   title: 'Balance' },
    thermohygro: { prefix: 'thh', formId: 'thermohygro-form',loadFn: 'loadEquipmentsForThermoHygro',title: 'Thermo-Hygrometer' },
    phmeter:     { prefix: 'ph',  formId: 'phmeter-form',    loadFn: 'loadEquipmentsForPHMeter',   title: 'pH Meter' },
    oven:        { prefix: 'ov',  formId: 'oven-form',       loadFn: 'loadEquipmentsForOven',      title: 'Oven/Incubator' },
    ruler:       { prefix: 'ru',  formId: 'ruler-form',      loadFn: 'loadEquipmentsForRuler',     title: 'Ruler' },
    bscfume:     { prefix: 'bsc', formId: 'bscfume-form',    loadFn: 'loadEquipmentsForBSCFume',   title: 'BSC/Fume Hood' },
    pcr:         { prefix: 'pcr', formId: 'pcr-form',        loadFn: 'loadEquipmentsForPCR',       title: 'PCR' },
  };

  window._calSheetMap = window._calSheetMap || {};
  const CAL_SHEET_MAP = {
    thermo:      'ThermometerCalibrations',
    pipette:     'PipetteCalibrations',
    centrifuge:  'CentrifugeCalibrations',
    balance:     'BalanceCalibrations',
    thermohygro: 'ThermoHygroCalibrations',
    phmeter:     'PHMeterCalibrations',
    oven:        'OvenCalibrations',
    ruler:       'RulerCalibrations',
    bscfume:     'BSCFumeCalibrations',
    pcr:         'PCRCalibrations',
  };
  window._calSheetMap = CAL_SHEET_MAP;

  function openCalFormPanel(pageId) {
    const cfg = CAL_PAGE_CONFIG[pageId];
    if (!cfg) return;
    const panel = document.getElementById('cal-panel-' + pageId);
    if (!panel) return;
    _clearCalEditMode(pageId); // เปิดฟอร์มใหม่ = ออกจากโหมดแก้ไขเสมอ

    // ซ่อน history card
    const histCard = document.getElementById(cfg.prefix + '-history-card');
    if (histCard) histCard.classList.add('hidden');

    // รีเซ็ตฟอร์ม
    const form = document.getElementById(cfg.formId);
    if (form) form.reset();

    panel.classList.remove('hidden');
    panel.scrollIntoView({ behavior: 'smooth', block: 'start' });
    try { if (typeof window[cfg.loadFn] === 'function') window[cfg.loadFn](); } catch(e) {}
  }

  function closeCalFormPanel(pageId) {
    _clearCalEditMode(pageId);
    const cfg = CAL_PAGE_CONFIG[pageId];
    const panel = document.getElementById('cal-panel-' + pageId);
    if (panel) panel.classList.add('hidden');

    // แสดง history card กลับมา
    if (cfg) {
      const histCard = document.getElementById(cfg.prefix + '-history-card');
      if (histCard) histCard.classList.remove('hidden');
      const form = document.getElementById(cfg.formId);
      if (form) form.reset();
    }
  }

   // ดึงประวัติจาก GAS — dynamic headers
  function loadCalHistory(pageId) {
    var cfg = CAL_PAGE_CONFIG[pageId];
    if (!cfg) return;
    var tbody = document.getElementById(cfg.prefix + '-history-tbody');
    var thead = document.getElementById(cfg.prefix + '-history-thead');
    var countEl = document.getElementById(cfg.prefix + '-history-count');
    if (!tbody) return;

    tbody.innerHTML = '<tr><td colspan="20" class="px-4 py-6 text-center text-slate-400 text-xs">กำลังโหลดประวัติ...</td></tr>';

    var _loadTimeout = setTimeout(function() {
      if (tbody.innerHTML.indexOf('กำลังโหลด') !== -1) {
        tbody.innerHTML = '<tr><td colspan="20" class="py-6 text-center text-slate-400 text-xs">'
          + '⏱ โหลดนานเกินไป <button onclick="loadCalHistory(\'' + pageId + '\')" class="ml-2 px-2 py-0.5 bg-primary-100 text-primary-700 rounded text-xs font-bold">🔄 ลองใหม่</button>'
          + '</td></tr>';
      }
    }, 20000);

    var sheetName = CAL_SHEET_MAP[pageId];
    if (!sheetName) { clearTimeout(_loadTimeout); tbody.innerHTML = '<tr><td colspan="20" class="text-center py-6 text-slate-400">ไม่พบชีตข้อมูล</td></tr>'; return; }

    google.script.run
      .withSuccessHandler(function(rows) {
        clearTimeout(_loadTimeout);
        if (rows && rows.error) {
          tbody.innerHTML = '<tr><td colspan="20" class="px-4 py-8 text-center text-red-400 text-sm">❌ ' + rows.error + '</td></tr>';
          if (countEl) countEl.textContent = 'เกิดข้อผิดพลาด';
          return;
        }
        if (!rows || rows.length < 1) {
          tbody.innerHTML = '<tr><td colspan="20" class="px-4 py-10 text-center text-slate-400"><span class="material-symbols-outlined text-3xl block mb-2 text-slate-300">history</span>ยังไม่มีประวัติ</td></tr>';
          if (countEl) countEl.textContent = 'ไม่มีข้อมูล';
          return;
        }

        var headers  = rows[0];
        var rowColIdx = headers.indexOf('_row');
        // กรองแถวว่าง — ไม่นับคอลัมน์ _row เพราะมีค่าทุกแถว
        var data = rows.slice(1).filter(function(r){
          return r && r.length > 0 && r.some(function(c, ci){
            return ci !== rowColIdx && c !== '' && c !== null && c !== undefined;
          });
        });

        var HIDDEN_COLS = ['ไฟล์ใบรับรอง (PDF)', 'หมายเหตุการแก้ไข', '_row'];
        var visHeaders  = headers.filter(function(h){ return HIDDEN_COLS.indexOf(h) === -1; });
        if (countEl) countEl.textContent = 'พบ ' + data.length + ' รายการ (ล่าสุดขึ้นก่อน) · ' + visHeaders.length + ' คอลัมน์';
        data.reverse();

        var colorMap = { thermo:'rose', pipette:'blue', centrifuge:'purple', balance:'amber', thermohygro:'teal', phmeter:'blue', oven:'orange', ruler:'green', bscfume:'slate', pcr:'indigo' };
        var color = colorMap[pageId] || 'primary';

        if (thead) {
          thead.innerHTML = '<tr>' + visHeaders.map(function(h){
            return '<th class="px-3 py-3 text-[10px] font-bold text-' + color + '-600 dark:text-' + color + '-400 uppercase tracking-wide whitespace-nowrap border-b dark:border-slate-700 bg-slate-50 dark:bg-slate-700/50">' + h + '</th>';
          }).join('') + '<th class="px-3 py-3 text-[10px] font-bold text-' + color + '-600 dark:text-' + color + '-400 uppercase border-b dark:border-slate-700 bg-slate-50 dark:bg-slate-700/50 text-center sticky right-0">จัดการ</th></tr>';
        }

        if (!window._calHistoryData) window._calHistoryData = {};
        window._calHistoryData[pageId] = { headers: headers, data: data };

        var pdfColIdx = headers.indexOf('ไฟล์ใบรับรอง (PDF)');
        var apColIdx  = headers.indexOf('สถานะการอนุมัติ');
        var apByIdx   = headers.indexOf('ผู้อนุมัติ');
        var apAtIdx   = headers.indexOf('วันที่อนุมัติ');
        var addedIdx  = headers.indexOf('ผู้บันทึก');

        // สิทธิ์อนุมัติ — คำนวณครั้งเดียว
        var _pos = String(currentUser ? currentUser.position || '' : '').toLowerCase();
        var isHead = !!currentUser && (
          currentUser.isHead === 'true' ||
          currentUser.isHead === true   ||
          currentUser.isHead === '1'    ||
          currentUser.role === 'Admin'  ||
          _pos.indexOf('หัวหน้า') !== -1
        );
        var myName = _getDisplayName();

        tbody.innerHTML = data.map(function(row, idx) {
          var cells = headers.map(function(h, hi) {
            if (HIDDEN_COLS.indexOf(h) !== -1) return '';
            var val = (row[hi] !== undefined && row[hi] !== null && row[hi] !== '') ? row[hi] : '-';
            var valStr = String(val);
            var isResultCol = ['ผลการตรวจสอบ','Evaluation','ผล','สรุปผล'].indexOf(h) !== -1;
            if (isResultCol) {
              var lines = valStr.split('\n');
              var summary = ((lines.find(function(l){ return l.indexOf('สรุป') !== -1; }) || lines[0] || valStr)).replace('สรุป:','').trim();
              if (valStr.indexOf('ยอมรับ') !== -1 || valStr.indexOf('Pass') !== -1 || valStr.indexOf('ผ่าน') !== -1)
                return '<td class="px-3 py-2.5"><span class="px-2 py-0.5 bg-green-100 text-green-700 dark:bg-green-900/30 dark:text-green-400 rounded-full text-[10px] font-bold whitespace-nowrap">' + summary + '</span></td>';
              if (valStr.indexOf('ปฏิเสธ') !== -1 || valStr.indexOf('Fail') !== -1 || valStr.indexOf('ไม่ผ่าน') !== -1)
                return '<td class="px-3 py-2.5"><span class="px-2 py-0.5 bg-red-100 text-red-700 dark:bg-red-900/30 dark:text-red-400 rounded-full text-[10px] font-bold whitespace-nowrap">' + summary + '</span></td>';
              return '<td class="px-3 py-2.5"><span class="px-2 py-0.5 bg-amber-100 text-amber-700 rounded-full text-[10px] font-bold whitespace-nowrap">' + valStr + '</span></td>';
            }
            var display = valStr.length > 35 ? valStr.substring(0, 33) + '…' : valStr;
            return '<td class="px-3 py-2.5 text-slate-600 dark:text-slate-300 whitespace-nowrap text-xs" title="' + valStr.replace(/"/g,"'") + '">' + display + '</td>';
          }).join('');

          var certUrl = (pdfColIdx !== -1 && row[pdfColIdx]) ? String(row[pdfColIdx]).trim() : '';
          var viewBtn = certUrl && certUrl.indexOf('http') === 0
            ? '<a href="' + certUrl + '" target="_blank" title="ดูใบรับรอง" class="inline-flex items-center justify-center w-7 h-7 rounded-lg bg-blue-50 hover:bg-blue-100 text-blue-500 hover:text-blue-700 transition border border-blue-200"><span class="material-symbols-outlined text-base leading-none">visibility</span></a>'
            : '<span class="inline-flex items-center justify-center w-7 h-7 rounded-lg bg-slate-100 text-slate-300 border border-slate-200"><span class="material-symbols-outlined text-base leading-none">visibility_off</span></span>';

          var apStatus = apColIdx >= 0 ? String(row[apColIdx]||'') : '';
          var apBy     = apByIdx  >= 0 ? String(row[apByIdx]||'')  : '';
          var apAt     = apAtIdx  >= 0 ? String(row[apAtIdx]||'')  : '';

          var approveBtn = '';
          if (apStatus === 'อนุมัติแล้ว') {
            approveBtn = '<span class="inline-flex items-center gap-1 px-2 py-0.5 bg-green-100 text-green-700 rounded-full text-[9px] font-bold" title="' + apBy + ' ' + apAt + '"><span class="material-symbols-outlined text-[12px]">verified</span> อนุมัติแล้ว</span>';
          } else {
            if (apStatus === 'ไม่อนุมัติ') approveBtn += '<span class="px-2 py-0.5 bg-red-100 text-red-700 rounded-full text-[9px] font-bold">ส่งกลับแก้ไข</span>';
            approveBtn += isHead
              ? '<button onclick="_approveCalRow(\'' + pageId + '\',' + idx + ')" class="inline-flex items-center gap-1 px-2 py-0.5 bg-amber-100 hover:bg-amber-200 text-amber-700 rounded text-[10px] font-bold border border-amber-300 transition"><span class="material-symbols-outlined text-[12px]">approval</span> อนุมัติ</button>'
              : (apStatus ? '' : '<span class="text-[9px] text-slate-300">รออนุมัติ</span>');
          }

          var recorder = addedIdx >= 0 ? String(row[addedIdx] || '').trim() : '';
          var isOwner  = !!recorder && (recorder === myName || (currentUser && recorder === currentUser.username));
          var editBtn  = (apStatus !== 'อนุมัติแล้ว' && (isOwner || isHead))
            ? '<button onclick="_editCalRow(\'' + pageId + '\',' + idx + ')" title="แก้ไขข้อมูล (ก่อนอนุมัติ)" class="inline-flex items-center justify-center w-7 h-7 rounded-lg bg-amber-50 hover:bg-amber-100 text-amber-600 border border-amber-200 transition"><span class="material-symbols-outlined text-base leading-none">edit</span></button>'
            : '';

          var actionCell = '<td class="px-3 py-2.5 text-center sticky right-0 bg-white dark:bg-slate-800">'
            + '<div class="flex items-center justify-center gap-1.5 flex-wrap">'
            + viewBtn
            + '<button onclick="generateRowPdf(\'' + pageId + '\',' + idx + ')" title="พิมพ์แบบฟอร์ม" class="inline-flex items-center justify-center w-7 h-7 rounded-lg bg-red-50 hover:bg-red-100 text-red-500 hover:text-red-700 transition border border-red-200"><span class="material-symbols-outlined text-base leading-none">print</span></button>'
            + editBtn
            + approveBtn
            + '</div></td>';

          var rowClass = idx % 2 === 0 ? '' : 'bg-slate-50/50 dark:bg-slate-700/20';
          return '<tr class="' + rowClass + ' hover:bg-primary-50/30 dark:hover:bg-slate-700/30 transition">' + cells + actionCell + '</tr>';
        }).join('');
      })
      .withFailureHandler(function(err) {
        clearTimeout(_loadTimeout);
        tbody.innerHTML = '<tr><td colspan="20" class="text-center py-6 text-red-400 text-xs">โหลดประวัติล้มเหลว: ' + (err.message || err) + '</td></tr>';
        if (countEl) countEl.textContent = 'เกิดข้อผิดพลาด';
      })
      .getSheetData(sheetName, currentUser ? currentUser.dept : '', currentUser ? currentUser.role : '');
  }

  function filterCalPageSearch(pageId, query) {
    const cfg = CAL_PAGE_CONFIG[pageId];
    if (!cfg) return;
    const q = query.trim().toLowerCase();
    const drop = document.getElementById(cfg.prefix + '-page-search-drop');
    if (!drop) return;
    if (!q || !cachedEquipments || !cachedEquipments.length) { drop.classList.add('hidden'); return; }
    const matches = cachedEquipments.filter(item =>
      (item['ชื่อเครื่องมือ'] || '').toLowerCase().includes(q) ||
      (item['รหัสครุภัณฑ์'] || '').toLowerCase().includes(q)
    ).slice(0, 8);
    if (!matches.length) {
      drop.innerHTML = '<div class="px-4 py-2 text-sm text-slate-400">ไม่พบรายการ</div>';
      drop.classList.remove('hidden'); return;
    }
    drop.innerHTML = matches.map(item =>
      '<div class="px-4 py-2.5 text-sm hover:bg-primary-50 dark:hover:bg-slate-700 cursor-pointer transition border-b dark:border-slate-700 last:border-0" onclick="document.getElementById(\'' + cfg.prefix + '-page-search\').value=\'\'; document.getElementById(\'' + cfg.prefix + '-page-search-drop\').classList.add(\'hidden\');">'
      + '<div class="font-bold text-slate-800 dark:text-slate-100">' + (item['ชื่อเครื่องมือ'] || '-') + '</div>'
      + '<div class="text-xs text-slate-400">' + (item['รหัสครุภัณฑ์'] || '') + '</div></div>'
    ).join('');
    drop.classList.remove('hidden');
  }


  var CAL_PAGE_TITLES = {
    thermo:      'แบบบันทึกผลการสอบเทียบ เครื่องวัดอุณหภูมิ (Thermometer)',
    pipette:     'แบบบันทึกผลการตรวจรับรอง Automatic Pipette',
    centrifuge:  'แบบบันทึกผลการสอบเทียบ เครื่องปั่นเหวี่ยง (Centrifuge)',
    balance:     'แบบบันทึกผลการสอบเทียบ เครื่องชั่ง (Balance)',
    thermohygro: 'แบบบันทึกผลการสอบเทียบ Thermo-Hygrometer',
    phmeter:     'แบบบันทึกผลการสอบเทียบ pH Meter',
    oven:        'แบบบันทึกผลการสอบเทียบ Oven / Incubator / Autoclave',
    ruler:       'แบบบันทึกผลการสอบเทียบ ไม้บรรทัด (Ruler)',
    bscfume:     'แบบบันทึกผลการสอบเทียบ BSC / Laminar air flow / Fume Hood',
    pcr:         'แบบบันทึกผลการสอบเทียบ เครื่อง PCR (Thermal Cycler)',
  };

  // Column definitions: key ตรงกับ header จริงในแต่ละชีต (จาก saveToSheetWithPdf ใน GAS)
  var CAL_COL_DEFS = {
    thermo: {
      headers: ['Testing Point (°C)', 'Temp. Standard (°C)', 'Reading (°C)', 'Uncertainty (±°C)', 'Reading ± Uncertainty (°C)', 'ช่วงควบคุม', 'ผล'],
      keys:    ['Testing Point (°C)', 'Temperature Standard (°C)', 'Thermometer Reading (°C)', 'Uncertainty (± °C)', 'Reading ± Uncertainty (°C)', 'ช่วงควบคุม (°C)', 'ผลการตรวจสอบ']
    },
    pipette: {
      headers: ['Test Vol. (µl)', 'Mean Vol. (µl)', 'Uncertainty (± µl)', 'Mean Vol. ± Uncertainty (µl)', 'ช่วงควบคุม (µl)', 'ผล'],
      keys:    ['Test Volume (ul.)', 'Mean Volume (ul.)', 'Uncertainty (± ul.)', 'Mean Volume ± Uncertainty (ul.)', 'ช่วงควบคุม (ul.)', 'ผลการตรวจสอบ']
    },
    centrifuge: {
      headers: ['Function', 'Setting', 'Std Reading', 'Uncertainty', 'Std ± U', 'ช่วงควบคุม', 'ผล'],
      keys:    ['Function', 'Setting', 'Std Reading', 'Uncertainty', 'Std ± U', 'ช่วงควบคุม', 'ผลการตรวจสอบ'],
      pos:     [5, 6, 7, 8, 9, 10, 11]   // ตำแหน่งคอลัมน์ในชีต (ตามลำดับที่บันทึก) — ใช้แทนการเดาชื่อหัวคอลัมน์
    },
    balance: {
      headers: ['Nominal (g)', 'Display (g)', 'Uncertainty', 'Display ± U', 'ช่วงควบคุม', 'ผล'],
      keys:    ['Nominal (g)', 'Display (g)', 'Uncertainty', 'Display ± U', 'ช่วงควบคุม', 'ผลการตรวจสอบ'],
      pos:     [5, 6, 7, 8, 9, 10]   // ตำแหน่งคอลัมน์ในชีต (ตามลำดับที่บันทึก) — ใช้แทนการเดาชื่อหัวคอลัมน์
    },
    thermohygro: {
      headers: ['Function', 'Std Reading (°C /%)', 'Inst Reading (°C /%)', 'Uncertainty (°C /%)', 'Inst Reading ± Uncertainty (°C /%)', 'ช่วงควบคุม (°C /%)', 'ผล'],
      keys:    ['Function', 'Standard reading (°C /%)', 'Instrument reading (°C /%)', 'Uncertainty (°C /%)', 'Instrument reading ± Uncertainty (°C /%)', 'ช่วงควบคุม (°C /%)', 'ผลการตรวจสอบ']
    },
    phmeter: {
      headers: ['Buffer', 'Std pH', 'Inst Reading', 'Uncertainty', 'Inst ± U', 'ช่วงควบคุม', 'ผล'],
      keys:    ['Function', 'Std Reading', 'Inst Reading', 'Uncertainty', 'Inst ± U', 'ช่วงควบคุม', 'ผลการตรวจสอบ'],
      pos:     [5, 6, 7, 8, 9, 10, 11]   // ตำแหน่งคอลัมน์ในชีต (ตามลำดับที่บันทึก) — ใช้แทนการเดาชื่อหัวคอลัมน์
    },
    oven: {
      headers: ['Thermo Reading (°C)', 'Temp Standard (°C)', 'Uncertainty (±°C)', 'Temp Standard ± Uncertainty (°C)', 'ช่วงควบคุม (°C)', 'ผล'],
      keys:    ['Thermometer reading (°C)', 'Temperature standard (°C)', 'Uncertainty (±°C)', 'Temperature reading ± Uncertainty (°C)', 'ช่วงควบคุม (°C)', 'ผลการตรวจสอบ']
    },
    ruler: {
      headers: ['Nominal (mm)', 'Display (mm)', 'Uncertainty', 'Display ± U', 'ช่วงควบคุม', 'ผล'],
      keys:    ['Nominal (mm)', 'Display (mm)', 'Uncertainty', 'Display ± U', 'ช่วงควบคุม', 'ผลการตรวจสอบ'],
      pos:     [5, 6, 7, 8, 9, 10]   // ตำแหน่งคอลัมน์ในชีต (ตามลำดับที่บันทึก) — ใช้แทนการเดาชื่อหัวคอลัมน์
    },
    pcr: {
      headers: ['Testing Point (°C)', 'Mean (°C)', 'Min (°C)', 'Max (°C)', 'Uniformity', 'Accuracy', 'ผล'],
      keys:    ['Testing Point (°C)', 'Mean (°C)', 'Min (°C)', 'Max (°C)', 'Uniformity', 'Accuracy', 'ผลการตรวจสอบ'],
      pos:     [5, 6, 7, 8, 9, 10, 11]   // ตำแหน่งคอลัมน์ในชีต (ตามลำดับที่บันทึก) — ใช้แทนการเดาชื่อหัวคอลัมน์
    },
  };

  function generateRowPdf(pageId, rowIdx) {
    var store = window._calHistoryData && window._calHistoryData[pageId];
    if (!store) { alert('ไม่พบข้อมูล กรุณารีเฟรชหน้า'); return; }
    var headers = store.headers;
    var row = store.data[rowIdx];
    if (!row) return;

    // map header → value (ใช้ชื่อ header จริงจากชีต)
    var d = {};
    headers.forEach(function(h, i) {
      d[h] = (row[i] !== undefined && row[i] !== null) ? String(row[i]) : '';
    });

    // helper: ค้นหา value จาก header ที่ใกล้เคียง (case-insensitive, trim)
    function val(key) {
      if (d[key] !== undefined) return d[key] || '-';
      // fuzzy match
      var k = key.toLowerCase().trim();
      for (var hdr in d) {
        if (hdr.toLowerCase().trim() === k) return d[hdr] || '-';
      }
      // partial match
      for (var hdr in d) {
        if (hdr.toLowerCase().indexOf(k) !== -1 || k.indexOf(hdr.toLowerCase()) !== -1) return d[hdr] || '-';
      }
      return '-';
    }

    var title = CAL_PAGE_TITLES[pageId] || 'แบบบันทึกผลการสอบเทียบ';
    var colDef = CAL_COL_DEFS[pageId];
    // ✅ ดึงค่าตามตำแหน่งคอลัมน์ (ชื่อหัวคอลัมน์ในชีตไม่ตรงกับชื่อใน PDF เช่น 'Nominal Test Value (mm.)')
    if (colDef && colDef.pos) {
      colDef.keys.forEach(function (k, i) {
        var v = row[colDef.pos[i]];
        d[k] = (v !== undefined && v !== null) ? String(v) : '';
      });
    }

    // ถ้าไม่มี colDef ให้ใช้ headers จริงทั้งหมด
    var thHeaders, dataKeys;
    if (colDef) {
      thHeaders = colDef.headers;
      dataKeys  = colDef.keys;
    } else {
      thHeaders = headers;
      dataKeys  = headers;
    }

    // =========================================================
    // สำหรับ thermohygro: normalize ข้อมูลที่อาจเก็บแบบ
    //   (A) multiline  → "Temperature\nHumidity\n..."
    //   (B) concatenated → "TemperatureHumidity..."  
    // ให้เป็น array ของแถวที่แยกชัดเจนก่อน buildTableRows
    // =========================================================
    if (pageId === 'thermohygro') {
      // ensure aliases exist for keys ที่อาจ lowercase ไม่ match
      for (var _hk in d) {
        var _lk = _hk.toLowerCase().replace(/\s+/g,' ').trim();
        if (_lk.indexOf('ช่วงควบคุม') !== -1 && !d['ช่วงควบคุม (°C /%)']) {
          d['ช่วงควบคุม (°C /%)'] = d[_hk];
        }
        if ((_lk.indexOf('ผลการตรวจสอบ') !== -1 || _lk === 'ผล') && !d['ผลการตรวจสอบ']) {
          d['ผลการตรวจสอบ'] = d[_hk];
        }
      }

      // ตรวจสอบว่า Function column ใช้ \n แยกหรือเปล่า
      var _funcRaw = val('Function');
      var _funcHasNewline = _funcRaw !== '-' && _funcRaw.indexOf('\n') !== -1;

      // ถ้าไม่มี \n = concatenated → ต้องแยก โดย parse จาก pattern Temperature/Humidity
      if (!_funcHasNewline && _funcRaw !== '-') {
        // แยก Function โดย match "Temperature" และ "Humidity"
        var _funcParts = [];
        var _funcStr = _funcRaw;
        var _funcRegex = /(Temperature|Humidity)/g;
        var _fmatch;
        while ((_fmatch = _funcRegex.exec(_funcStr)) !== null) {
          _funcParts.push(_fmatch[1]);
        }
        if (_funcParts.length > 1) {
          var _nRows = _funcParts.length;
          // helper: split numeric string into N parts
          function _splitNumStr(str, n) {
            if (!str || str === '-') return Array(n).fill('-');
            // ลอง split ด้วย newline ก่อน
            var _byNL = str.split('\n').filter(function(s){ return s.trim() !== ''; });
            if (_byNL.length === n) return _byNL.map(function(s){ return s.trim(); });
            // สำหรับ ranges เช่น "15 - 3215 - 3545 - 8545 - 85"
            // หา pattern "ตัวเลข - ตัวเลข" ซ้ำๆ (รองรับ decimal)
            var _rangeMatch = str.match(/[\d.]+\s*-\s*[\d.]+/g);
            if (_rangeMatch && _rangeMatch.length === n) {
              return _rangeMatch.map(function(s){ return s.trim(); });
            }
            // สำหรับ "23.57 - 24.4531.58 - 32.46..." → ต้อง split ที่จุดที่ตัวเลขแรกจบและตัวเลขใหม่เริ่ม
            // ใช้ regex แบบ lookahead: split ที่ตัวเลขสุดท้ายของ range ตามด้วยตัวเลขของ range ถัดไป
            // Pattern: (digits - digits)(digits - digits)... โดยไม่มีตัวคั่น
            var _concatRangeReg = /(-?[\d.]+\s*-\s*-?[\d.]+)/g;
            var _concatMatches = [];
            var _cm;
            // ลองใช้ regex ที่ match range ซ้ำๆ บน string
            var _workStr = str;
            // แทนที่ "XX.XXYY.YY" → "XX.XX YY.YY" ก่อน แล้ว match
            // Split แบบ greedy: match X.XX - Y.YY ทีละ range
            var _splitReg = /(-?[\d]+\.?[\d]*)\s*-\s*(-?[\d]+\.?[\d]*)/g;
            while ((_cm = _splitReg.exec(_workStr)) !== null) {
              _concatMatches.push(_cm[1].trim() + ' - ' + _cm[2].trim());
            }
            if (_concatMatches.length === n) return _concatMatches;
            // ตัวเลขเดี่ยวๆ
            var _numMatch = str.match(/-?[\d.]+/g);
            if (_numMatch && _numMatch.length === n) return _numMatch;
            // fallback: ใช้ค่าเดิม repeat
            return Array(n).fill(str);
          }

          // helper: split result string (ยอมรับ/ปฏิเสธ แยกด้วย \n)
          function _splitResultStr(str, n) {
            if (!str || str === '-') return Array(n).fill('-');
            var _lines = str.split('\n').filter(function(l){
              var t = l.trim();
              return t !== '' && t.indexOf('สรุป:') === -1;
            });
            if (_lines.length === n) return _lines;
            if (_lines.length > n) return _lines.slice(0, n);
            return Array(n).fill(str.split('\n')[0] || str);
          }

          // สร้าง normalized multiline values
          d['Function']                                = _funcParts.join('\n');
          d['Standard reading (°C /%)']                = _splitNumStr(val('Standard reading (°C /%)'), _nRows).join('\n');
          d['Instrument reading (°C /%)']              = _splitNumStr(val('Instrument reading (°C /%)'), _nRows).join('\n');
          d['Uncertainty (°C /%)']                     = _splitNumStr(val('Uncertainty (°C /%)'), _nRows).join('\n');
          d['Instrument reading ± Uncertainty (°C /%)'] = _splitNumStr(val('Instrument reading ± Uncertainty (°C /%)'), _nRows).join('\n');
          d['ช่วงควบคุม (°C /%)']                    = _splitNumStr(val('ช่วงควบคุม (°C /%)'), _nRows).join('\n');
          d['ผลการตรวจสอบ']                           = _splitResultStr(val('ผลการตรวจสอบ'), _nRows).join('\n');
        }
      }
    }

    // สร้างแถวตาราง — แต่ละ key อาจมีค่าหลายบรรทัด (join ด้วย \n)
    function buildTableRows(thH, dKeys) {
      var arrays = dKeys.map(function(k, ki) {
        var v = val(k);
        if (v === '-') return ['-'];
        var lines = v.split('\n');
        // สำหรับคอลัมน์ผล (คอลัมน์สุดท้าย) ให้กรอง blank lines และ สรุป: line ออก
        if (ki === dKeys.length - 1) {
          lines = lines.filter(function(l) {
            var t = l.trim();
            return t !== '' && t.indexOf('สรุป:') === -1;
          });
        }
        return lines.length ? lines : ['-'];
      });
      // จำกัด maxLen ด้วย column แรก (Function) เพื่อป้องกัน rows เกิน
      var maxLen = arrays[0].length;
      // ถ้า column อื่นยาวกว่า Function ก็จำกัดด้วย Function length
      // (ยกเว้นกรณี Function = ['-'] ซึ่งหมายถึงไม่มีข้อมูล)
      if (maxLen === 1 && arrays[0][0] === '-') {
        maxLen = Math.max.apply(null, arrays.map(function(a){ return a.length; }));
      }
      var html = '';
      for (var i = 0; i < maxLen; i++) {
        var lastVal = (arrays[arrays.length-1][i] || '').trim();
        var cls = '';
        if (lastVal.indexOf('ยอมรับ') !== -1 || lastVal.indexOf('ผ่าน') !== -1) cls = 'pass';
        else if (lastVal.indexOf('ปฏิเสธ') !== -1 || lastVal.indexOf('ไม่ผ่าน') !== -1) cls = 'fail';
        html += '<tr class="' + cls + '">';
        arrays.forEach(function(a, ai) {
          var cell = (a[i] !== undefined && a[i].trim() !== '') ? a[i].trim() : (i === 0 ? '-' : '');
          // badge สำหรับคอลัมน์ผล
          var isLastCol = ai === arrays.length - 1;
          if (isLastCol && cell) {
            if (cell.indexOf('ยอมรับ') !== -1 || cell.indexOf('ผ่าน') !== -1)
              cell = '<span style="color:#16a34a;font-weight:700">' + cell.replace('สรุป:','').trim() + '</span>';
            else if (cell.indexOf('ปฏิเสธ') !== -1 || cell.indexOf('ไม่ผ่าน') !== -1)
              cell = '<span style="color:#dc2626;font-weight:700">' + cell.replace('สรุป:','').trim() + '</span>';
          }
          html += '<td>' + cell + '</td>';
        });
        html += '</tr>';
      }
      return html;
    }

    var tableBodyHtml = buildTableRows(thHeaders, dataKeys);
    var theadHtml = '<thead><tr>' + thHeaders.map(function(h){ return '<th>' + h + '</th>'; }).join('') + '</tr></thead>';
    var tableHtml = '<table>' + theadHtml + '<tbody>' + tableBodyHtml + '</tbody></table>';

    // ✅ BSC / Laminar air flow / Fume Hood: แสดงผลเป็นรายการทีละบรรทัด (รายการทดสอบ | ผล | เกณฑ์)
    if (pageId === 'bscfume') {
      var _bfItems = [
        // [รายการ, header ผล, header เกณฑ์, ตำแหน่งคอลัมน์ผล (สำรองถ้าชื่อหัวคอลัมน์ในชีตถูกเปลี่ยน)]
        ['1. Leakage of Main Filter',    'Result 1.1 (Main)',    'Spec 1.1', 6],
        ['2. Leakage of Exhaust Filter', 'Result 1.2 (Exhaust)', 'Spec 1.2', 8],
        ['3. Air Velocity (Down Flow)',  'Result 2 (Velocity)',  'Spec 2',   10],
        ['4. Percentage (Down Flow)',    'Result 3 (%)',         'Spec 3',   12],
        ['5. UV Radiation Test',         'Result 4 (UV)',        'Spec 4',   14],
        ['6. Illumination Test',         'Result 5 (Illum)',     'Spec 5',   16]
      ];
      function _bfGet(k, pos) {
        var x = d[k];
        if (x === undefined) x = row[pos];
        return (x === undefined || x === null || String(x).trim() === '') ? '-' : String(x).replace(/\n/g, '<br>');
      }
      tableHtml = '<table><thead><tr>'
        + '<th style="width:45%">รายการทดสอบ (Test Item)</th><th>ผลการทดสอบ (Result)</th><th>เกณฑ์การผ่าน (Pass Spec.)</th>'
        + '</tr></thead><tbody>'
        + _bfItems.map(function (it) {
            return '<tr><td style="text-align:left;font-weight:600">' + it[0] + '</td>'
              + '<td style="text-align:center">' + _bfGet(it[1], it[3]) + '</td>'
              + '<td style="text-align:center">' + _bfGet(it[2], it[3] + 1) + '</td></tr>';
          }).join('')
        + '</tbody></table>';
    }

    // ผลสรุป
    var resultKey = null;
    ['ผลการตรวจสอบ','Evaluation','ผล','สรุปผล'].forEach(function(k){
      if (!resultKey && d[k] !== undefined) resultKey = k;
    });
    var resultVal = resultKey ? (d[resultKey] || '') : '';
    var rLines = resultVal.split('\n');
    var summaryLine = '';
    for (var ri = 0; ri < rLines.length; ri++) {
      if (rLines[ri].indexOf('สรุป') !== -1) { summaryLine = rLines[ri]; break; }
    }
    if (!summaryLine) summaryLine = rLines[rLines.length-1] || '';
    summaryLine = summaryLine.replace('สรุป:','').trim();

    var isPass = summaryLine.indexOf('ยอมรับ') !== -1 || summaryLine.indexOf('ผ่าน') !== -1 || summaryLine.indexOf('Pass') !== -1;
    var isFail = summaryLine.indexOf('ปฏิเสธ') !== -1 || summaryLine.indexOf('ไม่ผ่าน') !== -1 || summaryLine.indexOf('Fail') !== -1;
    var resColor = isPass ? '#16a34a' : isFail ? '#dc2626' : '#d97706';
    var resBg    = isPass ? '#f0fdf4' : isFail ? '#fef2f2' : '#fffbeb';

    // ข้อมูลหัวกระดาษ — ค้นหาจาก d ด้วย fuzzy
    var eqCode  = val('รหัสครุภัณฑ์/ยี่ห้อ') !== '-' ? val('รหัสครุภัณฑ์/ยี่ห้อ') : val('SERIAL NO.');
    var calDate = val('วันที่สอบเทียบ') !== '-' ? val('วันที่สอบเทียบ') : val('CAL DATE');
    var certNo  = val('CERTIFICATE NUMBER') !== '-' ? val('CERTIFICATE NUMBER') : val('Certificate No.');
    var addedBy = val('ผู้บันทึก');

    // Extra info ตามประเภท
    var extraRows = '';
    if (pageId === 'thermo')    extraRows = '<div class="info-row"><span class="lbl">ประเภท Thermometer:</span><span class="val">' + val('ประเภท Thermometer') + '</span></div>';
    if (pageId === 'oven')      extraRows = '<div class="info-row"><span class="lbl">ประเภทเครื่อง:</span><span class="val">' + val('ประเภทเครื่อง') + '</span></div>';
    if (pageId === 'pipette')   extraRows = '<div class="info-row"><span class="lbl">Capacity (µl):</span><span class="val">' + val('Capacity') + '</span></div>';
    if (pageId === 'centrifuge')extraRows = '<div class="info-row"><span class="lbl">Serial No.:</span><span class="val">' + val('SERIAL NO.') + '</span></div>';
    if (pageId === 'bscfume')   extraRows = '<div class="info-row"><span class="lbl">ประเภทตู้:</span><span class="val">' + val('ประเภทตู้') + '</span></div>';

    // ข้อมูลการอนุมัติ
    var approvedBy     = val('ผู้อนุมัติ');
    var approvedAt     = val('วันที่อนุมัติ');
    var approvalStatus = val('สถานะการอนุมัติ');
    var approvedColor  = approvalStatus === 'อนุมัติแล้ว' ? '#16a34a' : '#94a3b8';

    // ✅ ชื่อห้องปฏิบัติการที่ทำการสอบเทียบ: จากคอลัมน์ที่บันทึกไว้ → (รายการเก่า) สถานที่ติดตั้งของเครื่องมือ
    var _calLab = String(d['ห้องปฏิบัติการที่สอบเทียบ'] || '').trim();
    if (!_calLab) {
      var _eqKey = String(d['ID อ้างอิง'] || d['ID'] || '').trim();
      var _eqHit = (typeof cachedEquipments !== 'undefined' && cachedEquipments || []).find(function (e) {
        return _eqKey && (String(e['ID'] || '').trim() === _eqKey || String(e['รหัสครุภัณฑ์'] || '').trim() === _eqKey);
      });
      _calLab = (_eqHit && String(_eqHit['สถานที่ติดตั้ง'] || '').trim()) || '-';
    }
    // ✅ ผู้พิมพ์ + วันที่/เวลาที่พิมพ์ (แสดงท้ายแบบฟอร์ม)
    var _printBy = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser ? currentUser.username : '-');
    var _printAt = new Date().toLocaleString('th-TH', { year: 'numeric', month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit' }) + ' น.';
    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="UTF-8"><title>' + title + '</title><style>'
      + '@page{size:A4 landscape;margin:12mm 14mm}'
      + '*{box-sizing:border-box}'
      + 'body{font-family:"Sarabun","Tahoma",Arial,sans-serif;font-size:11pt;color:#1e293b;margin:0;padding:0}'
      + '.header{text-align:center;border-bottom:2.5px solid #1e40af;padding-bottom:8px;margin-bottom:12px}'
      + '.header h1{font-size:14pt;font-weight:700;color:#1e40af;margin:0}'
      + '.header .lab{font-size:11pt;font-weight:600;color:#334155;margin-top:3px}'
      + '.info-grid{display:grid;grid-template-columns:repeat(3,1fr);gap:5px 20px;margin-bottom:12px;border:1px solid #e2e8f0;border-radius:6px;padding:8px 14px;background:#f8fafc}'
      + '.info-row{display:flex;align-items:baseline;gap:6px;font-size:10pt}'
      + '.lbl{font-weight:600;color:#475569;min-width:140px;white-space:nowrap;flex-shrink:0}'
      + '.val{color:#1e293b;border-bottom:1px dashed #cbd5e1;flex:1;padding-bottom:1px}'
      + 'h2{font-size:11pt;font-weight:700;color:#1e40af;margin:12px 0 5px}'
      + 'table{width:100%;border-collapse:collapse;font-size:10pt;margin-bottom:10px}'
      + 'th{background:#1e40af;color:white;padding:6px 8px;text-align:center;font-weight:600;white-space:nowrap;border:1px solid #1e3a8a}'
      + 'td{padding:6px 8px;border:1px solid #e2e8f0;text-align:center;vertical-align:middle}'
      + 'tr:nth-child(even) td{background:#f8fafc}'
      + 'tr.pass td{background:#f0fdf4!important}'
      + 'tr.fail td{background:#fef2f2!important}'
      + '.result-box{border:2px solid ' + resColor + ';border-radius:8px;background:' + resBg + ';padding:10px 18px;margin:12px 0;display:flex;align-items:center;gap:12px}'
      + '.rlbl{font-size:10pt;font-weight:600;color:#64748b}'
      + '.rval{font-size:14pt;font-weight:700;color:' + resColor + '}'
      + '.print-meta{margin-top:14px;padding-top:6px;border-top:1px dashed #cbd5e1;text-align:right;font-size:9pt;color:#64748b}'
      + '.sigs{display:grid;grid-template-columns:1fr 1fr 1fr;gap:20px;margin-top:20px}'
      + '.sig{text-align:center;border-top:1px solid #94a3b8;padding-top:6px;font-size:9.5pt;color:#475569}'
      + '.sig .role{font-size:9pt;color:#94a3b8;margin-top:2px}'
      + '.sp{height:32px}'
      + '@media print{body{-webkit-print-color-adjust:exact;print-color-adjust:exact}}'
      + '</style></head><body>'
      + '<div class="header"><h1>' + title + '</h1><div class="lab">ห้องปฏิบัติการที่ทำการสอบเทียบ: ' + _calLab + '</div></div>'
      + '<div class="info-grid">'
      +   '<div class="info-row"><span class="lbl">รหัสครุภัณฑ์/ยี่ห้อ:</span><span class="val">' + eqCode + '</span></div>'
      +   '<div class="info-row"><span class="lbl">วันที่สอบเทียบ:</span><span class="val">' + calDate + '</span></div>'
      +   '<div class="info-row"><span class="lbl">Certificate No.:</span><span class="val">' + certNo + '</span></div>'
      +   extraRows
      +   '<div class="info-row"><span class="lbl">ผู้บันทึกข้อมูล:</span><span class="val">' + addedBy + '</span></div>'
      +   '<div class="info-row"><span class="lbl">สถานะการอนุมัติ:</span><span class="val" style="color:' + approvedColor + ';font-weight:700;">' + (approvalStatus||'รออนุมัติ') + '</span></div>'
      +   (approvedBy !== '-' ? '<div class="info-row"><span class="lbl">ผู้อนุมัติ:</span><span class="val">' + approvedBy + '</span></div>' : '')
      +   (approvedAt !== '-' ? '<div class="info-row"><span class="lbl">วันที่อนุมัติ:</span><span class="val">' + approvedAt + '</span></div>' : '')
      + '</div>'
      + '<h2>ผลการทดสอบ</h2>'
      + tableHtml
      + '<div class="result-box"><span class="rlbl">สรุปผลการสอบเทียบ:</span><span class="rval">' + (summaryLine || '-') + '</span></div>'
      + '<div class="print-meta">พิมพ์โดย: ' + _printBy + ' &nbsp;|&nbsp; วันที่พิมพ์: ' + _printAt + '</div>'
      + '</body></html>';

    var win = window.open('', '_blank', 'width=1000,height=750');
    if (!win) { alert('กรุณาอนุญาต Popup ในเบราว์เซอร์ก่อนครับ'); return; }
    win.document.write(html);
    win.document.close();
    win.focus();
    setTimeout(function(){ win.print(); }, 600);
  }
  // MutationObserver removed — switchPage() calls loadAdminData() directly



  // ============================================================
  // บันทึกประวัติเครื่องมือ — view / edit / print
  // ============================================================

  function _getItem(idx) {
    // ใช้ cachedEquipments ซึ่งมีข้อมูลครบและมี _rowIndex
    if (cachedEquipments && cachedEquipments[idx]) return cachedEquipments[idx];
    return _equipmentListData[idx] || null;
  }

  function _fmtDate(d) {
    if (!d) return '-';
    var dt = new Date(d);
    if (isNaN(dt)) return d;
    return dt.toLocaleDateString('th-TH', { year: 'numeric', month: 'long', day: 'numeric' });
  }

  // ✅ เครื่องมือที่มีรอบกำหนด (สอบเทียบ หรือ บำรุงรักษา) → คำนวณ Due Date และแจ้งเตือนเหมือนกัน
  // ============================================================
  // ✅ บันทึกเครื่องมือ PM — ปัญหา / การแก้ไข-ซ่อมแซม (เพิ่มต่อเนื่องต่อเครื่อง)
  // ============================================================
  var _pmEqList = [], _pmRecords = [], _pmRowSeq = 0;

  function _pmEsc(v) { return String(v == null ? '' : v).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;'); }
  function _pmFmtDate(s) {
    if (!s) return '-';
    var d = new Date(String(s).length === 10 ? s + 'T00:00:00' : s);
    return isNaN(d) ? s : d.toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
  }
  function _pmToday() {
    var d = new Date(); return d.getFullYear() + '-' + ('0' + (d.getMonth() + 1)).slice(-2) + '-' + ('0' + d.getDate()).slice(-2);
  }
  function _pmCurrentEq() {
    var id = (document.getElementById('pm-eqSelect') || {}).value;
    return _pmEqList.find(function (e) { return e['ID'] === id; }) || null;
  }

  function loadPMPage() {
    var sel = document.getElementById('pm-eqSelect');
    if (sel) sel.innerHTML = '<option value="">-- กำลังโหลดรายการเครื่องมือ --</option>';
    var dept = currentUser ? currentUser.dept : '', role = currentUser ? currentUser.role : '';
    google.script.run.withSuccessHandler(function (data) {
      // ✅ เฉพาะเครื่องที่ไม่ต้องสอบเทียบ (ไม่สอบเทียบ / บำรุงรักษา) และยังไม่ปลดระวาง — เครื่องสอบเทียบใช้แบบฟอร์มสอบเทียบแทน
      _pmEqList = (data || []).filter(function (e) {
        var need = String(e['สถานะการสอบเทียบ'] || e['การสอบเทียบหรือไม่'] || '').trim();
        return need !== 'สอบเทียบ' && String(e['สถานะเครื่องมือ'] || '').trim() !== 'จำหน่ายออก';
      }).sort(function (a, b) {
        var ma = _isMaint(a) ? 0 : 1, mb = _isMaint(b) ? 0 : 1;
        return ma - mb || String(a['รหัสครุภัณฑ์'] || '').localeCompare(String(b['รหัสครุภัณฑ์'] || ''));
      });
      var keep = sel ? sel.value : '';
      _pmFilterSelect((document.getElementById('pm-eqSearch') || {}).value || '');
      if (keep && _pmEqList.some(function (e) { return e['ID'] === keep; })) { sel.value = keep; pmSelectEquipment(); }
      else { document.getElementById('pm-body').classList.add('hidden'); document.getElementById('pm-eqInfo').classList.add('hidden'); }
    }).withFailureHandler(function (e) {
      if (sel) sel.innerHTML = '<option value="">โหลดไม่สำเร็จ</option>';
      showToast('โหลดรายการเครื่องมือไม่สำเร็จ: ' + (e && e.message || e), 'danger');
    }).getEquipments(dept, role);
  }

  function _pmFilterSelect(q) {
    var sel = document.getElementById('pm-eqSelect'); if (!sel) return;
    var keep = sel.value; q = String(q || '').trim().toLowerCase();
    var list = _pmEqList.filter(function (e) {
      return !q || [e['รหัสครุภัณฑ์'], e['ชื่อเครื่องมือ'], e['ID'], e['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง']].join(' ').toLowerCase().indexOf(q) !== -1;
    });
    var maint = list.filter(_isMaint), other = list.filter(function (e) { return !_isMaint(e); });
    function opts(arr) { return arr.map(function (e) { return '<option value="' + _pmEsc(e['ID']) + '">' + _pmEsc((e['รหัสครุภัณฑ์'] || '-') + ' | ' + (e['ชื่อเครื่องมือ'] || '-')) + '</option>'; }).join(''); }
    sel.innerHTML = '<option value="">' + (list.length ? '-- เลือกเครื่องมือที่ไม่ต้องสอบเทียบ (' + list.length + ' รายการ) --' : (_pmEqList.length ? '-- ไม่พบเครื่องมือที่ค้นหา --' : '-- ไม่มีเครื่องมือที่ไม่ต้องสอบเทียบในหน่วยนี้ --')) + '</option>'
      + (maint.length ? '<optgroup label="บำรุงรักษา (มีรอบ PM)">' + opts(maint) + '</optgroup>' : '')
      + (other.length ? '<optgroup label="ไม่สอบเทียบ">' + opts(other) + '</optgroup>' : '');
    if (keep && list.some(function (e) { return e['ID'] === keep; })) sel.value = keep;
  }

  function pmSelectEquipment() {
    var eq = _pmCurrentEq();
    var info = document.getElementById('pm-eqInfo'), body = document.getElementById('pm-body');
    if (!eq) { info.classList.add('hidden'); body.classList.add('hidden'); return; }
    var isM = _isMaint(eq);
    function f(lbl, val) { return '<div><span class="text-slate-500">' + lbl + ':</span> <b class="text-slate-800 dark:text-slate-100">' + _pmEsc(val || '-') + '</b></div>'; }
    info.innerHTML = f('รหัสครุภัณฑ์', eq['รหัสครุภัณฑ์']) + f('ชื่อเครื่องมือ', eq['ชื่อเครื่องมือ']) + f('ประเภท', eq['ประเภทเครื่องมือ'])
      + f('ยี่ห้อ/รุ่น/S/N', eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง']) + f('สถานที่ติดตั้ง', eq['สถานที่ติดตั้ง']) + f('สถานะเครื่องมือ', eq['สถานะเครื่องมือ'])
      + (isM ? f('บำรุงรักษาล่าสุด', _pmFmtDate(eq['วันที่สอบเทียบล่าสุด'])) + f('รอบบำรุงรักษา', eq['ระยะเวลาการสอบเทียบ'] ? eq['ระยะเวลาการสอบเทียบ'] + ' เดือน' : '-')
             + f('ครบกำหนดครั้งถัดไป', _pmFmtDate(eq['สอบเทียบครั้งถัดไป'])) : '');
    info.classList.remove('hidden');
    body.classList.remove('hidden');
    var sw = document.getElementById('pm-sched-wrap');
    sw.classList.toggle('hidden', !isM); sw.classList.toggle('flex', isM);
    document.getElementById('pm-date').value = _pmToday();
    document.getElementById('pm-rows').innerHTML = ''; addPMRow();
    loadPMHistory();
  }

  function loadPMHistory() {
    var eq = _pmCurrentEq(); if (!eq) return;
    var tb = document.getElementById('pm-history-body');
    tb.innerHTML = '<tr><td colspan="6" class="p-4 text-center text-slate-400">กำลังโหลด...</td></tr>';
    google.script.run.withSuccessHandler(function (res) {
      if (!res || !res.success) { tb.innerHTML = '<tr><td colspan="6" class="p-4 text-center text-red-500">' + _pmEsc(res && res.message || 'โหลดไม่สำเร็จ') + '</td></tr>'; return; }
      _pmRecords = res.data || [];
      document.getElementById('pm-count').textContent = '(' + _pmRecords.length + ' รายการ)';
      if (!_pmRecords.length) { tb.innerHTML = '<tr><td colspan="6" class="p-4 text-center text-slate-400 italic">ยังไม่มีบันทึก PM ของเครื่องนี้</td></tr>'; return; }
      tb.innerHTML = _pmRecords.map(function (r, i) {
        return '<tr class="border-t dark:border-slate-700 align-top">'
          + '<td class="p-2 text-center text-slate-500">' + (i + 1) + '</td>'
          + '<td class="p-2 text-center whitespace-nowrap">' + _pmFmtDate(r.date) + '</td>'
          + '<td class="p-2 whitespace-pre-line">' + _pmEsc(r.problem || '-') + '</td>'
          + '<td class="p-2 whitespace-pre-line">' + _pmEsc(r.fix || '-') + '</td>'
          + '<td class="p-2 text-xs text-slate-500">' + _pmEsc(r.addedBy || '-') + '</td>'
          + '<td class="p-2 text-center"><button onclick="deletePMRecord(' + i + ')" title="ลบ" class="text-red-400 hover:text-red-600"><span class="material-symbols-outlined text-lg">delete</span></button></td>'
          + '</tr>';
      }).join('');
    }).withFailureHandler(function (e) {
      tb.innerHTML = '<tr><td colspan="6" class="p-4 text-center text-red-500">' + _pmEsc(e && e.message || e) + '</td></tr>';
    }).getPMRecords(eq['ID']);
  }

  function addPMRow() {
    var id = ++_pmRowSeq;
    var tr = document.createElement('tr');
    tr.className = 'pm-row border-t dark:border-slate-700 align-top';
    tr.innerHTML = '<td class="p-1.5"><textarea rows="2" class="pm-problem w-full p-2 border rounded-md text-sm dark:bg-slate-700 dark:border-slate-600" placeholder="อาการ / ปัญหาที่พบ"></textarea></td>'
      + '<td class="p-1.5"><textarea rows="2" class="pm-fix w-full p-2 border rounded-md text-sm dark:bg-slate-700 dark:border-slate-600" placeholder="วิธีแก้ไข / การซ่อมแซม"></textarea></td>'
      + '<td class="p-1.5 text-center"><button type="button" onclick="this.closest(\'tr\').remove()" title="ลบแถว" class="text-red-400 hover:text-red-600 mt-2"><span class="material-symbols-outlined text-lg">close</span></button></td>';
    document.getElementById('pm-rows').appendChild(tr);
  }

  function submitPMRecords() {
    var eq = _pmCurrentEq();
    if (!eq) { alert('กรุณาเลือกเครื่องมือ'); return; }
    var date = document.getElementById('pm-date').value;
    if (!date) { alert('กรุณาระบุวันที่ดำเนินการ'); return; }
    var items = [];
    document.querySelectorAll('#pm-rows .pm-row').forEach(function (tr) {
      var p = tr.querySelector('.pm-problem').value.trim(), f = tr.querySelector('.pm-fix').value.trim();
      if (p || f) items.push({ problem: p, fix: f });
    });
    if (!items.length) { alert('กรุณากรอกปัญหา หรือ การแก้ไข/ซ่อมแซม อย่างน้อย 1 แถว'); return; }
    var upd = _isMaint(eq) && document.getElementById('pm-updateSched').checked;
    if (!confirm('บันทึก PM ' + items.length + ' รายการ ของ ' + (eq['รหัสครุภัณฑ์'] || eq['ID']) + ' ?')) return;
    showLoader();
    google.script.run.withSuccessHandler(function (res) {
      hideLoader();
      if (!res || !res.success) { alert('บันทึกไม่สำเร็จ: ' + (res && res.message || '')); return; }
      var msg = 'บันทึก PM เรียบร้อย ' + res.count + ' รายการ';
      if (res.schedule && res.schedule.updated) {
        msg += '\nอัปเดตบำรุงรักษาล่าสุด: ' + _pmFmtDate(res.schedule.last) + (res.schedule.next ? '  ครบกำหนดครั้งถัดไป: ' + _pmFmtDate(res.schedule.next) : '');
      }
      alert(msg);
      if (res.schedule && res.schedule.updated) { loadPMPage(); } // โหลดข้อมูลเครื่องใหม่ (วันที่รอบถัดไป)
      else { document.getElementById('pm-rows').innerHTML = ''; addPMRow(); loadPMHistory(); }
    }).withFailureHandler(function (e) {
      hideLoader(); alert('บันทึกไม่สำเร็จ: ' + (e && e.message || e));
    }).savePMRecords({
      eqId: eq['ID'], eqCode: eq['รหัสครุภัณฑ์'] || '', eqName: eq['ชื่อเครื่องมือ'] || '', dept: eq['สถานที่ติดตั้ง'] || '',
      date: date, items: items, updateSchedule: upd,
      addedBy: currentUser ? (_getDisplayName() || currentUser.username) : 'System'
    });
  }

  function deletePMRecord(i) {
    var r = _pmRecords[i]; if (!r) return;
    if (!confirm('ลบรายการ PM วันที่ ' + _pmFmtDate(r.date) + ' ?\n\nปัญหา: ' + (r.problem || '-'))) return;
    showLoader();
    google.script.run.withSuccessHandler(function (res) {
      hideLoader();
      if (res && res.success) { showToast('ลบรายการแล้ว', 'success'); loadPMHistory(); }
      else alert('ลบไม่สำเร็จ: ' + (res && res.message || ''));
    }).withFailureHandler(function (e) { hideLoader(); alert('ลบไม่สำเร็จ: ' + (e && e.message || e)); })
      .deletePMRecord(r.id, currentUser ? currentUser.username : '');
  }

  // พิมพ์ประวัติ PM ของเครื่อง (ตาราง ปัญหา | การแก้ไข/ซ่อมแซม)
  function printPMHistory() {
    var eq = _pmCurrentEq(); if (!eq) return;
    var now = new Date();
    var by = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser ? currentUser.username : '-');
    var at = now.toLocaleDateString('th-TH', { day: 'numeric', month: 'short', year: 'numeric' }) + ' ' + now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + ' น.';
    var rows = _pmRecords.map(function (r, i) {
      return '<tr><td class="c">' + (i + 1) + '</td><td class="c">' + _pmFmtDate(r.date) + '</td><td>' + _pmEsc(r.problem || '-').replace(/\n/g, '<br>')
        + '</td><td>' + _pmEsc(r.fix || '-').replace(/\n/g, '<br>') + '</td><td>' + _pmEsc(r.addedBy || '-') + '</td></tr>';
    }).join('');
    for (var k = _pmRecords.length; k < 6; k++) rows += '<tr class="blank"><td></td><td></td><td></td><td></td><td></td></tr>'; // ช่องว่างสำหรับเขียนเพิ่ม
    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="UTF-8"><title>บันทึก PM ' + _pmEsc(eq['รหัสครุภัณฑ์'] || '') + '</title><style>'
      + '@import url("https://fonts.googleapis.com/css2?family=Sarabun:wght@400;600;700&display=swap");'
      + '@page{size:A4;margin:14mm 12mm}*{box-sizing:border-box;margin:0;padding:0}'
      + 'body{font-family:"Sarabun",sans-serif;font-size:11pt;color:#1e293b}'
      + 'h1{font-size:15pt;text-align:center;color:#1e3a8a;margin-bottom:10px}'
      + '.info{display:grid;grid-template-columns:1fr 1fr;gap:4px 16px;border:1px solid #cbd5e1;border-radius:6px;padding:8px 12px;margin-bottom:12px;font-size:10.5pt}'
      + '.info span{color:#64748b}'
      + 'table{width:100%;border-collapse:collapse}th,td{border:1px solid #1e293b;padding:6px 8px;vertical-align:top}'
      + 'th{background:#f1f5f9;font-weight:600;text-align:center}.c{text-align:center;white-space:nowrap}'
      + 'tr.blank td{height:48px}tr{page-break-inside:avoid}'
      + '.foot{margin-top:10px;text-align:right;font-size:9pt;color:#64748b;border-top:1px dashed #cbd5e1;padding-top:4px}'
      + '</style></head><body>'
      + '<h1>บันทึกการบำรุงรักษา / ซ่อมแซมเครื่องมือ (PM)</h1>'
      + '<div class="info">'
      +   '<div><span>รหัสครุภัณฑ์:</span> <b>' + _pmEsc(eq['รหัสครุภัณฑ์'] || '-') + '</b></div>'
      +   '<div><span>ชื่อเครื่องมือ:</span> <b>' + _pmEsc(eq['ชื่อเครื่องมือ'] || '-') + '</b></div>'
      +   '<div><span>ยี่ห้อ/รุ่น/S/N:</span> ' + _pmEsc(eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || '-') + '</div>'
      +   '<div><span>สถานที่ติดตั้ง:</span> ' + _pmEsc(eq['สถานที่ติดตั้ง'] || '-') + '</div>'
      + '</div>'
      + '<table><thead><tr><th style="width:40px">ลำดับ</th><th style="width:90px">วันที่</th><th>ปัญหา</th><th>การแก้ไข/ซ่อมแซม</th><th style="width:120px">ผู้บันทึก</th></tr></thead>'
      + '<tbody>' + rows + '</tbody></table>'
      + '<div class="foot">พิมพ์โดย: ' + _pmEsc(by) + ' &nbsp;|&nbsp; วันที่พิมพ์: ' + at + '</div>'
      + '</body></html>';
    var win = window.open('', '_blank', 'width=900,height=750');
    if (!win) { showToast('เบราว์เซอร์บล็อกหน้าต่างใหม่ กรุณาอนุญาต Pop-up', 'warning'); return; }
    win.document.open(); win.document.write(html); win.document.close();
    win.onload = function () { setTimeout(function () { win.focus(); win.print(); }, 500); };
  }

  // ============================================================
  // ✅ คำนวณค่าเสื่อมราคา (Depreciation)
  //   หลักการ: คำนวณ "ค่าเสื่อมตามปีอายุสินทรัพย์" (asset-year) ก่อน แล้วปันส่วนเข้าปีงบประมาณ
  //   ตามจำนวนวันที่ใช้งานจริง · วิธีเส้นตรงคิดรายวัน (ค่าเสื่อมต่อปี ÷ 365 × วัน) ตามแนวปฏิบัติภาครัฐ
  //   - SL : (ราคาทุน − ราคาซาก) ÷ อายุ ต่อปี
  //   - DDB: อัตรา 2/อายุ × มูลค่าสุทธิต้นปี และเปลี่ยนเป็นเส้นตรงเมื่อเส้นตรงให้ค่ามากกว่า (ไม่ต่ำกว่าราคาซาก)
  //   - SYD: (ราคาทุน − ราคาซาก) × ปีที่เหลือ ÷ ผลรวมจำนวนปี
  //   ปีสุดท้ายปรับให้มูลค่าสุทธิเท่าราคาซากพอดี
  // ============================================================
  var _dpEqList = [], _dpLast = null;
  var DP_METHOD_NAME = { SL: 'วิธีเส้นตรง (Straight-line)', DDB: 'วิธียอดคงเหลือลดลงทวีคูณ (Double-declining balance)', SYD: 'วิธีผลรวมจำนวนปี (Sum-of-the-years\' digits)' };

  function _dpNum(v) { var n = parseFloat(String(v == null ? '' : v).replace(/[^0-9.\-]/g, '')); return isNaN(n) ? NaN : n; }
  function _dpMoney(n) { return (Math.round(n * 100) / 100).toLocaleString('th-TH', { minimumFractionDigits: 2, maximumFractionDigits: 2 }); }
  function _dpUTC(iso) { var p = String(iso).split('-'); return Date.UTC(+p[0], +p[1] - 1, +p[2]); }
  function _dpIso(t) { var d = new Date(t); return d.getUTCFullYear() + '-' + ('0' + (d.getUTCMonth() + 1)).slice(-2) + '-' + ('0' + d.getUTCDate()).slice(-2); }
  function _dpAddYears(t, y) {                         // รองรับอายุเป็นทศนิยม (เช่น 2.5 ปี = 30 เดือน)
    var d = new Date(t), m = Math.round(y * 12);
    var r = Date.UTC(d.getUTCFullYear(), d.getUTCMonth() + m, d.getUTCDate());
    if (new Date(r).getUTCDate() !== d.getUTCDate()) r = Date.UTC(d.getUTCFullYear(), d.getUTCMonth() + m + 1, 0); // 29 ก.พ. ฯลฯ
    return r;
  }
  function _dpFmt(iso) { if (!iso) return '-'; var d = new Date(iso + 'T00:00:00'); return isNaN(d) ? iso : d.toLocaleDateString('th-TH', { day: 'numeric', month: 'short', year: 'numeric' }); }
  var DAY = 86400000;

  // ค่าเสื่อมราย "ปีอายุสินทรัพย์"
  function _dpAssetYears(cost, salvage, life, method) {
    var base = cost - salvage, n = Math.ceil(life - 1e-9), out = [], nbv = cost;
    var sum = life * (life + 1) / 2;                                  // SYD
    for (var k = 1; k <= n; k++) {
      var frac = Math.min(1, life - (k - 1));                         // ปีสุดท้ายอาจไม่เต็มปี
      var d;
      if (method === 'DDB') {
        var remainYears = life - (k - 1);
        var ddb = nbv * (2 / life) * frac;
        var sl  = (nbv - salvage) / remainYears * frac;
        d = Math.max(ddb, sl);
      } else if (method === 'SYD') {
        d = base * (life - (k - 1)) / sum * frac;
      } else {
        d = base / life * frac;
      }
      if (k === n || nbv - d < salvage) d = nbv - salvage;             // ปิดยอดให้เหลือราคาซาก
      d = Math.max(0, d);
      nbv -= d;
      out.push(d);
    }
    return out;
  }

  // ปันส่วนเข้าปีงบประมาณ (1 ต.ค. – 30 ก.ย.) ตามจำนวนวัน
  function dpSchedule(cost, salvage, life, startIso, method) {
    var start = _dpUTC(startIso), end = _dpAddYears(start, life);     // [start, end)
    var ay = _dpAssetYears(cost, salvage, life, method);
    var seg = ay.map(function (amt, i) {
      var s = _dpAddYears(start, i), e = Math.min(_dpAddYears(start, i + 1), end);
      return { s: s, e: e, amt: amt, days: Math.round((e - s) / DAY) };
    });
    var sd = new Date(start), fy = sd.getUTCFullYear() + (sd.getUTCMonth() >= 9 ? 1 : 0);
    var rows = [], acc = 0;
    while (true) {
      var fs = Date.UTC(fy - 1, 9, 1), fe = Date.UTC(fy, 9, 1);
      if (fs >= end) break;
      var a = Math.max(fs, start), b = Math.min(fe, end), dep = 0;
      seg.forEach(function (g) {
        var ov = Math.min(g.e, b) - Math.max(g.s, a);
        if (ov > 0 && g.days > 0) dep += g.amt * Math.round(ov / DAY) / g.days;
      });
      if (method === 'SL') {
        // วิธีเส้นตรง: คิดรายวัน = ค่าเสื่อมต่อปี ÷ 365 × จำนวนวันที่ใช้งานในปีงบประมาณ (ปีสุดท้ายรับยอดคงเหลือ)
        var full = (a === fs && b === fe);                              // ใช้งานเต็มปีงบประมาณ → ค่าเสื่อมเต็มปี
        dep = full ? (cost - salvage) / life : (cost - salvage) / life / 365 * Math.round((b - a) / DAY);
        if (b >= end || acc + dep > cost - salvage) dep = cost - salvage - acc;
      }
      acc += dep;
      rows.push({ fy: fy, from: _dpIso(a), to: _dpIso(b - DAY), days: Math.round((b - a) / DAY), dep: dep, acc: acc, nbv: cost - acc });
      fy++;
    }
    return { rows: rows, endIso: _dpIso(end - DAY), assetYears: ay };
  }

  // ค่าเสื่อมสะสม ณ วันที่ (รวมวันที่นั้น)
  function dpAccumAt(cost, salvage, life, startIso, method, asOfIso) {
    var start = _dpUTC(startIso), end = _dpAddYears(start, life), cut = Math.min(_dpUTC(asOfIso) + DAY, end);
    if (cut <= start) return 0;
    if (method === 'SL') {
      if (cut >= end) return cost - salvage;
      var sch = dpSchedule(cost, salvage, life, startIso, 'SL'), acc0 = 0;
      for (var i = 0; i < sch.rows.length; i++) {
        var r = sch.rows[i], rs = _dpUTC(r.from), re = _dpUTC(r.to) + DAY;
        if (re <= cut) { acc0 += r.dep; continue; }                        // ปีงบที่ผ่านไปแล้วทั้งปี
        if (rs < cut) acc0 += (cost - salvage) / life / 365 * Math.round((cut - rs) / DAY); // ปีปัจจุบัน (บางส่วน)
        break;
      }
      return Math.min(cost - salvage, acc0);
    }
    var ay = _dpAssetYears(cost, salvage, life, method), acc = 0;
    ay.forEach(function (amt, i) {
      var s = _dpAddYears(start, i), e = Math.min(_dpAddYears(start, i + 1), end), days = Math.round((e - s) / DAY);
      var ov = Math.min(e, cut) - s;
      if (ov > 0 && days > 0) acc += amt * Math.min(days, Math.round(ov / DAY)) / days;
    });
    return acc;
  }

  function _dpStartOf(eq) {
    var raw = eq['วันเริ่มใช้เครื่อง'] || eq['วันรับเครื่อง'] || '';
    return (typeof _calToIsoDate === 'function') ? _calToIsoDate(raw) : String(raw).slice(0, 10);
  }
  function _dpTodayIso() { var d = new Date(); return d.getFullYear() + '-' + ('0' + (d.getMonth() + 1)).slice(-2) + '-' + ('0' + d.getDate()).slice(-2); }

  function loadDeprPage() {
    var asof = document.getElementById('dp-asof'); if (asof && !asof.value) asof.value = _dpTodayIso();
    var sel = document.getElementById('dp-eqSelect');
    var dept = currentUser ? currentUser.dept : '', role = currentUser ? currentUser.role : '';
    google.script.run.withSuccessHandler(function (data) {
      _dpEqList = (data || []).slice().sort(function (a, b) { return String(a['รหัสครุภัณฑ์'] || '').localeCompare(String(b['รหัสครุภัณฑ์'] || '')); });
      _dpFillSelect((document.getElementById('dp-eqSearch') || {}).value || '');
    }).withFailureHandler(function () { if (sel) sel.innerHTML = '<option value="">โหลดไม่สำเร็จ</option>'; }).getEquipments(dept, role);
  }

  function _dpFillSelect(q) {
    var sel = document.getElementById('dp-eqSelect'); if (!sel) return;
    var keep = sel.value; q = String(q || '').trim().toLowerCase();
    var list = _dpEqList.filter(function (e) { return !q || [e['รหัสครุภัณฑ์'], e['ชื่อเครื่องมือ'], e['ID']].join(' ').toLowerCase().indexOf(q) !== -1; });
    sel.innerHTML = '<option value="">-- เลือกเครื่องมือ (' + list.length + ') --</option>' + list.map(function (e) {
      return '<option value="' + _pmEsc(e['ID']) + '">' + _pmEsc((e['รหัสครุภัณฑ์'] || '-') + ' | ' + (e['ชื่อเครื่องมือ'] || '-')) + '</option>';
    }).join('');
    if (keep) sel.value = keep;
  }

  function dpPickEquipment() {
    var id = document.getElementById('dp-eqSelect').value;
    var eq = _dpEqList.find(function (e) { return e['ID'] === id; });
    if (!eq) return;
    var cost = _dpNum(eq['มูลค่าเครื่องมือ']), life = _dpNum(eq['อายุการใช้งาน(ปี)'] || eq['อายุการใช้งาน (ปี)']);
    document.getElementById('dp-cost').value  = isNaN(cost) ? '' : cost;
    document.getElementById('dp-life').value  = isNaN(life) ? '' : life;
    document.getElementById('dp-start').value = _dpStartOf(eq);
    var miss = [];
    if (isNaN(cost)) miss.push('มูลค่าเครื่องมือ'); if (isNaN(life)) miss.push('อายุการใช้งาน'); if (!_dpStartOf(eq)) miss.push('วันเริ่มใช้/วันรับเครื่อง');
    if (miss.length) showToast('ทะเบียนเครื่องมือยังไม่มี: ' + miss.join(', ') + ' — กรุณากรอกเอง', 'warning');
  }

  function dpCalculate() {
    var cost = _dpNum(document.getElementById('dp-cost').value), salvage = _dpNum(document.getElementById('dp-salvage').value);
    var life = _dpNum(document.getElementById('dp-life').value), start = document.getElementById('dp-start').value;
    var method = document.getElementById('dp-method').value, asof = document.getElementById('dp-asof').value || _dpTodayIso();
    if (isNaN(salvage)) salvage = 0;
    if (!(cost > 0)) { alert('กรุณากรอกราคาทุนมากกว่า 0'); return; }
    if (!(life > 0)) { alert('กรุณากรอกอายุการใช้งาน (ปี)'); return; }
    if (!start)      { alert('กรุณาระบุวันที่เริ่มคิดค่าเสื่อม'); return; }
    if (salvage >= cost) { alert('ราคาซากต้องน้อยกว่าราคาทุน'); return; }

    var sch = dpSchedule(cost, salvage, life, start, method);
    var acc = dpAccumAt(cost, salvage, life, start, method, asof);
    var eqSel = document.getElementById('dp-eqSelect');
    var eqLabel = eqSel.value ? eqSel.options[eqSel.selectedIndex].text : '';
    _dpLast = { cost: cost, salvage: salvage, life: life, start: start, method: method, asof: asof, sch: sch, acc: acc, eqLabel: eqLabel };

    document.getElementById('dp-result-title').textContent = (eqLabel ? '· ' + eqLabel + ' ' : '') + '· ' + DP_METHOD_NAME[method];
    function card(lbl, val, cls) { return '<div class="rounded-xl border dark:border-slate-700 p-3 ' + cls + '"><div class="text-xs text-slate-500">' + lbl + '</div><div class="text-lg font-bold">' + val + '</div></div>'; }
    document.getElementById('dp-cards').innerHTML =
        card('ค่าเสื่อมราคา/ปี ' + (method === 'SL' ? '' : '(ปีแรก)'), _dpMoney(sch.assetYears[0] || 0) + ' ฿', 'bg-amber-50 dark:bg-amber-900/20')
      + card('ค่าเสื่อมสะสม ณ ' + _dpFmt(asof), _dpMoney(acc) + ' ฿', 'bg-slate-50 dark:bg-slate-900/40')
      + card('มูลค่าสุทธิ ณ ' + _dpFmt(asof), _dpMoney(cost - acc) + ' ฿', 'bg-emerald-50 dark:bg-emerald-900/20')
      + card('ตัดค่าเสื่อมครบ', _dpFmt(sch.endIso), 'bg-sky-50 dark:bg-sky-900/20');

    var f = { SL: 'ค่าเสื่อม/ปี = (ราคาทุน − ราคาซาก) ÷ อายุการใช้งาน = (' + _dpMoney(cost) + ' − ' + _dpMoney(salvage) + ') ÷ ' + life + ' = <b>' + _dpMoney((cost - salvage) / life) + ' บาท</b>',
              DDB: 'อัตรา = 2 ÷ อายุ = ' + (200 / life).toFixed(2) + '% ต่อปี คูณมูลค่าสุทธิต้นปี · เปลี่ยนเป็นวิธีเส้นตรงเมื่อให้ค่ามากกว่า · ไม่ต่ำกว่าราคาซาก',
              SYD: 'ค่าเสื่อมปีที่ k = (ราคาทุน − ราคาซาก) × (อายุ − k + 1) ÷ ' + (life * (life + 1) / 2) + ' (ผลรวมจำนวนปี)' }[method];
    document.getElementById('dp-formula').innerHTML = '<b>สูตร:</b> ' + f
      + '<br>ปีงบประมาณคิดตามจำนวนวันที่ใช้งานจริง (ปีแรกและปีสุดท้ายไม่เต็มปี) · ปีสุดท้ายปรับให้มูลค่าสุทธิเหลือเท่าราคาซาก';

    document.getElementById('dp-table').innerHTML = sch.rows.map(function (r) {
      var cur = asof >= r.from && asof <= r.to;
      return '<tr class="border-t dark:border-slate-700' + (cur ? ' bg-amber-50 dark:bg-amber-900/20 font-semibold' : '') + '">'
        + '<td class="p-2 text-center">' + (r.fy + 543) + (cur ? ' ◀' : '') + '</td>'
        + '<td class="p-2 text-center text-xs text-slate-500">' + _dpFmt(r.from) + ' – ' + _dpFmt(r.to) + '</td>'
        + '<td class="p-2 text-right">' + r.days + '</td>'
        + '<td class="p-2 text-right">' + _dpMoney(r.dep) + '</td>'
        + '<td class="p-2 text-right">' + _dpMoney(r.acc) + '</td>'
        + '<td class="p-2 text-right">' + _dpMoney(r.nbv) + '</td></tr>';
    }).join('') + '<tr class="border-t-2 dark:border-slate-600 font-bold bg-slate-50 dark:bg-slate-900/40"><td class="p-2 text-center" colspan="3">รวม</td><td class="p-2 text-right">'
      + _dpMoney(cost - salvage) + '</td><td class="p-2 text-right" colspan="2">ราคาซาก ' + _dpMoney(salvage) + '</td></tr>';
    document.getElementById('dp-result').classList.remove('hidden');
  }

  function dpPrint() {
    var L = _dpLast; if (!L) return;
    var by = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser ? currentUser.username : '-'), now = new Date();
    var at = now.toLocaleDateString('th-TH', { day: 'numeric', month: 'short', year: 'numeric' }) + ' ' + now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + ' น.';
    var rows = L.sch.rows.map(function (r) {
      return '<tr><td class="c">' + (r.fy + 543) + '</td><td class="c">' + _dpFmt(r.from) + ' – ' + _dpFmt(r.to) + '</td><td class="r">' + r.days + '</td><td class="r">'
        + _dpMoney(r.dep) + '</td><td class="r">' + _dpMoney(r.acc) + '</td><td class="r">' + _dpMoney(r.nbv) + '</td></tr>';
    }).join('');
    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="UTF-8"><title>ตารางค่าเสื่อมราคา</title><style>'
      + '@import url("https://fonts.googleapis.com/css2?family=Sarabun:wght@400;600;700&display=swap");@page{size:A4;margin:14mm 12mm}'
      + 'body{font-family:"Sarabun",sans-serif;font-size:11pt;color:#1e293b;margin:0}h1{font-size:15pt;text-align:center;color:#1e3a8a;margin:0 0 10px}'
      + '.info{display:grid;grid-template-columns:1fr 1fr;gap:3px 16px;border:1px solid #cbd5e1;border-radius:6px;padding:8px 12px;margin-bottom:10px;font-size:10.5pt}.info span{color:#64748b}'
      + 'table{width:100%;border-collapse:collapse;font-size:10pt}th,td{border:1px solid #94a3b8;padding:4px 6px}th{background:#f1f5f9}.c{text-align:center}.r{text-align:right}'
      + '.foot{margin-top:10px;text-align:right;font-size:9pt;color:#64748b;border-top:1px dashed #cbd5e1;padding-top:4px}</style></head><body>'
      + '<h1>ตารางคำนวณค่าเสื่อมราคาครุภัณฑ์</h1><div class="info">'
      + '<div style="grid-column:1/-1"><span>ครุภัณฑ์:</span> <b>' + _pmEsc(L.eqLabel || '-') + '</b></div>'
      + '<div><span>ราคาทุน:</span> <b>' + _dpMoney(L.cost) + ' บาท</b></div><div><span>ราคาซาก:</span> ' + _dpMoney(L.salvage) + ' บาท</div>'
      + '<div><span>อายุการใช้งาน:</span> ' + L.life + ' ปี</div><div><span>วันที่เริ่มคิดค่าเสื่อม:</span> ' + _dpFmt(L.start) + '</div>'
      + '<div><span>วิธีคำนวณ:</span> ' + DP_METHOD_NAME[L.method] + '</div><div><span>ตัดค่าเสื่อมครบ:</span> ' + _dpFmt(L.sch.endIso) + '</div>'
      + '<div><span>ค่าเสื่อมสะสม ณ ' + _dpFmt(L.asof) + ':</span> <b>' + _dpMoney(L.acc) + ' บาท</b></div><div><span>มูลค่าสุทธิ ณ ' + _dpFmt(L.asof) + ':</span> <b>' + _dpMoney(L.cost - L.acc) + ' บาท</b></div>'
      + '</div><table><thead><tr><th>ปีงบประมาณ</th><th>ช่วงที่คิด</th><th>จำนวนวัน</th><th>ค่าเสื่อมราคาประจำปี</th><th>ค่าเสื่อมราคาสะสม</th><th>มูลค่าสุทธิตามบัญชี</th></tr></thead><tbody>'
      + rows + '</tbody></table><div class="foot">พิมพ์โดย: ' + _pmEsc(by) + ' &nbsp;|&nbsp; วันที่พิมพ์: ' + at + '</div></body></html>';
    var win = window.open('', '_blank', 'width=900,height=750');
    if (!win) { showToast('เบราว์เซอร์บล็อกหน้าต่างใหม่ กรุณาอนุญาต Pop-up', 'warning'); return; }
    win.document.open(); win.document.write(html); win.document.close();
    win.onload = function () { setTimeout(function () { win.focus(); win.print(); }, 500); };
  }

  var _dpSummary = [];
  function dpSummaryAll() {
    var asof = document.getElementById('dp-asof').value || _dpTodayIso(), salvage = 1;
    _dpSummary = _dpEqList.map(function (e) {
      var cost = _dpNum(e['มูลค่าเครื่องมือ']), life = _dpNum(e['อายุการใช้งาน(ปี)'] || e['อายุการใช้งาน (ปี)']), start = _dpStartOf(e);
      var r = { code: e['รหัสครุภัณฑ์'] || '-', name: e['ชื่อเครื่องมือ'] || '-', start: start, life: life, cost: cost, annual: NaN, acc: NaN, nbv: NaN, note: '' };
      var miss = [];
      if (!(cost > 0)) miss.push('ราคาทุน'); if (!(life > 0)) miss.push('อายุ'); if (!start) miss.push('วันเริ่มใช้');
      if (miss.length) { r.note = 'ขาด ' + miss.join('/'); return r; }
      var sv = Math.min(salvage, cost);
      r.annual = (cost - sv) / life;
      r.acc = dpAccumAt(cost, sv, life, start, 'SL', asof);
      r.nbv = cost - r.acc;
      if (asof < start) r.note = 'ยังไม่เริ่มคิด';
      else if (Math.abs(r.nbv - sv) < 0.005) r.note = 'ตัดครบแล้ว';
      return r;
    });
    var ok = _dpSummary.filter(function (r) { return !isNaN(r.acc); });
    var tot = ok.reduce(function (t, r) { t.cost += r.cost; t.annual += r.annual; t.acc += r.acc; t.nbv += r.nbv; return t; }, { cost: 0, annual: 0, acc: 0, nbv: 0 });
    function m(v) { return isNaN(v) ? '-' : _dpMoney(v); }
    document.getElementById('dp-sum-body').innerHTML = (_dpSummary.length ? _dpSummary.map(function (r, i) {
      return '<tr class="border-t dark:border-slate-700"><td class="p-2 text-center">' + (i + 1) + '</td><td class="p-2">' + _pmEsc(r.code) + '<div class="text-xs text-slate-500">' + _pmEsc(r.name) + '</div></td>'
        + '<td class="p-2 text-center">' + _dpFmt(r.start) + '</td><td class="p-2 text-right">' + (isNaN(r.life) ? '-' : r.life) + '</td>'
        + '<td class="p-2 text-right">' + m(r.cost) + '</td><td class="p-2 text-right">' + m(r.annual) + '</td><td class="p-2 text-right">' + m(r.acc) + '</td>'
        + '<td class="p-2 text-right font-semibold">' + m(r.nbv) + '</td><td class="p-2 text-center text-xs ' + (r.note.indexOf('ขาด') === 0 ? 'text-red-500' : 'text-slate-500') + '">' + (r.note || '') + '</td></tr>';
    }).join('') : '<tr><td colspan="9" class="p-4 text-center text-slate-400">ไม่มีเครื่องมือ</td></tr>');
    // ✅ แถวรวมย้ายไปไว้ใน tfoot เพื่อให้แสดงทุกหน้าเมื่อแบ่งหน้า
    document.getElementById('dp-sum-foot').innerHTML = ''
      + '<tr class="border-t-2 dark:border-slate-600 font-bold bg-slate-50 dark:bg-slate-900/40"><td class="p-2 text-center" colspan="4">รวม ' + ok.length + ' รายการที่คำนวณได้ ณ ' + _dpFmt(asof) + '</td>'
      + '<td class="p-2 text-right">' + _dpMoney(tot.cost) + '</td><td class="p-2 text-right">' + _dpMoney(tot.annual) + '</td><td class="p-2 text-right">' + _dpMoney(tot.acc) + '</td><td class="p-2 text-right">' + _dpMoney(tot.nbv) + '</td><td></td></tr>';
  }

  function dpExportSummary() {
    if (!_dpSummary.length) dpSummaryAll();
    if (!_dpSummary.length) { showToast('ไม่มีข้อมูล', 'warning'); return; }
    var asof = document.getElementById('dp-asof').value || _dpTodayIso();
    function n(v) { return isNaN(v) ? '' : (Math.round(v * 100) / 100).toFixed(2); }
    var h = '<html><head><meta charset="UTF-8"></head><body><table border="1">'
      + '<tr><td colspan="9" style="font-weight:bold;font-size:14pt">สรุปค่าเสื่อมราคาเครื่องมือ (วิธีเส้นตรง ราคาซาก 1 บาท) ณ ' + _dpFmt(asof) + '</td></tr>'
      + '<tr style="background:#e2e8f0;font-weight:bold"><td>ลำดับ</td><td>รหัสครุภัณฑ์</td><td>ชื่อเครื่องมือ</td><td>วันเริ่มคิด</td><td>อายุ (ปี)</td><td>ราคาทุน</td><td>ค่าเสื่อม/ปี</td><td>ค่าเสื่อมสะสม</td><td>มูลค่าสุทธิ</td><td>หมายเหตุ</td></tr>'
      + _dpSummary.map(function (r, i) {
          return '<tr><td>' + (i + 1) + '</td><td style="mso-number-format:\'\\@\'">' + _pmEsc(r.code) + '</td><td>' + _pmEsc(r.name) + '</td><td>' + (r.start || '') + '</td><td>' + (isNaN(r.life) ? '' : r.life)
            + '</td><td>' + n(r.cost) + '</td><td>' + n(r.annual) + '</td><td>' + n(r.acc) + '</td><td>' + n(r.nbv) + '</td><td>' + r.note + '</td></tr>';
        }).join('') + '</table></body></html>';
    var blob = new Blob(['\uFEFF' + h], { type: 'application/vnd.ms-excel;charset=utf-8;' });
    var a = document.createElement('a'); a.href = URL.createObjectURL(blob); a.download = 'ค่าเสื่อมราคา_' + asof + '.xls';
    document.body.appendChild(a); a.click(); document.body.removeChild(a);
  }

  // ============================================================
  // ✅ แบ่งหน้าแบบทั่วไป (ใช้กับ: ข้อมูล/สอบเทียบ, สถานะเครื่องมือ, เพิ่มเครื่องมือ, แผนและสรุปผล)
  //    - ตาราง/ตัวกรองเดิมไม่ต้องแก้: แถวที่ตัวกรองซ่อน (style.display='none') จะไม่ถูกนับ
  //    - แถวย่อยที่ตามหลังแถวหลัก (เช่น แผงกรอกข้อมูลที่กางออก) ถูกซ่อน/แสดงไปพร้อมแถวหลัก
  //    - เฝ้าดูการเปลี่ยนแปลงของตารางอัตโนมัติ (โหลดใหม่ / ค้นหา / กรอง)
  // ============================================================
  var _gPagers = {};

  function _gpEnsureCss() {
    if (document.getElementById('gp-css')) return;
    var st = document.createElement('style');
    st.id = 'gp-css';
    st.textContent = '.gp-hide{display:none!important}';
    document.head.appendChild(st);
  }

  function attachTablePager(key, tbodyId, unitSelector) {
    var tb = document.getElementById(tbodyId);
    if (!tb || _gPagers[key]) return;
    _gpEnsureCss();
    var wrap = tb.closest('.overflow-x-auto') || tb.closest('table');
    var box = document.createElement('div');
    box.id = 'gp-' + key;
    wrap.parentNode.insertBefore(box, wrap.nextSibling);
    var P = _gPagers[key] = { key: key, tb: tb, box: box, sel: unitSelector || 'tr', page: 1, size: 10, lastCount: -1, lastSig: '', busy: false };
    var mo = new MutationObserver(function (muts) {
      if (P.busy) return;
      var reset = false, relevant = false;
      muts.forEach(function (m) {
        if (m.type === 'childList' && m.target === tb) relevant = true;
        if (m.type === 'attributes' && m.attributeName === 'style' && m.target.parentNode === tb && m.target.matches && m.target.matches(P.sel)) { relevant = true; reset = true; }
      });
      if (!relevant) return;
      clearTimeout(P.t);
      P.t = setTimeout(function () { _gpRender(key, reset); }, 0);
    });
    mo.observe(tb, { childList: true, subtree: true, attributes: true, attributeFilter: ['style'] });
    _gpRender(key, true);
  }

  function _gpUnits(P) {
    return Array.prototype.filter.call(P.tb.children, function (tr) {
      if (tr.tagName !== 'TR' || !tr.matches(P.sel)) return false;
      return !(tr.cells.length === 1 && tr.cells[0].hasAttribute('colspan'));   // แถว "กำลังโหลด / ไม่พบข้อมูล"
    });
  }

  function _gpRender(key, resetPage) {
    var P = _gPagers[key]; if (!P) return;
    P.busy = true;
    try {
      var units = _gpUnits(P);
      var matched = units.filter(function (u) { return u.style.display !== 'none'; });
      var sig = matched.length + '/' + units.length;
      if (resetPage || sig !== P.lastSig) { if (resetPage || matched.length !== P.lastCount) P.page = 1; }
      P.lastSig = sig; P.lastCount = matched.length;

      var total = matched.length, size = P.size, pages = Math.max(1, Math.ceil(total / size));
      if (P.page > pages) P.page = pages;
      var from = (P.page - 1) * size, to = Math.min(from + size, total);
      var onPage = {};
      matched.slice(from, to).forEach(function (u) { onPage[units.indexOf(u)] = true; });

      // ซ่อน/แสดงแถวหลัก + แถวย่อยที่ตามหลัง
      var kids = P.tb.children, curShow = true, ui = -1;
      for (var i = 0; i < kids.length; i++) {
        var tr = kids[i];
        var isUnit = units.indexOf(tr) !== -1;
        if (isUnit) { ui = units.indexOf(tr); curShow = !!onPage[ui]; }
        else if (ui === -1) { tr.classList.remove('gp-hide'); continue; }      // แถว placeholder
        tr.classList.toggle('gp-hide', !curShow);
      }

      if (!units.length) { P.box.innerHTML = ''; return; }
      P.box.innerHTML = _gpHtml(key, P.page, pages, size, total, units.length, from, to);
    } finally {
      setTimeout(function () { P.busy = false; }, 0);
    }
  }

  function _gpHtml(key, p, pages, size, total, all, from, to) {
    var btn = function (label, target, o) {
      o = o || {};
      return '<button type="button" ' + (o.disabled ? 'disabled' : 'onclick="gpGo(\'' + key + '\',' + target + ')"') + ' title="' + (o.title || '') + '" class="min-w-[36px] h-9 px-2.5 rounded-full border text-sm transition flex items-center justify-center '
        + (o.active ? 'bg-slate-900 text-white border-slate-900 dark:bg-primary-600 dark:border-primary-600 font-bold'
           : o.disabled ? 'border-slate-200 dark:border-slate-700 text-slate-300 dark:text-slate-600 cursor-not-allowed'
           : 'border-slate-200 dark:border-slate-600 text-slate-600 dark:text-slate-300 hover:bg-slate-100 dark:hover:bg-slate-700') + '">' + label + '</button>';
    };
    var icon = function (n) { return '<span class="material-symbols-outlined text-lg">' + n + '</span>'; };
    var nums = [], seen = {};
    [1, pages, p - 1, p, p + 1].forEach(function (n) { if (n >= 1 && n <= pages && !seen[n]) { seen[n] = 1; nums.push(n); } });
    nums.sort(function (a, b) { return a - b; });
    var numHtml = '', prev = 0;
    nums.forEach(function (n) { if (prev && n - prev > 1) numHtml += '<span class="px-1 text-slate-400">…</span>'; numHtml += btn(n, n, { active: n === p }); prev = n; });
    return '<div class="flex flex-col md:flex-row md:items-center justify-between gap-3 px-5 py-3 border-t dark:border-slate-700 bg-slate-50/50 dark:bg-slate-800/50">'
      + '<div class="flex flex-wrap items-center gap-4 text-sm text-slate-600 dark:text-slate-300">'
      +   '<span>แสดง <b>' + (total ? from + 1 : 0) + '</b> ถึง <b>' + to + '</b> จากทั้งหมด <b>' + total.toLocaleString() + '</b> รายการ'
      +     (total !== all ? ' <span class="text-xs text-slate-400">(กรองจาก ' + all.toLocaleString() + ')</span>' : '') + '</span>'
      +   '<label class="flex items-center gap-2">แสดงต่อหน้า: <select onchange="gpSize(\'' + key + '\', this.value)" class="px-3 py-1.5 border border-slate-200 dark:border-slate-600 rounded-lg bg-white dark:bg-slate-700 text-sm">'
      +     [10, 25, 50, 100].map(function (s) { return '<option value="' + s + '"' + (s === size ? ' selected' : '') + '>' + s + '</option>'; }).join('')
      +   '</select></label>'
      + '</div>'
      + '<div class="flex items-center gap-1.5 flex-wrap">'
      +   btn(icon('keyboard_double_arrow_left'), 1, { disabled: p === 1, title: 'หน้าแรก' })
      +   btn(icon('chevron_left'), p - 1, { disabled: p === 1, title: 'ก่อนหน้า' })
      +   numHtml
      +   btn(icon('chevron_right'), p + 1, { disabled: p === pages, title: 'ถัดไป' })
      +   btn(icon('keyboard_double_arrow_right'), pages, { disabled: p === pages, title: 'หน้าสุดท้าย' })
      + '</div></div>';
  }

  function gpGo(key, n) {
    var P = _gPagers[key]; if (!P) return;
    P.page = n; _gpRender(key, false);
    var card = P.tb.closest('.rounded-xl') || P.tb.closest('table');
    if (card && card.scrollIntoView) card.scrollIntoView({ behavior: 'smooth', block: 'start' });
  }
  function gpSize(key, v) { var P = _gPagers[key]; if (!P) return; P.size = parseInt(v, 10) || 10; P.page = 1; _gpRender(key, false); }

  function _initTablePagers() {
    attachTablePager('table',     'equipment-table-body', 'tr[data-name]');   // ข้อมูล/สอบเทียบ (มีแถวกางออกตามหลัง)
    attachTablePager('eqstatus',  'eq-status-tbody',      'tr');              // สถานะเครื่องมือ
    attachTablePager('formsum',   'form-summary-tbody',   'tr');              // เพิ่มเครื่องมือ
    attachTablePager('report',    'cal-report-table-body','tr');              // แผนและสรุปผล
    attachTablePager('impact',    'ia-tbody',             'tr');              // ประเมินผลกระทบย้อนหลัง
    attachTablePager('verify',    'vr-tbody',             'tr');              // ทวนสอบก่อนใช้งาน
    attachTablePager('icheck',    'ic-tbody',             'tr');              // ตรวจสอบระหว่างรอบ
    attachTablePager('calreview', 'rv-tbody',             'tr');              // ทบทวนรอบสอบเทียบ
    attachTablePager('deprsum',   'dp-sum-body',          'tr');              // สรุปค่าเสื่อมราคาทั้งหน่วย
  }
  if (document.readyState === 'loading') document.addEventListener('DOMContentLoaded', _initTablePagers);
  else _initTablePagers();

  // ============================================================
  // ✅ ผู้ดูแลระบบ: แสดงทีละส่วน (สถานที่ตั้ง / บริษัทผู้ให้บริการ / ประเภทเครื่องมือ / บัญชีผู้ใช้)
  // ============================================================
  var ADMIN_SECTIONS = {
    location:   { icon: 'location_on',     color: 'text-emerald-600', title: 'จัดการสถานที่ตั้ง',       sub: 'เพิ่ม / แก้ไข / ลบ สถานที่ติดตั้งเครื่องมือ' },
    department: { icon: 'business',        color: 'text-amber-600',   title: 'บริษัทผู้ให้บริการ',       sub: 'เพิ่ม / แก้ไข / ลบ บริษัทหรือห้องปฏิบัติการผู้ให้บริการสอบเทียบ (ใช้ในช่อง "ห้องปฏิบัติการที่ทำการสอบเทียบ")' },
    category:   { icon: 'category',        color: 'text-purple-600',  title: 'จัดการประเภทเครื่องมือ',   sub: 'เพิ่ม / แก้ไข / ลบ ประเภทเครื่องมือ' },
    users:      { icon: 'manage_accounts', color: 'text-blue-600',    title: 'บัญชีผู้ใช้งาน',           sub: 'จัดการบัญชีผู้ใช้งานระบบ สิทธิ์ และหน่วยงาน' },
    execlink:   { icon: 'monitoring',      color: 'text-amber-600',   title: 'ลิงก์แดชบอร์ดผู้บริหาร',    sub: 'ลิงก์สำหรับผู้บริหารดูภาพรวมโดยไม่ต้องเข้าสู่ระบบ' }
  };
  function _showAdminSection(sec) {
    document.querySelectorAll('#page-admin .admin-sec').forEach(function (el) {
      el.style.display = (el.id === 'admin-sec-' + sec) ? '' : 'none';
    });
    var c = ADMIN_SECTIONS[sec] || ADMIN_SECTIONS.users;
    var ic = document.getElementById('admin-title-icon'), tt = document.getElementById('admin-title-text'), sb = document.getElementById('admin-title-sub');
    if (ic) { ic.textContent = c.icon; ic.className = 'material-symbols-outlined ' + c.color; }
    if (tt) tt.textContent = c.title;
    if (sb) sb.textContent = c.sub;
  }
  function _adminFilterList(listId, q) {
    q = String(q || '').toLowerCase().trim();
    document.querySelectorAll('#' + listId + ' > li').forEach(function (li) {
      li.style.display = !q || li.textContent.toLowerCase().indexOf(q) !== -1 ? '' : 'none';
    });
  }

  // ============================================================
  // ✅ แดชบอร์ดผู้บริหาร: อายุเครื่องมือ / มูลค่า / ค่าเสื่อมราคา (วิธีเส้นตรง ราคาซาก 1 บาท)
  //    ใช้ข้อมูลชุดเดียวกับแดชบอร์ด (กรองตามหน่วยงานของผู้ใช้แล้ว)
  // ============================================================
  var _execAgeChart = null, _execValChart = null;

  function _execShortBaht(n) {
    if (!isFinite(n)) return '-';
    var a = Math.abs(n);
    if (a >= 1e6) return (n / 1e6).toLocaleString('th-TH', { maximumFractionDigits: 2 }) + ' ล้าน';
    if (a >= 1e3) return (n / 1e3).toLocaleString('th-TH', { maximumFractionDigits: 1 }) + ' พัน';
    return n.toLocaleString('th-TH', { maximumFractionDigits: 0 });
  }

  function _execCompute(data) {
    var today = _dpTodayIso(), tNow = _dpUTC(today);
    var groups = {}, tot = { n: 0, valued: 0, aged: 0, ageSum: 0, over: 0, cost: 0, acc: 0, nbv: 0, missing: 0 };
    tot.retired = 0;
    data.forEach(function (e) {
      if (String(e['สถานะเครื่องมือ'] || '').trim() === 'จำหน่ายออก') { tot.retired++; return; }   // ปลดระวาง: ไม่นับเป็นสินทรัพย์
      var g = e['ประเภทเครื่องมือ'] || 'ไม่ระบุ';
      var G = groups[g] || (groups[g] = { name: g, n: 0, aged: 0, ageSum: 0, ageMax: 0, over: 0, cost: 0, acc: 0, nbv: 0, b: [0, 0, 0, 0, 0] });
      G.n++; tot.n++;
      var start = _dpStartOf(e), cost = _dpNum(e['มูลค่าเครื่องมือ']), life = _dpNum(e['อายุการใช้งาน(ปี)'] || e['อายุการใช้งาน (ปี)']);
      var age = null;
      if (start && /^\d{4}-\d{2}-\d{2}$/.test(start)) {
        age = Math.max(0, (tNow - _dpUTC(start)) / (365.25 * 86400000));
        G.aged++; G.ageSum += age; G.ageMax = Math.max(G.ageMax, age); tot.aged++; tot.ageSum += age;
        if (life > 0 && age > life) { G.over++; tot.over++; }
      }
      G.b[age === null ? 4 : age <= 3 ? 0 : age <= 5 ? 1 : age <= 10 ? 2 : 3]++;
      if (cost > 0) {
        G.cost += cost; tot.cost += cost;
        var acc = 0;
        if (life > 0 && start) { var sv = Math.min(1, cost); acc = dpAccumAt(cost, sv, life, start, 'SL', today); tot.valued++; }
        else tot.missing++;
        G.acc += acc; G.nbv += cost - acc; tot.acc += acc; tot.nbv += cost - acc;
      } else tot.missing++;
    });
    return { groups: Object.keys(groups).map(function (k) { return groups[k]; }), tot: tot };
  }

  function renderDashboardExec(data) {
    var wrap = document.getElementById('dash-exec');
    if (!wrap) return;
    if (_execAgeChart) { try { _execAgeChart.destroy(); } catch (e) {} _execAgeChart = null; }
    if (_execValChart) { try { _execValChart.destroy(); } catch (e) {} _execValChart = null; }
    var cards = document.getElementById('dash-exec-cards'), tb = document.getElementById('dash-exec-table'), note = document.getElementById('dash-exec-note');
    if (!data || !data.length) {
      cards.innerHTML = ''; tb.innerHTML = '<tr><td colspan="9" class="px-3 py-6 text-center text-slate-400 italic">ยังไม่มีเครื่องมือของหน่วยนี้</td></tr>';
      if (note) note.textContent = ''; return;
    }
    var R = _execCompute(data), T = R.tot;
    var avgAge = T.aged ? T.ageSum / T.aged : NaN, pctNbv = T.cost ? T.nbv / T.cost * 100 : NaN;
    function card(icon, color, label, val, sub) {
      return '<div class="bg-white dark:bg-slate-800 rounded-xl shadow-sm border border-slate-200 dark:border-slate-700 p-4">'
        + '<div class="flex items-center gap-2 text-xs text-slate-500 mb-1"><span class="material-symbols-outlined text-lg ' + color + '">' + icon + '</span>' + label + '</div>'
        + '<div class="text-xl font-black text-slate-800 dark:text-white">' + val + '</div>'
        + (sub ? '<div class="text-[11px] text-slate-400 mt-0.5">' + sub + '</div>' : '') + '</div>';
    }
    cards.innerHTML =
        card('payments', 'text-blue-600', 'มูลค่าเครื่องมือรวม', _execShortBaht(T.cost) + ' ฿', T.cost.toLocaleString('th-TH', { maximumFractionDigits: 2 }) + ' บาท')
      + card('trending_down', 'text-red-500', 'ค่าเสื่อมราคาสะสม', _execShortBaht(T.acc) + ' ฿', 'ณ ' + _dpFmt(_dpTodayIso()))
      + card('account_balance_wallet', 'text-emerald-600', 'มูลค่าสุทธิคงเหลือ', _execShortBaht(T.nbv) + ' ฿', isFinite(pctNbv) ? 'คงเหลือ ' + pctNbv.toFixed(1) + '% ของมูลค่า' : '')
      + card('hourglass_bottom', 'text-amber-600', 'อายุเฉลี่ยเครื่องมือ', isFinite(avgAge) ? avgAge.toFixed(1) + ' ปี' : '-', 'จาก ' + T.aged + ' เครื่องที่มีวันเริ่มใช้')
      + card('warning', 'text-orange-600', 'เกินอายุการใช้งาน', T.over + ' เครื่อง', T.aged ? (T.over / T.aged * 100).toFixed(1) + '% ควรพิจารณาจัดหาทดแทน' : '');
    if (note) note.textContent = 'คำนวณค่าเสื่อมได้ ' + T.valued + ' จาก ' + T.n + ' เครื่อง' + (T.missing ? ' · ข้อมูลมูลค่า/อายุ/วันเริ่มใช้ไม่ครบ ' + T.missing + ' เครื่อง' : '') + (T.retired ? ' · ไม่รวมเครื่องปลดระวาง ' + T.retired + ' เครื่อง' : '');

    // ตาราง (เรียงตามมูลค่า)
    var G = R.groups.slice().sort(function (a, b) { return b.cost - a.cost || b.n - a.n; });
    function m(v) { return v ? v.toLocaleString('th-TH', { minimumFractionDigits: 2, maximumFractionDigits: 2 }) : '-'; }
    tb.innerHTML = G.map(function (g) {
      var pct = g.cost ? g.nbv / g.cost * 100 : NaN;
      return '<tr class="hover:bg-slate-50 dark:hover:bg-slate-700/30"><td class="px-3 py-2">' + _pmEsc(g.name) + '</td><td class="px-3 py-2 text-right">' + g.n + '</td>'
        + '<td class="px-3 py-2 text-right">' + (g.aged ? (g.ageSum / g.aged).toFixed(1) : '-') + '</td><td class="px-3 py-2 text-right">' + (g.aged ? g.ageMax.toFixed(1) : '-') + '</td>'
        + '<td class="px-3 py-2 text-right ' + (g.over ? 'text-orange-600 font-bold' : 'text-slate-400') + '">' + g.over + '</td>'
        + '<td class="px-3 py-2 text-right">' + m(g.cost) + '</td><td class="px-3 py-2 text-right text-red-500">' + m(g.acc) + '</td>'
        + '<td class="px-3 py-2 text-right font-semibold">' + m(g.nbv) + '</td>'
        + '<td class="px-3 py-2 text-right">' + (isFinite(pct) ? '<span class="' + (pct < 20 ? 'text-red-500' : pct < 50 ? 'text-amber-600' : 'text-emerald-600') + ' font-bold">' + pct.toFixed(1) + '%</span>' : '-') + '</td></tr>';
    }).join('') + '<tr class="bg-slate-50 dark:bg-slate-900/40 font-bold border-t-2 dark:border-slate-600"><td class="px-3 py-2">รวม</td><td class="px-3 py-2 text-right">' + T.n + '</td>'
      + '<td class="px-3 py-2 text-right">' + (isFinite(avgAge) ? avgAge.toFixed(1) : '-') + '</td><td></td><td class="px-3 py-2 text-right">' + T.over + '</td>'
      + '<td class="px-3 py-2 text-right">' + m(T.cost) + '</td><td class="px-3 py-2 text-right text-red-500">' + m(T.acc) + '</td><td class="px-3 py-2 text-right">' + m(T.nbv) + '</td>'
      + '<td class="px-3 py-2 text-right">' + (isFinite(pctNbv) ? pctNbv.toFixed(1) + '%' : '-') + '</td></tr>';

    if (typeof Chart === 'undefined') return;
    var dark = document.documentElement.classList.contains('dark'), tick = dark ? '#94a3b8' : '#64748b', grid = dark ? 'rgba(148,163,184,.15)' : 'rgba(148,163,184,.25)';
    function short(s) { s = String(s); return s.length > 22 ? s.slice(0, 21) + '…' : s; }

    // กราฟอายุ: แท่งซ้อนตามช่วงอายุ (กลุ่มที่มีจำนวนมากสุด 8 กลุ่ม)
    var topN = R.groups.slice().sort(function (a, b) { return b.n - a.n; }).slice(0, 8);
    var buckets = [['≤ 3 ปี', '#10b981'], ['3–5 ปี', '#3b82f6'], ['5–10 ปี', '#f59e0b'], ['> 10 ปี', '#ef4444'], ['ไม่ระบุวันเริ่มใช้', '#cbd5e1']];
    var ageCtx = document.getElementById('dash-exec-age');
    if (ageCtx) _execAgeChart = new Chart(ageCtx, {
      type: 'bar',
      data: { labels: topN.map(function (g) { return short(g.name); }),
              datasets: buckets.map(function (b, i) { return { label: b[0], data: topN.map(function (g) { return g.b[i]; }), backgroundColor: b[1], borderRadius: 3 }; }) },
      options: { indexAxis: 'y', responsive: true, maintainAspectRatio: false,
        plugins: { legend: { position: 'bottom', labels: { boxWidth: 10, font: { size: 11 }, color: tick } },
                   tooltip: { callbacks: { afterBody: function (it) { var g = topN[it[0].dataIndex]; return g.aged ? 'อายุเฉลี่ย ' + (g.ageSum / g.aged).toFixed(1) + ' ปี' : ''; } } } },
        scales: { x: { stacked: true, ticks: { precision: 0, color: tick }, grid: { color: grid }, title: { display: true, text: 'จำนวนเครื่อง', color: tick } },
                  y: { stacked: true, ticks: { color: tick, font: { size: 11 } }, grid: { display: false } } } }
    });

    // กราฟมูลค่า: มูลค่ารวม vs มูลค่าสุทธิ (กลุ่มที่มูลค่าสูงสุด 8 กลุ่ม)
    var topV = R.groups.filter(function (g) { return g.cost > 0; }).sort(function (a, b) { return b.cost - a.cost; }).slice(0, 8);
    var valCtx = document.getElementById('dash-exec-value');
    if (valCtx) _execValChart = new Chart(valCtx, {
      type: 'bar',
      data: { labels: topV.map(function (g) { return short(g.name); }),
              datasets: [{ label: 'มูลค่ารวม', data: topV.map(function (g) { return Math.round(g.cost); }), backgroundColor: '#93c5fd', borderRadius: 3 },
                         { label: 'มูลค่าสุทธิคงเหลือ', data: topV.map(function (g) { return Math.round(g.nbv); }), backgroundColor: '#10b981', borderRadius: 3 }] },
      options: { indexAxis: 'y', responsive: true, maintainAspectRatio: false,
        plugins: { legend: { position: 'bottom', labels: { boxWidth: 10, font: { size: 11 }, color: tick } },
                   tooltip: { callbacks: { label: function (c) { return c.dataset.label + ': ' + Number(c.raw).toLocaleString('th-TH') + ' บาท'; } } } },
        scales: { x: { ticks: { color: tick, callback: function (v) { return _execShortBaht(v); } }, grid: { color: grid } },
                  y: { ticks: { color: tick, font: { size: 11 } }, grid: { display: false } } } }
    });
  }

  // ============================================================
  // ✅ หยุดใช้งาน / ป้าย "ห้ามใช้งาน" SD-IN-06-Patho-QMU-003 / นำกลับมาใช้ / ปลดระวาง
  // ============================================================
  var TAG_DOC = {
    code: 'SD-IN-06-Patho-QMU-003', title: 'ป้ายบ่งชี้เครื่องมืออุปกรณ์', org: 'สาขาวิชาพยาธิวิทยา',
    rev: '00', effective: '15 ต.ค. 2567',
    preparer: ['จิรัสม์ตชา นิลโมจน์', 'นางสาวจิรัสม์ตชา นิลโมจน์'],
    reviewer: ['รุ่งเรือง จารุมโนกุล', 'ทนพ.รุ่งเรือง จารุมโนกุล'],
    approver: ['คณุตม์ จารุธรรมโสภณ', 'ผศ.ดร.นพ.คณุตม์ จารุธรรมโสภณ']
  };
  var STOP_CAUSES = ['เครื่องชำรุด', 'รอการซ่อม', 'ไม่ผ่านการสอบเทียบ', 'อื่นๆ'];
  var STOPPED_STATUSES = ['สอบเทียบไม่ผ่านเกณฑ์', 'เครื่องมือเสีย', 'ส่งซ่อม', 'ระงับการใช้งาน'];
  var RETIRE_REASONS = ['ชำรุด ซ่อมไม่คุ้มค่า', 'เกินอายุการใช้งาน', 'ล้าสมัย / ไม่มีอะไหล่', 'อื่นๆ'];

  function _isRetired(item) { return String((item && item['สถานะเครื่องมือ']) || '').trim() === 'จำหน่ายออก'; }
  function _isStopped(item) { return STOPPED_STATUSES.indexOf(String((item && item['สถานะเครื่องมือ']) || '').trim()) !== -1; }
  function _canRetire() {
    return !!currentUser && (currentUser.role === 'Admin' || currentUser.isHead === 'true' || currentUser.isHead === true || String(currentUser.isHead) === '1');
  }
  function _todayIso() { var d = new Date(); return d.getFullYear() + '-' + ('0' + (d.getMonth() + 1)).slice(-2) + '-' + ('0' + d.getDate()).slice(-2); }
  function _thDate(iso) {
    if (!iso) return '';
    var d = new Date(String(iso).slice(0, 10) + 'T00:00:00');
    return isNaN(d) ? iso : d.toLocaleDateString('th-TH', { day: 'numeric', month: 'long', year: 'numeric' });
  }
  function _eqByIdx(i) { return cachedEquipments && cachedEquipments[i]; }

  // modal พื้นฐาน (สไตล์เดียวกับหน้าต่างอัปเดตสถานะเดิม)
  function _miniModal(title, sub, color, bodyHtml, saveLabel, onSave) {
    var ov = document.createElement('div');
    ov.style.cssText = 'position:fixed;inset:0;background:rgba(0,0,0,.6);z-index:9999;display:flex;align-items:center;justify-content:center;padding:16px;';
    ov.innerHTML = '<div style="background:#fff;border-radius:16px;width:100%;max-width:540px;max-height:92vh;overflow:auto;box-shadow:0 20px 60px rgba(0,0,0,.3);color:#1e293b;">'
      + '<div style="background:' + color + ';color:#fff;padding:16px 20px;border-radius:16px 16px 0 0;display:flex;justify-content:space-between;align-items:center;">'
      +   '<div><div style="font-size:16px;font-weight:700;">' + title + '</div><div style="font-size:11px;opacity:.85;margin-top:2px;">' + sub + '</div></div>'
      +   '<button data-x style="background:none;border:none;color:#fff;font-size:22px;cursor:pointer;">✕</button></div>'
      + '<div style="padding:18px 20px;font-size:13px;">' + bodyHtml + '<div data-err style="color:#dc2626;font-size:12px;margin-top:8px;"></div></div>'
      + '<div style="padding:14px 20px;border-top:1px solid #e2e8f0;display:flex;gap:10px;justify-content:flex-end;">'
      +   '<button data-x style="padding:9px 20px;border:1.5px solid #94a3b8;background:#fff;color:#64748b;border-radius:8px;font-size:13px;font-weight:600;cursor:pointer;">ยกเลิก</button>'
      +   '<button data-save style="padding:9px 24px;border:none;background:' + color + ';color:#fff;border-radius:8px;font-size:13px;font-weight:700;cursor:pointer;">' + saveLabel + '</button>'
      + '</div></div>';
    document.body.appendChild(ov);
    var close = function () { if (ov.parentNode) ov.parentNode.removeChild(ov); };
    ov.querySelectorAll('[data-x]').forEach(function (b) { b.onclick = close; });
    ov.addEventListener('click', function (e) { if (e.target === ov) close(); });
    var errEl = ov.querySelector('[data-err]'), btn = ov.querySelector('[data-save]');
    btn.onclick = function () {
      errEl.textContent = '';
      onSave(ov, function (msg) { errEl.textContent = msg; }, function (busy) { btn.disabled = busy; btn.style.opacity = busy ? .6 : 1; }, close);
    };
    return ov;
  }
  var _fs = 'width:100%;padding:8px 10px;border:1.5px solid #cbd5e1;border-radius:6px;font-size:13px;box-sizing:border-box;font-family:inherit;';
  var _lb = 'display:block;font-size:12px;font-weight:600;color:#475569;margin:10px 0 4px;';
  function _esc2(v) { return String(v == null ? '' : v).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;'); }

  // ---------- หยุดใช้งาน ----------
  function openStopUseModal(eqIdx, preset) {
    var it = _eqByIdx(eqIdx); if (!it) return;
    if (_isRetired(it)) { showToast('เครื่องนี้ปลดระวางแล้ว', 'warning'); return; }
    var me = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser && currentUser.username) || '';
    var body =
        '<div style="display:grid;grid-template-columns:1fr 1fr;gap:0 12px;">'
      +   '<div><label style="' + _lb + '">ชื่อเครื่องมือ</label><input data-f="eqName" value="' + _esc2(it['ชื่อเครื่องมือ']) + '" style="' + _fs + '"></div>'
      +   '<div><label style="' + _lb + '">รหัสครุภัณฑ์</label><input data-f="eqCode" value="' + _esc2(it['รหัสครุภัณฑ์']) + '" style="' + _fs + '"></div>'
      +   '<div><label style="' + _lb + '">ห้องปฏิบัติการ</label><input data-f="lab" value="' + _esc2(it['สถานที่ติดตั้ง']) + '" style="' + _fs + '"></div>'
      +   '<div><label style="' + _lb + '">ผู้รับผิดชอบ</label><input data-f="owner" value="' + _esc2(me) + '" style="' + _fs + '"></div>'
      +   '<div><label style="' + _lb + '">วันที่หยุดใช้เครื่องมือ *</label><input type="date" data-f="date" value="' + _todayIso() + '" style="' + _fs + '"></div>'
      + '</div>'
      + '<label style="' + _lb + '">สาเหตุ * (เลือกได้มากกว่า 1 ข้อ)</label>'
      + '<div style="display:flex;flex-wrap:wrap;gap:8px 18px;">'
      +   STOP_CAUSES.map(function (c) { return '<label style="display:flex;align-items:center;gap:6px;cursor:pointer;"><input type="checkbox" data-cause value="' + c + '" style="width:16px;height:16px;accent-color:#dc2626;">' + c + '</label>'; }).join('')
      + '</div>'
      + '<input data-f="causeOther" placeholder="ระบุสาเหตุอื่นๆ" style="' + _fs + 'margin-top:8px;display:none;">'
      + '<div style="font-size:11px;color:#92400e;background:#fffbeb;border:1px solid #fde68a;border-radius:6px;padding:6px 10px;margin-top:8px;">📋 สาเหตุ "เครื่องชำรุด" หรือ "ไม่ผ่านการสอบเทียบ" ระบบจะสร้างแบบประเมินผลกระทบย้อนหลังให้อัตโนมัติ และต้องอนุมัติก่อนนำเครื่องกลับมาใช้</div>'
      + '<label style="' + _lb + '">หมายเหตุ</label><textarea data-f="note" rows="2" style="' + _fs + 'resize:vertical;"></textarea>'
      + '<label style="display:flex;align-items:center;gap:8px;margin-top:12px;font-weight:600;color:#b91c1c;cursor:pointer;"><input type="checkbox" data-print checked style="width:16px;height:16px;accent-color:#dc2626;"> พิมพ์ป้าย "ห้ามใช้งาน" ทันทีหลังบันทึก</label>';
    var ov = _miniModal('⛔ หยุดใช้งานเครื่องมือ', _esc2((it['รหัสครุภัณฑ์'] || '-') + ' · ' + (it['ชื่อเครื่องมือ'] || '-')), '#b91c1c', body, '💾 บันทึกหยุดใช้งาน',
      function (ov, err, busy, close) {
        var f = function (k) { return (ov.querySelector('[data-f="' + k + '"]') || {}).value || ''; };
        var causes = Array.prototype.map.call(ov.querySelectorAll('[data-cause]:checked'), function (x) { return x.value; });
        if (!f('date')) return err('กรุณาระบุวันที่หยุดใช้');
        if (!causes.length) return err('กรุณาเลือกสาเหตุอย่างน้อย 1 ข้อ');
        if (causes.indexOf('อื่นๆ') !== -1 && !f('causeOther').trim()) return err('กรุณาระบุสาเหตุอื่นๆ');
        var payload = { eqId: it['ID'], eqCode: f('eqCode'), eqName: f('eqName'), lab: f('lab'), owner: f('owner'), date: f('date'),
                        causes: causes, causeOther: f('causeOther').trim(), note: f('note').trim(), by: me };
        var wantPrint = ov.querySelector('[data-print]').checked;
        busy(true);
        google.script.run.withSuccessHandler(function (res) {
          busy(false);
          if (!res || !res.success) return err((res && res.message) || 'บันทึกไม่สำเร็จ');
          close();
          showToast('⛔ บันทึกหยุดใช้งานแล้ว · สถานะ "' + res.status + '"' + (res.iaId ? ' · สร้างแบบประเมินผลกระทบ ' + res.iaId + ' แล้ว' : ''), 'success');
          if (wantPrint) printStopTag({ eqName: payload.eqName, eqCode: payload.eqCode, lab: payload.lab, owner: payload.owner,
                                        date: payload.date, causes: causes, causeOther: payload.causeOther, returnDate: '' });
          _reloadStatusViews();
        }).withFailureHandler(function (e) { busy(false); err(e && e.message || String(e)); }).stopEquipmentUse(payload);
      });
    var other = ov.querySelector('[data-f="causeOther"]');
    ov.querySelectorAll('[data-cause]').forEach(function (cb) {
      cb.addEventListener('change', function () { var on = !!ov.querySelector('[data-cause][value="อื่นๆ"]:checked'); other.style.display = on ? '' : 'none'; });
    });
    if (preset) {   // เปิดจากผลตรวจระหว่างรอบไม่ผ่าน
      (preset.causes || []).forEach(function (c) { var cb = ov.querySelector('[data-cause][value="' + c + '"]'); if (cb) cb.checked = true; });
      if (preset.other) { other.value = preset.other; other.style.display = ''; }
    }
  }

  // ---------- นำกลับมาใช้ใหม่ ----------
  function _openReturnToServiceModalLegacy(eqIdx) {
    var it = _eqByIdx(eqIdx); if (!it) return;
    var me = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser && currentUser.username) || '';
    var warn = it['สถานะเครื่องมือ'] === 'สอบเทียบไม่ผ่านเกณฑ์'
      ? '<div style="background:#fef3c7;border:1px solid #fcd34d;color:#92400e;border-radius:8px;padding:10px 12px;margin-bottom:6px;font-size:12px;">⚠️ เครื่องหยุดใช้เพราะ <b>ไม่ผ่านการสอบเทียบ</b> — ควรซ่อม/ปรับตั้ง และสอบเทียบหรือทวนสอบให้ผ่านเกณฑ์ก่อนนำกลับมาใช้</div>' : '';
    var body = warn
      + '<label style="' + _lb + '">วันที่นำมาใช้ใหม่ *</label><input type="date" data-f="date" value="' + _todayIso() + '" style="' + _fs + '">'
      + '<label style="' + _lb + '">หลักฐาน / หมายเหตุ</label><textarea data-f="note" rows="3" placeholder="เช่น ซ่อมแล้ว ใบรับรองสอบเทียบเลขที่..., ผลทวนสอบผ่าน" style="' + _fs + 'resize:vertical;"></textarea>'
      + '<label style="display:flex;align-items:flex-start;gap:8px;margin-top:12px;cursor:pointer;"><input type="checkbox" data-ok style="width:16px;height:16px;margin-top:2px;accent-color:#16a34a;"> <span>ยืนยันว่าเครื่องได้รับการแก้ไขและตรวจสอบแล้วว่าใช้งานได้ตามเกณฑ์</span></label>';
    _miniModal('✅ นำเครื่องกลับมาใช้งาน', _esc2((it['รหัสครุภัณฑ์'] || '-') + ' · ' + (it['ชื่อเครื่องมือ'] || '-')), '#15803d', body, '💾 บันทึกนำกลับมาใช้',
      function (ov, err, busy, close) {
        var date = ov.querySelector('[data-f="date"]').value, note = ov.querySelector('[data-f="note"]').value.trim();
        if (!date) return err('กรุณาระบุวันที่นำมาใช้ใหม่');
        if (!ov.querySelector('[data-ok]').checked) return err('กรุณาติ๊กยืนยันการตรวจสอบก่อนนำกลับมาใช้');
        busy(true);
        google.script.run.withSuccessHandler(function (res) {
          busy(false);
          if (!res || !res.success) {
            if (res && res.blockedBy) { close(); showToast(res.message, 'warning'); switchPage('impact'); var _s = document.getElementById('ia-search'); if (_s) _s.value = res.blockedBy; return; }
            return err((res && res.message) || 'บันทึกไม่สำเร็จ');
          }
          close(); showToast('✅ นำเครื่องกลับมาใช้งานแล้ว', 'success'); _reloadStatusViews();
        }).withFailureHandler(function (e) { busy(false); err(e && e.message || String(e)); })
          .returnEquipmentToService(it['ID'], date, note, me);
      });
  }

  // ---------- ปลดระวาง ----------
  function openRetireModal(eqIdx) {
    var it = _eqByIdx(eqIdx); if (!it) return;
    if (!_canRetire()) { showToast('ปลดระวางได้เฉพาะหัวหน้าหน่วย / Admin', 'warning'); return; }
    var me = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser && currentUser.username) || '';
    var body =
        '<div style="background:#f1f5f9;border:1px solid #cbd5e1;border-radius:8px;padding:10px 12px;font-size:12px;color:#334155;">เมื่อปลดระวางแล้ว เครื่องจะถูกนำออกจากแผนสอบเทียบ การแจ้งเตือน และหยุดคิดค่าเสื่อม · ประวัติทั้งหมดยังเปิดดูและพิมพ์ได้</div>'
      + '<label style="' + _lb + '">วันที่ปลดระวาง *</label><input type="date" data-f="date" value="' + _todayIso() + '" style="' + _fs + '">'
      + '<label style="' + _lb + '">เหตุผล *</label><div style="display:flex;flex-wrap:wrap;gap:8px 18px;">'
      +   RETIRE_REASONS.map(function (r) { return '<label style="display:flex;align-items:center;gap:6px;cursor:pointer;"><input type="radio" name="rt-reason" value="' + r + '" style="width:16px;height:16px;">' + r + '</label>'; }).join('')
      + '</div><input data-f="reasonOther" placeholder="ระบุเหตุผลอื่นๆ" style="' + _fs + 'margin-top:8px;display:none;">'
      + '<div style="display:grid;grid-template-columns:1fr 1fr;gap:0 12px;">'
      +   '<div><label style="' + _lb + '">เลขที่หนังสือจำหน่ายพัสดุ</label><input data-f="docNo" style="' + _fs + '"></div>'
      +   '<div><label style="' + _lb + '">ผู้อนุมัติ</label><input data-f="approver" value="' + _esc2(me) + '" style="' + _fs + '"></div>'
      + '</div>'
      + '<label style="' + _lb + '">หมายเหตุ</label><textarea data-f="note" rows="2" style="' + _fs + 'resize:vertical;"></textarea>';
    var ov = _miniModal('📦 ปลดระวางเครื่องมือ', _esc2((it['รหัสครุภัณฑ์'] || '-') + ' · ' + (it['ชื่อเครื่องมือ'] || '-')), '#475569', body, '📦 ยืนยันปลดระวาง',
      function (ov, err, busy, close) {
        var f = function (k) { return (ov.querySelector('[data-f="' + k + '"]') || {}).value || ''; };
        var r = ov.querySelector('input[name="rt-reason"]:checked');
        if (!f('date')) return err('กรุณาระบุวันที่ปลดระวาง');
        if (!r) return err('กรุณาเลือกเหตุผล');
        if (r.value === 'อื่นๆ' && !f('reasonOther').trim()) return err('กรุณาระบุเหตุผลอื่นๆ');
        if (!confirm('ยืนยันปลดระวาง ' + (it['รหัสครุภัณฑ์'] || it['ID']) + ' ?')) return;
        busy(true);
        google.script.run.withSuccessHandler(function (res) {
          busy(false);
          if (!res || !res.success) return err((res && res.message) || 'บันทึกไม่สำเร็จ');
          close(); showToast('📦 ปลดระวางเรียบร้อย', 'success'); _reloadStatusViews();
        }).withFailureHandler(function (e) { busy(false); err(e && e.message || String(e)); })
          .retireEquipment({ eqId: it['ID'], date: f('date'), reason: r.value, reasonOther: f('reasonOther').trim(), docNo: f('docNo').trim(),
                             approver: f('approver').trim(), note: f('note').trim(), username: currentUser ? currentUser.username : '', by: me });
      });
    var other = ov.querySelector('[data-f="reasonOther"]');
    ov.querySelectorAll('input[name="rt-reason"]').forEach(function (x) { x.addEventListener('change', function () { other.style.display = x.value === 'อื่นๆ' && x.checked ? '' : (ov.querySelector('input[name="rt-reason"][value="อื่นๆ"]:checked') ? '' : 'none'); }); });
  }

  // ---------- พิมพ์ป้ายจากบันทึกล่าสุด ----------
  function reprintStopTag(eqIdx) {
    var it = _eqByIdx(eqIdx); if (!it) return;
    showLoader();
    google.script.run.withSuccessHandler(function (res) {
      hideLoader();
      var log = res && res.success && res.data && res.data[0];
      if (!log) { showToast('ยังไม่มีบันทึกการหยุดใช้ของเครื่องนี้ — กด "หยุดใช้งาน" ก่อน', 'warning'); return; }
      printStopTag({ eqName: log['ชื่อเครื่องมือ'], eqCode: log['รหัสครุภัณฑ์'], lab: log['ห้องปฏิบัติการ'], owner: log['ผู้รับผิดชอบ'],
                     date: log['วันที่หยุดใช้'], causes: String(log['สาเหตุ'] || '').split(/\s*,\s*/), causeOther: log['สาเหตุอื่นๆ'],
                     returnDate: log['วันที่นำมาใช้ใหม่'] });
    }).withFailureHandler(function (e) { hideLoader(); showToast('โหลดบันทึกไม่สำเร็จ: ' + (e && e.message || e), 'danger'); }).getStopUseLogs(it['ID']);
  }

  // ---------- ป้าย "ห้ามใช้งาน" ตามแบบ SD-IN-06-Patho-QMU-003 (1 ป้ายต่อ A5 แนวนอน) ----------
  function printStopTag(t) {
    var D = TAG_DOC, causes = t.causes || [];
    var _pBy = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser ? (currentUser.fullname || currentUser.username) : '-');
    var _now = new Date();
    var _pAt = _now.toLocaleDateString('th-TH', { day: 'numeric', month: 'short', year: 'numeric' }) + ' ' + _now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + ' น.';
    function box(c) { return '<span class="cb">' + (causes.indexOf(c) !== -1 ? '✓' : '') + '</span> ' + c; }
    function fill(v) { return '<span class="fill">' + (v ? _esc2(v) : '&nbsp;') + '</span>'; }
    var one =
        '<div class="tag">'
      + '<table class="hd"><tr>'
      +   '<td class="l"><b>' + D.title + '</b><br>' + D.org + '</td>'
      +   '<td class="r"><b>' + D.code + '</b><br>แก้ไขครั้งที่ ' + D.rev + ' &nbsp; วันที่ประกาศใช้ ' + D.effective + '<br>หน้า 1 จากทั้งหมด 1 หน้า</td>'
      + '</tr></table>'
      + '<div class="ban">ห้ามใช้งาน</div>'
      + '<div class="ttl">' + D.title + '</div>'
      + '<div class="row"><div>ชื่อเครื่องมือ ' + fill(t.eqName) + '</div><div>รหัสครุภัณฑ์ ' + fill(t.eqCode) + '</div></div>'
      + '<div class="row"><div>ห้องปฏิบัติการ ' + fill(t.lab) + '</div><div>ผู้รับผิดชอบ ' + fill(t.owner) + '</div></div>'
      + '<div class="row"><div>วันที่หยุดใช้เครื่องมือ ' + fill(_thDate(t.date)) + '</div><div>วันที่นำมาใช้ใหม่ ' + fill(_thDate(t.returnDate)) + '</div></div>'
      + '<div class="cause"><b>สาเหตุ :</b> ระบุเครื่องหมาย ✓<div class="cbs">' + box('เครื่องชำรุด') + box('รอการซ่อม') + box('ไม่ผ่านการสอบเทียบ') + '</div>'
      +   '<div class="cbs">' + box('อื่นๆ') + ' ระบุ ' + fill(t.causeOther) + '</div></div>'
      + '<div class="pby">พิมพ์โดย: ' + _esc2(_pBy) + ' &nbsp;|&nbsp; วันที่พิมพ์: ' + _pAt + '</div>'
      + '</div>';
    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="UTF-8"><title>' + D.code + ' ป้ายห้ามใช้งาน ' + _esc2(t.eqCode || '') + '</title><style>'
      + '@import url("https://fonts.googleapis.com/css2?family=Sarabun:wght@400;600;700;800&display=swap");'
      + '@page{size:A5 landscape;margin:8mm}*{box-sizing:border-box;margin:0;padding:0}'
      + 'body{font-family:"Sarabun",sans-serif;font-size:12pt;color:#111;-webkit-print-color-adjust:exact;print-color-adjust:exact}'
      + '.tag{border:2px solid #111;padding:8px 12px 8px;page-break-inside:avoid;display:flex;flex-direction:column;gap:7px}'
      + '.pby{margin-top:4px;padding-top:4px;border-top:1px dashed #999;text-align:right;font-size:8.5pt;color:#555}'
      + 'table.hd{width:100%;border-collapse:collapse;font-size:10pt}table.hd td{border:1px solid #111;padding:4px 8px;vertical-align:middle}'
      + 'table.hd td.l{width:45%}table.hd td.r{text-align:right}'
      + '.ban{background:#dc2626;color:#fff;text-align:center;font-size:40pt;font-weight:800;letter-spacing:4px;border-radius:6px;line-height:1.25;padding:2px 0}'
      + '.ttl{text-align:center;font-weight:700;font-size:14pt}'
      + '.row{display:grid;grid-template-columns:1fr 1fr;gap:16px}.row>div{display:flex;align-items:flex-end;gap:6px;white-space:nowrap}'
      + '.fill{flex:1;border-bottom:1px dotted #111;min-height:1.3em;padding:0 4px;white-space:normal;font-weight:600}'
      + '.cause .cbs{display:flex;flex-wrap:wrap;gap:6px 28px;margin:5px 0 0 22px;align-items:flex-end}.cause .cbs .fill{min-width:200px}'
      + '.cb{display:inline-flex;width:15px;height:15px;border:1.5px solid #111;align-items:center;justify-content:center;font-weight:800;font-size:11pt;line-height:1;vertical-align:-2px}'
      + '</style></head><body>' + one + '</body></html>';
    var win = window.open('', '_blank', 'width=860,height=640');
    if (!win) { showToast('เบราว์เซอร์บล็อกหน้าต่างใหม่ กรุณาอนุญาต Pop-up', 'warning'); return; }
    win.document.open(); win.document.write(html); win.document.close();
    win.onload = function () { setTimeout(function () { win.focus(); win.print(); }, 500); };
  }

  function _reloadStatusViews() {
    if (typeof loadEqStatusPage === 'function' && document.getElementById('page-eq-status') && document.getElementById('page-eq-status').style.display !== 'none') loadEqStatusPage();
    else if (typeof loadEquipments === 'function') loadEquipments();
  }

  // ปุ่มจัดการในตารางสถานะเครื่องมือ (ปุ่มไอคอนกลมขนาดเล็ก มี tooltip)
  function _eqStatusActionButtons(item, gi) {
    var b = function (fn, icon, tip, cls) {
      return '<button onclick="' + fn + '(' + gi + ')" title="' + tip + '" aria-label="' + tip + '" '
        + 'class="inline-flex items-center justify-center w-8 h-8 rounded-full border transition hover:scale-110 hover:shadow-sm ' + cls + '">'
        + '<span class="material-symbols-outlined" style="font-size:17px">' + icon + '</span></button>';
    };
    var out = [];
    if (_isRetired(item)) {
      return '<span title="' + _esc2('ปลดระวาง ' + (item['วันที่ปลดระวาง'] || '') + ' · ' + (item['เหตุผลปลดระวาง'] || '')) + '" '
        + 'class="inline-flex items-center gap-1 px-2 py-1 rounded-full bg-slate-100 dark:bg-slate-700 text-slate-500 text-[10px] font-bold whitespace-nowrap">'
        + '<span class="material-symbols-outlined" style="font-size:14px">inventory_2</span>ปลดระวางแล้ว</span>';
    }
    if (_isStopped(item)) {
      out.push(b('openVerificationModal', 'verified', 'ทวนสอบก่อนนำกลับมาใช้', 'bg-emerald-50 border-emerald-200 text-emerald-600 hover:bg-emerald-100 dark:bg-emerald-900/20 dark:border-emerald-800'));
      out.push(b('reprintStopTag', 'print', 'พิมพ์ป้ายห้ามใช้งาน', 'bg-rose-50 border-rose-200 text-rose-600 hover:bg-rose-100 dark:bg-rose-900/20 dark:border-rose-800'));
      out.push(b('openImpactForEquipment', 'assignment_late', 'ประเมินผลกระทบย้อนหลัง', 'bg-amber-50 border-amber-200 text-amber-600 hover:bg-amber-100 dark:bg-amber-900/20 dark:border-amber-800'));
    } else if (String(item['สถานะเครื่องมือ'] || '').trim() === 'รอทวนสอบก่อนใช้') {
      out.push('<button onclick="switchPage(\'verify\')" title="รอหัวหน้าอนุมัติผลทวนสอบ" class="inline-flex items-center justify-center w-8 h-8 rounded-full border bg-sky-50 border-sky-200 text-sky-600 hover:bg-sky-100 transition hover:scale-110"><span class="material-symbols-outlined" style="font-size:17px">pending_actions</span></button>');
    } else {
      out.push(b('openVerificationModal', 'verified', 'ทวนสอบก่อนใช้งาน (เครื่องใหม่ / ย้ายที่ / หยุดใช้นาน)', 'bg-emerald-50 border-emerald-200 text-emerald-600 hover:bg-emerald-100 dark:bg-emerald-900/20 dark:border-emerald-800'));
      out.push(b('openStopUseModal', 'block', 'หยุดใช้งาน + พิมพ์ป้าย', 'bg-rose-50 border-rose-200 text-rose-600 hover:bg-rose-100 dark:bg-rose-900/20 dark:border-rose-800'));
    }
    out.push(b('openEquipmentStatusModal', 'edit', 'อัปเดตสถานะ', 'bg-sky-50 border-sky-200 text-sky-600 hover:bg-sky-100 dark:bg-sky-900/20 dark:border-sky-800'));
    if (_canRetire()) out.push(b('openRetireModal', 'inventory_2', 'ปลดระวาง', 'bg-slate-50 border-slate-200 text-slate-500 hover:bg-slate-100 dark:bg-slate-700 dark:border-slate-600 dark:text-slate-300'));
    return '<div class="inline-flex items-center gap-1.5 whitespace-nowrap">' + out.join('') + '</div>';
  }


  // ============================================================
  // ✅ ประเมินผลกระทบย้อนหลัง (Impact assessment)
  // ============================================================
  var _iaList = [];
  var IA_RESULTS = ['ไม่มีผลกระทบต่อผลตรวจ', 'มีผลกระทบแต่ไม่มีนัยสำคัญทางคลินิก', 'มีผลกระทบต่อผลตรวจ'];
  var IA_ACTIONS = ['ทบทวนผลตรวจย้อนหลัง', 'ตรวจซ้ำจากตัวอย่างที่เก็บไว้', 'แจ้งแพทย์ / ผู้ส่งตรวจ', 'ออกรายงานผลแก้ไข', 'เปิด NC / CAPA', 'อื่นๆ'];
  var IA_STATUS_CLS = {
    'รอประเมิน':   'bg-amber-100 text-amber-700 border-amber-200',
    'ส่งกลับแก้ไข': 'bg-rose-100 text-rose-700 border-rose-200',
    'รออนุมัติ':    'bg-sky-100 text-sky-700 border-sky-200',
    'อนุมัติแล้ว':   'bg-emerald-100 text-emerald-700 border-emerald-200'
  };

  function loadImpactPage() {
    var tb = document.getElementById('ia-tbody');
    if (tb) tb.innerHTML = '<tr><td colspan="7" class="px-4 py-10 text-center text-slate-400">กำลังโหลด...</td></tr>';
    google.script.run.withSuccessHandler(function (res) {
      _iaList = (res && res.success && res.data) || [];
      if (res && !res.success) showToast('โหลดไม่สำเร็จ: ' + res.message, 'danger');
      renderImpactTable();
    }).withFailureHandler(function (e) { showToast('โหลดไม่สำเร็จ: ' + (e && e.message || e), 'danger'); })
      .getImpactAssessments(currentUser ? currentUser.dept : '', currentUser ? currentUser.role : '');
  }

  function _iaFmt(iso) { return iso ? _thDate(String(iso).slice(0, 10)) : '-'; }

  function renderImpactTable() {
    var tb = document.getElementById('ia-tbody'); if (!tb) return;
    var q = ((document.getElementById('ia-search') || {}).value || '').toLowerCase().trim();
    var fl = (document.getElementById('ia-filter') || {}).value || 'all';
    var cnt = { all: _iaList.length }; Object.keys(IA_STATUS_CLS).forEach(function (k) { cnt[k] = 0; });
    _iaList.forEach(function (r) { cnt[r['สถานะ']] = (cnt[r['สถานะ']] || 0) + 1; });
    var sum = document.getElementById('ia-summary');
    if (sum) sum.innerHTML = [['all', 'ทั้งหมด', 'inventory', 'bg-white dark:bg-slate-800 border-slate-200 text-slate-700 dark:text-slate-200']]
      .concat(Object.keys(IA_STATUS_CLS).map(function (k) { return [k, k, k === 'อนุมัติแล้ว' ? 'task_alt' : k === 'รออนุมัติ' ? 'pending' : k === 'ส่งกลับแก้ไข' ? 'undo' : 'edit_note', IA_STATUS_CLS[k]]; }))
      .map(function (c) {
        return '<div onclick="document.getElementById(\'ia-filter\').value=\'' + c[0] + '\';renderImpactTable()" class="cursor-pointer rounded-xl border p-3 hover:shadow-md transition ' + c[3] + '">'
          + '<div class="flex items-center gap-1.5 text-[11px] font-bold"><span class="material-symbols-outlined" style="font-size:16px">' + c[2] + '</span>' + c[1] + '</div>'
          + '<div class="text-2xl font-black mt-1">' + (cnt[c[0]] || 0) + '</div></div>';
      }).join('');
    var list = _iaList.filter(function (r) {
      return (fl === 'all' || r['สถานะ'] === fl) && (!q || [r['IAID'], r['รหัสครุภัณฑ์'], r['ชื่อเครื่องมือ']].join(' ').toLowerCase().indexOf(q) !== -1);
    });
    if (!list.length) { tb.innerHTML = '<tr><td colspan="7" class="px-4 py-10 text-center text-slate-400 italic">ไม่มีแบบประเมินในเงื่อนไขที่เลือก</td></tr>'; return; }
    var canDecide = _canRetire();
    tb.innerHTML = list.map(function (r) {
      var i = _iaList.indexOf(r), st = r['สถานะ'];
      var btn = function (fn, icon, tip, cls) {
        return '<button onclick="' + fn + '(' + i + ')" title="' + tip + '" class="inline-flex items-center justify-center w-8 h-8 rounded-full border transition hover:scale-110 ' + cls + '"><span class="material-symbols-outlined" style="font-size:17px">' + icon + '</span></button>';
      };
      var acts = [];
      if (st !== 'อนุมัติแล้ว' && st !== 'รออนุมัติ') acts.push(btn('openImpactForm', 'edit_note', 'ประเมิน / แก้ไข', 'bg-amber-50 border-amber-200 text-amber-600 hover:bg-amber-100'));
      if (st === 'รออนุมัติ' && canDecide) acts.push(btn('openImpactDecision', 'gavel', 'พิจารณาอนุมัติ', 'bg-sky-50 border-sky-200 text-sky-600 hover:bg-sky-100'));
      if (st === 'รออนุมัติ' || st === 'อนุมัติแล้ว') acts.push(btn('openImpactForm', 'visibility', 'ดูรายละเอียด', 'bg-slate-50 border-slate-200 text-slate-500 hover:bg-slate-100'));
      acts.push(btn('printImpactAssessment', 'print', 'พิมพ์แบบบันทึก', 'bg-slate-50 border-slate-200 text-slate-500 hover:bg-slate-100'));
      return '<tr class="hover:bg-slate-50 dark:hover:bg-slate-700/30">'
        + '<td class="px-3 py-2 font-mono text-xs text-slate-500 whitespace-nowrap">' + _esc2(r['IAID']) + '</td>'
        + '<td class="px-3 py-2"><div class="font-semibold text-slate-800 dark:text-slate-100">' + _esc2(r['รหัสครุภัณฑ์'] || '-') + '</div><div class="text-[11px] text-slate-400">' + _esc2(r['ชื่อเครื่องมือ'] || '') + '</div></td>'
        + '<td class="px-3 py-2 text-xs">' + _esc2(r['สาเหตุ'] || '-') + '</td>'
        + '<td class="px-3 py-2 text-xs text-center whitespace-nowrap">' + _iaFmt(r['วันที่ผ่านครั้งล่าสุด']) + ' – ' + _iaFmt(r['วันที่หยุดใช้'])
        +   (r['จำนวนวันที่อาจได้รับผลกระทบ'] ? '<div class="text-[11px] text-slate-400">' + r['จำนวนวันที่อาจได้รับผลกระทบ'] + ' วัน</div>' : '') + '</td>'
        + '<td class="px-3 py-2 text-xs text-center">' + _esc2(r['ผลการประเมิน'] || '-') + '</td>'
        + '<td class="px-3 py-2 text-center"><span class="inline-block px-2 py-0.5 rounded-full border text-[11px] font-bold ' + (IA_STATUS_CLS[st] || '') + '">' + _esc2(st) + '</span>'
        +   (st === 'ส่งกลับแก้ไข' && r['ความเห็นผู้อนุมัติ'] ? '<div class="text-[10px] text-rose-500 mt-0.5 max-w-[160px] mx-auto truncate" title="' + _esc2(r['ความเห็นผู้อนุมัติ']) + '">' + _esc2(r['ความเห็นผู้อนุมัติ']) + '</div>' : '') + '</td>'
        + '<td class="px-3 py-2 text-center"><div class="inline-flex gap-1.5">' + acts.join('') + '</div></td></tr>';
    }).join('');
  }

  function _iaDaysClient(a, b) {
    if (!/^\d{4}-\d{2}-\d{2}$/.test(a || '') || !/^\d{4}-\d{2}-\d{2}$/.test(b || '')) return '';
    return Math.max(0, Math.round((new Date(b + 'T00:00:00') - new Date(a + 'T00:00:00')) / 86400000));
  }

  function openImpactForm(i) {
    var r = _iaList[i]; if (!r) return;
    var ro = r['สถานะ'] === 'อนุมัติแล้ว' || r['สถานะ'] === 'รออนุมัติ';
    var dis = ro ? ' disabled' : '';
    var acts = String(r['การดำเนินการ'] || '').split(/\s*,\s*/);
    var info = '<div style="background:#fffbeb;border:1px solid #fde68a;border-radius:8px;padding:10px 12px;font-size:12px;line-height:1.8;">'
      + '<b>' + _esc2(r['รหัสครุภัณฑ์']) + ' · ' + _esc2(r['ชื่อเครื่องมือ']) + '</b> (' + _esc2(r['ห้องปฏิบัติการ'] || '-') + ')<br>'
      + 'สาเหตุหยุดใช้: <b>' + _esc2(r['สาเหตุ']) + '</b> · วันที่หยุดใช้: <b>' + _iaFmt(r['วันที่หยุดใช้']) + '</b>'
      + (r['ความเห็นผู้อนุมัติ'] && r['สถานะ'] === 'ส่งกลับแก้ไข' ? '<div style="color:#be123c;margin-top:4px">↩ ส่งกลับแก้ไข: ' + _esc2(r['ความเห็นผู้อนุมัติ']) + '</div>' : '')
      + '</div>';
    var body = info
      + '<div style="display:grid;grid-template-columns:1fr 1fr;gap:0 12px;">'
      +   '<div><label style="' + _lb + '">วันที่ผ่านการสอบเทียบครั้งล่าสุด</label><input type="date" data-f="วันที่ผ่านครั้งล่าสุด" value="' + _esc2(r['วันที่ผ่านครั้งล่าสุด']) + '" style="' + _fs + '"' + dis + '></div>'
      +   '<div><label style="' + _lb + '">ช่วงที่อาจได้รับผลกระทบ</label><div data-days style="' + _fs + 'background:#f8fafc;"></div></div>'
      +   '<div><label style="' + _lb + '">ความคลาดเคลื่อนที่พบ</label><input data-f="ความคลาดเคลื่อนที่พบ" value="' + _esc2(r['ความคลาดเคลื่อนที่พบ']) + '" placeholder="เช่น อ่านค่าสูงกว่าจริง +1.8 °C" style="' + _fs + '"' + dis + '></div>'
      +   '<div><label style="' + _lb + '">เกณฑ์ที่ยอมรับได้</label><input data-f="เกณฑ์ที่ยอมรับได้" value="' + _esc2(r['เกณฑ์ที่ยอมรับได้']) + '" placeholder="เช่น ±1.0 °C" style="' + _fs + '"' + dis + '></div>'
      + '</div>'
      + '<label style="' + _lb + '">รายการทดสอบที่ใช้เครื่องนี้</label><textarea data-f="รายการทดสอบที่เกี่ยวข้อง" rows="2" placeholder="เช่น การเพาะเชื้อ, การบ่มตัวอย่าง..." style="' + _fs + 'resize:vertical;"' + dis + '>' + _esc2(r['รายการทดสอบที่เกี่ยวข้อง']) + '</textarea>'
      + '<label style="' + _lb + '">จำนวนตัวอย่าง / รายงานผลโดยประมาณในช่วงนั้น</label><input data-f="จำนวนตัวอย่างโดยประมาณ" value="' + _esc2(r['จำนวนตัวอย่างโดยประมาณ']) + '" style="' + _fs + '"' + dis + '>'
      + '<label style="' + _lb + '">ผลการประเมิน *</label><div style="display:flex;flex-direction:column;gap:6px;">'
      +   IA_RESULTS.map(function (x) { return '<label style="display:flex;gap:8px;align-items:center;cursor:pointer;"><input type="radio" name="ia-res" value="' + x + '"' + (r['ผลการประเมิน'] === x ? ' checked' : '') + dis + ' style="width:16px;height:16px;">' + x + '</label>'; }).join('')
      + '</div>'
      + '<label style="' + _lb + '">เหตุผลประกอบการประเมิน *</label><textarea data-f="เหตุผลประกอบ" rows="3" placeholder="เทียบขนาดความคลาดเคลื่อนกับเกณฑ์ / ผลต่อการแปลผล / หลักฐานจาก QC ในช่วงเดียวกัน" style="' + _fs + 'resize:vertical;"' + dis + '>' + _esc2(r['เหตุผลประกอบ']) + '</textarea>'
      + '<label style="' + _lb + '">การดำเนินการ (บังคับเมื่อ "มีผลกระทบต่อผลตรวจ")</label><div style="display:grid;grid-template-columns:1fr 1fr;gap:6px;">'
      +   IA_ACTIONS.map(function (x) { return '<label style="display:flex;gap:8px;align-items:center;cursor:pointer;font-size:12px;"><input type="checkbox" data-act value="' + x + '"' + (acts.indexOf(x) !== -1 ? ' checked' : '') + dis + ' style="width:15px;height:15px;">' + x + '</label>'; }).join('')
      + '</div>'
      + '<div style="display:grid;grid-template-columns:2fr 1fr;gap:0 12px;">'
      +   '<div><label style="' + _lb + '">รายละเอียดการดำเนินการ</label><textarea data-f="รายละเอียดการดำเนินการ" rows="2" style="' + _fs + 'resize:vertical;"' + dis + '>' + _esc2(r['รายละเอียดการดำเนินการ']) + '</textarea></div>'
      +   '<div><label style="' + _lb + '">เลขที่ NC / CAPA</label><input data-f="เลขที่ NC/CAPA" value="' + _esc2(r['เลขที่ NC/CAPA']) + '" style="' + _fs + '"' + dis + '></div>'
      + '</div>'
      + (ro ? '<div style="margin-top:10px;font-size:12px;color:#475569;">ผู้ประเมิน: ' + _esc2(r['ผู้ประเมิน'] || '-') + (r['ผู้อนุมัติ'] ? ' · ผู้อนุมัติ: ' + _esc2(r['ผู้อนุมัติ']) + ' (' + _esc2(r['วันที่อนุมัติ']) + ')' : '') + '</div>' : '');
    var me = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser && currentUser.username) || '';
    var ov = _miniModal('📋 ประเมินผลกระทบย้อนหลัง ' + _esc2(r['IAID']), _esc2(r['รหัสครุภัณฑ์'] + ' · ' + r['ชื่อเครื่องมือ']), '#b45309', body,
      ro ? 'ปิด' : '📨 ส่งให้หัวหน้าหน่วยอนุมัติ',
      function (ov, err, busy, close) {
        if (ro) return close();
        _iaSave(ov, r, true, me, err, busy, close);
      });
    ov.firstChild.style.maxWidth = '680px';
    var dEl = ov.querySelector('[data-days]'), lastEl = ov.querySelector('[data-f="วันที่ผ่านครั้งล่าสุด"]');
    var upd = function () {
      var n = _iaDaysClient(lastEl.value, r['วันที่หยุดใช้']);
      dEl.textContent = lastEl.value ? _iaFmt(lastEl.value) + ' – ' + _iaFmt(r['วันที่หยุดใช้']) + (n !== '' ? ' (' + n + ' วัน)' : '') : 'ไม่พบประวัติผ่านครั้งก่อน — กรุณาระบุ';
    };
    lastEl.addEventListener('change', upd); upd();
    if (!ro) {   // ปุ่มบันทึกร่าง
      var bar = ov.querySelector('[data-save]').parentNode, draft = document.createElement('button');
      draft.textContent = '💾 บันทึกร่าง';
      draft.style.cssText = 'padding:9px 18px;border:1.5px solid #b45309;background:#fff;color:#b45309;border-radius:8px;font-size:13px;font-weight:700;cursor:pointer;';
      bar.insertBefore(draft, ov.querySelector('[data-save]'));
      draft.onclick = function () { _iaSave(ov, r, false, me, function (m) { ov.querySelector('[data-err]').textContent = m; }, function (b) { draft.disabled = b; }, function () { if (ov.parentNode) ov.parentNode.removeChild(ov); }); };
    }
  }

  function _iaCollect(ov) {
    var f = {};
    ov.querySelectorAll('[data-f]').forEach(function (el) { f[el.getAttribute('data-f')] = String(el.value || '').trim(); });
    var res = ov.querySelector('input[name="ia-res"]:checked'); f['ผลการประเมิน'] = res ? res.value : '';
    f['การดำเนินการ'] = Array.prototype.map.call(ov.querySelectorAll('[data-act]:checked'), function (x) { return x.value; }).join(', ');
    return f;
  }

  function _iaSave(ov, r, submit, me, err, busy, close) {
    var f = _iaCollect(ov);
    if (submit) {
      if (!f['ผลการประเมิน']) return err('กรุณาเลือกผลการประเมิน');
      if (!f['เหตุผลประกอบ']) return err('กรุณาระบุเหตุผลประกอบการประเมิน');
      if (f['ผลการประเมิน'] === 'มีผลกระทบต่อผลตรวจ' && !f['การดำเนินการ']) return err('มีผลกระทบต่อผลตรวจ — กรุณาเลือกการดำเนินการอย่างน้อย 1 ข้อ');
      if (!confirm('ส่งแบบประเมินให้หัวหน้าหน่วยอนุมัติ?')) return;
    }
    busy(true);
    google.script.run.withSuccessHandler(function (res) {
      busy(false);
      if (!res || !res.success) return err((res && res.message) || 'บันทึกไม่สำเร็จ');
      close();
      showToast(submit ? '📨 ส่งให้หัวหน้าหน่วยอนุมัติแล้ว' + (res.mail && res.mail.success ? ' (แจ้งทางอีเมล)' : '') : '💾 บันทึกร่างแล้ว', 'success');
      loadImpactPage();
    }).withFailureHandler(function (e) { busy(false); err(e && e.message || String(e)); })
      .saveImpactAssessment({ iaId: r['IAID'], fields: f, submit: submit, by: me });
  }

  function openImpactDecision(i) {
    var r = _iaList[i]; if (!r) return;
    var row = function (k, v) { return '<tr><td style="padding:4px 8px;color:#64748b;white-space:nowrap;vertical-align:top">' + k + '</td><td style="padding:4px 8px">' + _esc2(v || '-') + '</td></tr>'; };
    var body = '<table style="width:100%;font-size:12px;border-collapse:collapse;background:#f8fafc;border-radius:8px">'
      + row('เครื่องมือ', r['รหัสครุภัณฑ์'] + ' · ' + r['ชื่อเครื่องมือ']) + row('สาเหตุ', r['สาเหตุ'])
      + row('ช่วงที่อาจกระทบ', _iaFmt(r['วันที่ผ่านครั้งล่าสุด']) + ' – ' + _iaFmt(r['วันที่หยุดใช้']) + (r['จำนวนวันที่อาจได้รับผลกระทบ'] ? ' (' + r['จำนวนวันที่อาจได้รับผลกระทบ'] + ' วัน)' : ''))
      + row('รายการทดสอบ', r['รายการทดสอบที่เกี่ยวข้อง']) + row('จำนวนตัวอย่าง', r['จำนวนตัวอย่างโดยประมาณ'])
      + row('ความคลาดเคลื่อน / เกณฑ์', (r['ความคลาดเคลื่อนที่พบ'] || '-') + ' / ' + (r['เกณฑ์ที่ยอมรับได้'] || '-'))
      + row('ผลการประเมิน', r['ผลการประเมิน']) + row('เหตุผล', r['เหตุผลประกอบ'])
      + row('การดำเนินการ', r['การดำเนินการ']) + row('รายละเอียด', r['รายละเอียดการดำเนินการ']) + row('NC / CAPA', r['เลขที่ NC/CAPA']) + row('ผู้ประเมิน', r['ผู้ประเมิน'])
      + '</table>'
      + '<label style="' + _lb + '">ความเห็นผู้อนุมัติ (บังคับเมื่อส่งกลับแก้ไข)</label><textarea data-f="cm" rows="2" style="' + _fs + 'resize:vertical;"></textarea>';
    var ov = _miniModal('⚖️ พิจารณาแบบประเมิน ' + _esc2(r['IAID']), _esc2(r['รหัสครุภัณฑ์'] + ' · ' + r['ชื่อเครื่องมือ']), '#0369a1', body, '✅ อนุมัติ',
      function (ov, err, busy, close) { _iaDecide(r, 'approve', ov.querySelector('[data-f="cm"]').value, err, busy, close); });
    ov.firstChild.style.maxWidth = '640px';
    var bar = ov.querySelector('[data-save]').parentNode, back = document.createElement('button');
    back.textContent = '↩ ส่งกลับแก้ไข';
    back.style.cssText = 'padding:9px 18px;border:1.5px solid #be123c;background:#fff;color:#be123c;border-radius:8px;font-size:13px;font-weight:700;cursor:pointer;';
    bar.insertBefore(back, ov.querySelector('[data-save]'));
    back.onclick = function () {
      _iaDecide(r, 'return', ov.querySelector('[data-f="cm"]').value, function (m) { ov.querySelector('[data-err]').textContent = m; },
        function (b) { back.disabled = b; }, function () { if (ov.parentNode) ov.parentNode.removeChild(ov); });
    };
  }

  function _iaDecide(r, decision, comment, err, busy, close) {
    if (decision !== 'approve' && !String(comment || '').trim()) return err('กรุณาระบุเหตุผลที่ส่งกลับแก้ไข');
    busy(true);
    google.script.run.withSuccessHandler(function (res) {
      busy(false);
      if (!res || !res.success) return err((res && res.message) || 'บันทึกไม่สำเร็จ');
      close(); showToast(decision === 'approve' ? '✅ อนุมัติแบบประเมินแล้ว — นำเครื่องกลับมาใช้ได้เมื่อแก้ไขเสร็จ' : '↩ ส่งกลับแก้ไขแล้ว', 'success'); loadImpactPage();
    }).withFailureHandler(function (e) { busy(false); err(e && e.message || String(e)); })
      .decideImpactAssessment(r['IAID'], decision, comment, currentUser ? currentUser.username : '');
  }

  function printImpactAssessment(i) {
    var r = _iaList[i]; if (!r) return;
    var by = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser ? currentUser.username : '-'), now = new Date();
    var at = now.toLocaleDateString('th-TH', { day: 'numeric', month: 'short', year: 'numeric' }) + ' ' + now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + ' น.';
    var acts = String(r['การดำเนินการ'] || '').split(/\s*,\s*/);
    var tr = function (k, v) { return '<tr><th>' + k + '</th><td>' + (v ? _esc2(v).replace(/\n/g, '<br>') : '-') + '</td></tr>'; };
    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="UTF-8"><title>' + _esc2(r['IAID']) + ' ประเมินผลกระทบย้อนหลัง</title><style>'
      + '@import url("https://fonts.googleapis.com/css2?family=Sarabun:wght@400;600;700&display=swap");@page{size:A4;margin:14mm}'
      + 'body{font-family:"Sarabun",sans-serif;font-size:11pt;color:#1e293b;margin:0}h1{font-size:15pt;text-align:center;color:#92400e;margin:0 0 2px}'
      + '.sub{text-align:center;font-size:10pt;color:#64748b;margin-bottom:10px}table{width:100%;border-collapse:collapse}th,td{border:1px solid #94a3b8;padding:5px 8px;vertical-align:top;text-align:left}'
      + 'th{background:#f8fafc;width:30%;font-weight:600}.cb{display:inline-block;width:12px;height:12px;border:1.2px solid #111;margin-right:4px;text-align:center;line-height:11px;font-size:10px;font-weight:700}'
      + '.st{display:inline-block;padding:1px 10px;border-radius:10px;background:#ecfdf5;color:#047857;font-weight:700}.foot{margin-top:12px;text-align:right;font-size:9pt;color:#64748b;border-top:1px dashed #cbd5e1;padding-top:4px}'
      + '</style></head><body><h1>แบบบันทึกการประเมินผลกระทบย้อนหลังจากเครื่องมือ</h1><div class="sub">เลขที่ ' + _esc2(r['IAID']) + ' · สถานะ <span class="st">' + _esc2(r['สถานะ']) + '</span></div><table>'
      + tr('เครื่องมือ', (r['รหัสครุภัณฑ์'] || '-') + ' / ' + (r['ชื่อเครื่องมือ'] || '-')) + tr('ห้องปฏิบัติการ', r['ห้องปฏิบัติการ']) + tr('สาเหตุที่หยุดใช้', r['สาเหตุ'])
      + tr('ช่วงที่อาจได้รับผลกระทบ', _iaFmt(r['วันที่ผ่านครั้งล่าสุด']) + ' ถึง ' + _iaFmt(r['วันที่หยุดใช้']) + (r['จำนวนวันที่อาจได้รับผลกระทบ'] ? ' (' + r['จำนวนวันที่อาจได้รับผลกระทบ'] + ' วัน)' : ''))
      + tr('รายการทดสอบที่เกี่ยวข้อง', r['รายการทดสอบที่เกี่ยวข้อง']) + tr('จำนวนตัวอย่างโดยประมาณ', r['จำนวนตัวอย่างโดยประมาณ'])
      + tr('ความคลาดเคลื่อนที่พบ', r['ความคลาดเคลื่อนที่พบ']) + tr('เกณฑ์ที่ยอมรับได้', r['เกณฑ์ที่ยอมรับได้'])
      + '<tr><th>ผลการประเมิน</th><td>' + IA_RESULTS.map(function (x) { return '<div><span class="cb">' + (r['ผลการประเมิน'] === x ? '✓' : '') + '</span>' + x + '</div>'; }).join('') + '</td></tr>'
      + tr('เหตุผลประกอบ', r['เหตุผลประกอบ'])
      + '<tr><th>การดำเนินการ</th><td>' + IA_ACTIONS.map(function (x) { return '<span style="display:inline-block;min-width:46%"><span class="cb">' + (acts.indexOf(x) !== -1 ? '✓' : '') + '</span>' + x + '</span>'; }).join('') + '</td></tr>'
      + tr('รายละเอียดการดำเนินการ', r['รายละเอียดการดำเนินการ']) + tr('เลขที่ NC / CAPA', r['เลขที่ NC/CAPA'])
      + tr('ผู้ประเมิน', (r['ผู้ประเมิน'] || '-') + (r['วันที่ประเมิน'] ? ' · ' + r['วันที่ประเมิน'] : ''))
      + tr('ผู้อนุมัติ', (r['ผู้อนุมัติ'] || '-') + (r['วันที่อนุมัติ'] ? ' · ' + r['วันที่อนุมัติ'] : '') + (r['ความเห็นผู้อนุมัติ'] ? ' · ความเห็น: ' + r['ความเห็นผู้อนุมัติ'] : ''))
      + '</table><div class="foot">พิมพ์โดย: ' + _esc2(by) + ' &nbsp;|&nbsp; วันที่พิมพ์: ' + at + '</div></body></html>';
    var win = window.open('', '_blank', 'width=900,height=800');
    if (!win) { showToast('เบราว์เซอร์บล็อกหน้าต่างใหม่ กรุณาอนุญาต Pop-up', 'warning'); return; }
    win.document.open(); win.document.write(html); win.document.close();
    win.onload = function () { setTimeout(function () { win.focus(); win.print(); }, 500); };
  }

  // จากหน้าสถานะเครื่องมือ: เปิดแบบประเมินของเครื่อง (สร้างให้ถ้ายังไม่มี)
  function openImpactForEquipment(eqIdx) {
    var it = _eqByIdx(eqIdx); if (!it) return;
    showLoader();
    google.script.run.withSuccessHandler(function (res) {
      hideLoader();
      if (!res || !res.success) { showToast(res && res.message || 'ไม่สำเร็จ', 'warning'); return; }
      switchPage('impact');
      var s = document.getElementById('ia-search'); if (s) s.value = it['รหัสครุภัณฑ์'] || '';
    }).withFailureHandler(function (e) { hideLoader(); showToast(e && e.message || String(e), 'danger'); })
      .createImpactAssessmentForEquipment(it['ID']);
  }

  // ============================================================
  // ✅ ข้อ 3: ทวนสอบก่อนใช้งาน
  // ============================================================
  var _vrList = [];
  var VR_REASONS = ['เครื่องใหม่', 'หลังซ่อม', 'ย้ายสถานที่', 'หยุดใช้นาน', 'นำกลับมาใช้หลังหยุดใช้', 'อื่นๆ'];
  var VR_CHECKS = ['ติดตั้งและสภาพแวดล้อมเหมาะสม (ไฟฟ้า อุณหภูมิ พื้นที่)', 'สภาพภายนอกและความปลอดภัยไฟฟ้า', 'ฟังก์ชันการทำงานครบถ้วน',
                   'ทดสอบความถูกต้อง / ความแม่นยำผ่านเกณฑ์', 'มีวิธีใช้งาน และผู้ใช้ได้รับการอบรม', 'ติดป้ายบ่งชี้และสถานะเครื่อง'];
  var VR_STATUS_CLS = { 'รออนุมัติ': 'bg-sky-100 text-sky-700 border-sky-200', 'ส่งกลับแก้ไข': 'bg-rose-100 text-rose-700 border-rose-200',
                        'อนุมัติแล้ว': 'bg-emerald-100 text-emerald-700 border-emerald-200', 'ไม่ผ่าน': 'bg-slate-100 text-slate-600 border-slate-200' };

  function loadVerifyPage() {
    var tb = document.getElementById('vr-tbody');
    if (tb) tb.innerHTML = '<tr><td colspan="7" class="px-4 py-10 text-center text-slate-400">กำลังโหลด...</td></tr>';
    google.script.run.withSuccessHandler(function (res) {
      _vrList = (res && res.success && res.data) || []; renderVerifyTable();
    }).withFailureHandler(function (e) { showToast('โหลดไม่สำเร็จ: ' + (e && e.message || e), 'danger'); })
      .getVerifications(currentUser ? currentUser.dept : '', currentUser ? currentUser.role : '');
  }

  function _iconBtn(fn, arg, icon, tip, cls) {
    return '<button onclick="' + fn + '(' + arg + ')" title="' + tip + '" aria-label="' + tip + '" class="inline-flex items-center justify-center w-8 h-8 rounded-full border transition hover:scale-110 ' + cls + '"><span class="material-symbols-outlined" style="font-size:17px">' + icon + '</span></button>';
  }
  function _chips(elId, list, keys, filterId, rerender) {
    var el = document.getElementById(elId); if (!el) return;
    var cnt = { all: list.length }; list.forEach(function (r) { cnt[r['สถานะ']] = (cnt[r['สถานะ']] || 0) + 1; });
    el.innerHTML = [['all', 'ทั้งหมด', 'bg-white dark:bg-slate-800 border-slate-200 text-slate-700 dark:text-slate-200']].concat(keys.map(function (k) { return [k, k, VR_STATUS_CLS[k] || IA_STATUS_CLS[k] || '']; }))
      .map(function (c) {
        return '<div onclick="document.getElementById(\'' + filterId + '\').value=\'' + c[0] + '\';' + rerender + '()" class="cursor-pointer rounded-xl border p-3 hover:shadow-md transition ' + c[2] + '">'
          + '<div class="text-[11px] font-bold">' + c[1] + '</div><div class="text-2xl font-black mt-1">' + (cnt[c[0]] || 0) + '</div></div>';
      }).join('');
  }

  function renderVerifyTable() {
    var tb = document.getElementById('vr-tbody'); if (!tb) return;
    _chips('vr-summary', _vrList, ['รออนุมัติ', 'ส่งกลับแก้ไข', 'อนุมัติแล้ว', 'ไม่ผ่าน'], 'vr-filter', 'renderVerifyTable');
    var q = ((document.getElementById('vr-search') || {}).value || '').toLowerCase().trim(), fl = (document.getElementById('vr-filter') || {}).value || 'all';
    var list = _vrList.filter(function (r) { return (fl === 'all' || r['สถานะ'] === fl) && (!q || [r['VRID'], r['รหัสครุภัณฑ์'], r['ชื่อเครื่องมือ']].join(' ').toLowerCase().indexOf(q) !== -1); });
    if (!list.length) { tb.innerHTML = '<tr><td colspan="7" class="px-4 py-10 text-center text-slate-400 italic">ยังไม่มีรายการทวนสอบ</td></tr>'; return; }
    var head = _canRetire();
    tb.innerHTML = list.map(function (r) {
      var i = _vrList.indexOf(r), st = r['สถานะ'], a = [];
      if (st === 'รออนุมัติ' && head) a.push(_iconBtn('openVerifyDecision', i, 'gavel', 'พิจารณาอนุมัติ', 'bg-sky-50 border-sky-200 text-sky-600 hover:bg-sky-100'));
      if (st === 'ส่งกลับแก้ไข') a.push(_iconBtn('editVerification', i, 'edit_note', 'แก้ไขแล้วส่งใหม่', 'bg-amber-50 border-amber-200 text-amber-600 hover:bg-amber-100'));
      a.push(_iconBtn('printVerification', i, 'print', 'พิมพ์บันทึกทวนสอบ', 'bg-slate-50 border-slate-200 text-slate-500 hover:bg-slate-100'));
      return '<tr class="hover:bg-slate-50 dark:hover:bg-slate-700/30">'
        + '<td class="px-3 py-2 font-mono text-xs text-slate-500 whitespace-nowrap">' + _esc2(r['VRID']) + '</td>'
        + '<td class="px-3 py-2"><div class="font-semibold">' + _esc2(r['รหัสครุภัณฑ์'] || '-') + '</div><div class="text-[11px] text-slate-400">' + _esc2(r['ชื่อเครื่องมือ'] || '') + '</div></td>'
        + '<td class="px-3 py-2 text-xs">' + _esc2(r['เหตุผล']) + '</td>'
        + '<td class="px-3 py-2 text-xs text-center whitespace-nowrap">' + _iaFmt(r['วันที่ทวนสอบ']) + '</td>'
        + '<td class="px-3 py-2 text-center text-xs font-bold ' + (r['ผลสรุป'] === 'ผ่าน' ? 'text-emerald-600' : 'text-rose-600') + '">' + _esc2(r['ผลสรุป']) + '</td>'
        + '<td class="px-3 py-2 text-center"><span class="inline-block px-2 py-0.5 rounded-full border text-[11px] font-bold ' + (VR_STATUS_CLS[st] || '') + '">' + _esc2(st) + '</span>'
        +   (st === 'ส่งกลับแก้ไข' && r['ความเห็นผู้อนุมัติ'] ? '<div class="text-[10px] text-rose-500 mt-0.5 max-w-[160px] mx-auto truncate" title="' + _esc2(r['ความเห็นผู้อนุมัติ']) + '">' + _esc2(r['ความเห็นผู้อนุมัติ']) + '</div>' : '') + '</td>'
        + '<td class="px-3 py-2 text-center"><div class="inline-flex gap-1.5">' + a.join('') + '</div></td></tr>';
    }).join('');
  }

  // ฟอร์มทวนสอบ (เปิดจากหน้าสถานะเครื่องมือ หรือแก้ไขรายการที่ถูกส่งกลับ)
  function openVerificationModal(eqIdx, existing) {
    var it = existing ? { 'ID': existing['ID อ้างอิง'], 'รหัสครุภัณฑ์': existing['รหัสครุภัณฑ์'], 'ชื่อเครื่องมือ': existing['ชื่อเครื่องมือ'] } : _eqByIdx(eqIdx);
    if (!it) return;
    var src = existing ? (cachedEquipments || []).find(function (e) { return e['ID'] === it['ID']; }) : it;
    var stopped = src && _isStopped(src);
    var prevChecks = {}; if (existing) { try { JSON.parse(existing['รายการตรวจ'] || '[]').forEach(function (c) { prevChecks[c.item] = c.ok; }); } catch (x) {} }
    var defReason = existing ? existing['เหตุผล'] : (stopped ? 'หลังซ่อม' : 'เครื่องใหม่');
    var me = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser && currentUser.username) || '';
    var v = function (k) { return existing ? _esc2(existing[k] || '') : ''; };
    var body =
        (stopped ? '<div style="background:#fffbeb;border:1px solid #fde68a;color:#92400e;border-radius:8px;padding:8px 12px;font-size:12px;margin-bottom:4px;">เครื่องอยู่ในสถานะ <b>' + _esc2(src['สถานะเครื่องมือ']) + '</b> — เมื่อผลทวนสอบได้รับอนุมัติ ระบบจะนำเครื่องกลับมาใช้และเติม "วันที่นำมาใช้ใหม่" ให้</div>' : '')
      + (existing && existing['ความเห็นผู้อนุมัติ'] ? '<div style="color:#be123c;font-size:12px;margin-bottom:4px;">↩ ส่งกลับแก้ไข: ' + _esc2(existing['ความเห็นผู้อนุมัติ']) + '</div>' : '')
      + '<div style="display:grid;grid-template-columns:2fr 1fr;gap:0 12px;">'
      +   '<div><label style="' + _lb + '">เหตุผลที่ทวนสอบ *</label><select data-f="reason" style="' + _fs + '">' + VR_REASONS.map(function (r) { return '<option' + (r === defReason ? ' selected' : '') + '>' + r + '</option>'; }).join('') + '</select></div>'
      +   '<div><label style="' + _lb + '">วันที่ทวนสอบ *</label><input type="date" data-f="date" value="' + (existing ? v('วันที่ทวนสอบ') : _todayIso()) + '" style="' + _fs + '"></div>'
      + '</div>'
      + '<label style="' + _lb + '">รายการตรวจ (ติ๊กเมื่อผ่าน)</label><div style="display:flex;flex-direction:column;gap:5px;">'
      +   VR_CHECKS.map(function (c) { return '<label style="display:flex;gap:8px;align-items:center;cursor:pointer;font-size:12.5px;"><input type="checkbox" data-chk value="' + _esc2(c) + '"' + (prevChecks[c] ? ' checked' : '') + ' style="width:16px;height:16px;accent-color:#16a34a;">' + c + '</label>'; }).join('')
      + '</div>'
      + '<label style="' + _lb + '">ผลทดสอบความถูกต้อง / ความแม่นยำ</label><textarea data-f="accuracy" rows="2" placeholder="เช่น อ่าน 37.2 °C เทียบ ref 37.0 °C (ต่าง +0.2) / ผล QC 2 ระดับอยู่ในช่วง" style="' + _fs + 'resize:vertical;">' + v('ผลทดสอบความถูกต้อง/แม่น') + '</textarea>'
      + '<div style="display:grid;grid-template-columns:1fr 1fr;gap:0 12px;">'
      +   '<div><label style="' + _lb + '">เกณฑ์ที่ใช้</label><input data-f="criteria" value="' + v('เกณฑ์ที่ใช้') + '" placeholder="เช่น ±1.0 °C" style="' + _fs + '"></div>'
      +   '<div><label style="' + _lb + '">หลักฐาน / เลขที่เอกสาร</label><input data-f="evidence" value="' + v('หลักฐาน/เลขที่เอกสาร') + '" placeholder="เลขใบรับรอง / รายงานติดตั้ง" style="' + _fs + '"></div>'
      + '</div>'
      + '<label style="' + _lb + '">ผลสรุป *</label><div style="display:flex;gap:20px;">'
      +   '<label style="display:flex;gap:6px;align-items:center;cursor:pointer;font-weight:700;color:#15803d;"><input type="radio" name="vr-res" value="ผ่าน"' + (existing && existing['ผลสรุป'] === 'ผ่าน' ? ' checked' : '') + ' style="width:16px;height:16px;">ผ่าน — พร้อมใช้งาน</label>'
      +   '<label style="display:flex;gap:6px;align-items:center;cursor:pointer;font-weight:700;color:#be123c;"><input type="radio" name="vr-res" value="ไม่ผ่าน"' + (existing && existing['ผลสรุป'] === 'ไม่ผ่าน' ? ' checked' : '') + ' style="width:16px;height:16px;">ไม่ผ่าน</label>'
      + '</div>'
      + '<label style="' + _lb + '">หมายเหตุ</label><textarea data-f="note" rows="2" style="' + _fs + 'resize:vertical;">' + v('หมายเหตุ') + '</textarea>';
    var ov = _miniModal('✅ ทวนสอบก่อนใช้งาน', _esc2((it['รหัสครุภัณฑ์'] || '-') + ' · ' + (it['ชื่อเครื่องมือ'] || '-')), '#15803d', body, '📨 บันทึก / ส่งอนุมัติ',
      function (ov, err, busy, close) {
        var f = function (k) { return String((ov.querySelector('[data-f="' + k + '"]') || {}).value || '').trim(); };
        var res = ov.querySelector('input[name="vr-res"]:checked');
        var checks = Array.prototype.map.call(ov.querySelectorAll('[data-chk]'), function (x) { return { item: x.value, ok: x.checked }; });
        if (!f('date')) return err('กรุณาระบุวันที่ทวนสอบ');
        if (!res) return err('กรุณาเลือกผลสรุป');
        if (res.value === 'ผ่าน' && checks.some(function (c) { return !c.ok; })) return err('ผลสรุป "ผ่าน" ต้องติ๊กรายการตรวจครบทุกข้อ');
        if (res.value === 'ผ่าน' && !f('accuracy') && !f('evidence')) return err('กรุณาระบุผลทดสอบความถูกต้อง หรือหลักฐาน อย่างน้อย 1 อย่าง');
        busy(true);
        google.script.run.withSuccessHandler(function (r) {
          busy(false);
          if (!r || !r.success) {
            if (r && r.blockedBy) { close(); showToast(r.message, 'warning'); switchPage('impact'); var s = document.getElementById('ia-search'); if (s) s.value = r.blockedBy; return; }
            return err((r && r.message) || 'บันทึกไม่สำเร็จ');
          }
          close();
          showToast(r.status === 'รออนุมัติ' ? '📨 ส่งผลทวนสอบให้หัวหน้าหน่วยอนุมัติแล้ว (' + r.vrId + ')' : 'บันทึกผลทวนสอบ "ไม่ผ่าน" แล้ว — เครื่องยังไม่พร้อมใช้งาน', r.status === 'รออนุมัติ' ? 'success' : 'warning');
          _reloadStatusViews(); if (document.getElementById('page-verify').style.display !== 'none') loadVerifyPage();
        }).withFailureHandler(function (e) { busy(false); err(e && e.message || String(e)); })
          .saveVerification({ vrId: existing ? existing['VRID'] : '', eqId: it['ID'], reason: f('reason'), date: f('date'), checks: checks,
                              accuracy: f('accuracy'), criteria: f('criteria'), evidence: f('evidence'), result: res.value, note: f('note'), by: me });
      });
    ov.firstChild.style.maxWidth = '640px';
  }
  // ปุ่ม "นำกลับมาใช้" เดิม → ต้องผ่านการทวนสอบก่อนเสมอ
  function openReturnToServiceModal(eqIdx) { openVerificationModal(eqIdx); }
  function editVerification(i) { openVerificationModal(null, _vrList[i]); }

  function openVerifyDecision(i) {
    var r = _vrList[i]; if (!r) return;
    var checks = []; try { checks = JSON.parse(r['รายการตรวจ'] || '[]'); } catch (x) {}
    var row = function (k, v) { return '<tr><td style="padding:4px 8px;color:#64748b;white-space:nowrap;vertical-align:top">' + k + '</td><td style="padding:4px 8px">' + v + '</td></tr>'; };
    var body = '<table style="width:100%;font-size:12px;border-collapse:collapse;background:#f8fafc">'
      + row('เครื่องมือ', _esc2(r['รหัสครุภัณฑ์'] + ' · ' + r['ชื่อเครื่องมือ'])) + row('เหตุผล', _esc2(r['เหตุผล'])) + row('วันที่ทวนสอบ', _iaFmt(r['วันที่ทวนสอบ']))
      + row('รายการตรวจ', checks.map(function (c) { return (c.ok ? '✅ ' : '❌ ') + _esc2(c.item); }).join('<br>'))
      + row('ผลทดสอบ', _esc2(r['ผลทดสอบความถูกต้อง/แม่น'] || '-')) + row('เกณฑ์', _esc2(r['เกณฑ์ที่ใช้'] || '-')) + row('หลักฐาน', _esc2(r['หลักฐาน/เลขที่เอกสาร'] || '-'))
      + row('ผลสรุป', '<b style="color:#15803d">' + _esc2(r['ผลสรุป']) + '</b>') + row('ผู้ทวนสอบ', _esc2(r['ผู้ทวนสอบ'] || '-')) + '</table>'
      + '<div style="font-size:12px;color:#15803d;margin-top:8px;">เมื่ออนุมัติ เครื่องจะเปลี่ยนเป็น "พร้อมใช้งาน" ทันที</div>'
      + '<label style="' + _lb + '">ความเห็นผู้อนุมัติ (บังคับเมื่อส่งกลับแก้ไข)</label><textarea data-f="cm" rows="2" style="' + _fs + 'resize:vertical;"></textarea>';
    var ov = _miniModal('⚖️ พิจารณาผลทวนสอบ ' + _esc2(r['VRID']), _esc2(r['รหัสครุภัณฑ์'] + ' · ' + r['ชื่อเครื่องมือ']), '#0369a1', body, '✅ อนุมัติ · พร้อมใช้งาน',
      function (ov, err, busy, close) { _vrDecide(r, 'approve', ov.querySelector('[data-f="cm"]').value, err, busy, close); });
    var bar = ov.querySelector('[data-save]').parentNode, back = document.createElement('button');
    back.textContent = '↩ ส่งกลับแก้ไข';
    back.style.cssText = 'padding:9px 18px;border:1.5px solid #be123c;background:#fff;color:#be123c;border-radius:8px;font-size:13px;font-weight:700;cursor:pointer;';
    bar.insertBefore(back, ov.querySelector('[data-save]'));
    back.onclick = function () { _vrDecide(r, 'return', ov.querySelector('[data-f="cm"]').value, function (m) { ov.querySelector('[data-err]').textContent = m; }, function (b) { back.disabled = b; }, function () { if (ov.parentNode) ov.parentNode.removeChild(ov); }); };
  }
  function _vrDecide(r, decision, comment, err, busy, close) {
    if (decision !== 'approve' && !String(comment || '').trim()) return err('กรุณาระบุเหตุผลที่ส่งกลับแก้ไข');
    busy(true);
    google.script.run.withSuccessHandler(function (res) {
      busy(false);
      if (!res || !res.success) return err((res && res.message) || 'บันทึกไม่สำเร็จ');
      close(); showToast(decision === 'approve' ? '✅ อนุมัติแล้ว — เครื่องพร้อมใช้งาน' : '↩ ส่งกลับแก้ไขแล้ว', 'success'); loadVerifyPage();
    }).withFailureHandler(function (e) { busy(false); err(e && e.message || String(e)); })
      .decideVerification(r['VRID'], decision, comment, currentUser ? currentUser.username : '');
  }

  function printVerification(i) {
    var r = _vrList[i]; if (!r) return;
    var checks = []; try { checks = JSON.parse(r['รายการตรวจ'] || '[]'); } catch (x) {}
    var by = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser ? currentUser.username : '-'), now = new Date();
    var at = now.toLocaleDateString('th-TH', { day: 'numeric', month: 'short', year: 'numeric' }) + ' ' + now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + ' น.';
    var tr = function (k, v) { return '<tr><th>' + k + '</th><td>' + (v || '-') + '</td></tr>'; };
    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="UTF-8"><title>' + _esc2(r['VRID']) + ' ทวนสอบก่อนใช้งาน</title><style>'
      + '@import url("https://fonts.googleapis.com/css2?family=Sarabun:wght@400;600;700&display=swap");@page{size:A4;margin:14mm}'
      + 'body{font-family:"Sarabun",sans-serif;font-size:11pt;color:#1e293b;margin:0}h1{font-size:15pt;text-align:center;color:#15803d;margin:0 0 2px}.sub{text-align:center;font-size:10pt;color:#64748b;margin-bottom:10px}'
      + 'table{width:100%;border-collapse:collapse}th,td{border:1px solid #94a3b8;padding:5px 8px;vertical-align:top;text-align:left}th{background:#f8fafc;width:30%;font-weight:600}'
      + '.cb{display:inline-block;width:12px;height:12px;border:1.2px solid #111;margin-right:5px;text-align:center;line-height:11px;font-size:10px;font-weight:700}'
      + '.foot{margin-top:12px;text-align:right;font-size:9pt;color:#64748b;border-top:1px dashed #cbd5e1;padding-top:4px}</style></head><body>'
      + '<h1>บันทึกการทวนสอบเครื่องมือก่อนใช้งาน</h1><div class="sub">เลขที่ ' + _esc2(r['VRID']) + ' · สถานะ ' + _esc2(r['สถานะ']) + '</div><table>'
      + tr('เครื่องมือ', _esc2((r['รหัสครุภัณฑ์'] || '-') + ' / ' + (r['ชื่อเครื่องมือ'] || '-'))) + tr('ห้องปฏิบัติการ', _esc2(r['ห้องปฏิบัติการ']))
      + tr('เหตุผลที่ทวนสอบ', _esc2(r['เหตุผล'])) + tr('วันที่ทวนสอบ', _iaFmt(r['วันที่ทวนสอบ']))
      + tr('รายการตรวจ', checks.map(function (c) { return '<div><span class="cb">' + (c.ok ? '✓' : '') + '</span>' + _esc2(c.item) + '</div>'; }).join(''))
      + tr('ผลทดสอบความถูกต้อง / แม่นยำ', _esc2(r['ผลทดสอบความถูกต้อง/แม่น']).replace(/\n/g, '<br>')) + tr('เกณฑ์ที่ใช้', _esc2(r['เกณฑ์ที่ใช้']))
      + tr('หลักฐาน / เลขที่เอกสาร', _esc2(r['หลักฐาน/เลขที่เอกสาร'])) + tr('ผลสรุป', '<b>' + _esc2(r['ผลสรุป']) + '</b>') + tr('หมายเหตุ', _esc2(r['หมายเหตุ']))
      + tr('ผู้ทวนสอบ', _esc2((r['ผู้ทวนสอบ'] || '-') + ' · ' + (r['วันที่บันทึก'] || '')))
      + tr('ผู้อนุมัติ', _esc2((r['ผู้อนุมัติ'] || '-') + (r['วันที่อนุมัติ'] ? ' · ' + r['วันที่อนุมัติ'] : '') + (r['ความเห็นผู้อนุมัติ'] ? ' · ' + r['ความเห็นผู้อนุมัติ'] : '')))
      + '</table><div class="foot">พิมพ์โดย: ' + _esc2(by) + ' &nbsp;|&nbsp; วันที่พิมพ์: ' + at + '</div></body></html>';
    var win = window.open('', '_blank', 'width=900,height=800');
    if (!win) { showToast('เบราว์เซอร์บล็อกหน้าต่างใหม่ กรุณาอนุญาต Pop-up', 'warning'); return; }
    win.document.open(); win.document.write(html); win.document.close();
    win.onload = function () { setTimeout(function () { win.focus(); win.print(); }, 500); };
  }

  // ============================================================
  // ✅ ข้อ 4: ตรวจสอบระหว่างรอบ
  // ============================================================
  var _icChart = null, _icCurrent = null;
  function _icPlan(item) { try { var p = JSON.parse(item['แผนตรวจระหว่างรอบ'] || ''); return p && p.points && p.points.length ? p : null; } catch (x) { return null; } }
  function _icNextDue(item) {
    var p = _icPlan(item); if (!p) return null;
    var last = String(item['ตรวจระหว่างรอบล่าสุด'] || '').slice(0, 10);
    if (!/^\d{4}-\d{2}-\d{2}$/.test(last)) return { due: null, days: -99999, never: true };
    var d = new Date(last + 'T00:00:00'); d.setDate(d.getDate() + p.freqDays);
    var t = new Date(); t.setHours(0, 0, 0, 0);
    return { due: d, days: Math.round((d - t) / 86400000) };
  }
  function _icActive(item) { return !_isRetired(item) && !_isStopped(item); }
  function _icOverdueCount(list) {
    return (list || []).filter(function (e) { var n = _icActive(e) && _icNextDue(e); return n && n.days < 0; }).length;
  }

  function loadIcPage() {
    google.script.run.withSuccessHandler(function (data) {
      cachedEquipments = data || [];
      cachedEquipments.forEach(function (it, i) { it._index = i; });
      renderIcTable();
      if (_icCurrent) { var again = cachedEquipments.find(function (e) { return e['ID'] === _icCurrent; }); if (again) showIcDetail(cachedEquipments.indexOf(again)); }
    }).withFailureHandler(function (e) { showToast('โหลดไม่สำเร็จ: ' + (e && e.message || e), 'danger'); })
      .getEquipments(currentUser ? currentUser.dept : '', currentUser ? currentUser.role : '');
  }

  function renderIcTable() {
    var tb = document.getElementById('ic-tbody'); if (!tb) return;
    var all = (cachedEquipments || []).filter(function (e) { return !_isRetired(e); });
    var withPlan = all.filter(_icPlan), over = withPlan.filter(function (e) { var n = _icNextDue(e); return _icActive(e) && n.days < 0; });
    var soon = withPlan.filter(function (e) { var n = _icNextDue(e); return _icActive(e) && n.days >= 0 && n.days <= 7; });
    var sum = document.getElementById('ic-summary');
    if (sum) sum.innerHTML = [['plan', 'มีแผนตรวจระหว่างรอบ', withPlan.length, 'bg-sky-50 border-sky-200 text-sky-700'], ['overdue', 'เลยกำหนดตรวจ', over.length, 'bg-rose-50 border-rose-200 text-rose-700'],
      ['soon', 'ครบกำหนดใน 7 วัน', soon.length, 'bg-amber-50 border-amber-200 text-amber-700'], ['noplan', 'ยังไม่มีแผน', all.length - withPlan.length, 'bg-slate-50 border-slate-200 text-slate-600']]
      .map(function (c) { return '<div onclick="document.getElementById(\'ic-filter\').value=\'' + c[0] + '\';renderIcTable()" class="cursor-pointer rounded-xl border p-3 hover:shadow-md transition ' + c[3] + '"><div class="text-[11px] font-bold">' + c[1] + '</div><div class="text-2xl font-black mt-1">' + c[2] + '</div></div>'; }).join('');
    var q = ((document.getElementById('ic-search') || {}).value || '').toLowerCase().trim(), fl = (document.getElementById('ic-filter') || {}).value || 'plan';
    var list = all.filter(function (e) {
      var p = _icPlan(e), n = p && _icNextDue(e);
      var okF = fl === 'all' || (fl === 'plan' && p) || (fl === 'noplan' && !p) || (fl === 'overdue' && p && _icActive(e) && n.days < 0) || (fl === 'soon' && p && _icActive(e) && n.days >= 0 && n.days <= 7);
      return okF && (!q || [e['รหัสครุภัณฑ์'], e['ชื่อเครื่องมือ']].join(' ').toLowerCase().indexOf(q) !== -1);
    });
    list.sort(function (a, b) { var na = _icNextDue(a), nb = _icNextDue(b); return (na ? na.days : 99999) - (nb ? nb.days : 99999); });
    if (!list.length) { tb.innerHTML = '<tr><td colspan="6" class="px-4 py-10 text-center text-slate-400 italic">' + (fl === 'plan' ? 'ยังไม่มีเครื่องที่ตั้งแผน — เลือก "ยังไม่มีแผน" แล้วกด ⚙️ เพื่อตั้งแผน' : 'ไม่พบรายการ') + '</td></tr>'; return; }
    tb.innerHTML = list.map(function (e) {
      var gi = cachedEquipments.indexOf(e), p = _icPlan(e), n = p && _icNextDue(e), dueTxt = '-', cls = 'text-slate-400';
      if (p) {
        if (!_icActive(e)) { dueTxt = 'เครื่องหยุดใช้'; cls = 'text-slate-400'; }
        else if (n.never) { dueTxt = 'ยังไม่เคยตรวจ'; cls = 'text-rose-600 font-bold'; }
        else { dueTxt = _iaFmt(n.due.toISOString().slice(0, 10)) + (n.days < 0 ? ' (เลย ' + -n.days + ' วัน)' : n.days <= 7 ? ' (อีก ' + n.days + ' วัน)' : ''); cls = n.days < 0 ? 'text-rose-600 font-bold' : n.days <= 7 ? 'text-amber-600 font-bold' : 'text-emerald-600'; }
      }
      var acts = [];
      if (p && _icActive(e)) acts.push(_iconBtn('openIcRecord', gi, 'edit_square', 'บันทึกผลตรวจ', 'bg-sky-50 border-sky-200 text-sky-600 hover:bg-sky-100'));
      if (p) acts.push(_iconBtn('showIcDetail', gi, 'monitoring', 'ประวัติและกราฟแนวโน้ม', 'bg-slate-50 border-slate-200 text-slate-500 hover:bg-slate-100'));
      acts.push(_iconBtn('openIcPlan', gi, 'settings', p ? 'แก้ไขแผน' : 'ตั้งแผนตรวจระหว่างรอบ', 'bg-slate-50 border-slate-200 text-slate-500 hover:bg-slate-100'));
      return '<tr class="hover:bg-slate-50 dark:hover:bg-slate-700/30">'
        + '<td class="px-3 py-2"><div class="font-semibold">' + _esc2(e['รหัสครุภัณฑ์'] || '-') + '</div><div class="text-[11px] text-slate-400">' + _esc2(e['ชื่อเครื่องมือ'] || '') + '</div></td>'
        + '<td class="px-3 py-2 text-xs">' + (p ? p.points.map(function (x) { return _esc2(x.name) + ' ' + x.ref + ' ±' + x.tol + ' ' + _esc2(x.unit || ''); }).join('<br>') : '<span class="text-slate-300">—</span>') + '</td>'
        + '<td class="px-3 py-2 text-xs text-center">' + (p ? 'ทุก ' + p.freqDays + ' วัน' : '-') + '</td>'
        + '<td class="px-3 py-2 text-xs text-center">' + (e['ตรวจระหว่างรอบล่าสุด'] ? _iaFmt(e['ตรวจระหว่างรอบล่าสุด']) : '-') + '</td>'
        + '<td class="px-3 py-2 text-xs text-center ' + cls + '">' + dueTxt + '</td>'
        + '<td class="px-3 py-2 text-center"><div class="inline-flex gap-1.5">' + acts.join('') + '</div></td></tr>';
    }).join('');
  }

  function openIcPlan(gi) {
    var it = _eqByIdx(gi); if (!it) return;
    var p = _icPlan(it) || { freqDays: 30, method: '', points: [{ name: '', ref: '', tol: '', unit: '' }] };
    var ptRow = function (x) {
      return '<div data-pt style="display:grid;grid-template-columns:2fr 1fr 1fr 1fr auto;gap:6px;margin-bottom:6px;">'
        + '<input data-k="name" placeholder="ชื่อจุดตรวจ เช่น อุณหภูมิ" value="' + _esc2(x.name) + '" style="' + _fs + '">'
        + '<input data-k="ref" type="number" step="any" placeholder="ค่าอ้างอิง" value="' + _esc2(x.ref) + '" style="' + _fs + '">'
        + '<input data-k="tol" type="number" step="any" min="0" placeholder="เกณฑ์ ±" value="' + _esc2(x.tol) + '" style="' + _fs + '">'
        + '<input data-k="unit" placeholder="หน่วย" value="' + _esc2(x.unit) + '" style="' + _fs + '">'
        + '<button type="button" onclick="this.parentNode.remove()" style="border:none;background:none;color:#e11d48;cursor:pointer;font-size:18px;">✕</button></div>';
    };
    var body = '<div style="display:grid;grid-template-columns:1fr 2fr;gap:0 12px;">'
      + '<div><label style="' + _lb + '">ความถี่ *</label><select data-f="freq" style="' + _fs + '">'
      +   [[1, 'ทุกวัน'], [7, 'ทุกสัปดาห์'], [14, 'ทุก 2 สัปดาห์'], [30, 'ทุกเดือน'], [90, 'ทุก 3 เดือน'], [180, 'ทุก 6 เดือน']].map(function (o) { return '<option value="' + o[0] + '"' + (+p.freqDays === o[0] ? ' selected' : '') + '>' + o[1] + '</option>'; }).join('')
      + '</select></div>'
      + '<div><label style="' + _lb + '">วิธีตรวจ / ตัวอ้างอิงที่ใช้</label><input data-f="method" value="' + _esc2(p.method) + '" placeholder="เช่น เทียบเทอร์โมมิเตอร์อ้างอิง SN xxx" style="' + _fs + '"></div></div>'
      + '<label style="' + _lb + '">จุดตรวจ * (ค่าอ้างอิง และเกณฑ์ยอมรับ ±)</label><div data-pts>' + p.points.map(ptRow).join('') + '</div>'
      + '<button type="button" data-addpt style="font-size:12px;border:1px dashed #94a3b8;background:#fff;border-radius:6px;padding:5px 12px;cursor:pointer;color:#475569;">+ เพิ่มจุดตรวจ</button>';
    var ov = _miniModal('⚙️ แผนตรวจระหว่างรอบ', _esc2((it['รหัสครุภัณฑ์'] || '-') + ' · ' + (it['ชื่อเครื่องมือ'] || '-')), '#0369a1', body, '💾 บันทึกแผน',
      function (ov, err, busy, close) {
        var pts = Array.prototype.map.call(ov.querySelectorAll('[data-pt]'), function (d) {
          var g = function (k) { return d.querySelector('[data-k="' + k + '"]').value.trim(); };
          return { name: g('name'), ref: g('ref'), tol: g('tol'), unit: g('unit') };
        }).filter(function (x) { return x.name || x.ref || x.tol; });
        if (!pts.length) return err('กรุณาเพิ่มจุดตรวจอย่างน้อย 1 จุด');
        if (pts.some(function (x) { return !x.name || x.ref === '' || !(parseFloat(x.tol) > 0); })) return err('ทุกจุดต้องมี ชื่อ ค่าอ้างอิง และเกณฑ์ ± มากกว่า 0');
        busy(true);
        google.script.run.withSuccessHandler(function (r) {
          busy(false);
          if (!r || !r.success) return err((r && r.message) || 'บันทึกไม่สำเร็จ');
          close(); showToast('💾 บันทึกแผนตรวจระหว่างรอบแล้ว', 'success'); loadIcPage();
        }).withFailureHandler(function (e) { busy(false); err(e && e.message || String(e)); })
          .setIntermediatePlan(it['ID'], { freqDays: ov.querySelector('[data-f="freq"]').value, method: ov.querySelector('[data-f="method"]').value, points: pts }, currentUser ? currentUser.username : '');
      });
    ov.firstChild.style.maxWidth = '640px';
    ov.querySelector('[data-addpt]').onclick = function () { ov.querySelector('[data-pts]').insertAdjacentHTML('beforeend', ptRow({ name: '', ref: '', tol: '', unit: '' })); };
  }

  function openIcRecord(gi) {
    var it = _eqByIdx(gi), p = it && _icPlan(it); if (!p) return;
    var me = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser && currentUser.username) || '';
    var body = (p.method ? '<div style="font-size:12px;color:#475569;margin-bottom:4px;">วิธีตรวจ: ' + _esc2(p.method) + '</div>' : '')
      + '<label style="' + _lb + '">วันที่ตรวจ *</label><input type="date" data-f="date" value="' + _todayIso() + '" style="' + _fs + '">'
      + '<table style="width:100%;border-collapse:collapse;margin-top:10px;font-size:12.5px;"><tr style="background:#f1f5f9"><th style="padding:6px;text-align:left">จุดตรวจ</th><th style="padding:6px">ค่าอ้างอิง</th><th style="padding:6px">ค่าที่อ่านได้ *</th><th style="padding:6px">ส่วนต่าง</th><th style="padding:6px">ผล</th></tr>'
      + p.points.map(function (x, k) {
          return '<tr><td style="padding:6px">' + _esc2(x.name) + '</td><td style="padding:6px;text-align:center">' + x.ref + ' ±' + x.tol + ' ' + _esc2(x.unit || '') + '</td>'
            + '<td style="padding:6px"><input data-read="' + k + '" type="number" step="any" style="' + _fs + 'text-align:center"></td>'
            + '<td data-diff="' + k + '" style="padding:6px;text-align:center">-</td><td data-ok="' + k + '" style="padding:6px;text-align:center">-</td></tr>';
        }).join('') + '</table>'
      + '<label style="' + _lb + '">หมายเหตุ</label><input data-f="note" style="' + _fs + '">';
    var ov = _miniModal('📝 บันทึกผลตรวจระหว่างรอบ', _esc2((it['รหัสครุภัณฑ์'] || '-') + ' · ' + (it['ชื่อเครื่องมือ'] || '-')), '#0369a1', body, '💾 บันทึกผล',
      function (ov, err, busy, close) {
        var reads = p.points.map(function (x, k) { return ov.querySelector('[data-read="' + k + '"]').value; });
        if (reads.some(function (v) { return v === '' || !isFinite(parseFloat(v)); })) return err('กรุณากรอกค่าที่อ่านได้ให้ครบทุกจุด');
        busy(true);
        google.script.run.withSuccessHandler(function (r) {
          busy(false);
          if (!r || !r.success) return err((r && r.message) || 'บันทึกไม่สำเร็จ');
          close(); loadIcPage();
          if (r.result === 'ผ่าน') showToast('✅ ผลตรวจระหว่างรอบ: ผ่าน', 'success');
          else if (confirm('❌ ผลตรวจระหว่างรอบ "ไม่ผ่าน" (' + r.icId + ')\n\nควรหยุดใช้เครื่องนี้และประเมินผลกระทบย้อนหลัง\nต้องการบันทึกหยุดใช้งานตอนนี้หรือไม่?'))
            openStopUseModal(gi, { causes: ['ไม่ผ่านการสอบเทียบ', 'อื่นๆ'], other: 'ตรวจระหว่างรอบไม่ผ่าน (' + r.icId + ')' });
        }).withFailureHandler(function (e) { busy(false); err(e && e.message || String(e)); })
          .saveIntermediateCheck({ eqId: it['ID'], date: ov.querySelector('[data-f="date"]').value, readings: reads, note: ov.querySelector('[data-f="note"]').value.trim(), by: me });
      });
    ov.querySelectorAll('[data-read]').forEach(function (inp) {
      inp.addEventListener('input', function () {
        var k = +inp.getAttribute('data-read'), x = p.points[k], v = parseFloat(inp.value), dEl = ov.querySelector('[data-diff="' + k + '"]'), oEl = ov.querySelector('[data-ok="' + k + '"]');
        if (!isFinite(v)) { dEl.textContent = '-'; oEl.textContent = '-'; return; }
        var diff = Math.round((v - x.ref) * 10000) / 10000, ok = Math.abs(diff) <= x.tol + 1e-9;
        dEl.textContent = (diff > 0 ? '+' : '') + diff; oEl.innerHTML = ok ? '<b style="color:#15803d">ผ่าน</b>' : '<b style="color:#be123c">ไม่ผ่าน</b>';
      });
    });
  }

  function showIcDetail(gi) {
    var it = _eqByIdx(gi), p = it && _icPlan(it); if (!p) return;
    _icCurrent = it['ID'];
    var box = document.getElementById('ic-detail'); box.classList.remove('hidden');
    document.getElementById('ic-detail-title').textContent = (it['รหัสครุภัณฑ์'] || '-') + ' · ' + (it['ชื่อเครื่องมือ'] || '');
    document.getElementById('ic-detail-plan').textContent = 'ตรวจทุก ' + p.freqDays + ' วัน' + (p.method ? ' · ' + p.method : '');
    document.getElementById('ic-history').innerHTML = '<tr><td colspan="4" class="px-3 py-6 text-center text-slate-400">กำลังโหลด...</td></tr>';
    google.script.run.withSuccessHandler(function (res) {
      var list = (res && res.success && res.data) || [];
      document.getElementById('ic-history').innerHTML = list.length ? list.slice().reverse().map(function (c) {
        return '<tr><td class="px-3 py-2 text-center text-xs whitespace-nowrap">' + _iaFmt(c.date) + '</td><td class="px-3 py-2 text-xs">'
          + c.detail.map(function (d) { return _esc2(d.name) + ': <b>' + d.read + '</b> (' + (d.diff > 0 ? '+' : '') + d.diff + ' / ±' + d.tol + ' ' + _esc2(d.unit || '') + ')' + (d.ok ? '' : ' <span class="text-rose-600 font-bold">✕</span>'); }).join('<br>')
          + '</td><td class="px-3 py-2 text-center text-xs font-bold ' + (c.result === 'ผ่าน' ? 'text-emerald-600' : 'text-rose-600') + '">' + c.result + '</td>'
          + '<td class="px-3 py-2 text-xs text-slate-500">' + _esc2(c.by || '-') + (c.note ? ' · ' + _esc2(c.note) : '') + '</td></tr>';
      }).join('') : '<tr><td colspan="4" class="px-3 py-6 text-center text-slate-400 italic">ยังไม่มีผลตรวจ</td></tr>';
      if (_icChart) { try { _icChart.destroy(); } catch (x) {} _icChart = null; }
      var ctx = document.getElementById('ic-chart');
      if (typeof Chart === 'undefined' || !ctx || !list.length) return;
      var colors = ['#0284c7', '#d97706', '#7c3aed', '#059669'], ds = [];
      var _maxTol = Math.max.apply(null, p.points.map(function (x) { return x.tol; }));
      p.points.forEach(function (pt, k) {
        var col = colors[k % colors.length];
        ds.push({ label: pt.name + ' (ส่วนต่าง)', data: list.map(function (c) { var d = c.detail.filter(function (x) { return x.name === pt.name; })[0]; return d ? d.diff : null; }), borderColor: col, backgroundColor: col, tension: .2, spanGaps: true, pointRadius: 4 });
        ds.push({ label: pt.name + ' เกณฑ์ +' + pt.tol, data: list.map(function () { return pt.tol; }), borderColor: col, borderDash: [5, 5], borderWidth: 1, pointRadius: 0, fill: false });
        ds.push({ label: pt.name + ' เกณฑ์ −' + pt.tol, data: list.map(function () { return -pt.tol; }), borderColor: col, borderDash: [5, 5], borderWidth: 1, pointRadius: 0, fill: false });
      });
      _icChart = new Chart(ctx, { type: 'line', data: { labels: list.map(function (c) { return _iaFmt(c.date); }), datasets: ds },
        options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom', labels: { boxWidth: 10, font: { size: 11 } } } },
                   scales: { y: { suggestedMin: -_maxTol * 1.4, suggestedMax: _maxTol * 1.4, title: { display: true, text: 'ค่าที่อ่านได้ − ค่าอ้างอิง' } } } } });
      box.scrollIntoView({ behavior: 'smooth', block: 'start' });
    }).getIntermediateChecks(it['ID']);
  }

  // แถบเตือนบนแดชบอร์ด: ตรวจระหว่างรอบเลยกำหนด / ทวนสอบรออนุมัติ
  function renderDashOpsAlert(data) {
    var el = document.getElementById('dash-ops-alert'); if (!el) return;
    var over = _icOverdueCount(data || []);
    var pend = (data || []).filter(function (e) { return String(e['สถานะเครื่องมือ'] || '').trim() === 'รอทวนสอบก่อนใช้'; }).length;
    var items = [];
    if (over) items.push('<button onclick="switchPage(\'intercheck\');setTimeout(function(){var f=document.getElementById(\'ic-filter\');if(f){f.value=\'overdue\';renderIcTable();}},600)" class="flex items-center gap-2 px-3 py-2 rounded-lg bg-rose-50 border border-rose-200 text-rose-700 text-sm font-semibold hover:bg-rose-100"><span class="material-symbols-outlined" style="font-size:18px">fact_check</span>ตรวจระหว่างรอบเลยกำหนด ' + over + ' เครื่อง</button>');
    if (pend) items.push('<button onclick="switchPage(\'verify\')" class="flex items-center gap-2 px-3 py-2 rounded-lg bg-sky-50 border border-sky-200 text-sky-700 text-sm font-semibold hover:bg-sky-100"><span class="material-symbols-outlined" style="font-size:18px">verified</span>รอทวนสอบก่อนใช้ / รออนุมัติ ' + pend + ' เครื่อง</button>');
    el.innerHTML = items.join(''); el.style.display = items.length ? 'flex' : 'none';
  }

  // ============================================================
  // ✅ ทบทวนรอบสอบเทียบ & ลดค่าใช้จ่าย
  // ============================================================
  var _rv = null;
  var RV_CLS = { 'ยืดรอบได้': 'bg-emerald-100 text-emerald-700 border-emerald-200', 'คงรอบ': 'bg-slate-100 text-slate-600 border-slate-200',
                 'ควรหดรอบ': 'bg-rose-100 text-rose-700 border-rose-200', 'ข้อมูลไม่พอ': 'bg-amber-100 text-amber-700 border-amber-200' };
  function _baht(n) { return n == null || !isFinite(n) ? '-' : Math.round(n).toLocaleString('th-TH'); }

  function loadCalReview() {
    var tb = document.getElementById('rv-tbody');
    if (tb) tb.innerHTML = '<tr><td colspan="7" class="px-4 py-10 text-center text-slate-400">กำลังวิเคราะห์ประวัติสอบเทียบ...</td></tr>';
    google.script.run.withSuccessHandler(function (res) {
      if (!res || !res.success) { showToast('วิเคราะห์ไม่สำเร็จ: ' + (res && res.message), 'danger'); return; }
      _rv = res; renderCalReview();
    }).withFailureHandler(function (e) { showToast('วิเคราะห์ไม่สำเร็จ: ' + (e && e.message || e), 'danger'); })
      .getCalIntervalReview(currentUser ? currentUser.dept : '', currentUser ? currentUser.role : '');
  }

  function renderCalReview() {
    if (!_rv) return;
    var R = _rv.rules, D = _rv.data;
    document.getElementById('rv-rules').innerHTML = '<b>เกณฑ์ที่ใช้:</b> '
      + '<span class="text-emerald-700 font-semibold">ยืดรอบได้</span> = ผ่านติดกัน ≥ ' + R.minHistory + ' ครั้ง และใช้เกณฑ์ไม่เกิน ' + R.extendMaxUsed + '% และไม่มีตรวจระหว่างรอบไม่ผ่านใน 12 เดือน (เสนอ ×' + R.extendFactor + ' สูงสุด ' + R.maxInterval + ' เดือน) · '
      + '<span class="text-rose-700 font-semibold">ควรหดรอบ</span> = ไม่ผ่านใน 2 ครั้งล่าสุด หรือใช้เกณฑ์เกิน ' + R.shortenUsed + '% หรือตรวจระหว่างรอบไม่ผ่าน · '
      + '"% เกณฑ์ที่ใช้" = ระยะจากกึ่งกลางช่วงควบคุมถึงขอบของ (ค่าที่อ่านได้ ± U) เทียบครึ่งช่วงควบคุม (0% ตรงกลาง, 100% ชนขอบ) · '
      + '<b>การปรับรอบต้องมีเหตุผลบันทึกไว้ และควรตั้งแผนตรวจระหว่างรอบควบคู่</b>';

    var cnt = {}, saveYr = 0, annualAll = 0, withCost = 0;
    D.forEach(function (x) {
      cnt[x.rec] = (cnt[x.rec] || 0) + 1;
      if (x.annual != null) { annualAll += x.annual; withCost++; }
      if (x.rec === 'ยืดรอบได้' && x.annual != null) saveYr += x.annual - x.annualNew;
    });
    var card = function (k, label, val, sub, cls) {
      return '<div onclick="document.getElementById(\'rv-filter\').value=\'' + k + '\';renderCalReview()" class="cursor-pointer rounded-xl border p-3 hover:shadow-md transition ' + cls + '"><div class="text-[11px] font-bold">' + label + '</div>'
        + '<div class="text-2xl font-black mt-1">' + val + '</div>' + (sub ? '<div class="text-[11px] opacity-80 mt-0.5">' + sub + '</div>' : '') + '</div>';
    };
    document.getElementById('rv-cards').innerHTML =
        card('ยืดรอบได้', 'ยืดรอบได้', (cnt['ยืดรอบได้'] || 0) + ' เครื่อง', saveYr ? 'ประหยัดได้ ≈ ' + _baht(saveYr) + ' บาท/ปี' : 'ยังไม่มีข้อมูลค่าใช้จ่าย', RV_CLS['ยืดรอบได้'])
      + card('คงรอบ', 'คงรอบ', (cnt['คงรอบ'] || 0) + ' เครื่อง', '', RV_CLS['คงรอบ'])
      + card('ควรหดรอบ', 'ควรหดรอบ / เฝ้าระวัง', (cnt['ควรหดรอบ'] || 0) + ' เครื่อง', 'ค่าเลื่อนหรือเคยไม่ผ่าน', RV_CLS['ควรหดรอบ'])
      + card('ข้อมูลไม่พอ', 'ข้อมูลไม่พอ', (cnt['ข้อมูลไม่พอ'] || 0) + ' เครื่อง', 'ประวัติน้อยกว่า ' + R.minHistory + ' ครั้ง', RV_CLS['ข้อมูลไม่พอ'])
      + card('all', 'ค่าสอบเทียบต่อปี (ประมาณ)', _baht(annualAll) + ' ฿', 'จาก ' + withCost + ' เครื่องที่มีข้อมูลค่าใช้จ่าย', 'bg-white dark:bg-slate-800 border-slate-200 text-slate-700 dark:text-slate-200');

    var q = ((document.getElementById('rv-search') || {}).value || '').toLowerCase().trim(), fl = (document.getElementById('rv-filter') || {}).value || 'all';
    var order = { 'ยืดรอบได้': 0, 'ควรหดรอบ': 1, 'คงรอบ': 2, 'ข้อมูลไม่พอ': 3 };
    var list = D.filter(function (x) { return (fl === 'all' || x.rec === fl) && (!q || [x.code, x.name, x.type].join(' ').toLowerCase().indexOf(q) !== -1); })
      .sort(function (a, b) { return order[a.rec] - order[b.rec] || ((b.annual || 0) - (b.annualNew || 0)) - ((a.annual || 0) - (a.annualNew || 0)); });
    var head = _canRetire();
    var tb = document.getElementById('rv-tbody');
    tb.innerHTML = list.length ? list.map(function (x) {
      var i = D.indexOf(x);
      var bars = x.history.slice(-3).map(function (h) {
        if (h.used == null) return '<span class="inline-block text-[10px] text-slate-400 mr-1">' + (h.pass ? 'ผ่าน' : 'ไม่ผ่าน') + '</span>';
        var w = Math.min(100, h.used), col = !h.pass || h.used > R.shortenUsed ? '#e11d48' : h.used > R.extendMaxUsed ? '#d97706' : '#059669';
        return '<div class="flex items-center gap-1.5 text-[10px]" title="' + _esc2(h.date) + '"><div style="width:90px;height:7px;background:#e2e8f0;border-radius:4px;overflow:hidden"><div style="width:' + w + '%;height:7px;background:' + col + '"></div></div>' + h.used + '%</div>';
      }).join('');
      var change = x.rec === 'ยืดรอบได้' || x.rec === 'ควรหดรอบ' ? ' → <b>' + x.suggested + ' เดือน</b>' : '';
      var costTxt = x.annual == null ? '<span class="text-slate-300">—</span>'
        : _baht(x.annual) + (x.suggested !== x.interval && (x.rec === 'ยืดรอบได้' || x.rec === 'ควรหดรอบ') ? '<div class="text-[11px] ' + (x.annualNew < x.annual ? 'text-emerald-600' : 'text-rose-600') + '">→ ' + _baht(x.annualNew) + '</div>' : '');
      var acts = _iconBtn('showCalReviewHistory', i, 'history', 'ประวัติ', 'bg-slate-50 border-slate-200 text-slate-500 hover:bg-slate-100');
      if (head) acts += _iconBtn('openApplyInterval', i, 'tune', 'ปรับรอบสอบเทียบ', 'bg-emerald-50 border-emerald-200 text-emerald-600 hover:bg-emerald-100');
      return '<tr class="hover:bg-slate-50 dark:hover:bg-slate-700/30">'
        + '<td class="px-3 py-2"><div class="font-semibold">' + _esc2(x.code || '-') + '</div><div class="text-[11px] text-slate-400">' + _esc2(x.name || '') + ' · ' + _esc2(x.type || '') + '</div></td>'
        + '<td class="px-3 py-2 text-center text-xs">' + x.interval + ' เดือน</td>'
        + '<td class="px-3 py-2 text-center text-xs">' + x.streak + ' / ' + x.count + '</td>'
        + '<td class="px-3 py-2">' + (bars || '<span class="text-slate-300 text-xs">—</span>') + '</td>'
        + '<td class="px-3 py-2"><span class="inline-block px-2 py-0.5 rounded-full border text-[11px] font-bold ' + RV_CLS[x.rec] + '">' + x.rec + '</span><span class="text-xs">' + change + '</span>'
        +   '<div class="text-[11px] text-slate-500 mt-0.5">' + _esc2(x.why) + (x.rec === 'ยืดรอบได้' && !x.hasIcPlan ? ' · <span class="text-amber-600">ควรตั้งแผนตรวจระหว่างรอบก่อน</span>' : '') + '</div></td>'
        + '<td class="px-3 py-2 text-right text-xs">' + costTxt + '</td>'
        + '<td class="px-3 py-2 text-center"><div class="inline-flex gap-1.5">' + acts + '</div></td></tr>';
    }).join('') : '<tr><td colspan="7" class="px-4 py-10 text-center text-slate-400 italic">ไม่พบรายการ</td></tr>';

    // รวมนัดสอบเทียบ
    var today = new Date(); today.setHours(0, 0, 0, 0);
    var lim = new Date(today.getTime() + 180 * 86400000), groups = {};
    D.forEach(function (x) {
      var nd = x.nextDue ? new Date(String(x.nextDue).slice(0, 10) + 'T00:00:00') : null;
      if (!nd || isNaN(nd) || nd > lim) return;
      var key = (x.vendor || 'ไม่ระบุผู้ให้บริการ') + '||' + (x.location || '-');
      (groups[key] = groups[key] || []).push({ x: x, d: nd });
    });
    var clusters = [];
    Object.keys(groups).forEach(function (k) {
      var arr = groups[k].sort(function (a, b) { return a.d - b.d; }), cur = null;
      arr.forEach(function (it) {
        if (cur && (it.d - cur.start) / 86400000 <= 60) cur.items.push(it);
        else { cur = { key: k, start: it.d, items: [it] }; clusters.push(cur); }
      });
    });
    clusters = clusters.filter(function (c) { return c.items.length >= 2; }).sort(function (a, b) { return a.start - b.start; });
    document.getElementById('rv-batch').innerHTML = clusters.length ? clusters.map(function (c) {
      var p = c.key.split('||'), last = c.items[c.items.length - 1].d, iso = function (d) { return d.getFullYear() + '-' + ('0' + (d.getMonth() + 1)).slice(-2) + '-' + ('0' + d.getDate()).slice(-2); };
      return '<div class="rounded-lg border border-sky-200 bg-sky-50/60 dark:bg-sky-900/20 dark:border-sky-800 px-3 py-2">'
        + '<div class="text-sm font-semibold text-sky-800 dark:text-sky-300">' + c.items.length + ' เครื่อง · นัดวันที่ ' + _iaFmt(iso(c.start)) + (c.start < today ? ' <span class="text-rose-600">(เลยกำหนดแล้ว)</span>' : '') + '</div>'
        + '<div class="text-[11px] text-slate-500">' + _esc2(p[0]) + ' · ' + _esc2(p[1]) + ' · ครบกำหนด ' + _iaFmt(iso(c.start)) + ' – ' + _iaFmt(iso(last)) + '</div>'
        + '<div class="text-[11px] text-slate-600 dark:text-slate-300 mt-1">' + c.items.map(function (it) { return _esc2(it.x.code || it.x.eqId); }).join(', ') + '</div></div>';
    }).join('') : '<p class="text-xs text-slate-400 italic">ไม่มีกลุ่มที่รวมนัดได้ในช่วง 180 วันข้างหน้า</p>';

    // ค่าใช้จ่ายตามผู้ให้บริการ (12 เดือน)
    var cut = new Date(today.getTime() - 365 * 86400000), V = {};
    (_rv.costs || []).forEach(function (c) {
      var d = c.date ? new Date(c.date + 'T00:00:00') : null; if (!d || d < cut) return;
      var v = V[c.vendor || 'ไม่ระบุ'] = V[c.vendor || 'ไม่ระบุ'] || { n: 0, sum: 0 }; v.n++; v.sum += c.cost;
    });
    var vk = Object.keys(V).sort(function (a, b) { return V[b].sum - V[a].sum; });
    document.getElementById('rv-vendor').innerHTML = vk.length ? vk.map(function (k) {
      return '<tr><td class="px-3 py-2">' + _esc2(k) + '</td><td class="px-3 py-2 text-right">' + V[k].n + '</td><td class="px-3 py-2 text-right">' + _baht(V[k].sum) + '</td><td class="px-3 py-2 text-right">' + _baht(V[k].sum / V[k].n) + '</td></tr>';
    }).join('') : '<tr><td colspan="4" class="px-3 py-6 text-center text-xs text-slate-400 italic">ยังไม่มีข้อมูลในเมนู "ค่าใช้จ่ายสอบเทียบ" ช่วง 12 เดือน</td></tr>';
  }

  function showCalReviewHistory(i) {
    var x = _rv && _rv.data[i]; if (!x) return;
    var rows = x.history.slice().reverse().map(function (h) {
      return '<tr><td style="padding:5px 8px">' + _iaFmt(h.date) + '</td><td style="padding:5px 8px;font-weight:700;color:' + (h.pass ? '#15803d' : '#be123c') + '">' + (h.pass ? 'ผ่าน' : 'ไม่ผ่าน') + '</td>'
        + '<td style="padding:5px 8px;text-align:right">' + (h.used == null ? '-' : h.used + '%') + '</td><td style="padding:5px 8px">' + _esc2(h.lab || '-') + '</td></tr>';
    }).join('');
    _miniModal('📈 ประวัติผลสอบเทียบ', _esc2((x.code || '-') + ' · ' + (x.name || '')), '#334155',
      '<table style="width:100%;border-collapse:collapse;font-size:12.5px"><tr style="background:#f1f5f9"><th style="padding:6px 8px;text-align:left">วันที่</th><th style="padding:6px 8px;text-align:left">ผล</th><th style="padding:6px 8px;text-align:right">% เกณฑ์ที่ใช้</th><th style="padding:6px 8px;text-align:left">ผู้ให้บริการ</th></tr>'
      + (rows || '<tr><td colspan="4" style="padding:10px;text-align:center;color:#94a3b8">ยังไม่มีประวัติ</td></tr>') + '</table>'
      + '<div style="font-size:12px;color:#475569;margin-top:10px">คำแนะนำ: <b>' + x.rec + '</b> · ' + _esc2(x.why) + ' · ตรวจระหว่างรอบ: ' + (x.hasIcPlan ? 'มีแผน' + (x.icFail ? ' (ไม่ผ่าน ' + x.icFail + ' ครั้งใน 12 เดือน)' : '') : 'ยังไม่มีแผน') + '</div>',
      'ปิด', function (ov, err, busy, close) { close(); });
  }

  function openApplyInterval(i) {
    var x = _rv && _rv.data[i]; if (!x) return;
    var opts = [3, 6, 9, 12, 18, 24, 36].map(function (m) { return '<option value="' + m + '"' + (m === x.suggested ? ' selected' : '') + '>' + m + ' เดือน' + (m === x.interval ? ' (ปัจจุบัน)' : '') + (m === x.suggested && m !== x.interval ? ' (แนะนำ)' : '') + '</option>'; }).join('');
    var ev = 'ประวัติ ' + x.count + ' ครั้ง ผ่านติดกัน ' + x.streak + ' ครั้ง ใช้เกณฑ์สูงสุด ' + (x.maxUsed == null ? '-' : x.maxUsed + '%') + ' (3 ครั้งล่าสุด) · ตรวจระหว่างรอบ: ' + (x.hasIcPlan ? 'มีแผน, ไม่ผ่าน ' + x.icFail + ' ครั้ง/12 เดือน' : 'ไม่มีแผน');
    var defReason = x.rec === 'ยืดรอบได้' ? 'ผลสอบเทียบผ่านต่อเนื่องและค่าเลื่อนน้อยเมื่อเทียบเกณฑ์ ยืดรอบตามแนวทาง ILAC-G24 โดยติดตามด้วยการตรวจระหว่างรอบ'
      : x.rec === 'ควรหดรอบ' ? 'ผลสอบเทียบเลื่อนเข้าใกล้ขอบเกณฑ์ / เคยไม่ผ่าน จึงหดรอบเพื่อลดความเสี่ยงต่อผลตรวจ' : '';
    var body = '<div style="background:#f8fafc;border:1px solid #e2e8f0;border-radius:8px;padding:8px 12px;font-size:12px;color:#334155">หลักฐาน: ' + _esc2(ev) + '</div>'
      + (x.rec === 'ยืดรอบได้' && !x.hasIcPlan ? '<div style="background:#fffbeb;border:1px solid #fde68a;color:#92400e;border-radius:8px;padding:8px 12px;font-size:12px;margin-top:8px">⚠️ แนะนำให้ตั้งแผนตรวจระหว่างรอบก่อนยืดรอบ เพื่อยืนยันว่าเครื่องยังอยู่ในเกณฑ์ระหว่างรอบที่ยาวขึ้น</div>' : '')
      + '<label style="' + _lb + '">รอบสอบเทียบใหม่</label><select data-f="m" style="' + _fs + '">' + opts + '</select>'
      + '<label style="' + _lb + '">เหตุผลการปรับรอบ * (บันทึกเป็นหลักฐาน)</label><textarea data-f="r" rows="3" style="' + _fs + 'resize:vertical;">' + _esc2(defReason) + '</textarea>'
      + '<div style="font-size:11px;color:#64748b;margin-top:6px">ระบบจะคำนวณ "สอบเทียบครั้งถัดไป" ใหม่จากวันที่สอบเทียบล่าสุด และบันทึกในชีต IntervalChanges</div>';
    _miniModal('⚙️ ปรับรอบสอบเทียบ', _esc2((x.code || '-') + ' · ' + (x.name || '') + ' · ปัจจุบัน ' + x.interval + ' เดือน'), '#047857', body, '💾 บันทึกการปรับรอบ',
      function (ov, err, busy, close) {
        var m = ov.querySelector('[data-f="m"]').value, r = ov.querySelector('[data-f="r"]').value.trim();
        if (+m === x.interval) return err('รอบใหม่เท่ากับรอบปัจจุบัน');
        if (r.length < 10) return err('กรุณาระบุเหตุผลการปรับรอบ');
        busy(true);
        google.script.run.withSuccessHandler(function (res) {
          busy(false);
          if (!res || !res.success) return err((res && res.message) || 'บันทึกไม่สำเร็จ');
          close(); showToast('✅ ปรับรอบเป็น ' + m + ' เดือน' + (res.next ? ' · ครบกำหนดครั้งถัดไป ' + _iaFmt(res.next) : ''), 'success'); loadCalReview();
        }).withFailureHandler(function (e) { busy(false); err(e && e.message || String(e)); })
          .applyCalInterval(x.eqId, m, r, ev, currentUser ? currentUser.username : '');
      });
  }

  function printCalReview() {
    if (!_rv) { showToast('กด "วิเคราะห์ใหม่" ก่อน', 'warning'); return; }
    var by = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser ? currentUser.username : '-'), now = new Date();
    var at = now.toLocaleDateString('th-TH', { day: 'numeric', month: 'short', year: 'numeric' }) + ' ' + now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + ' น.';
    var order = { 'ยืดรอบได้': 0, 'ควรหดรอบ': 1, 'คงรอบ': 2, 'ข้อมูลไม่พอ': 3 };
    var rows = _rv.data.slice().sort(function (a, b) { return order[a.rec] - order[b.rec]; }).map(function (x, k) {
      return '<tr><td class="c">' + (k + 1) + '</td><td>' + _esc2(x.code || '-') + '<br><small>' + _esc2(x.name || '') + '</small></td><td class="c">' + x.interval + '</td><td class="c">' + x.streak + '/' + x.count + '</td>'
        + '<td class="c">' + (x.maxUsed == null ? '-' : x.maxUsed + '%') + '</td><td><b>' + x.rec + '</b>' + (x.suggested !== x.interval ? ' → ' + x.suggested + ' ด.' : '') + '<br><small>' + _esc2(x.why) + '</small></td>'
        + '<td class="r">' + _baht(x.annual) + '</td><td class="r">' + (x.suggested !== x.interval ? _baht(x.annualNew) : '-') + '</td></tr>';
    }).join('');
    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="UTF-8"><title>รายงานทบทวนรอบสอบเทียบ</title><style>'
      + '@import url("https://fonts.googleapis.com/css2?family=Sarabun:wght@400;600;700&display=swap");@page{size:A4 landscape;margin:10mm}'
      + 'body{font-family:"Sarabun",sans-serif;font-size:10.5pt;color:#1e293b;margin:0}h1{font-size:15pt;text-align:center;margin:0 0 4px;color:#047857}.sub{text-align:center;font-size:9.5pt;color:#64748b;margin-bottom:8px}'
      + 'table{width:100%;border-collapse:collapse}th,td{border:1px solid #94a3b8;padding:4px 6px;vertical-align:top}th{background:#e2e8f0}.c{text-align:center}.r{text-align:right}small{color:#64748b}'
      + '.foot{margin-top:8px;text-align:right;font-size:9pt;color:#64748b}</style></head><body><h1>รายงานทบทวนรอบสอบเทียบเครื่องมือ</h1>'
      + '<div class="sub">แนวทาง ILAC-G24 / OIML D10 · ยืดรอบเมื่อผ่านติดกัน ≥ ' + _rv.rules.minHistory + ' ครั้ง และใช้เกณฑ์ ≤ ' + _rv.rules.extendMaxUsed + '% · หดรอบเมื่อไม่ผ่าน หรือใช้เกณฑ์ > ' + _rv.rules.shortenUsed + '%</div>'
      + '<table><thead><tr><th>#</th><th>เครื่องมือ</th><th>รอบ (เดือน)</th><th>ผ่านติดกัน</th><th>% เกณฑ์สูงสุด</th><th>คำแนะนำ</th><th>ค่าใช้จ่าย/ปี ปัจจุบัน</th><th>ค่าใช้จ่าย/ปี ใหม่</th></tr></thead><tbody>' + rows + '</tbody></table>'
      + '<div class="foot">พิมพ์โดย: ' + _esc2(by) + ' &nbsp;|&nbsp; วันที่พิมพ์: ' + at + '</div></body></html>';
    var win = window.open('', '_blank', 'width=1100,height=800');
    if (!win) { showToast('เบราว์เซอร์บล็อกหน้าต่างใหม่ กรุณาอนุญาต Pop-up', 'warning'); return; }
    win.document.open(); win.document.write(html); win.document.close();
    win.onload = function () { setTimeout(function () { win.focus(); win.print(); }, 500); };
  }

  // ✅ ลิงก์แดชบอร์ดผู้บริหาร (Admin)
  function execLinkAction(act) {
    var inp = document.getElementById('exec-link');
    if (act === 'copy' || act === 'open') {
      if (!inp.value) { showToast('กด "แสดงลิงก์" ก่อน', 'warning'); return; }
      if (act === 'open') { window.open(inp.value, '_blank'); return; }
      inp.select(); try { navigator.clipboard ? navigator.clipboard.writeText(inp.value) : document.execCommand('copy'); } catch (e) { document.execCommand('copy'); }
      showToast('📋 คัดลอกลิงก์แล้ว', 'success'); return;
    }
    if (act === 'new' && !confirm('สร้างลิงก์ใหม่? ลิงก์เดิมที่เคยส่งให้ผู้บริหารจะใช้ไม่ได้ทันที')) return;
    showLoader();
    google.script.run.withSuccessHandler(function (r) {
      hideLoader();
      if (!r || !r.success) { showToast((r && r.message) || 'ไม่สำเร็จ', 'danger'); return; }
      inp.value = r.url; if (act === 'new') showToast('🔑 สร้างลิงก์ใหม่แล้ว — ส่งลิงก์นี้ให้ผู้บริหารแทนลิงก์เดิม', 'success');
    }).withFailureHandler(function (e) { hideLoader(); showToast(e && e.message || String(e), 'danger'); })
      [act === 'new' ? 'regenerateExecDashKey' : 'getExecDashLink'](currentUser ? currentUser.username : '');
  }

  // เปลี่ยนหัวข้อช่องวันที่ตามประเภท: สอบเทียบ / บำรุงรักษา
  function _relabelScheduleFields() {
    var sel = document.getElementById('isCalibrated');
    var w = sel && sel.value === 'บำรุงรักษา' ? 'บำรุงรักษา' : 'สอบเทียบ';
    var a = document.getElementById('lbl-lastCalDate'), b = document.getElementById('lbl-calInterval'), c = document.getElementById('lbl-nextCalDate');
    if (a) a.textContent = 'วันที่' + w + 'ล่าสุด';
    if (b) b.textContent = 'ระยะเวลาการ' + w + ' (รอบ)';
    if (c) c.textContent = 'วันที่' + w + 'รอบถัดไป (Due Date)';
  }

  function _needsSchedule(v, item) {
    if (item && String(item['สถานะเครื่องมือ'] || '').trim() === 'จำหน่ายออก') return false;   // ปลดระวางแล้ว → ไม่อยู่ในแผน/แจ้งเตือน
    v = String(v || '').trim(); return v === 'สอบเทียบ' || v === 'บำรุงรักษา';
  }
  function _isMaint(item) { return String((item && (item['สถานะการสอบเทียบ'] || item['การสอบเทียบหรือไม่'])) || '').trim() === 'บำรุงรักษา'; }

  // ✅ ป้ายกำกับเครื่องมือที่ไม่ต้องสอบเทียบ: "บำรุงรักษา" หรือ "ไม่สอบเทียบ"
  function _noCalLabel(item) {
    var v = String((item && (item['สถานะการสอบเทียบ'] || item['การสอบเทียบหรือไม่'])) || '').trim();
    return v === 'บำรุงรักษา' ? 'บำรุงรักษา' : 'ไม่สอบเทียบ';
  }

  function _buildHistoryHTML(item) {
    if (!item) return '<p class="text-red-400">ไม่พบข้อมูลเครื่องมือ</p>';

    // ✅ แก้ชื่อ column ให้ตรงกับ Sheet จริง
    var isCal   = _needsSchedule(item['สถานะการสอบเทียบ'] || item['การสอบเทียบหรือไม่'] || '', item);
    var lastCal = item['วันที่สอบเทียบล่าสุด'] || item['วันที่ทำการสอบเทียบ (Cal-Date)'] || '';
    var intv    = item['ระยะเวลาการสอบเทียบ'] || '';
    // ✅ ใช้ Due Date จาก Sheet ก่อน (col 20)
    var dueDate = item['สอบเทียบครั้งถัดไป'] || item['Due Date'] || '';
    if (!dueDate && isCal && lastCal && intv && !/^\d{4}-\d{2}-\d{2}/.test(intv)) {
      var d = new Date(lastCal);
      if (!isNaN(d)) {
        d.setMonth(d.getMonth() + (parseInt(intv) || 12));
        d.setDate(d.getDate() - 1);
        dueDate = _fmtDate(d.toISOString().slice(0,10));
      }
    } else if (dueDate) {
      dueDate = _fmtDate(dueDate);
    }
    var compRaw   = item['ชื่อบริษัทผู้แทน/โทรศัพท์'] || '';
    var compParts = compRaw.split('/');
    var compName  = compParts[0] ? compParts[0].trim() : (item['ชื่อบริษัท'] || '-');
    var compRep   = compParts[1] ? compParts[1].trim() : (item['เซลล์'] || '-');
    var compPhone = compParts[2] ? compParts[2].trim() : (item['เบอร์ติดต่อ'] || '-');

    function row(no, label, val) {
      return '<div class="flex gap-3 py-2 border-b border-slate-100 dark:border-slate-700/60">'
        + '<span class="w-5 text-xs font-bold text-primary-600 shrink-0">' + no + '.</span>'
        + '<span class="w-44 text-xs font-semibold text-slate-600 dark:text-slate-400 shrink-0">' + label + '</span>'
        + '<span class="flex-1 text-sm text-slate-800 dark:text-slate-100 font-medium">' + (val || '-') + '</span>'
        + '</div>';
    }

    var html = '';
    html += row(1,  'ชื่อเครื่องมือ',               item['ชื่อเครื่องมือ']);
    html += row(2,  'ประเภทเครื่องมือ',              item['ประเภทเครื่องมือ']);
    html += row(3,  'รหัสครุภัณฑ์',                  item['รหัสครุภัณฑ์']);
    html += row(4,  'ยี่ห้อ / รุ่น / รหัสเครื่อง',  item['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || item['ยี่ห้อ/รุ่น/Serial']);
    html += row(5,  'คุณลักษณะเครื่อง',              item['คุณลักษณะเครื่อง'] || item['คุณลักษณะ'] || item['รายละเอียด/สเปค']);
    html += row(6,  'บริษัท / ผู้แทน / เบอร์โทร',   compName + (compRep !== '-' ? ' / ' + compRep : '') + (compPhone !== '-' ? ' / ' + compPhone : ''));
    html += row(7,  'ช่างซ่อมบำรุง / เบอร์',        item['รายชื่อช่างซ่อมบำรุง'] || item['ช่างซ่อม'] || item['ชื่อช่าง'] || '-');
    html += row(8,  'วันที่รับ / เริ่มใช้เครื่อง',  (_fmtDate(item['วันรับเครื่อง'] || item['วันที่รับเครื่อง']) || '-') + (item['วันเริ่มใช้เครื่อง'] || item['วันที่เริ่มใช้เครื่อง'] ? ' / ' + _fmtDate(item['วันเริ่มใช้เครื่อง'] || item['วันที่เริ่มใช้เครื่อง']) : ''));
    html += row(9,  'สถานที่ติดตั้ง / วางเครื่อง',  item['สถานที่ติดตั้ง']);
    html += row(10, 'สถานภาพเครื่องเมื่อรับ',        item['สถานะเครื่องเมื่อรับ'] || item['สถานภาพ'] || item['สถานะเครื่อง']);
    html += row(11, 'คู่มือการใช้งาน',               item['คู่มือการใช้งาน'] || item['คู่มือ'] ? '✅ มีคู่มือ' : '-');
    html += row(12, 'กำหนดอายุการใช้งาน', (item['อายุการใช้งาน(ปี)'] || item['อายุการใช้งาน (ปี)'] || '-') + (item['อายุการใช้งาน(ปี)'] || item['อายุการใช้งาน (ปี)'] ? ' ปี' : ''));
    html += row(13, 'มูลค่าเครื่องมือ',              item['มูลค่าเครื่องมือ'] ? parseFloat(String(item['มูลค่าเครื่องมือ']).replace(/,/g,'')).toLocaleString('th-TH') + ' บาท' : '-');

    // ข้อ 14 สอบเทียบ
    html += '<div class="flex gap-3 py-2">'
      + '<span class="w-5 text-xs font-bold text-primary-600 shrink-0">14.</span>'
      + '<span class="w-44 text-xs font-semibold text-slate-600 dark:text-slate-400 shrink-0">ความจำเป็นในการสอบเทียบ</span>'
      + '<span class="flex-1">'
      + (isCal
          ? (_isMaint(item) ? '<span class="text-sm font-medium text-sky-700">☑ บำรุงรักษา (Preventive Maintenance)</span>' : '<span class="text-sm font-medium text-emerald-700">☑ ต้องการสอบเทียบ</span>')
            + '<div class="mt-1 text-xs text-slate-600 dark:text-slate-400 space-y-0.5">'
            + '<div>วันที่' + (_isMaint(item) ? 'บำรุงรักษา' : 'สอบเทียบ') + 'ล่าสุด: <strong>' + _fmtDate(lastCal) + '</strong></div>'
            + '<div>ระยะเวลาการ' + (_isMaint(item) ? 'บำรุงรักษา' : 'สอบเทียบ') + ': <strong>' + intv + (intv && !isNaN(intv) ? ' เดือน' : '') + '</strong></div>'
            + '<div>Due Date (รอบถัดไป): <strong class="text-red-600">' + dueDate + '</strong></div>'
            + '</div>'
          : (_noCalLabel(item) === 'บำรุงรักษา'
              ? '<span class="text-sm font-medium text-sky-700">☑ บำรุงรักษา (ไม่ต้องสอบเทียบ)</span>'
              : '<span class="text-sm font-medium text-slate-500">☐ ไม่สอบเทียบ</span>'))
      + '</span></div>';

    return html;
  }

  function viewEquipmentHistory(idx) {
    var item = _getItem(idx);
    var modal = document.getElementById('history-record-modal');
    var cont = document.getElementById('history-record-content');
    if (!modal || !cont || !item) return;
    cont.innerHTML = _buildHistoryHTML(item);
    modal.setAttribute('data-idx', idx);
    modal.classList.remove('hidden');
    modal.scrollTop = 0;
  }

  function closeHistoryModal() {
    var modal = document.getElementById('history-record-modal');
    if (modal) modal.classList.add('hidden');
  }

  function editEquipmentRecord(idx) {
    var item = _getItem(idx);
    if (!item) { console.error('ไม่พบข้อมูล index:', idx); return; }
    console.log('[edit] item:', item['ชื่อเครื่องมือ'], '_index:', item._index, '_rowIndex:', item._rowIndex);
    // ปิด history modal ถ้าเปิดอยู่
    closeHistoryModal();
    // สลับไปหน้า form แล้วค่อย populate
    switchPage('form');
    setTimeout(function() {
      _openFormForEdit(item, idx);
    }, 600);
  }

  function _openFormForEdit(item, idx) {
    // เปิด panel
    var panel = document.getElementById('equipment-form-panel');
    if (!panel) return;
    panel.classList.remove('hidden');
    var summaryCard = document.getElementById('form-summary-card');
    if (summaryCard) summaryCard.classList.add('hidden');

    // เปลี่ยน title/icon เป็น "แก้ไข"
    var titleEl = document.getElementById('form-panel-title');
    var iconEl  = document.getElementById('form-panel-icon');
    if (titleEl) titleEl.textContent = 'แก้ไขข้อมูลเครื่องมือ';
    if (iconEl)  iconEl.textContent  = 'edit';

    // reset ก่อนแล้ว populate
    var form = document.getElementById('equipment-form');
    if (form) form.reset();

    // เก็บ row index ไว้ใน hidden field
    var rowIndexEl = document.getElementById('eqRowIndex');
    if (rowIndexEl) rowIndexEl.value = (item._rowIndex !== undefined ? item._rowIndex : (item._index !== undefined ? item._index + 2 : ''));

    // ---- helper: set value พร้อม log ถ้า field ไม่มี ----
    function setVal(id, val) {
      var el = document.getElementById(id);
      if (el) el.value = (val !== undefined && val !== null) ? String(val) : '';
    }

    // ---- parse ชื่อบริษัทผู้แทน/โทรศัพท์ (เก็บเป็น "บริษัท / เซลล์ / โทร") ----
    var compRaw   = item['ชื่อบริษัทผู้แทน/โทรศัพท์'] || '';
    var compParts = compRaw.split('/');
    var compName  = (compParts[0] || '').trim();
    var compRep   = (compParts[1] || '').trim();
    var compPhone = (compParts[2] || '').trim();

    // ---- parse ช่างซ่อมบำรุง (เก็บเป็น "ชื่อช่าง / เบอร์") ----
    var techRaw   = item['รายชื่อช่างซ่อมบำรุง'] || item['ช่างซ่อม'] || item['ชื่อช่าง'] || '';
    var techParts = techRaw.split('/');
    var techName  = (techParts[0] || '').trim();
    var techPhone = (techParts[1] || '').trim();

    // ---- parse ผลิตภัณฑ์/รุ่น/รหัสเครื่อง ----
    var modelRaw   = item['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || item['ยี่ห้อ/รุ่น/Serial'] || '';
    var modelParts = modelRaw.split('/');
    var brandVal   = (modelParts[0] || '').trim();
    var modelVal   = (modelParts[1] || '').trim();
    var snVal      = (modelParts[2] || '').trim();

    // ---- calibration ----
    var calStatus = item['สถานะการสอบเทียบ'] || item['การสอบเทียบหรือไม่'] || '';
    var isCal     = _needsSchedule(calStatus);
    var lastCal   = item['วันที่สอบเทียบล่าสุด'] || item['วันที่ทำการสอบเทียบ (Cal-Date)'] || '';
    var intv      = item['ระยะเวลาการสอบเทียบ'] || '';
    var nextCal   = item['สอบเทียบครั้งถัดไป'] || item['Due Date'] || item['วันที่สอบเทียบรอบถัดไป'] || '';

    // ---- populate ทุก field ตาม column จริงใน Sheet ----
    setVal('eqName',       item['ชื่อเครื่องมือ']);
    setVal('eqCode',       item['รหัสครุภัณฑ์']);
    setVal('eqSpec',       item['คุณลักษณะเครื่อง'] || item['คุณลักษณะ'] || item['รายละเอียด/สเปค']);
    setVal('compName',     compName);
    setVal('compRep',      compRep);
    setVal('compPhone',    compPhone);
    setVal('techName',     techName);
    setVal('techPhone',    techPhone);
    setVal('dateReceive',  item['วันรับเครื่อง']     || item['วันที่รับเครื่อง'] || '');
    setVal('dateStart',    item['วันเริ่มใช้เครื่อง'] || item['วันที่เริ่มใช้เครื่อง'] || '');
    setVal('status',       item['สถานะเครื่องเมื่อรับ'] || item['สถานภาพ'] || item['สถานะเครื่อง'] || '');
    setVal('lifespan',     item['อายุการใช้งาน(ปี)']   || item['อายุการใช้งาน (ปี)'] || '');
    setVal('value',        item['มูลค่าเครื่องมือ'] || '');
    setVal('brand',        brandVal);
    setVal('model',        modelVal);
    setVal('serialNumber', snVal);
    setVal('isCalibrated', calStatus);
    setVal('lastCalDate',  lastCal);
    setVal('nextCalDate',  nextCal);
    setVal('calInterval',  intv);

    // ---- eqType select (รอ options โหลดก่อน) ----
    var typeEl     = document.getElementById('eqType');
    var targetType = item['ประเภทเครื่องมือ'] || '';
    if (typeEl) {
      (function trySetType(tries) {
        if (typeEl.options.length > 1) { typeEl.value = targetType; }
        else if (tries > 0) { setTimeout(function() { trySetType(tries - 1); }, 200); }
      })(20);
    }

    // ---- location select (รอ options โหลดก่อน) ----
    var locEl     = document.getElementById('location');
    var targetLoc = item['สถานที่ติดตั้ง'] || '';
    if (locEl) {
      (function trySetLoc(tries) {
        if (locEl.options.length > 1) { locEl.value = targetLoc; }
        else if (tries > 0) { setTimeout(function() { trySetLoc(tries - 1); }, 200); }
      })(20);
    }

    // ---- แสดง/ซ่อน cal fields ----
    var calContainer = document.getElementById('calDateFieldContainer');
    if (calContainer) {
      if (isCal) calContainer.classList.remove('hidden');
      else       calContainer.classList.add('hidden');
    }
    _relabelScheduleFields(); // หัวข้อช่องวันที่: สอบเทียบ / บำรุงรักษา

    // scroll ไปที่ form
    panel.scrollIntoView({ behavior: 'smooth', block: 'start' });
  }

  function printEquipmentHistory(idx) {
    var item = _getItem(idx);
    if (!item) return;
    _doPrintHistory(item);
  }

  function printHistoryModal() {
    var content = document.getElementById('history-record-content');
    if (!content) return;
    // ดึง item จาก data ที่ render อยู่ใน modal
    var activeIdx = parseInt(document.getElementById('history-record-modal').getAttribute('data-idx') || '0');
    var item = _getItem(activeIdx);
    if (item) _doPrintHistory(item);
  }

  function _doPrintHistory(item) {
    // ✅ แก้ชื่อ column ให้ตรงกับ Sheet จริง
    var calNeed = _needsSchedule(item['สถานะการสอบเทียบ'] || item['การสอบเทียบหรือไม่'] || '', item);
    var lastCal = item['วันที่สอบเทียบล่าสุด'] || item['วันที่ทำการสอบเทียบ (Cal-Date)'] || '';
    var intv    = item['ระยะเวลาการสอบเทียบ'] || '';
    // ✅ ใช้ Due Date จาก Sheet ก่อน
    var dueDate = item['สอบเทียบครั้งถัดไป'] || item['Due Date'] || '';
    if (!dueDate && calNeed && lastCal && intv) {
      var d2 = new Date(lastCal);
      if (!isNaN(d2)) {
        d2.setMonth(d2.getMonth() + (parseInt(intv) || 12));
        d2.setDate(d2.getDate() - 1);
        dueDate = _fmtDate(d2.toISOString().slice(0,10));
      }
    } else if (dueDate) {
      dueDate = _fmtDate(dueDate);
    }
    var compRaw   = item['ชื่อบริษัทผู้แทน/โทรศัพท์'] || '';
    var compParts = compRaw.split('/');
    var compName  = compParts[0] ? compParts[0].trim() : '-';
    var compRep   = compParts[1] ? compParts[1].trim() : (item['เซลล์'] || '');
    var compPhone = compParts[2] ? compParts[2].trim() : (item['เบอร์ติดต่อ'] || '');

    var now = new Date();
    var printedBy = (typeof currentUser !== 'undefined' && currentUser) ? (currentUser.fullname || currentUser.username) : '';
    var printedDate = now.toLocaleDateString('th-TH', {day:'numeric', month:'short', year:'numeric'})
                    + ' เวลา ' + now.toLocaleTimeString('th-TH', {hour:'2-digit', minute:'2-digit'}) + ' น.';

    // SVG logo (ไอคอนกล้องจุลทรรศน์ สีขาว บน พื้น navy)
    var logoSVG = '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 48 48" width="38" height="38">'
      + '<rect width="48" height="48" rx="6" fill="#1e3a8a"/>'
      + '<path d="M24 10 L28 18 L20 18 Z" fill="white"/>'
      + '<rect x="22" y="18" width="4" height="12" rx="2" fill="white"/>'
      + '<ellipse cx="24" cy="33" rx="8" ry="3" fill="none" stroke="white" stroke-width="2"/>'
      + '<line x1="16" y1="38" x2="32" y2="38" stroke="white" stroke-width="2.5" stroke-linecap="round"/>'
      + '</svg>';

    function row(no, label, val) {
      var v = (val === undefined || val === null || val === '') ? '-' : String(val);
      return '<tr>'
        + '<td class="c-no">' + no + '.</td>'
        + '<td class="c-label">' + label + '</td>'
        + '<td class="c-val">' + v + '</td>'
        + '</tr>';
    }

    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="utf-8">'
      + '<title>บันทึกประวัติเครื่องมือ</title>'
      + '<style>'
      + '@import url("https://fonts.googleapis.com/css2?family=Sarabun:wght@400;600;700&display=swap");'
      + '*{box-sizing:border-box;margin:0;padding:0;}'
      + 'body{font-family:"Sarabun","Noto Sans Thai",Arial,sans-serif;font-size:13px;color:#1e293b;background:#fff;}'
      + '.page{width:210mm;min-height:297mm;padding:14mm 16mm 14mm 16mm;margin:0 auto;}'

      /* ===== HEADER ===== */
      + '.doc-header{display:flex;align-items:stretch;border:1.5px solid #1e3a8a;margin-bottom:14px;}'
      + '.hd-logo{width:70px;min-width:70px;background:#1e3a8a;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:6px 4px;gap:3px;}'
      + '.hd-logo-text{color:#fff;font-size:8px;font-weight:700;text-align:center;line-height:1.3;}'
      + '.hd-title{flex:1;display:flex;align-items:center;justify-content:center;padding:8px 12px;border-left:1.5px solid #1e3a8a;border-right:1.5px solid #1e3a8a;}'
      + '.hd-title h1{font-size:18px;font-weight:700;color:#1e293b;text-align:center;line-height:1.4;}'
      + '.hd-meta{width:160px;min-width:160px;font-size:10px;padding:6px 8px;display:flex;flex-direction:column;justify-content:center;gap:3px;}'
      + '.hd-meta-row{display:flex;gap:4px;}'
      + '.hd-meta-key{color:#64748b;font-weight:600;white-space:nowrap;}'
      + '.hd-meta-val{color:#1e293b;font-weight:700;}'

      /* ===== CONTENT TABLE ===== */
      + '.content-table{width:100%;border-collapse:collapse;border:1.5px solid #1e3a8a;}'
      + '.content-table tr{border-bottom:1px solid #cbd5e1;}'
      + '.content-table tr:last-child{border-bottom:none;}'
      + '.content-table td{padding:6px 8px;vertical-align:top;}'
      + '.c-no{width:30px;font-weight:700;color:#1e3a8a;text-align:center;border-right:1px solid #cbd5e1;background:#f8fafc;}'
      + '.c-label{width:195px;font-weight:600;color:#334155;border-right:1px solid #cbd5e1;background:#f8fafc;}'
      + '.c-val{color:#1e293b;font-weight:500;}'

      /* ===== CAL SECTION ===== */
      + '.cal-row td{background:#fff !important;vertical-align:top;}'
      + '.cal-need-box{margin-top:4px;}'
      + '.cal-grid{display:grid;grid-template-columns:1fr 1fr 1fr;gap:6px;margin-top:6px;border:1px solid #bfdbfe;border-radius:4px;padding:6px 10px;background:#eff6ff;}'
      + '.cal-item-label{font-size:10px;color:#64748b;font-weight:600;margin-bottom:1px;}'
      + '.cal-item-val{font-size:12px;font-weight:700;color:#1e293b;}'
      + '.cal-item-val.due{color:#dc2626;}'
      + '.no-cal{color:#64748b;font-style:italic;}'

      /* ===== FOOTER ===== */
      + '.footer{margin-top:10px;font-size:9px;color:#94a3b8;text-align:right;border-top:1px solid #e2e8f0;padding-top:5px;}'
      + '@page{size:A4 portrait;margin:0;}'
      + '@media print{body{margin:0;}}'
      + '</style>'
      + '</head><body><div class="page">'

      /* ---- Header ---- */
      + '<div class="doc-header">'
      + '  <div class="hd-logo">' + logoSVG + '<div class="hd-logo-text">สาขาวิชา<br>พยาธิวิทยา</div></div>'
      + '  <div class="hd-title"><h1>บันทึกประวัติเครื่องมือ</h1></div>'
      + '  <div class="hd-meta">'
      + '    <div class="hd-meta-row"><span class="hd-meta-key">รหัสเอกสาร&nbsp;:</span><span class="hd-meta-val" style="font-size:9px;letter-spacing:-0.3px;">SD-Fo-06-Patho-QMU-007</span></div>'
      + '    <div class="hd-meta-row"><span class="hd-meta-key">แก้ไขครั้งที่ :</span><span class="hd-meta-val">01</span></div>'
      + '    <div class="hd-meta-row"><span class="hd-meta-key">วันที่ประกาศใช้ :</span><span class="hd-meta-val">31 ก.ค. 2569</span></div>'
      + '  </div>'
      + '</div>'

      /* ---- Content Table ---- */
      + '<table class="content-table">'
      + row(1,  'ชื่อเครื่องมือ',                          item['ชื่อเครื่องมือ'])
      + row(2,  'ประเภทเครื่องมือ',                        item['ประเภทเครื่องมือ'])
      + row(3,  'รหัสครุภัณฑ์',                            item['รหัสครุภัณฑ์'])
      + row(4,  'ข้อมูลผลิตภัณฑ์ (ยี่ห้อ / รุ่น / รหัสเครื่อง)', item['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || item['ยี่ห้อ/รุ่น/Serial'])
      + row(5,  'คุณลักษณะเครื่อง',                        item['คุณลักษณะเครื่อง'] || item['คุณลักษณะ'] || item['รายละเอียด/สเปค'])
      + row(6,  'ชื่อบริษัท / ผู้แทน-เซลล์ / เบอร์โทรบริษัท', compName + (compRep ? ' / ' + compRep : '') + (compPhone ? ' / ' + compPhone : ''))
      + row(7,  'รายชื่อช่างซ่อมบำรุง / เบอร์โทรช่าง',    item['รายชื่อช่างซ่อมบำรุง'] || item['ช่างซ่อม'] || item['ชื่อช่าง'] || '-')
      + row(8,  'วันที่รับเครื่อง / เริ่มใช้เครื่อง',     (_fmtDate(item['วันรับเครื่อง'] || item['วันที่รับเครื่อง']) || '-') + (item['วันเริ่มใช้เครื่อง'] || item['วันที่เริ่มใช้เครื่อง'] ? ' / ' + _fmtDate(item['วันเริ่มใช้เครื่อง'] || item['วันที่เริ่มใช้เครื่อง']) : ''))
      + row(9,  'สถานที่ติดตั้ง / วางเครื่อง',            item['สถานที่ติดตั้ง'])
      + row(10, 'สถานภาพเครื่องเมื่อรับ',                  item['สถานะเครื่องเมื่อรับ'] || item['สถานภาพ'] || item['สถานะเครื่อง'])
      + row(11, 'คู่มือการใช้งาน',                         item['คู่มือการใช้งาน'] || item['คู่มือ'] ? '✅ มีคู่มือ' : '-')
      + row(12, 'กำหนดอายุการใช้งาน', (item['อายุการใช้งาน(ปี)'] || item['อายุการใช้งาน (ปี)'] || '-') + (item['อายุการใช้งาน(ปี)'] || item['อายุการใช้งาน (ปี)'] ? ' ปี' : ''))
      + row(13, 'มูลค่าเครื่องมือ',                        item['มูลค่าเครื่องมือ'] ? parseFloat(String(item['มูลค่าเครื่องมือ']).replace(/,/g,'')).toLocaleString('th-TH') + ' บาท' : '-')

      /* ---- row 14 ไม่แสดงใน PDF ---- */
      + '</table>'

      /* ---- Footer ---- */
      + '<div class="footer">'
      + (printedBy ? 'พิมพ์โดย: ' + printedBy + ' | ' : '')
      + 'วันที่พิมพ์: ' + printedDate
      + '</div>'

      + '</div></body></html>';

    var pw = window.open('', '_blank', 'width=850,height=1000');
    if (!pw) { alert("กรุณาอนุญาต Pop-up เพื่อพิมพ์"); return; }
    pw.document.write(html);
    pw.document.close();
    pw.focus();
    setTimeout(function() { pw.print(); }, 800);
  }

  // Event delegation สำหรับ expand panel buttons
  document.addEventListener('click', function(e) {
    var saveBtn  = e.target.closest('.ep-save-btn');
    var closeBtn = e.target.closest('.ep-close-btn');
    var editBtn  = e.target.closest('.ep-edit-btn');
    var clearBtn = e.target.closest('.ep-clear-btn');
    if (saveBtn)  { saveParamRow(saveBtn.getAttribute('data-save-id'));       return; }
    if (closeBtn) { toggleExpand(closeBtn.getAttribute('data-close-id'));     return; }
    if (editBtn)  { toggleExpand(editBtn.getAttribute('data-edit-id'));       return; }
    if (clearBtn) { clearParamRow(clearBtn.getAttribute('data-clear-id'));    return; }
    var sendBtn = e.target.closest('.ep-send-btn');
    if (sendBtn) {
      var eid = sendBtn.getAttribute('data-send-eqid');
      var eidx = (cachedEquipments||[]).findIndex(function(e){ return e['ID'] === eid; });
      if (eidx >= 0) updateCalSendStatus(eidx);
      return;
    }
    // ep-view-btn handled in separate listener below
  });

  // กด Escape ปิด modal
  document.addEventListener('keydown', function(e) {
    if (e.key === 'Escape') {
      var m = document.getElementById('history-record-modal');
      if (m && !m.classList.contains('hidden')) closeHistoryModal();
    }
  });


  // ============================================================
  // CAL SPEC VIEW MODAL — view / แก้ไข / PDF / Excel
  // ============================================================
  var _calSpecCurrentId = null;

  function openCalSpecModal(id) {
    var item = cachedEquipments ? cachedEquipments.find(function(e){ return e['ID'] === id; }) : null;
    if (!item) return;
    _calSpecCurrentId = id;

    var ptype  = _getParamType(item['ประเภทเครื่องมือ']);
    var params = _calParamConfig[ptype] || _calParamConfig['default'];

    var savedRaw = item['ช่วงใช้งาน'] || '';
    var savedTP  = item['จุดทดสอบ']   || '';
    var limitObj = {}, pointsObj = {};
    try { limitObj  = JSON.parse(savedRaw); } catch(e) { if (savedRaw && params[0]) limitObj[params[0].key + '_limit'] = savedRaw; }
    try { pointsObj = JSON.parse(savedTP);  } catch(e) { if (savedTP  && params[0]) pointsObj[params[0].key + '_points'] = savedTP; }

    // title
    document.getElementById('csm-title').textContent    = item['ชื่อเครื่องมือ'] || '-';
    document.getElementById('csm-subtitle').textContent = (item['รหัสครุภัณฑ์']||'') + ' · ' + (item['ประเภทเครื่องมือ']||'');

    // table
    var tbody = document.getElementById('csm-tbody');
    tbody.innerHTML = params.map(function(p) {
      var lv = limitObj[p.key  + '_limit']  || '-';
      var pv = pointsObj[p.key + '_points'] || '-';
      // แปลง comma-separated → badge list
      var ptBadges = pv === '-' ? '-' : pv.split(',').map(function(v) {
        return '<span class="inline-block bg-primary-50 dark:bg-primary-900/30 text-primary-700 dark:text-primary-300 font-mono text-[11px] rounded px-1.5 py-0.5 mr-1">' + v.trim() + '</span>';
      }).join('');
      return '<tr class="hover:bg-slate-50 dark:hover:bg-slate-700/30 transition">'
        + '<td class="px-3 py-3"><span class="inline-flex items-center gap-1 text-[11px] font-bold text-primary-700 dark:text-primary-300 bg-primary-50 dark:bg-primary-900/30 rounded px-2 py-1">'
        +   p.label + ' <span class="font-normal text-primary-400 text-[10px]">(' + p.unit + ')</span>'
        + '</span></td>'
        + '<td class="px-3 py-3 font-mono text-sm font-semibold text-slate-700 dark:text-slate-200">' + lv + '</td>'
        + '<td class="px-3 py-3">' + ptBadges + '</td>'
        + '</tr>';
    }).join('');

    // info section
    // คำนวณ Due Date จาก lastCal + interval (เหมือน renderTableBody)
    var _lastCal  = item['วันที่ทำการสอบเทียบ (Cal-Date)'] || item['วันที่สอบเทียบล่าสุด'] || '';
    var _intv     = item['ระยะเวลาการสอบเทียบ'] || '';
    var _isCal    = _needsSchedule(item['การสอบเทียบหรือไม่'] || item['สถานะการสอบเทียบ'] || '', item);
    var _dueTxt   = item['สอบเทียบครั้งถัดไป'] || item['Due Date'] || item['วันที่สอบเทียบรอบถัดไป'] || '';
    var _dueColor = '#64748b';

    if (!_dueTxt && _isCal && _lastCal && _intv) {
      var _d = new Date(_lastCal);
      if (!isNaN(_d)) {
        _d.setMonth(_d.getMonth() + (parseInt(_intv) || 12));
        _d.setDate(_d.getDate() - 1);
        _dueTxt = _d.toLocaleDateString('th-TH', { year:'numeric', month:'long', day:'numeric' });
        var _diff = Math.ceil((_d - new Date()) / (1000*60*60*24));
        _dueColor = _diff < 0 ? '#dc2626' : _diff <= 30 ? '#ea580c' : _diff <= 60 ? '#d97706' : '#16a34a';
      }
    } else if (_dueTxt) {
      var _d2 = new Date(_dueTxt);
      if (!isNaN(_d2)) {
        _dueTxt = _d2.toLocaleDateString('th-TH', { year:'numeric', month:'long', day:'numeric' });
        var _diff2 = Math.ceil((_d2 - new Date()) / (1000*60*60*24));
        _dueColor = _diff2 < 0 ? '#dc2626' : _diff2 <= 30 ? '#ea580c' : _diff2 <= 60 ? '#d97706' : '#16a34a';
      }
    }

    var _lastCalFmt = _lastCal
      ? (function(){ var d = new Date(_lastCal); return isNaN(d) ? _lastCal : d.toLocaleDateString('th-TH', {year:'numeric',month:'long',day:'numeric'}); })()
      : '-';

    var info = document.getElementById('csm-info');
    info.innerHTML = [
      { label: 'สถานที่ติดตั้ง',      val: item['สถานที่ติดตั้ง'] || '-',         color: '' },
      { label: 'วันที่สอบเทียบล่าสุด', val: _lastCalFmt,                             color: '' },
      { label: 'ระยะเวลาสอบเทียบ',     val: _intv ? _intv + ' เดือน' : '-',         color: '' },
      { label: 'Due Date (รอบถัดไป)',   val: _dueTxt || '-',                          color: _dueColor },
    ].map(function(f) {
      return '<div class="bg-slate-50 dark:bg-slate-700/50 rounded-lg px-3 py-2">'
        + '<div class="text-[10px] text-slate-400 mb-0.5">' + f.label + '</div>'
        + '<div class="text-xs font-semibold" style="color:' + (f.color || 'inherit') + '">' + f.val + '</div>'
        + '</div>';
    }).join('');

    // updated by
    var updEl = document.getElementById('csm-updated');
    if (item['แก้ไขโดย'] || item['วันที่แก้ไข']) {
      updEl.textContent = 'บันทึกล่าสุด: ' + (item['แก้ไขโดย']||'') + (item['วันที่แก้ไข'] ? ' · ' + item['วันที่แก้ไข'] : '');
    } else {
      updEl.textContent = '';
    }

    // bind buttons
    document.getElementById('csm-edit-btn').onclick  = function() { closeCalSpecModal(); toggleExpand(id); };
    document.getElementById('csm-pdf-btn').onclick   = function() { _printCalSpecPdf(item, params, limitObj, pointsObj); };
    document.getElementById('csm-excel-btn').onclick = function() { _exportCalSpecExcel(item, params, limitObj, pointsObj); };

    // show modal
    var modal = document.getElementById('cal-spec-modal');
    modal.classList.remove('hidden');
    modal.style.display = 'flex';
  }

  function closeCalSpecModal() {
    var modal = document.getElementById('cal-spec-modal');
    modal.classList.add('hidden');
    modal.style.display = '';
    _calSpecCurrentId = null;
  }

  // ===== helper: คำนวณ Due Date จาก item =====
  function _calcDueDate(item) {
    var raw = item['สอบเทียบครั้งถัดไป'] || item['Due Date'] || item['วันที่สอบเทียบรอบถัดไป'] || '';
    var lastCal = item['วันที่ทำการสอบเทียบ (Cal-Date)'] || item['วันที่สอบเทียบล่าสุด'] || '';
    var intv    = item['ระยะเวลาการสอบเทียบ'] || '';
    var isCal   = _needsSchedule(item['การสอบเทียบหรือไม่'] || item['สถานะการสอบเทียบ'] || '', item);
    var d = null;

    if (raw) {
      d = new Date(raw);
      if (isNaN(d)) d = null;
    }
    if (!d && isCal && lastCal && intv) {
      d = new Date(lastCal);
      if (!isNaN(d)) {
        d.setMonth(d.getMonth() + (parseInt(intv) || 12));
        d.setDate(d.getDate() - 1);
      } else { d = null; }
    }
    if (!d) return '-';
    return d.toLocaleDateString('th-TH', { year: 'numeric', month: 'long', day: 'numeric' });
  }

  function _printCalSpecPdf(item, params, limitObj, pointsObj) {
    var now = new Date().toLocaleDateString('th-TH', { day:'numeric', month:'long', year:'numeric' })
            + ' เวลา ' + new Date().toLocaleTimeString('th-TH', { hour:'2-digit', minute:'2-digit' }) + ' น.';
    var printedBy = currentUser ? (currentUser.fullname || currentUser.username) : '';
    var dueDate   = _calcDueDate(item);
    var lastCalRaw = item['วันที่ทำการสอบเทียบ (Cal-Date)'] || item['วันที่สอบเทียบล่าสุด'] || '';
    var lastCal = lastCalRaw ? (function(){
      var d = new Date(lastCalRaw);
      return isNaN(d) ? lastCalRaw : d.toLocaleDateString('th-TH',{year:'numeric',month:'long',day:'numeric'});
    })() : '-';
    var intv = item['ระยะเวลาการสอบเทียบ'] || '';

    // สร้าง table rows — แต่ละ parameter = 1 แถว (rowspan ลำดับ/ชื่อ/รหัส/due)
    var rowspan = params.length;
    var tableRows = '';
    params.forEach(function(p, pi) {
      var lv = limitObj[p.key  + '_limit']  || '-';
      var pv = pointsObj[p.key + '_points'] || '-';
      var pts = pv === '-' ? '-' : pv.split(',').map(function(v){ return v.trim(); }).join(', ');

      if (pi === 0) {
        tableRows += '<tr>'
          + '<td class="no" rowspan="' + rowspan + '">1</td>'
          + '<td class="name" rowspan="' + rowspan + '">' + (item['ชื่อเครื่องมือ']||'-') + '</td>'
          + '<td class="code" rowspan="' + rowspan + '">' + (item['รหัสครุภัณฑ์']||'-') + '</td>'
          + '<td class="param"><span class="ptag">' + p.label + '</span> <span class="unit">(' + p.unit + ')</span></td>'
          + '<td class="range">' + lv + '</td>'
          + '<td class="test">' + pts + '</td>'
          + '<td class="due" rowspan="' + rowspan + '">' + dueDate + '</td>'
          + '</tr>';
      } else {
        tableRows += '<tr>'
          + '<td class="param"><span class="ptag">' + p.label + '</span> <span class="unit">(' + p.unit + ')</span></td>'
          + '<td class="range">' + lv + '</td>'
          + '<td class="test">' + pts + '</td>'
          + '</tr>';
      }
    });

    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="utf-8">'
      + '<title>SD-Fo-06-Patho-QMU-017</title>'
      + '<style>'
      + '@import url("https://fonts.googleapis.com/css2?family=Sarabun:wght@400;600;700&display=swap");'
      + '*{box-sizing:border-box;margin:0;padding:0}'
      + 'body{font-family:"Sarabun","Noto Sans Thai",Arial,sans-serif;font-size:13px;color:#1e293b;}'
      + '.page{width:210mm;padding:10mm 12mm;margin:0 auto;}'

      /* === DOC STRIP (3-column) เหมือน SD-017 === */
      + '.doc-strip{display:grid;grid-template-columns:1fr auto 1fr;align-items:center;border-bottom:2px solid #1e3a8a;padding-bottom:5px;margin-bottom:4px;gap:8px;}'
      + '.ds-left{font-size:11px;}'
      + '.ds-left-title{font-weight:700;}'
      + '.ds-left-sub{font-size:9px;color:#64748b;margin-top:1px;}'
      + '.ds-center{text-align:center;font-size:15px;font-weight:700;}'
      + '.ds-right{text-align:right;font-size:9px;color:#64748b;line-height:1.7;}'
      + '.ds-right b{color:#1e293b;font-weight:700;}'

      /* === INFO SECTION === */
      + '.info-section{margin:8px 0;}'
      + '.info-grid{display:grid;grid-template-columns:1fr 1fr;gap:6px;margin-bottom:8px;}'
      + '.info-card{background:#f8fafc;border:1px solid #e2e8f0;border-radius:5px;padding:6px 10px;}'
      + '.info-label{font-size:9px;color:#64748b;font-weight:600;text-transform:uppercase;letter-spacing:.3px;margin-bottom:2px;}'
      + '.info-val{font-size:12px;font-weight:700;}'
      + '.info-val.due{color:#dc2626;}'

      /* === TABLE === */
      + 'table{width:100%;border-collapse:collapse;margin-top:2px;}'
      + 'th,td{border:1px solid #64748b;padding:5px 6px;vertical-align:middle;}'
      + 'th{background:#1e3a8a;color:#fff;font-size:11px;font-weight:700;text-align:center;white-space:nowrap;}'
      + 'td.no{text-align:center;width:24px;font-size:11px;color:#475569;}'
      + 'td.name{font-size:12px;font-weight:600;width:24%;}'
      + 'td.code{font-size:11px;font-family:monospace;text-align:center;width:13%;}'
      + 'td.param{width:14%;}'
      + '.ptag{font-size:11px;font-weight:700;color:#1d4ed8;}'
      + '.unit{font-size:9px;color:#94a3b8;}'
      + 'td.range{text-align:center;font-size:11px;font-family:monospace;font-weight:600;}'
      + 'td.test{text-align:center;font-size:11px;}'
      + 'td.due{text-align:center;font-size:11px;font-weight:700;color:#dc2626;width:14%;}'
      + 'tr:nth-child(even) td{background:#f8fafc;}'

      /* === SIGNATURE === */
      + '.sig-section{display:grid;grid-template-columns:1fr 1fr 1fr;gap:16px;margin-top:16px;}'
      + '.sig-box{text-align:center;}'
      + '.sig-line{border-bottom:1px solid #94a3b8;margin:28px 8px 4px;}'
      + '.sig-label{font-size:10px;color:#64748b;}'

      /* === FOOTER === */
      + '.footer{margin-top:8px;display:flex;justify-content:space-between;font-size:9px;color:#94a3b8;border-top:1px solid #e2e8f0;padding-top:4px;}'
      + '@page{size:A4 portrait;margin:0}'
      + '@media print{body{margin:0;}.page{padding:8mm 10mm;}}'
      + '</style></head><body><div class="page">'

      /* DOC STRIP */
      + '<div class="doc-strip">'
      +   '<div class="ds-left">'
      +     '<div class="ds-left-title">แบบบันทึกการระบุรายละเอียดการสอบเทียบ</div>'
      +     '<div class="ds-left-sub">สาขาวิชาพยาธิวิทยา</div>'
      +   '</div>'
      +   '<div class="ds-center">แบบบันทึกการระบุรายละเอียดการสอบเทียบ</div>'
      +   '<div class="ds-right">'
      +     '<b>รหัสเอกสาร :</b> SD-Fo-06-Patho-QMU-017<br>'
      +     '<b>แก้ไขครั้งที่ :</b> 00<br>'
      +     '<b>วันที่ประกาศใช้ :</b> 15 ต.ค. 2567'
      +   '</div>'
      + '</div>'

      /* INFO SECTION */
      + '<div class="info-section">'
      +   '<div class="info-grid">'
      +     '<div class="info-card"><div class="info-label">ชื่อเครื่องมือ</div><div class="info-val">' + (item['ชื่อเครื่องมือ']||'-') + '</div></div>'
      +     '<div class="info-card"><div class="info-label">รหัสครุภัณฑ์</div><div class="info-val">' + (item['รหัสครุภัณฑ์']||'-') + '</div></div>'
      +     '<div class="info-card"><div class="info-label">ประเภทเครื่องมือ</div><div class="info-val" style="font-size:11px;">' + (item['ประเภทเครื่องมือ']||'-') + '</div></div>'
      +     '<div class="info-card"><div class="info-label">สถานที่ติดตั้ง</div><div class="info-val" style="font-size:11px;">' + (item['สถานที่ติดตั้ง']||'-') + '</div></div>'
      +     '<div class="info-card"><div class="info-label">วันที่สอบเทียบล่าสุด</div><div class="info-val">' + lastCal + '</div></div>'
      +     '<div class="info-card"><div class="info-label">ระยะเวลาสอบเทียบ</div><div class="info-val">' + (intv ? intv + ' เดือน' : '-') + '</div></div>'
      +   '</div>'
      + '</div>'

      /* TABLE */
      + '<table><thead><tr>'
      +   '<th style="width:24px">ลำดับที่</th>'
      +   '<th style="width:24%">ชื่อเครื่อง</th>'
      +   '<th style="width:13%">รหัสครุภัณฑ์</th>'
      +   '<th style="width:14%">Parameter</th>'
      +   '<th>ช่วงใช้งาน<br><span style="font-weight:400;font-size:9px;">(Control limit)</span></th>'
      +   '<th>จุดทดสอบ<br><span style="font-weight:400;font-size:9px;">(Testing points)</span></th>'
      +   '<th style="width:14%">Due Date</th>'
      + '</tr></thead>'
      + '<tbody>' + tableRows + '</tbody></table>'

      /* FOOTER */
      + '<div class="footer">'
      +   '<span>SD-Fo-06-Patho-QMU-017</span>'
      +   '<span>' + (printedBy ? 'พิมพ์โดย: ' + printedBy + ' | ' : '') + 'วันที่พิมพ์: ' + now + '</span>'
      + '</div>'
      + '</div></body></html>';

    var pw = window.open('', '_blank', 'width=860,height=1000');
    if (!pw) { alert('กรุณาอนุญาต Pop-up เพื่อพิมพ์'); return; }
    pw.document.write(html);
    pw.document.close();
    pw.focus();
    setTimeout(function() { pw.print(); }, 700);
  }
  function _exportCalSpecExcel(item, params, limitObj, pointsObj) {
    // สร้าง CSV แล้ว download (Excel เปิด CSV ได้)
    var rows = [
      ['รหัสครุภัณฑ์', 'ชื่อเครื่องมือ', 'ประเภท', 'สถานที่ติดตั้ง', 'Parameter', 'หน่วย', 'ช่วงใช้งาน (Control limit)', 'จุดทดสอบ (Testing points)', 'Due Date']
    ];

    params.forEach(function(p) {
      var lv = limitObj[p.key  + '_limit']  || '';
      var pv = pointsObj[p.key + '_points'] || '';
      rows.push([
        item['รหัสครุภัณฑ์']  || '',
        item['ชื่อเครื่องมือ'] || '',
        item['ประเภทเครื่องมือ'] || '',
        item['สถานที่ติดตั้ง'] || '',
        p.label,
        p.unit,
        lv,
        pv,
        _calcDueDate(item)
      ]);
    });

    // UTF-8 BOM เพื่อให้ Excel แสดงภาษาไทยถูกต้อง
    var csv = '﻿' + rows.map(function(r) {
      return r.map(function(c) {
        var s = String(c).replace(/"/g, '""');
        return s.indexOf(',') !== -1 || s.indexOf('"') !== -1 || s.indexOf('\n') !== -1 ? '"' + s + '"' : s;
      }).join(',');
    }).join('\r\n');

    var blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
    var url  = URL.createObjectURL(blob);
    var a    = document.createElement('a');
    a.href     = url;
    a.download = 'CalSpec_' + (item['รหัสครุภัณฑ์']||item['ID']) + '.csv';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    showToast('Export Excel สำเร็จ', 'success');
  }

  // handle ep-view-btn click via delegation
  document.addEventListener('click', function(e) {
    var viewBtn = e.target.closest('.ep-view-btn');
    if (viewBtn) {
      openCalSpecModal(viewBtn.getAttribute('data-view-id'));
    }
  });

  // Escape ปิด cal-spec-modal
  document.addEventListener('keydown', function(e) {
    if (e.key === 'Escape') {
      var m = document.getElementById('cal-spec-modal');
      if (m && !m.classList.contains('hidden')) closeCalSpecModal();
    }
  });


  // ============================================================
  // พิมพ์แบบฟอร์ม SD-Fo-06-Patho-QMU-017
  // แบบบันทึกการระบุรายละเอียดการสอบเทียบ
  // ============================================================
  // ===== SD-017 Filter Modal =====
  function printSD017Form() {
    if (!cachedEquipments || cachedEquipments.length === 0) {
      showToast('กรุณาโหลดข้อมูลก่อนพิมพ์', 'warning'); return;
    }

    var isAdmin = currentUser && currentUser.role === 'Admin';

    // รวบรวม location และ type จาก cachedEquipments
    var locations = [], types = [];
    cachedEquipments.forEach(function(eq) {
      var loc  = eq['สถานที่ติดตั้ง'] || '';
      var type = eq['ประเภทเครื่องมือ'] || '';
      if (loc  && locations.indexOf(loc)  === -1) locations.push(loc);
      if (type && types.indexOf(type) === -1) types.push(type);
    });
    locations.sort(); types.sort();

    // สร้าง options
    function mkOpts(arr, allLabel) {
      return '<option value="">' + allLabel + '</option>'
        + arr.map(function(v){ return '<option value="' + v + '">' + v + '</option>'; }).join('');
    }

    // แสดง filter dialog (inline modal)
    var overlay = document.createElement('div');
    overlay.id = 'sd017-filter-overlay';
    overlay.style.cssText = 'position:fixed;inset:0;background:rgba(0,0,0,.55);z-index:9000;display:flex;align-items:center;justify-content:center;';
    overlay.innerHTML = '<div style="background:#fff;border-radius:16px;padding:28px 32px;width:460px;max-width:95vw;box-shadow:0 20px 60px rgba(0,0,0,.25);">'
      + '<h3 style="font-size:16px;font-weight:700;color:#1e293b;margin-bottom:4px;">พิมพ์แบบฟอร์ม SD-017</h3>'
      + '<p style="font-size:12px;color:#64748b;margin-bottom:20px;">เลือกเงื่อนไขการพิมพ์</p>'
      + (isAdmin
          ? '<label style="display:block;font-size:12px;font-weight:600;color:#374151;margin-bottom:4px;">ห้องปฏิบัติการ / สถานที่ติดตั้ง</label>'
          + '<select id="sd017-loc-sel" style="width:100%;padding:8px 10px;border:1px solid #d1d5db;border-radius:8px;font-size:13px;margin-bottom:14px;">'
          + mkOpts(locations, 'ทุกสถานที่')
          + '</select>'
          : '')
      + '<label style="display:block;font-size:12px;font-weight:600;color:#374151;margin-bottom:4px;">ประเภทเครื่องมือ</label>'
      + '<select id="sd017-type-sel" style="width:100%;padding:8px 10px;border:1px solid #d1d5db;border-radius:8px;font-size:13px;margin-bottom:20px;">'
      + mkOpts(types, 'ทุกประเภท')
      + '</select>'
      + '<div style="display:flex;gap:10px;justify-content:flex-end;">'
      +   '<button id="sd017-cancel-btn" style="padding:8px 20px;border:1px solid #d1d5db;border-radius:8px;font-size:13px;font-weight:600;background:#fff;cursor:pointer;color:#64748b;">ยกเลิก</button>'
      +   '<button id="sd017-print-btn" style="padding:8px 22px;border:none;border-radius:8px;font-size:13px;font-weight:700;background:#dc2626;color:#fff;cursor:pointer;display:flex;align-items:center;gap:6px;">'
      +     '<span class="material-symbols-outlined" style="font-size:16px;">print</span> พิมพ์'
      +   '</button>'
      + '</div>'
      + '</div>';

    document.body.appendChild(overlay);

    document.getElementById('sd017-cancel-btn').onclick = function() {
      document.body.removeChild(overlay);
    };
    overlay.addEventListener('click', function(e) {
      if (e.target === overlay) document.body.removeChild(overlay);
    });

    document.getElementById('sd017-print-btn').onclick = function() {
      var locFilter  = (document.getElementById('sd017-loc-sel')  || {}).value  || '';
      var typeFilter = (document.getElementById('sd017-type-sel') || {}).value || '';
      document.body.removeChild(overlay);
      _doSD017Print(locFilter, typeFilter);
    };
  }

  function _doSD017Print(locFilter, typeFilter) {
    var role      = currentUser ? currentUser.role : '';
    var userDept  = currentUser ? currentUser.dept : '';
    var now       = new Date().toLocaleDateString('th-TH', { day:'numeric', month:'long', year:'numeric' });
    var printedBy = currentUser ? (currentUser.fullname || currentUser.username) : '';

    // กรองข้อมูล
    var filtered = cachedEquipments.filter(function(eq) {
      var loc  = eq['สถานที่ติดตั้ง'] || '';
      var type = eq['ประเภทเครื่องมือ'] || '';
      var locOk  = !locFilter  || loc  === locFilter;
      var typeOk = !typeFilter || type === typeFilter;
      // user ธรรมดาเห็นแค่ของตัวเอง
      var deptOk = role === 'Admin' || loc === userDept;
      return locOk && typeOk && deptOk;
    });

    if (filtered.length === 0) {
      showToast('ไม่พบข้อมูลในเงื่อนไขที่เลือก', 'warning'); return;
    }

    // label สำหรับ header
    var scopeLabel = locFilter ? locFilter : (role === 'Admin' ? 'ทุกสถานที่' : userDept);
    var typeLabel  = typeFilter ? typeFilter : 'ทุกประเภท';

    // สร้าง table rows (ไม่มีคอลัมน์สถานที่)
    var tableRows = '';
    var no = 1;
    filtered.forEach(function(item) {
      var ptype  = _getParamType(item['ประเภทเครื่องมือ']);
      var params = _calParamConfig[ptype] || _calParamConfig['default'];

      var savedRaw = item['ช่วงใช้งาน'] || '';
      var savedTP  = item['จุดทดสอบ']   || '';
      var limitObj = {}, pointsObj = {};
      try { limitObj  = JSON.parse(savedRaw); } catch(e) { if (savedRaw && params[0]) limitObj[params[0].key  + '_limit']  = savedRaw; }
      try { pointsObj = JSON.parse(savedTP);  } catch(e) { if (savedTP  && params[0]) pointsObj[params[0].key + '_points'] = savedTP;  }

      var rowspan  = params.length;
      var firstRow = true;
      var dueText  = _calcDueDate(item);

      params.forEach(function(p) {
        var lv = limitObj[p.key  + '_limit']  || '';
        var pv = pointsObj[p.key + '_points'] || '';
        var pvDisplay = pv ? pv.split(',').map(function(v){ return v.trim(); }).join(', ') : '';

        if (firstRow) {
          tableRows += '<tr>'
            + '<td class="no" rowspan="' + rowspan + '">' + no++ + '</td>'
            + '<td class="name" rowspan="' + rowspan + '"><div class="name-th">' + (item['ชื่อเครื่องมือ']||'-') + '</div></td>'
            + '<td class="code" rowspan="' + rowspan + '">' + (item['รหัสครุภัณฑ์']||'-') + '</td>'
            + '<td class="param"><span class="param-tag">' + p.label + '</span> <span class="unit">(' + p.unit + ')</span></td>'
            + '<td class="range">' + lv + '</td>'
            + '<td class="test">'  + pvDisplay + '</td>'
            + '<td class="due" rowspan="' + rowspan + '">' + dueText + '</td>'
            + '</tr>';
          firstRow = false;
        } else {
          tableRows += '<tr>'
            + '<td class="param"><span class="param-tag">' + p.label + '</span> <span class="unit">(' + p.unit + ')</span></td>'
            + '<td class="range">' + lv + '</td>'
            + '<td class="test">'  + pvDisplay + '</td>'
            + '</tr>';
        }
      });
    });

    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="utf-8">'
      + '<title>SD-Fo-06-Patho-QMU-017</title>'
      + '<style>'
      + '@import url("https://fonts.googleapis.com/css2?family=Sarabun:wght@400;600;700&display=swap");'
      + '*{box-sizing:border-box;margin:0;padding:0}'
      + 'body{font-family:"Sarabun","Noto Sans Thai",Arial,sans-serif;font-size:13px;color:#1e293b;}'
      + '.page{width:210mm;padding:10mm 12mm;margin:0 auto;}'
      + '.doc-strip{display:flex;justify-content:space-between;align-items:flex-start;border-bottom:2px solid #1e3a8a;padding-bottom:5px;margin-bottom:6px;}'
      + '.ds-left{font-size:13px;font-weight:700;}'
      + '.ds-left-sub{font-size:9px;color:#64748b;margin-top:2px;}'
      + '.ds-right{text-align:right;font-size:9px;color:#64748b;line-height:1.7;}'
      + '.ds-right b{color:#1e293b;}'
      + '.form-title{text-align:center;font-size:17px;font-weight:700;margin:8px 0 6px;}'
      + '.scope-line{font-size:11px;color:#475569;margin-bottom:8px;}'
      + '.scope-line b{color:#1e293b;}'
      + 'table{width:100%;border-collapse:collapse;}'
      + 'th,td{border:1px solid #64748b;padding:5px 6px;vertical-align:middle;}'
      + 'th{background:#1e3a8a;color:#fff;font-size:11px;font-weight:700;text-align:center;}'
      + 'td.no{text-align:center;width:26px;font-size:11px;color:#475569;}'
      + 'td.name{width:26%;font-size:12px;font-weight:600;}'
      + 'td.code{width:13%;font-size:11px;font-family:monospace;text-align:center;}'
      + 'td.param{width:14%;}'
      + '.param-tag{font-size:11px;font-weight:700;color:#1d4ed8;}'
      + '.unit{font-size:9px;color:#94a3b8;}'
      + 'td.range,td.test{text-align:center;font-size:11px;}'
      + 'td.due{text-align:center;font-size:11px;font-weight:700;color:#dc2626;width:13%;}'
      + 'tr:nth-child(even) td{background:#f8fafc;}'
      + '.footer{margin-top:8px;display:flex;justify-content:space-between;font-size:9px;color:#94a3b8;}'
      + '@page{size:A4 portrait;margin:0}'
      + '@media print{body{margin:0;}.page{padding:8mm 10mm;}}'
      + '</style></head><body><div class="page">'

      + '<div class="doc-strip">'
      +   '<div><div class="ds-left">แบบบันทึกการระบุรายละเอียดการสอบเทียบ</div>'
      +   '<div class="ds-left-sub">สาขาวิชาพยาธิวิทยา</div></div>'
      +   '<div class="ds-right">'
      +     '<b>รหัสเอกสาร :</b> SD-Fo-06-Patho-QMU-017<br>'
      +     '<b>แก้ไขครั้งที่ :</b> 00<br>'
      +     '<b>วันที่ประกาศใช้ :</b> 15 ต.ค. 2567'
      +   '</div>'
      + '</div>'
      + '<div class="form-title">แบบบันทึกการระบุรายละเอียดการสอบเทียบ</div>'
      + '<div class="scope-line">ห้องปฏิบัติการ : <b>' + scopeLabel + '</b>'
      +   (typeFilter ? ' &nbsp;|&nbsp; ประเภทเครื่องมือ : <b>' + typeLabel + '</b>' : '')
      + '</div>'

      + '<table><thead><tr>'
      +   '<th style="width:26px">ลำดับที่</th>'
      +   '<th style="width:26%">ชื่อเครื่อง</th>'
      +   '<th style="width:13%">รหัสครุภัณฑ์</th>'
      +   '<th style="width:14%">Parameter</th>'
      +   '<th>ช่วงใช้งาน</th>'
      +   '<th>จุดทดสอบ</th>'
      +   '<th style="width:13%">Due Date</th>'
      + '</tr></thead>'
      + '<tbody>' + tableRows + '</tbody>'
      + '</table>'

      + '<div class="footer">'
      +   '<span>SD-Fo-06-Patho-QMU-017 | พิมพ์วันที่: ' + now + '</span>'
      +   '<span>' + (printedBy ? 'พิมพ์โดย: ' + printedBy : '') + '</span>'
      + '</div>'
      + '</div></body></html>';

    var pw = window.open('', '_blank', 'width=870,height=1100');
    if (!pw) { showToast('กรุณาอนุญาต Pop-up เพื่อพิมพ์', 'warning'); return; }
    pw.document.write(html);
    pw.document.close();
    pw.focus();
    setTimeout(function() { pw.print(); }, 700);
  }


  // ============================================================
  // สถานะการส่งสอบเทียบ — รอส่ง / รับเครื่อง / รีเซ็ต
  // ============================================================
  function updateCalSendStatus(idx) {
    var item = _getItem(idx);
    if (!item) return;

    var isAdmin   = currentUser && currentUser.role === 'Admin';
    var sendStatus = item['สถานะการส่งสอบ'] || '';
    var eqName    = item['ชื่อเครื่องมือ'] || '';
    var eqCode    = item['รหัสครุภัณฑ์'] || '';

    // กำหนดว่าสถานะอะไรกดได้บ้าง
    // ห้องปฏิบัติการ: กด "รอส่งการสอบเทียบ" หรือ "รีเซ็ต"
    // Admin/หน่วยจ่ายกลาง: กด "รับเครื่องส่งสอบเทียบ" ได้เพิ่ม

    var overlay = document.createElement('div');
    overlay.style.cssText = 'position:fixed;inset:0;background:rgba(0,0,0,.55);z-index:9999;display:flex;align-items:center;justify-content:center;';

    var statusBadgeHtml = sendStatus
      ? '<span style="display:inline-block;padding:3px 10px;border-radius:20px;font-size:11px;font-weight:700;background:#ede9fe;color:#7c3aed;">สถานะปัจจุบัน: ' + sendStatus + '</span>'
      : '<span style="display:inline-block;padding:3px 10px;border-radius:20px;font-size:11px;background:#f1f5f9;color:#64748b;">ยังไม่มีสถานะ</span>';

    var adminBtns = isAdmin
      ? '<button id="css-receive-btn" style="width:100%;padding:12px;border:1.5px solid #7c3aed;border-radius:10px;background:#faf5ff;color:#7c3aed;font-size:14px;font-weight:700;cursor:pointer;display:flex;align-items:center;gap:8px;justify-content:center;margin-bottom:8px;">'
        + '<span class="material-symbols-outlined">inventory_2</span> รับเครื่องส่งสอบเทียบ</button>'
      : '';

    var resetBtn = sendStatus
      ? '<button id="css-reset-btn" style="width:100%;padding:10px;border:1px solid #e2e8f0;border-radius:10px;background:#fff;color:#94a3b8;font-size:13px;cursor:pointer;margin-bottom:8px;">↩ รีเซ็ตสถานะ</button>'
      : '';

    overlay.innerHTML = '<div style="background:#fff;border-radius:16px;padding:24px 28px;width:420px;max-width:95vw;box-shadow:0 20px 60px rgba(0,0,0,.25);">'
      + '<div style="display:flex;align-items:center;gap:10px;margin-bottom:4px;">'
      +   '<span class="material-symbols-outlined" style="color:#7c3aed;font-size:22px;">local_shipping</span>'
      +   '<h3 style="font-size:16px;font-weight:700;color:#1e293b;">อัปเดตสถานะการส่งสอบเทียบ</h3>'
      + '</div>'
      + '<p style="font-size:12px;color:#64748b;margin-bottom:12px;">' + eqName + ' · ' + eqCode + '</p>'
      + '<div style="margin-bottom:14px;">' + statusBadgeHtml + '</div>'
      + '<div style="display:flex;flex-direction:column;gap:0;">'
      +   '<button id="css-wait-btn" style="width:100%;padding:12px;border:1.5px solid #3b82f6;border-radius:10px;background:#eff6ff;color:#1d4ed8;font-size:14px;font-weight:700;cursor:pointer;display:flex;align-items:center;gap:8px;justify-content:center;margin-bottom:8px;">'
      +     '<span class="material-symbols-outlined">schedule_send</span> รอส่งการสอบเทียบ</button>'
      +   adminBtns
      +   resetBtn
      +   '<button id="css-cancel-btn" style="width:100%;padding:9px;border:1px solid #e2e8f0;border-radius:10px;background:#f8fafc;color:#64748b;font-size:13px;cursor:pointer;">ยกเลิก</button>'
      + '</div>'
      + '</div>';

    document.body.appendChild(overlay);

    function doUpdate(newStatus) {
      document.body.removeChild(overlay);
      showLoader();
      google.script.run
        .withSuccessHandler(function(res) {
          hideLoader();
          if (res && res.success) {
            item['สถานะการส่งสอบ'] = newStatus;
            if (typeof loadEquipments === 'function') loadEquipments();
                        if (typeof _renderEqStatusPage === 'function') _renderEqStatusPage();
            showToast(newStatus ? '✅ อัปเดตสถานะ: ' + newStatus : '✅ รีเซ็ตสถานะเรียบร้อย', 'success');
            
          } else {
            showToast('เกิดข้อผิดพลาด', 'danger');
          }
        })
        .withFailureHandler(function(err) {
          hideLoader();
          showToast('เซิร์ฟเวอร์ขัดข้อง: ' + err, 'danger');
        })
        .updateCalSendStatus({
          eqId:      item['ID'],
          rowIndex:  item._rowIndex || (item._index !== undefined ? item._index + 2 : null),
          status:    newStatus,
          updatedBy: currentUser ? _getDisplayName() : 'System'
        });
    }

    document.getElementById('css-wait-btn').onclick    = function() { doUpdate('รอส่งการสอบเทียบ'); };
    document.getElementById('css-cancel-btn').onclick  = function() { document.body.removeChild(overlay); };
    overlay.addEventListener('click', function(e) { if (e.target === overlay) document.body.removeChild(overlay); });

    var receiveBtn = document.getElementById('css-receive-btn');
    if (receiveBtn) receiveBtn.onclick = function() { doUpdate('รับเครื่องส่งสอบเทียบ'); };

    var resetBtnEl = document.getElementById('css-reset-btn');
    if (resetBtnEl) resetBtnEl.onclick = function() { doUpdate(''); };
  }


  
  function loadEquipmentsForPHMeter() {
    var dept = currentUser ? currentUser.dept : '';
    var role = currentUser ? currentUser.role : '';
    google.script.run.withSuccessHandler(function(data) {
      cachedEquipments = data;
      cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
      var select = document.getElementById('ph-eqSelect');
      if (!select) return;
      select.innerHTML = '<option value="">-- เลือกรายการเครื่องมือ --</option>';
      // ใช้ _getParamType เหมือนกับระบบอื่น
      var filtered = data.filter(function(item) {
        return _getParamType(item['ประเภทเครื่องมือ'] || '') === 'ph';
      });
      filtered.forEach(function(item) {
        select.innerHTML += '<option value="' + item['ID'] + '">' + (item['รหัสครุภัณฑ์']||'-') + ' | ' + (item['ชื่อเครื่องมือ']||'-') + '</option>';
      });
    }).withFailureHandler(function(){}).getEquipments(dept, role);
  }

  function loadEquipmentsForRuler() {
    var dept = currentUser ? currentUser.dept : '';
    var role = currentUser ? currentUser.role : '';
    google.script.run.withSuccessHandler(function(data) {
      cachedEquipments = data;
      cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
      var select = document.getElementById('ru-eqSelect');
      if (!select) return;
      select.innerHTML = '<option value="">-- เลือกรายการเครื่องมือ --</option>';
      // ใช้ _getParamType เหมือนกับระบบอื่น
      var filtered = data.filter(function(item) {
        return _getParamType(item['ประเภทเครื่องมือ'] || '') === 'ruler';
      });
      filtered.forEach(function(item) {
        select.innerHTML += '<option value="' + item['ID'] + '">' + (item['รหัสครุภัณฑ์']||'-') + ' | ' + (item['ชื่อเครื่องมือ']||'-') + '</option>';
      });
    }).withFailureHandler(function(){}).getEquipments(dept, role);
  }

  // ============================================================
  // ✅ BSC / Laminar air flow / Fume Hood
  //    - ดึงรายการตู้ทั้ง 3 ชนิด แล้วกรองตามประเภทตู้ที่เลือก
  //    - แก้ id dropdown (ฟอร์มใช้ bf-eqSelect แต่โค้ดเดิมหา bsc-eqSelect → รายการไม่ขึ้น)
  // ============================================================
  var _bscFumeList = [];

  function _cabinetKind(typeStr) {
    var t = String(typeStr || '').toLowerCase();
    if (t.indexOf('laminar') !== -1)                                              return 'Laminar air flow';
    if (t.indexOf('fume') !== -1 || t.indexOf('ตู้ดูด') !== -1)                    return 'Fume Hood';
    if (t.indexOf('bsc') !== -1 || t.indexOf('biological safety') !== -1 || t.indexOf('biosafety') !== -1) return 'BSC';
    return '';
  }

  function _bscFumeSelectEl() {
    return document.getElementById('bf-eqSelect') || document.getElementById('bsc-eqSelect');
  }

  function _filterBSCFumeList() {
    var select = _bscFumeSelectEl();
    if (!select) return;
    var picked = document.querySelector('input[name="cabinetType"]:checked');
    var kind = picked ? picked.value : '';
    var keep = select.value;
    var list = _bscFumeList.filter(function (it) { return !kind || _cabinetKind(it['ประเภทเครื่องมือ']) === kind; });
    select.innerHTML = '<option value="">' + (list.length ? '-- เลือกรายการตู้ --' : '-- ไม่พบตู้ประเภทนี้ในหน่วย --') + '</option>';
    list.forEach(function (it) {
      var o = document.createElement('option');
      o.value = it['ID'];
      o.textContent = (it['รหัสครุภัณฑ์'] || '-') + ' | ' + (it['ชื่อเครื่องมือ'] || '-');
      select.appendChild(o);
    });
    if (keep && list.some(function (it) { return it['ID'] === keep; })) select.value = keep;
    else { select.value = ''; ['bf-serialNo', 'bf-assetBrand'].forEach(function (id) { var el = document.getElementById(id); if (el) el.value = ''; }); }
  }

  function loadEquipmentsForBSCFume() {
    var dept = currentUser ? currentUser.dept : '';
    var role = currentUser ? currentUser.role : '';
    google.script.run.withSuccessHandler(function(data) {
      cachedEquipments = data || [];
      cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
      _bscFumeList = cachedEquipments.filter(function (item) { return _cabinetKind(item['ประเภทเครื่องมือ']) !== ''; });
      _filterBSCFumeList();
    }).withFailureHandler(function(){}).getEquipments(dept, role);
  }

  // ============================================================
  // ✅ บันทึกผล BSC / Laminar air flow / Fume Hood (เดิมไม่มีฟังก์ชันนี้ → กดบันทึกแล้ว error)
  //    ใช้ saveBSCFumeWithPdf() ฝั่งเซิร์ฟเวอร์ที่มีอยู่แล้ว (ชีต BSCFumeCalibrations)
  // ============================================================
  function submitBSCFumeForm() {
    var form = document.getElementById('bscfume-form');
    if (form && !form.checkValidity()) { form.reportValidity(); return; }

    var eqSel  = _bscFumeSelectEl();
    var eqId   = eqSel ? eqSel.value : '';
    var cab    = document.querySelector('input[name="cabinetType"]:checked');
    var ev     = document.querySelector('input[name="bfEval"]:checked');
    if (!cab)  { alert('กรุณาเลือกประเภทตู้'); return; }
    if (!eqId) { alert('กรุณาเลือกรายการตู้ที่ต้องการบันทึก'); return; }
    if (!ev)   { alert('กรุณาเลือกผลการประเมินสรุป (Pass / Fail)'); return; }

    function v(id) { var el = document.getElementById(id); return el ? String(el.value || '').trim() : ''; }
    var items = ['r1_1', 's1_1', 'r1_2', 's1_2', 'r2', 's2', 'r3', 's3', 'r4', 's4', 'r5', 's5'];
    var hasAny = items.some(function (k) { return k.charAt(0) === 'r' && v('bf-' + k); });
    if (!hasAny && !confirm('ยังไม่ได้กรอกผลการทดสอบในตารางเลย ต้องการบันทึกต่อหรือไม่?')) return;

    // ข้อความผล: ห้ามให้ Fail มีคำว่า "ยอมรับ" เพราะระบบใช้คำนี้ตัดสินว่าผ่าน
    var resultText = ev.value === 'Pass' ? 'ยอมรับและนำไปใช้งานได้ (Pass)' : 'ปฏิเสธ ต้องปรับปรุง (Fail)';
    var eq = (cachedEquipments || []).find(function (it) { return it['ID'] === eqId; }) || {};

    var payload = {
      cabinetType: cab.value,
      calType:     'BSC / Laminar air flow / Fume Hood',
      calDept:     v('bf-cal-dept'),
      eqId:        eqId,
      eqName:      (eq['รหัสครุภัณฑ์'] || '') + ' / ' + (eq['ชื่อเครื่องมือ'] || ''),
      serialNo:    v('bf-serialNo'),
      assetBrand:  v('bf-assetBrand'),
      calDate:     v('bf-calDate'),
      certNo:      v('bf-certNo'),
      evaluation:  resultText,
      result:      resultText,
      overallResult: resultText,
      addedBy:     currentUser ? _getDisplayName() : 'System'
    };
    items.forEach(function (k) { payload[k] = v('bf-' + k); });
    _calEditInject('bscfume', payload);

    var fileInput = document.getElementById('bf-pdfFile');
    if (fileInput && fileInput.files.length > 0) {
      var file = fileInput.files[0];
      if (file.size > 20 * 1024 * 1024) { alert('ไฟล์ PDF ใหญ่เกิน 20 MB'); return; }
      showLoader();
      var reader = new FileReader();
      reader.onload = function (e) {
        payload.pdfBase64 = e.target.result.split(',')[1];
        payload.pdfName = file.name;
        _executeSubmitBSCFume(payload);
      };
      reader.readAsDataURL(file);
    } else if (confirm('ยืนยันบันทึกผลการตรวจสอบ ' + cab.value + ' โดยไม่มีไฟล์เอกสารแนบใช่หรือไม่?')) {
      showLoader();
      _executeSubmitBSCFume(payload);
    }
  }

  function _executeSubmitBSCFume(payload) {
    google.script.run
      .withSuccessHandler(function (res) {
        hideLoader();
        if (res && res.success) {
          alert((res.edited ? 'แก้ไข' : 'บันทึก') + 'ผลตรวจรับรองเรียบร้อย'
            + (res.mail && res.mail.success ? ' และส่งอีเมลแจ้งหัวหน้าหน่วยแล้ว' : ''));
          var form = document.getElementById('bscfume-form');
          if (form) form.reset();
          ['bf-serialNo', 'bf-assetBrand'].forEach(function (id) { var el = document.getElementById(id); if (el) el.value = ''; });
          _filterBSCFumeList();
          _afterCalSave('bscfume');
        } else {
          alert('เกิดข้อผิดพลาด: ' + ((res && res.message) || 'ไม่ทราบสาเหตุ'));
        }
      })
      .withFailureHandler(function (err) {
        hideLoader();
        alert('บันทึกไม่สำเร็จ: ' + (err && err.message || err));
      })
      .saveBSCFumeWithPdf(payload);
  }

  // เลือกตู้ → เติม Serial / รหัสครุภัณฑ์ และเลือกประเภทตู้ให้อัตโนมัติ
  function autoFillBSCFumeInfo() {
    var select = _bscFumeSelectEl();
    var eq = (cachedEquipments || []).find(function (it) { return select && it['ID'] === select.value; });
    var sn = document.getElementById('bf-serialNo'), ab = document.getElementById('bf-assetBrand');
    if (!eq) { if (sn) sn.value = ''; if (ab) ab.value = ''; return; }
    if (sn) sn.value = eq['ผลิตภัณฑ์/รุ่น/รหัสเครื่อง'] || 'ไม่ระบุ';
    if (ab) ab.value = (eq['รหัสครุภัณฑ์'] || '') + ' / ' + (eq['ชื่อเครื่องมือ'] || '');
    var kind = _cabinetKind(eq['ประเภทเครื่องมือ']);
    var r = kind && document.querySelector('input[name="cabinetType"][value="' + kind + '"]');
    if (r && !r.checked) r.checked = true;
  }


  // ===== Preview & อนุมัติผลการสอบเทียบ (isHead) =====
  // ============================================================
  // ✅ แก้ไขข้อมูลผลสอบเทียบ (user ที่บันทึก / isHead / Admin)
  // ============================================================
  // ============================================================
  // ✅ อัปเดตสถานะเครื่องมือ (ไม่ผ่าน/เสีย/ส่งซ่อม/จำหน่าย)
  // ============================================================
  function openEquipmentStatusModal(eqIdx) {
    var item = cachedEquipments && cachedEquipments[eqIdx];
    if (!item) { showToast('ไม่พบข้อมูลเครื่องมือ', 'danger'); return; }
    var eqName    = item['ชื่อเครื่องมือ'] || '-';
    var eqCode    = item['รหัสครุภัณฑ์'] || '-';
    var curStatus = item['สถานะการส่งสอบ'] || item['สถานะเครื่องมือ'] || '';

    var statusOptions = [
      { val: 'สอบเทียบไม่ผ่านเกณฑ์',   label: '⚠️ สอบเทียบไม่ผ่านเกณฑ์',    color: '#dc2626' },
      { val: 'เครื่องมือเสีย',           label: '🔧 เครื่องมือเสีย',             color: '#9a3412' },
      { val: 'ส่งซ่อม',                 label: '🛠️ ส่งซ่อม',                   color: '#ea580c' },
      { val: 'ระงับการใช้งาน',           label: '🚫 ระงับการใช้งาน',             color: '#7c3aed' },
      { val: 'พร้อมใช้งาน',             label: '✅ พร้อมใช้งาน',               color: '#16a34a' },
    ];

    var optHTML = statusOptions.map(function(s) {
      var checked = curStatus === s.val ? 'checked' : '';
      return '<label style="display:flex;align-items:center;gap:10px;padding:10px 14px;border:1.5px solid #e2e8f0;border-radius:8px;cursor:pointer;margin-bottom:8px;transition:border-color .15s;" '
        + 'onmouseover="this.style.borderColor=\'' + s.color + '\'" onmouseout="this.style.borderColor=\'#e2e8f0\'">'
        + '<input type="radio" name="eq-status-opt" value="' + s.val + '" ' + checked + ' style="accent-color:' + s.color + ';width:16px;height:16px;">'
        + '<span style="font-size:14px;font-weight:600;color:' + s.color + ';">' + s.label + '</span>'
        + '</label>';
    }).join('');

    var overlay = document.createElement('div');
    overlay.style.cssText = 'position:fixed;inset:0;background:rgba(0,0,0,.6);z-index:9999;display:flex;align-items:center;justify-content:center;padding:16px;';
    overlay.innerHTML =
      '<div style="background:#fff;border-radius:16px;width:100%;max-width:500px;box-shadow:0 20px 60px rgba(0,0,0,.3);">'
      + '<div style="background:#1e3a8a;color:#fff;padding:16px 20px;border-radius:16px 16px 0 0;display:flex;align-items:center;justify-content:space-between;">'
      +   '<div><div style="font-size:16px;font-weight:700;">📋 อัปเดตสถานะเครื่องมือ</div>'
      +   '<div style="font-size:11px;opacity:.8;margin-top:2px;">' + eqCode + ' · ' + eqName + '</div></div>'
      +   '<button id="eq-status-close" style="background:none;border:none;color:#fff;font-size:22px;cursor:pointer;">✕</button>'
      + '</div>'
      + '<div style="padding:20px;">'
      +   '<p style="font-size:12px;color:#64748b;margin:0 0 14px;">เลือกสถานะปัจจุบันของเครื่องมือ</p>'
      +   optHTML
      +   '<div style="margin-top:14px;">'
      +     '<label style="font-size:12px;font-weight:600;color:#475569;display:block;margin-bottom:5px;">หมายเหตุ (ถ้ามี)</label>'
      +     '<textarea id="eq-status-note" rows="3" placeholder="เช่น ส่งซ่อมวันที่... คาดรับคืนวันที่..." style="width:100%;padding:8px 10px;border:1.5px solid #cbd5e1;border-radius:6px;font-size:12px;box-sizing:border-box;font-family:inherit;resize:vertical;"></textarea>'
      +   '</div>'
      + '</div>'
      + '<div style="padding:14px 20px;border-top:1px solid #e2e8f0;display:flex;gap:10px;justify-content:flex-end;">'
      +   '<button id="eq-status-cancel" style="padding:9px 20px;border:1.5px solid #94a3b8;background:#fff;color:#64748b;border-radius:8px;font-size:13px;font-weight:600;cursor:pointer;">ยกเลิก</button>'
      +   '<button id="eq-status-save" style="padding:9px 24px;border:none;background:#1e3a8a;color:#fff;border-radius:8px;font-size:13px;font-weight:700;cursor:pointer;">💾 บันทึกสถานะ</button>'
      + '</div>'
      + '</div>';

    document.body.appendChild(overlay);

    function closeOverlay() { if (overlay.parentNode) document.body.removeChild(overlay); }
    document.getElementById('eq-status-close').onclick  = closeOverlay;
    document.getElementById('eq-status-cancel').onclick = closeOverlay;
    overlay.addEventListener('click', function(e) { if (e.target === overlay) closeOverlay(); });

    document.getElementById('eq-status-save').onclick = function() {
      var selected = overlay.querySelector('input[name="eq-status-opt"]:checked');
      if (!selected) { showToast('กรุณาเลือกสถานะ', 'warning'); return; }
      var newStatus = selected.value;
      var note      = document.getElementById('eq-status-note').value.trim();
      var saveBtn   = document.getElementById('eq-status-save');
      saveBtn.disabled = true; saveBtn.textContent = '⏳ กำลังบันทึก...';

      google.script.run
        .withSuccessHandler(function(res) {
          closeOverlay();
          if (res && res.success) {
            // อัปเดต cachedEquipments ทันที
            if (cachedEquipments[eqIdx]) {
              cachedEquipments[eqIdx]['สถานะเครื่องมือ'] = newStatus;
              cachedEquipments[eqIdx]['สถานะการส่งสอบ'] = '';
            }
            showToast('✅ อัปเดตสถานะ "' + newStatus + '" สำเร็จ', 'success');
            loadEquipments();
          } else {
            showToast('เกิดข้อผิดพลาด: ' + (res && res.message ? res.message : ''), 'danger');
          }
        })
        .withFailureHandler(function(err) {
          closeOverlay();
          showToast('เกิดข้อผิดพลาด: ' + (err.message || err), 'danger');
        })
        .updateEquipmentStatus(item['ID'], newStatus, note, _getDisplayName() || (currentUser && currentUser.username) || '');
    };
  }

    function _approveCalRow(pageId, idx) {
    var histData = window._calHistoryData && window._calHistoryData[pageId];
    if (!histData) { showToast('ไม่พบข้อมูล', 'danger'); return; }
    var headers = histData.headers;
    var row     = histData.data[idx];
    if (!row) { showToast('ไม่พบแถวข้อมูล', 'danger'); return; }

    // ค่าดิบ — ใช้ส่งให้ server (ว่าง = '')
    function getRaw(names) {
      for (var i = 0; i < names.length; i++) {
        var ci = headers.indexOf(names[i]);
        if (ci >= 0 && row[ci] !== '' && row[ci] !== null && row[ci] !== undefined) return String(row[ci]);
      }
      return '';
    }
    // ค่าสำหรับแสดงผล (ว่าง = '-')
    function getVal(names) { return getRaw(names) || '-'; }

    var eqName   = getVal(['รหัสครุภัณฑ์/ยี่ห้อ','ชื่อเครื่องมือ','Serial No.']);
    var calDate  = getVal(['วันที่สอบเทียบ']);
    var result   = getVal(['ผลการตรวจสอบ','Evaluation','ผล','สรุปผล']);
    var approver = _getDisplayName();

    var sheetName = (typeof CAL_SHEET_MAP !== 'undefined' && CAL_SHEET_MAP[pageId])
      || (window._calSheetMap && window._calSheetMap[pageId]);
    if (!sheetName) { showToast('ไม่พบชื่อชีตของหน้านี้', 'danger'); return; }

    var HIDE_IN_PREVIEW = ['ไฟล์ใบรับรอง (PDF)', '_row', 'หมายเหตุการแก้ไข'];
    var previewRows = headers.map(function(h, hi){
      if (!h || HIDE_IN_PREVIEW.indexOf(h) !== -1) return '';
      var v = (row[hi] !== undefined && row[hi] !== null && row[hi] !== '') ? String(row[hi]) : '-';
      return '<tr><td style="padding:5px 8px;color:#64748b;font-size:12px;border-bottom:1px solid #f1f5f9;white-space:nowrap;width:160px;">' + h + '</td>'
        + '<td style="padding:5px 8px;font-size:12px;border-bottom:1px solid #f1f5f9;font-weight:500;">' + v + '</td></tr>';
    }).join('');

    var overlay = document.createElement('div');
    overlay.style.cssText = 'position:fixed;inset:0;background:rgba(0,0,0,.6);z-index:9999;display:flex;align-items:center;justify-content:center;padding:16px;';
    overlay.innerHTML =
      '<div style="background:#fff;border-radius:16px;width:100%;max-width:680px;max-height:90vh;display:flex;flex-direction:column;box-shadow:0 20px 60px rgba(0,0,0,.3);">'
      + '<div style="background:#1e3a8a;color:#fff;padding:16px 20px;border-radius:16px 16px 0 0;display:flex;align-items:center;justify-content:space-between;">'
      +   '<div><div style="font-size:16px;font-weight:700;">📋 ตรวจสอบก่อนอนุมัติ</div>'
      +   '<div style="font-size:12px;opacity:.8;margin-top:2px;">' + eqName + ' · ' + calDate + '</div></div>'
      +   '<button id="ap-close-btn" style="background:none;border:none;color:#fff;font-size:22px;cursor:pointer;opacity:.8;padding:0 4px;">✕</button>'
      + '</div>'
      + '<div style="overflow-y:auto;flex:1;padding:0;">'
      +   '<table style="width:100%;border-collapse:collapse;">' + previewRows + '</table>'
      + '</div>'
      + '<div style="padding:14px 20px;border-top:1px solid #e2e8f0;display:flex;align-items:center;gap:10px;flex-wrap:wrap;">'
      +   '<div style="flex:1;font-size:12px;color:#64748b;">ผู้อนุมัติ: <strong>' + approver + '</strong></div>'
      +   '<button id="ap-reject-btn" style="padding:9px 20px;border:1.5px solid #dc2626;background:#fff;color:#dc2626;border-radius:8px;font-size:13px;font-weight:700;cursor:pointer;">❌ ไม่อนุมัติ</button>'
      +   '<button id="ap-approve-btn" style="padding:9px 24px;border:none;background:#16a34a;color:#fff;border-radius:8px;font-size:13px;font-weight:700;cursor:pointer;">✅ อนุมัติผลการสอบเทียบ</button>'
      + '</div>'
      + '</div>';

    document.body.appendChild(overlay);

    function closeOverlay() { if (overlay.parentNode) overlay.parentNode.removeChild(overlay); }
    document.getElementById('ap-close-btn').onclick = closeOverlay;
    overlay.addEventListener('click', function(e){ if (e.target === overlay) closeOverlay(); });

    function doApprove(decision) {
      closeOverlay();
      showLoader();
      google.script.run
        .withSuccessHandler(function(res) {
          hideLoader();
          if (res && res.success) {
            showToast(decision === 'approved' ? '✅ อนุมัติเรียบร้อย — ' + approver : '❌ ไม่อนุมัติ — ส่งกลับให้ผู้บันทึกแก้ไข',
                      decision === 'approved' ? 'success' : 'warning');
            setTimeout(function(){
              loadCalHistory(pageId);
              loadEquipments();
            }, 1500);
          } else {
            showToast('เกิดข้อผิดพลาด: ' + (res && res.message || ''), 'danger');
          }
        })
        .withFailureHandler(function(err){ hideLoader(); showToast('ขัดข้อง: ' + (err && err.message || err), 'danger'); })
        .approveCalibrationRow({
          sheetName:  sheetName,
          sheetRow:   getRaw(['_row']),
          eqId:       getRaw(['ID อ้างอิง']),
          calDate:    getRaw(['วันที่สอบเทียบ']),
          calType:    pageId,
          eqName:     getRaw(['รหัสครุภัณฑ์/ยี่ห้อ','ชื่อเครื่องมือ','Serial No.']),
          result:     getRaw(['ผลการตรวจสอบ','Evaluation','ผล','สรุปผล']),
          addedBy:    getRaw(['ผู้บันทึก']),
          approvedBy: approver,
          decision:   decision
        });
    }

    document.getElementById('ap-approve-btn').onclick = function(){ doApprove('approved'); };
    document.getElementById('ap-reject-btn').onclick  = function(){ doApprove('rejected'); };
  }

  
  // คำนวณ due diff เหมือน loadTableData
  function _calCostGetDiff(item) {
    var lastCal = item['วันที่สอบเทียบล่าสุด'] || item['วันที่ทำการสอบเทียบ (Cal-Date)'] || '';
    var intv    = item['ระยะเวลาการสอบเทียบ'] || '';
    var isCal   = _needsSchedule(item['สถานะการสอบเทียบ'] || item['การสอบเทียบหรือไม่'] || '', item);
    if (!isCal || !lastCal || !intv) return null;
    var d = new Date(lastCal);
    if (isNaN(d)) return null;
    // ใช้ Due Date จาก sheet ก่อน
    var dueSheet = item['สอบเทียบครั้งถัดไป'] || '';
    if (dueSheet) { var ds = new Date(dueSheet); if (!isNaN(ds)) d = ds; else { d.setMonth(d.getMonth() + (parseInt(intv)||12)); d.setDate(d.getDate()-1); } }
    else { d.setMonth(d.getMonth() + (parseInt(intv)||12)); d.setDate(d.getDate()-1); }
    var today = new Date(); today.setHours(0,0,0,0);
    return { diff: Math.ceil((d - today)/(1000*60*60*24)), dueDate: d };
  }

   function loadCalCost() {
    var tbody = document.getElementById('cal-cost-tbody');
    if (!tbody) return;
    tbody.innerHTML = '<tr><td colspan="8" class="px-4 py-10 text-center text-slate-400">กำลังโหลด...</td></tr>';

    google.script.run
      .withSuccessHandler(function(res) {
        _calCostData = (res && res.success && res.data) ? res.data : [];
        _renderCalCostSummary();
        _renderCalCostTable();
        _populateCalCostYearFilter();
        var cntEl = document.getElementById('cal-cost-count');
        if (cntEl) cntEl.textContent = 'พบ ' + _calCostData.length + ' รายการ';
      })
      .withFailureHandler(function(err) {
        _calCostData = [];
        _renderCalCostSummary();
        _renderCalCostTable();
        showToast('โหลดข้อมูลล้มเหลว: ' + err, 'warning');
      })
      .getCalCostData();
  }

  function _loadCalCostWithEquipments() {
    // สร้าง due list จาก cachedEquipments
    _calCostDueList = (cachedEquipments || []).filter(function(item){
      var r = _calCostGetDiff(item);
      return r !== null; // รวมทุกเครื่องที่มีข้อมูลสอบเทียบ
    }).map(function(item){
      var r = _calCostGetDiff(item);
      return Object.assign({}, item, {
        _diff:    r ? r.diff    : null,
        _dueDate: r ? r.dueDate : null
      });
    }).sort(function(a,b){ return (a._diff||9999) - (b._diff||9999); });

    // โหลดค่าใช้จ่ายที่บันทึกไว้
    google.script.run
      .withSuccessHandler(function(res) {
        _calCostData = (res && res.success && res.data) ? res.data : [];
        _renderCalCostPage();
      })
      .withFailureHandler(function(err) {
        _calCostData = [];
        _renderCalCostPage();
        showToast('โหลดค่าใช้จ่ายล้มเหลว: ' + err, 'warning');
      })
      .getCalCostData();
  }

    function _renderCalCostPage() {
    _renderCalCostSummary();
    _renderCalCostTable();
    _populateCalCostYearFilter();
    var cntEl = document.getElementById('cal-cost-count');
    if (cntEl) cntEl.textContent = 'พบ ' + _calCostData.length + ' รายการ';
  }

  function _populateCalCostEquipmentDropdown() {
    var sel = document.getElementById('cc-equipment');
    if (!sel) return;
    var src = (_calCostDueList.length > 0 ? _calCostDueList : cachedEquipments) || [];
    sel.innerHTML = '<option value="">— เลือกเครื่องมือ —</option>';
    src.forEach(function(item){
      var id   = item['ID'] || '';
      var code = item['รหัสครุภัณฑ์'] || '';
      var name = item['ชื่อเครื่องมือ'] || '-';
      var diff = item._diff;
      var tag  = diff !== undefined && diff !== null
        ? (diff < 0 ? ' 🔴 เกิน ' + Math.abs(diff) + 'วัน' : diff <= 60 ? ' ⚠️ อีก ' + diff + 'วัน' : '')
        : '';
      sel.innerHTML += '<option value="' + id + '">' + code + ' — ' + name + tag + '</option>';
    });
  }

  function _populateCalCostYearFilter() {
    var sel = document.getElementById('cal-cost-year-filter');
    if (!sel) return;
    var years = [...new Set(_calCostData.map(function(r){ return r.year||''; }).filter(Boolean))].sort().reverse();
    sel.innerHTML = '<option value="all">— ทุกปี —</option>';
    years.forEach(function(y){ sel.innerHTML += '<option value="' + y + '">' + y + '</option>'; });
  }

  function _getDueBadge(diff) {
    if (diff === null || diff === undefined) return '';
    if (diff < 0)       return '<span class="ml-1 px-1.5 py-0.5 rounded-full text-[9px] font-bold bg-red-100 text-red-700">🔴 เกิน ' + Math.abs(diff) + 'วัน</span>';
    if (diff === 0)     return '<span class="ml-1 px-1.5 py-0.5 rounded-full text-[9px] font-bold bg-red-100 text-red-700">🔴 ครบวันนี้</span>';
    if (diff <= 30)     return '<span class="ml-1 px-1.5 py-0.5 rounded-full text-[9px] font-bold bg-orange-100 text-orange-700">🔴 อีก ' + diff + 'วัน</span>';
    if (diff <= 60)     return '<span class="ml-1 px-1.5 py-0.5 rounded-full text-[9px] font-bold bg-amber-100 text-amber-700">⚠️ อีก ' + diff + 'วัน</span>';
    return '<span class="ml-1 px-1.5 py-0.5 rounded-full text-[9px] font-bold bg-emerald-50 text-emerald-600">✅ อีก ' + diff + 'วัน</span>';
  }

    function _renderCalCostSummary() {
    var el = document.getElementById('cal-cost-summary');
    if (!el) return;

    var totalCost = _calCostData.reduce(function(s, r) { return s + (parseFloat(r.cost) || 0); }, 0);
    var _now      = new Date();
    var _m        = _now.getMonth();
    var thisYear  = _m >= 9 ? (_now.getFullYear() + 544) : (_now.getFullYear() + 543);
    var totalTY   = _calCostData.filter(function(r) { return r.year == thisYear; })
                                .reduce(function(s, r) { return s + (parseFloat(r.cost) || 0); }, 0);
    var totalItems = _calCostData.length;

    // นับตาม vendor
    var vendorMap = {};
    _calCostData.forEach(function(r) {
      var v = r.vendor || 'ไม่ระบุ';
      vendorMap[v] = (vendorMap[v] || 0) + 1;
    });
    var topVendor = Object.keys(vendorMap).sort(function(a, b) { return vendorMap[b] - vendorMap[a]; })[0] || '-';

    function fmt(n) { return n.toLocaleString('th-TH', { minimumFractionDigits: 2, maximumFractionDigits: 2 }); }

    el.innerHTML = [
      {
        label: 'รายการทั้งหมด',
        val:   totalItems + ' รายการ',
        icon:  'receipt_long',
        cls:   'text-primary-700 bg-primary-50 border-primary-200 dark:bg-primary-900/20 dark:text-primary-400'
      },
      {
        label: 'ค่าใช้จ่ายรวม (ทั้งหมด)',
        val:   fmt(totalCost) + ' ฿',
        icon:  'account_balance_wallet',
        cls:   'text-slate-700 bg-slate-50 border-slate-200 dark:bg-slate-700/50 dark:text-slate-300'
      },
      {
        label: 'ค่าใช้จ่ายปีงบประมาณ ' + thisYear,
        val:   fmt(totalTY) + ' ฿',
        icon:  'calendar_today',
        cls:   'text-emerald-700 bg-emerald-50 border-emerald-200 dark:bg-emerald-900/20 dark:text-emerald-400'
      },
      {
        label: 'ผู้ให้บริการหลัก',
        val:   topVendor,
        icon:  'business',
        cls:   'text-amber-700 bg-amber-50 border-amber-200 dark:bg-amber-900/20 dark:text-amber-400'
      },
    ].map(function(c) {
      return '<div class="rounded-xl border p-4 ' + c.cls + '">'
        + '<div class="flex items-center gap-2 mb-1">'
        +   '<span class="material-symbols-outlined text-lg">' + c.icon + '</span>'
        +   '<span class="text-xs font-bold uppercase tracking-wide">' + c.label + '</span>'
        + '</div>'
        + '<div class="text-xl font-bold truncate">' + c.val + '</div>'
        + '</div>';
    }).join('');
  }

  function filterCalCostTable(query, mode) {
    if (mode !== undefined) _calCostFilterMode = mode;
    var q    = (query !== undefined ? query : (document.getElementById('cal-cost-search')||{}).value || '').toLowerCase().trim();
    var year = ((document.getElementById('cal-cost-year-filter')||{}).value) || 'all';
    var rows = document.querySelectorAll('#cal-cost-tbody tr[data-rowtype]');
    var vis  = 0;
    rows.forEach(function(row){
      var nameMatch = !q || (row.getAttribute('data-name')||'').includes(q) || (row.getAttribute('data-code')||'').includes(q);
      var yearMatch = year === 'all' || (row.getAttribute('data-year')||'') === year;
      var modeMatch = true;
      if (_calCostFilterMode === 'overdue') modeMatch = row.getAttribute('data-diff') !== null && parseInt(row.getAttribute('data-diff')) < 0;
      if (_calCostFilterMode === 'near60')  modeMatch = row.getAttribute('data-diff') !== null && parseInt(row.getAttribute('data-diff')) >= 0 && parseInt(row.getAttribute('data-diff')) <= 60;
      var show = nameMatch && yearMatch && modeMatch;
      row.style.display = show ? '' : 'none';
      if (show) vis++;
    });
    var cntEl = document.getElementById('cal-cost-count');
    if (cntEl) cntEl.textContent = 'แสดง ' + vis + ' รายการ';
  }

    function _renderCalCostTable() {
    var tbody = document.getElementById('cal-cost-tbody');
    if (!tbody) return;

    if (!_calCostData || _calCostData.length === 0) {
      tbody.innerHTML = '<tr><td colspan="8" class="px-4 py-14 text-center text-slate-400">'
        + '<span class="material-symbols-outlined text-4xl block mb-3 text-slate-300">receipt_long</span>'
        + '<div class="font-medium">ยังไม่มีรายการค่าใช้จ่าย</div>'
        + '<div class="text-xs mt-1">กดปุ่ม "+ เพิ่มรายการ" เพื่อบันทึกค่าใช้จ่ายสอบเทียบ</div>'
        + '</td></tr>';
      return;
    }

    // เรียงจากล่าสุด
    var sorted = _calCostData.slice().sort(function(a, b) {
      return (b.calDate || '').localeCompare(a.calDate || '');
    });

    var html = '';
    sorted.forEach(function(rec, idx) {
      var cost    = parseFloat(rec.cost) || 0;
      var costFmt = cost.toLocaleString('th-TH', { minimumFractionDigits: 2, maximumFractionDigits: 2 });
      var cdFmt   = rec.calDate
        ? (function() {
            var d = new Date(rec.calDate);
            return isNaN(d) ? rec.calDate : d.toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' });
          })()
        : '-';
      var rowBg = idx % 2 === 0 ? 'bg-white dark:bg-slate-800' : 'bg-slate-50/60 dark:bg-slate-700/20';

      html += '<tr class="' + rowBg + ' hover:bg-primary-50/30 dark:hover:bg-slate-700/30 transition border-b border-slate-100 dark:border-slate-700"'
        + ' data-rowtype="cost"'
        + ' data-name="' + (rec.eqName || '').toLowerCase() + '"'
        + ' data-code="' + (rec.eqCode || '').toLowerCase() + '"'
        + ' data-diff="9999"'
        + ' data-year="' + (rec.year || '') + '">'
        // # 
        + '<td class="px-4 py-3 text-center text-xs text-slate-400 font-medium">' + (idx + 1) + '</td>'
        // เครื่องมือ
        + '<td class="px-4 py-3">'
        +   '<div class="font-semibold text-sm text-slate-800 dark:text-slate-100">' + (rec.eqName || '-') + '</div>'
        +   (rec.eqCode ? '<div class="text-[11px] text-slate-400 font-mono mt-0.5">' + rec.eqCode + '</div>' : '')
        + '</td>'
        // วันที่สอบ
        + '<td class="px-4 py-3 text-xs text-slate-500 text-center whitespace-nowrap">' + cdFmt + '</td>'
        // ปีงบ
        + '<td class="px-4 py-3 text-center">'
        +   '<span class="px-2 py-0.5 bg-indigo-50 text-indigo-700 dark:bg-indigo-900/30 dark:text-indigo-400 rounded-full text-[11px] font-bold">' + (rec.year || '-') + '</span>'
        + '</td>'
        // ค่าใช้จ่าย
        + '<td class="px-4 py-3 text-center">'
        +   '<span class="font-bold text-emerald-700 dark:text-emerald-400 text-sm">' + costFmt + '</span>'
        +   '<span class="text-[10px] text-slate-400 ml-0.5">฿</span>'
        + '</td>'
        // ผู้ให้บริการ
        + '<td class="px-4 py-3 text-xs text-slate-500 text-center whitespace-nowrap">' + (rec.vendor || '<span class="text-slate-300">—</span>') + '</td>'
        // ใบเสร็จ / หมายเหตุ
        + '<td class="px-4 py-3 text-xs text-slate-400 max-w-[180px] truncate" title="' + (rec.note || '') + '">'
        +   (rec.invoice ? '<span class="font-mono text-[10px] bg-slate-100 dark:bg-slate-700 px-1.5 py-0.5 rounded mr-1">' + rec.invoice + '</span>' : '')
        +   (rec.note || '')
        + '</td>'
        // จัดการ
        + '<td class="px-4 py-3 text-center">'
        +   '<div class="flex items-center justify-center gap-1.5">'
        +     '<button onclick="openCalCostModal(' + JSON.stringify(rec).replace(/\"/g, "'") + ', null)" '
        +       'title="แก้ไข" class="w-7 h-7 rounded-lg flex items-center justify-center bg-blue-50 hover:bg-blue-100 text-blue-600 border border-blue-200 transition">'
        +       '<span class="material-symbols-outlined text-sm">edit</span>'
        +     '</button>'
        +     '<button onclick="deleteCalCost(\'' + (rec.id || '') + '\')" '
        +       'title="ลบ" class="w-7 h-7 rounded-lg flex items-center justify-center bg-red-50 hover:bg-red-100 text-red-500 border border-red-200 transition">'
        +       '<span class="material-symbols-outlined text-sm">delete</span>'
        +     '</button>'
        +   '</div>'
        + '</td>'
        + '</tr>';
    });

    tbody.innerHTML = html;
  }

  function _populateCalCostVendorDropdown(selectedVal) {
    var sel = document.getElementById('cc-vendor');
    if (!sel) return;
    // โหลด departments จาก GAS
    google.script.run
      .withSuccessHandler(function(depts) {
        var val = selectedVal || sel.value || '';
        sel.innerHTML = '<option value="">— เลือกผู้ให้บริการ —</option>';
        (depts || []).forEach(function(d) {
          sel.innerHTML += '<option value="' + d + '"' + (d === val ? ' selected' : '') + '>' + d + '</option>';
        });
        // ถ้าค่าที่เลือกไม่อยู่ในรายการให้เพิ่มเป็น option พิเศษ
        if (val && !(depts||[]).includes(val)) {
          sel.innerHTML += '<option value="' + val + '" selected>' + val + ' (กรอกเอง)</option>';
        }
        if (val) sel.value = val;
      })
      .withFailureHandler(function() {
        // fallback: ถ้าโหลดไม่ได้ให้คงค่าเดิม
      })
      .getDepartmentList();
  }

        // =========================================================
  // เมื่อเลือกเครื่องมือ — แสดงรหัสครุภัณฑ์ใต้ dropdown
  // =========================================================
  function onCalCostEqChange() {
    var sel  = document.getElementById('cc-eqname');
    var disp = document.getElementById('cc-eqcode-display');
    var hidCode = document.getElementById('cc-eqcode');
    var hidId   = document.getElementById('cc-eq-id');
    if (!sel || !disp) return;

    var eqId = sel.value;
    var item = (cachedEquipments || []).find(function(e) { return e['ID'] === eqId; });
    if (item) {
      var code = item['รหัสครุภัณฑ์'] || '';
      disp.textContent = code ? '📌 รหัสครุภัณฑ์: ' + code : '';
      disp.classList.toggle('hidden', !code);
      if (hidCode) hidCode.value = code;
      if (hidId)   hidId.value   = eqId;
    } else {
      disp.textContent = '';
      disp.classList.add('hidden');
      if (hidCode) hidCode.value = '';
      if (hidId)   hidId.value   = '';
    }
  }

  // =========================================================
  // โหลด dropdown เครื่องมือ + ผู้ให้บริการ
  // =========================================================
  function _loadCalCostDropdowns(presetEqId, presetVendor) {
    // --- dropdown เครื่องมือ ---
    var eqSel = document.getElementById('cc-eqname');
    if (eqSel) {
      eqSel.innerHTML = '<option value="">— เลือกเครื่องมือ —</option>';

      // ถ้า cachedEquipments มีข้อมูลแล้ว ใช้เลย
      var src = cachedEquipments || [];
      if (src.length > 0) {
        _fillEqDropdown(eqSel, src, presetEqId);
      } else {
        // โหลดจาก GAS ก่อน
        var dept = currentUser ? currentUser.dept : '';
        var role = currentUser ? currentUser.role : '';
        google.script.run
          .withSuccessHandler(function(data) {
            cachedEquipments = data || [];
            cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
            _fillEqDropdown(eqSel, cachedEquipments, presetEqId);
          })
          .withFailureHandler(function() {})
          .getEquipments(dept, role);
      }
    }

    // --- dropdown ผู้ให้บริการ (ดึงจาก Departments sheet) ---
    var vendorSel = document.getElementById('cc-vendor');
    if (vendorSel) {
      vendorSel.innerHTML = '<option value="">— เลือกผู้ให้บริการ —</option>';
      google.script.run
        .withSuccessHandler(function(depts) {
          (depts || []).forEach(function(d) {
            var opt = document.createElement('option');
            opt.value = d;
            opt.textContent = d;
            if (d === presetVendor) opt.selected = true;
            vendorSel.appendChild(opt);
          });
          // ถ้า presetVendor ไม่อยู่ในรายการ ให้เพิ่มเป็น option พิเศษ
          if (presetVendor && !Array.from(vendorSel.options).some(function(o) { return o.value === presetVendor; })) {
            var extra = document.createElement('option');
            extra.value = presetVendor;
            extra.textContent = presetVendor;
            extra.selected = true;
            vendorSel.appendChild(extra);
          }
        })
        .withFailureHandler(function() {})
        .getDepartmentList();
    }
  }

  function _fillEqDropdown(sel, equipments, presetEqId) {
    equipments.forEach(function(item) {
      var id   = item['ID'] || '';
      var code = item['รหัสครุภัณฑ์'] || '';
      var name = item['ชื่อเครื่องมือ'] || '-';
      var opt  = document.createElement('option');
      opt.value = id;
      opt.textContent = name + (code ? ' (' + code + ')' : '');
      if (id === presetEqId) opt.selected = true;
      sel.appendChild(opt);
    });
    // trigger แสดงรหัส ถ้ามี preset
    if (presetEqId) onCalCostEqChange();
  }

  // =========================================================
  // เปิด Modal
  // =========================================================
  function openCalCostModal(rec, presetEqId) {
    var modal = document.getElementById('cal-cost-modal');
    var title = document.getElementById('cal-cost-modal-title');
    var todayStr = new Date().toISOString().slice(0, 10);
    var presetVendor = '';

    if (rec && rec.id) {
      // แก้ไขรายการเดิม
      if (title) title.textContent = 'แก้ไขค่าใช้จ่ายสอบเทียบ';
      document.getElementById('cc-record-id').value = rec.id      || '';
      document.getElementById('cc-eq-id').value     = rec.eqId    || '';
      document.getElementById('cc-eqcode').value    = rec.eqCode  || '';
      document.getElementById('cc-caldate').value   = rec.calDate || todayStr;
      document.getElementById('cc-cost').value      = rec.cost    || '';
      document.getElementById('cc-invoice').value   = rec.invoice || '';
      document.getElementById('cc-note').value      = rec.note    || '';
      presetEqId   = rec.eqId   || '';
      presetVendor = rec.vendor || '';

      // แสดงรหัสครุภัณฑ์ (ถ้ามี)
      var disp = document.getElementById('cc-eqcode-display');
      if (disp && rec.eqCode) {
        disp.textContent = '📌 รหัสครุภัณฑ์: ' + rec.eqCode;
        disp.classList.remove('hidden');
      }
    } else {
      // เพิ่มใหม่
      if (title) title.textContent = 'บันทึกค่าใช้จ่ายสอบเทียบ';
      document.getElementById('cc-record-id').value = '';
      document.getElementById('cc-eq-id').value     = '';
      document.getElementById('cc-eqcode').value    = '';
      document.getElementById('cc-caldate').value   = todayStr;
      document.getElementById('cc-cost').value      = '';
      document.getElementById('cc-invoice').value   = '';
      document.getElementById('cc-note').value      = '';
      var disp2 = document.getElementById('cc-eqcode-display');
      if (disp2) { disp2.textContent = ''; disp2.classList.add('hidden'); }
    }

    // โหลด dropdowns
    _loadCalCostDropdowns(presetEqId || '', presetVendor);

    modal.style.display = 'flex';
    modal.classList.remove('hidden');
  }
  function closeCalCostModal() {
    var modal = document.getElementById('cal-cost-modal');
    if (modal) { modal.style.display = 'none'; modal.classList.add('hidden'); }
  }

       function saveCalCost() {
    var eqSel   = document.getElementById('cc-eqname');
    var eqId    = (document.getElementById('cc-eq-id')   || {}).value || '';
    var eqCode  = (document.getElementById('cc-eqcode')  || {}).value || '';
    var calDate = (document.getElementById('cc-caldate') || {}).value || '';
    var cost    = (document.getElementById('cc-cost')    || {}).value || '';
    var vendor  = (document.getElementById('cc-vendor')  || {}).value || '';

    // ดึงชื่อเครื่องมือจาก select option text (ตัด "(รหัส)" ออก)
    var eqName = '';
    if (eqSel && eqSel.selectedIndex > 0) {
      eqName = (eqSel.options[eqSel.selectedIndex].text || '').replace(/\s*\([^)]*\)\s*$/, '').trim();
    }

    // ดึง eqCode จาก cachedEquipments ถ้ายังไม่มี
    if (!eqCode && eqId) {
      var item = (cachedEquipments || []).find(function(e) { return e['ID'] === eqId; });
      if (item) eqCode = item['รหัสครุภัณฑ์'] || '';
    }

    // validation
    if (!eqId) {
      showToast('กรุณาเลือกเครื่องมือ', 'warning');
      if (eqSel) eqSel.focus();
      return;
    }
    if (!calDate) {
      showToast('กรุณาระบุวันที่สอบเทียบ', 'warning');
      document.getElementById('cc-caldate').focus();
      return;
    }
    if (cost === '' || isNaN(parseFloat(cost))) {
      showToast('กรุณากรอกค่าใช้จ่าย', 'warning');
      document.getElementById('cc-cost').focus();
      return;
    }
    if (!vendor) {
      showToast('กรุณาเลือกผู้ให้บริการสอบเทียบ', 'warning');
      document.getElementById('cc-vendor').focus();
      return;
    }

    // คำนวณปีงบประมาณไทย
    var calDateObj = new Date(calDate);
    var calMonth   = calDateObj.getMonth();
    var calYear    = calDateObj.getFullYear();
    var fiscalYear = calMonth >= 9 ? (calYear + 544) : (calYear + 543);

    var payload = {
      id:      (document.getElementById('cc-record-id') || {}).value || '',
      eqId:    eqId,
      eqName:  eqName,
      eqCode:  eqCode,
      calDate: calDate,
      year:    fiscalYear,
      cost:    parseFloat(cost) || 0,
      vendor:  vendor,
      invoice: (document.getElementById('cc-invoice') || {}).value || '',
      note:    (document.getElementById('cc-note')    || {}).value || '',
      savedBy: _getDisplayName() || (currentUser && currentUser.username) || '',
      savedAt: new Date().toISOString()
    };

    showLoader();
    google.script.run
      .withSuccessHandler(function(res) {
        hideLoader();
        if (res && res.success) {
          showToast('✅ บันทึกค่าใช้จ่ายเรียบร้อยแล้ว', 'success');
          closeCalCostModal();
          loadCalCost();
        } else {
          showToast('เกิดข้อผิดพลาด: ' + (res && res.message || ''), 'danger');
        }
      })
      .withFailureHandler(function(err) {
        hideLoader();
        showToast('ขัดข้อง: ' + err, 'danger');
      })
      .saveCalCostData(payload);
  }

  function deleteCalCost(id) {
    if (!id || !confirm('ยืนยันลบรายการค่าใช้จ่ายนี้?')) return;
    showLoader();
    google.script.run
      .withSuccessHandler(function(res){
        hideLoader();
        if (res && res.success) { showToast('🗑️ ลบรายการเรียบร้อย', 'success'); loadCalCost(); }
        else showToast('เกิดข้อผิดพลาด: ' + (res&&res.message||''), 'danger');
      })
      .withFailureHandler(function(err){ hideLoader(); showToast('ขัดข้อง: ' + err, 'danger'); })
      .deleteCalCostData(id);
  }

  function exportCalCostCSV() {
    if (!_calCostData || _calCostData.length === 0) { showToast('ยังไม่มีข้อมูลค่าใช้จ่ายสำหรับ Export', 'warning'); return; }
    var headers = ['ลำดับ','รหัสครุภัณฑ์','ชื่อเครื่องมือ','ปีงบประมาณ','วันที่สอบเทียบ','ค่าใช้จ่าย (บาท)','ผู้ให้บริการ','เลขที่ใบเสร็จ','หมายเหตุ','บันทึกโดย'];
    var rows = _calCostData.map(function(r, i){
      return [i+1, r.eqCode||'', r.eqName||'', r.year||'', r.calDate||'', r.cost||0, r.vendor||'', r.invoice||'', r.note||'', r.savedBy||'']
        .map(function(v){ return '"' + String(v).replace(/"/g,'""') + '"'; }).join(',');
    });
    var csv  = '\uFEFF' + headers.join(',') + '\n' + rows.join('\n');
    var blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
    var url  = URL.createObjectURL(blob);
    var a    = document.createElement('a'); a.href = url;
    a.download = 'cal_cost_' + new Date().toISOString().slice(0,10) + '.csv';
    document.body.appendChild(a); a.click(); document.body.removeChild(a);
    URL.revokeObjectURL(url);
    showToast('📥 Export CSV เรียบร้อย', 'success');
  }

  // ปุ่ม + เพิ่มรายการ (Add manually without preset equipment)
  document.addEventListener('DOMContentLoaded', function(){
    var pg = document.getElementById('page-cal-cost');
    if (!pg) return;
    var addBtn = document.createElement('button');
    addBtn.className = 'flex items-center gap-1.5 px-4 py-2 rounded-lg bg-primary-700 hover:bg-primary-800 text-white text-sm font-bold transition';
    addBtn.innerHTML = '<span class="material-symbols-outlined text-base">add</span> เพิ่มรายการ';
    addBtn.onclick = function(){ openCalCostModal(null, null); };
    var hdr = pg.querySelector('.flex.gap-2');
    if (hdr) hdr.insertBefore(addBtn, hdr.firstChild);
  });

  // =====================================================
  // ✅ Export แผนและสรุปผลการสอบเทียบ → PDF (แบบฟอร์ม SD-Fo-06-Patho-QMU-005)
  //    หัวกระดาษแบบฟอร์มควบคุม + ตัวกรองที่เลือก + พิมพ์โดย / วันที่พิมพ์
  //    ใช้เงื่อนไขกรองชุดเดียวกับ Export Excel
  // =====================================================
  var REPORT_PDF_DOC = { code: 'SD-Fo-06-Patho-QMU-005', rev: '01', effective: '1 ต.ค. 2569' };

  function _collectReportRows() {
    var fyFilter     = (document.getElementById('report-fy-filter')     || {}).value || 'all';
    var deptFilter   = (document.getElementById('report-dept-filter')   || {}).value || 'all';
    var typeFilter   = (document.getElementById('report-type-filter')   || {}).value || 'all';
    var statusFilter = (document.getElementById('report-status-filter') || {}).value || 'all';
    var today = new Date(); today.setHours(0,0,0,0);
    var thaiM = ["ม.ค.","ก.พ.","มี.ค.","เม.ย.","พ.ค.","มิ.ย.","ก.ค.","ส.ค.","ก.ย.","ต.ค.","พ.ย.","ธ.ค."];
    var sumData = (typeof calSummaryData !== 'undefined' && calSummaryData) || {};
    var out = [];
    function addIv(d, iv) { var m = parseInt(iv); if (!isNaN(m)) d.setMonth(d.getMonth() + m); return d; }

    (cachedEquipments || []).forEach(function (item) {
      if (typeFilter !== 'all' && item['ประเภทเครื่องมือ'] !== typeFilter) return;
      if (deptFilter !== 'all' && (item['สถานที่ติดตั้ง'] || '') !== deptFilter) return;
      var calRec = sumData[item['ID'] || ''] || {};
      var result = calRec.result || 'รอผล';
      var isCal = _needsSchedule(item['สถานะการสอบเทียบ'], item);
      var intervalRaw = String(item['ระยะเวลาการสอบเทียบ'] || '').trim();
      var planText = '-', eqFy = 'all', dueDate = null, dueDiff = null;
      if (isCal) {
        var dr = item['สอบเทียบครั้งถัดไป'] || '';
        if (dr) { var t = new Date(dr); if (!isNaN(t)) dueDate = t; }
        if (!dueDate && calRec.calDate && intervalRaw) { var t2 = new Date(calRec.calDate); if (!isNaN(t2)) dueDate = addIv(t2, intervalRaw); }
        if (!dueDate && item['วันที่สอบเทียบล่าสุด'] && intervalRaw) { var t3 = new Date(item['วันที่สอบเทียบล่าสุด']); if (!isNaN(t3)) dueDate = addIv(t3, intervalRaw); }
        if (dueDate) {
          var m = dueDate.getMonth(), fy = dueDate.getFullYear(); if (m >= 9) fy += 1;
          eqFy = String(fy); planText = thaiM[m] + ' ' + (fy + 543) + (_isMaint(item) ? ' (บำรุงรักษา)' : '');
          dueDiff = Math.ceil((dueDate - today) / 86400000);
        }
      }
      if (fyFilter !== 'all' && eqFy !== fyFilter) return;
      var isPass = /ยอมรับ|ผ่าน|Pass/.test(result) && !/ไม่ผ่าน/.test(result);
      var isFail = /ปฏิเสธ|ไม่ผ่าน|Fail/.test(result);
      var isOverdue = dueDiff !== null && dueDiff < 0;
      var isNear60 = dueDiff !== null && dueDiff >= 0 && dueDiff <= 60;
      if (statusFilter === 'pass'    && !isPass) return;
      if (statusFilter === 'fail'    && !isFail) return;
      if (statusFilter === 'pending' && (isPass || isFail)) return;
      if (statusFilter === 'overdue' && !isOverdue) return;
      if (statusFilter === 'near60'  && !isNear60) return;
      var cd = calRec.calDate || '';
      var cdFmt = cd ? (function () { var d = new Date(cd); return isNaN(d) ? cd : d.toLocaleDateString('th-TH', { year: 'numeric', month: 'short', day: 'numeric' }); })() : '-';
      out.push({
        name: item['ชื่อเครื่องมือ'] || '-', code: item['รหัสครุภัณฑ์'] || '-', type: item['ประเภทเครื่องมือ'] || '-',
        dept: item['สถานที่ติดตั้ง'] || '-',
        plan: planText + (isOverdue ? ' (เกิน ' + Math.abs(dueDiff) + ' วัน)' : isNear60 ? ' (อีก ' + dueDiff + ' วัน)' : ''),
        cert: calRec.certNo || '-', calDate: cdFmt,
        result: isPass ? 'ผ่าน / ยอมรับ' : isFail ? 'ไม่ผ่าน / ปฏิเสธ' : result, pass: isPass, fail: isFail,
        approval: calRec.approvalStatus || '-', cond: item['สถานะเครื่องมือ'] || '-'
      });
    });
    return out;
  }

  function exportReportPdf() {
    var rows = _collectReportRows();
    if (!rows.length) { showToast('ไม่มีข้อมูลที่ตรงเงื่อนไขสำหรับ Export', 'warning'); return; }

    function selText(id) { var el = document.getElementById(id); return el && el.options && el.selectedIndex >= 0 ? el.options[el.selectedIndex].text : '-'; }
    function esc(v) { return String(v == null ? '' : v).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;'); }

    var now = new Date();
    var printBy = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser ? (currentUser.fullname || currentUser.username) : '-');
    var printAt = now.toLocaleDateString('th-TH', { day: 'numeric', month: 'long', year: 'numeric' })
                + ' เวลา ' + now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + ' น.';
    var filterLine = 'ปีงบประมาณ: ' + selText('report-fy-filter') + ' &nbsp;|&nbsp; หน่วยงาน: ' + selText('report-dept-filter')
                   + ' &nbsp;|&nbsp; ประเภท: ' + selText('report-type-filter') + ' &nbsp;|&nbsp; สถานะ: ' + selText('report-status-filter');

    var logo = '<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 48 48" width="34" height="34">'
      + '<rect width="48" height="48" rx="6" fill="#1e3a8a"/><path d="M24 10 L28 18 L20 18 Z" fill="white"/>'
      + '<rect x="22" y="18" width="4" height="12" rx="2" fill="white"/>'
      + '<ellipse cx="24" cy="33" rx="8" ry="3" fill="none" stroke="white" stroke-width="2"/>'
      + '<line x1="16" y1="38" x2="32" y2="38" stroke="white" stroke-width="2.5" stroke-linecap="round"/></svg>';

    var body = rows.map(function (r, i) {
      var rc = r.pass ? 'color:#15803d' : r.fail ? 'color:#dc2626' : 'color:#64748b';
      return '<tr>'
        + '<td class="c">' + (i + 1) + '</td><td><b>' + esc(r.name) + '</b></td><td>' + esc(r.code) + '</td>'
        + '<td>' + esc(r.type) + '</td><td>' + esc(r.dept) + '</td><td class="c" style="color:#4338ca;font-weight:700">' + esc(r.plan) + '</td>'
        + '<td class="c">' + esc(r.cert) + '</td><td class="c">' + esc(r.calDate) + '</td>'
        + '<td class="c" style="font-weight:700;' + rc + '">' + esc(r.result) + '</td>'
        + '<td class="c">' + esc(r.approval) + '</td><td class="c">' + esc(r.cond) + '</td></tr>';
    }).join('');

    var html = '<!DOCTYPE html><html lang="th"><head><meta charset="UTF-8"><title>' + REPORT_PDF_DOC.code + ' แผนและสรุปผลการสอบเทียบ</title><style>'
      + '@import url("https://fonts.googleapis.com/css2?family=Sarabun:wght@400;600;700&display=swap");'
      + '@page{size:A4 landscape;margin:10mm 10mm 14mm 10mm}'
      + '*{box-sizing:border-box;margin:0;padding:0}'
      + 'body{font-family:"Sarabun","Noto Sans Thai",Arial,sans-serif;font-size:10.5px;color:#1e293b;-webkit-print-color-adjust:exact;print-color-adjust:exact}'
      + '.doc-header{display:flex;align-items:stretch;border:1.5px solid #1e3a8a}'
      + '.hd-logo{width:70px;min-width:70px;background:#1e3a8a;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:5px 4px;gap:2px}'
      + '.hd-logo-text{color:#fff;font-size:8px;font-weight:700;text-align:center;line-height:1.3}'
      + '.hd-title{flex:1;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:6px 12px;border-left:1.5px solid #1e3a8a;border-right:1.5px solid #1e3a8a}'
      + '.hd-title h1{font-size:16px;font-weight:700;text-align:center}'
      + '.hd-title h2{font-size:12px;font-weight:600;color:#334155;margin-top:2px;text-align:center}'
      + '.hd-meta{width:215px;min-width:215px;font-size:10px;padding:5px 8px;display:flex;flex-direction:column;justify-content:center;gap:2px}'
      + '.hd-meta div{display:flex;gap:4px}.hd-meta span{color:#64748b;font-weight:600;white-space:nowrap}.hd-meta b{color:#1e293b;white-space:nowrap}'
      + '.filters{text-align:center;font-size:10px;color:#475569;padding:5px 0 7px}'
      + 'table.rep{width:100%;border-collapse:collapse}'
      + 'table.rep th,table.rep td{border:1px solid #94a3b8;padding:4px 5px;vertical-align:top}'
      + 'table.rep thead .cols th{background:#e2e8f0;font-weight:700;text-align:center;vertical-align:middle}'
      + 'table.rep thead .hd th{border:none;padding:0;font-weight:normal;text-align:left}'
      + 'table.rep tbody tr:nth-child(even) td{background:#f8fafc}'
      + 'table.rep tr{page-break-inside:avoid}'
      + '.c{text-align:center}'
      + '.total td{background:#f1f5f9!important;font-weight:700}'
      + 'table.rep tfoot td{border:none;padding:4px 0 0}'
      + '.foot{display:flex;justify-content:space-between;font-size:9px;color:#64748b;border-top:1px solid #cbd5e1;padding-top:3px}'
      + '</style></head><body>'
      + '<table class="rep"><thead>'
      +   '<tr class="hd"><th colspan="11">'
      +     '<div class="doc-header">'
      +       '<div class="hd-logo">' + logo + '<div class="hd-logo-text">สาขาวิชา<br>พยาธิวิทยา</div></div>'
      +       '<div class="hd-title"><h1>แผนและสรุปผลการสอบเทียบเครื่องมือห้องปฏิบัติการ</h1></div>'
      +       '<div class="hd-meta">'
      +         '<div><span>รหัสเอกสาร :</span><b>' + REPORT_PDF_DOC.code + '</b></div>'
      +         '<div><span>แก้ไขครั้งที่ :</span><b>' + REPORT_PDF_DOC.rev + '</b></div>'
      +         '<div><span>วันที่ประกาศใช้ :</span><b>' + REPORT_PDF_DOC.effective + '</b></div>'
      +       '</div>'
      +     '</div>'
      +     '<div class="filters">' + filterLine + '</div>'
      +   '</th></tr>'
      +   '<tr class="cols"><th style="width:32px">ลำดับ</th><th>ชื่อเครื่องมือ</th><th>รหัสครุภัณฑ์</th><th>ประเภท</th><th>หน่วยงาน / สถานที่</th>'
      +   '<th>แผนปีงบประมาณ</th><th>Certificate Number</th><th>วันที่สอบเทียบ</th><th>ผลการตรวจรับรอง</th><th>สถานะการอนุมัติ</th><th>สถานะเครื่องมือ</th></tr>'
      + '</thead><tbody>' + body
      +   '<tr class="total"><td colspan="5" style="text-align:right">รวมทั้งหมด</td><td colspan="6">' + rows.length + ' รายการ</td></tr>'
      + '</tbody><tfoot><tr><td colspan="11"><div class="foot"><span>' + REPORT_PDF_DOC.code + '</span><span>พิมพ์โดย: ' + esc(printBy) + ' &nbsp;|&nbsp; วันที่พิมพ์: ' + printAt + '</span></div></td></tr></tfoot></table>'
      + '</body></html>';

    var win = window.open('', '_blank', 'width=1200,height=800');
    if (!win) { showToast('เบราว์เซอร์บล็อกหน้าต่างใหม่ กรุณาอนุญาต Pop-up', 'warning'); return; }
    win.document.open(); win.document.write(html); win.document.close();
    win.onload = function () { setTimeout(function () { win.focus(); win.print(); }, 600); };
    showToast('📄 เตรียม PDF เรียบร้อย (' + rows.length + ' รายการ) — เลือก "บันทึกเป็น PDF" ในหน้าต่างพิมพ์', 'success');
  }

  // =====================================================
  // Export แผนและสรุปผลการสอบเทียบ → Excel (.xlsx via CSV)
  // =====================================================
  function exportReportExcel() {
    // รวบรวมแถวที่แสดงอยู่ในตาราง
    var tbody = document.getElementById('cal-report-table-body');
    if (!tbody) { showToast('ไม่มีข้อมูลสำหรับ Export', 'warning'); return; }

    var rows = tbody.querySelectorAll('tr[data-report-status], tr');
    if (!rows || rows.length === 0) { showToast('ไม่มีข้อมูลสำหรับ Export', 'warning'); return; }

    // อ่านค่า filter ปัจจุบัน
    var fyText   = (document.getElementById('report-fy-filter')     || {}).options;
    var deptText = (document.getElementById('report-dept-filter')   || {}).options;
    var typeText = (document.getElementById('report-type-filter')   || {}).options;
    var stText   = (document.getElementById('report-status-filter') || {}).options;
    var fyLabel   = fyText   ? fyText[fyText.selectedIndex].text   : '';
    var deptLabel = deptText ? deptText[deptText.selectedIndex].text : '';
    var typeLabel = typeText ? typeText[typeText.selectedIndex].text : '';

    // สร้าง HTML table สำหรับ Excel
    var now = new Date().toLocaleDateString('th-TH',{year:'numeric',month:'long',day:'numeric'});
    // ✅ หัวกระดาษแบบฟอร์มควบคุม (เหมือน Export PDF) — SD-Fo-06-Patho-QMU-005
    var stText2  = (document.getElementById('report-status-filter') || {}).options;
    var stLabel  = stText2 ? stText2[stText2.selectedIndex].text : '';
    var _xBy = (typeof _getDisplayName === 'function' && _getDisplayName()) || (currentUser ? (currentUser.fullname || currentUser.username) : '-');
    var _xNow = new Date();
    var _xAt = _xNow.toLocaleDateString('th-TH', { day: 'numeric', month: 'long', year: 'numeric' })
             + ' เวลา ' + _xNow.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + ' น.';
    var _bd = 'border:1.5px solid #1e3a8a;';
    var htmlTable = '<!DOCTYPE html><html xmlns:x="urn:schemas-microsoft-com:office:excel"><head><meta charset="UTF-8">'
      + '<style>@page{mso-page-orientation:landscape;margin:0.4in 0.3in 0.5in 0.3in}td{mso-number-format:"\\@";}</style></head><body>'
      + '<table border="1" style="border-collapse:collapse;font-family:Sarabun,sans-serif;font-size:11pt;">'
      // แถว 1–3: โลโก้/สาขา | ชื่อแบบฟอร์ม | รหัสเอกสาร/แก้ไขครั้งที่/วันที่ประกาศใช้
      + '<tr>'
      +   '<td colspan="2" rowspan="3" style="' + _bd + 'background:#1e3a8a;color:#ffffff;font-weight:bold;text-align:center;vertical-align:middle;font-size:12pt;">สาขาวิชา<br>พยาธิวิทยา</td>'
      +   '<td colspan="6" rowspan="2" style="' + _bd + 'border-bottom:none;font-size:16pt;font-weight:bold;text-align:center;vertical-align:middle;padding:6px;">แผนและสรุปผลการสอบเทียบเครื่องมือห้องปฏิบัติการ</td>'
      +   '<td colspan="3" style="' + _bd + 'border-bottom:none;font-size:10pt;padding:3px 6px;"><font color="#64748b">รหัสเอกสาร :</font> <b>' + REPORT_PDF_DOC.code + '</b></td>'
      + '</tr>'
      + '<tr>'
      +   '<td colspan="3" style="' + _bd + 'border-top:none;border-bottom:none;font-size:10pt;padding:3px 6px;"><font color="#64748b">แก้ไขครั้งที่ :</font> <b>' + REPORT_PDF_DOC.rev + '</b></td>'
      + '</tr>'
      + '<tr>'
      +   '<td colspan="6" style="' + _bd + 'border-top:none;font-size:9pt;color:#475569;text-align:center;">ปีงบประมาณ: ' + fyLabel + ' | หน่วยงาน: ' + deptLabel + ' | ประเภท: ' + typeLabel + ' | สถานะ: ' + stLabel + '</td>'
      +   '<td colspan="3" style="' + _bd + 'border-top:none;font-size:10pt;padding:3px 6px;"><font color="#64748b">วันที่ประกาศใช้ :</font> <b>' + REPORT_PDF_DOC.effective + '</b></td>'
      + '</tr>'
      + '<tr><td colspan="11" style="border:none;height:6px;"></td></tr>'
      + '<tr style="background:#e2e8f0;font-weight:bold;">'
      + '<td style="padding:6px 10px;text-align:center;width:40px;">ลำดับ</td>'
      + '<td style="padding:6px 10px;">ชื่อเครื่องมือ</td>'
      + '<td style="padding:6px 10px;">รหัสครุภัณฑ์</td>'
      + '<td style="padding:6px 10px;">ประเภท</td>'
      + '<td style="padding:6px 10px;">หน่วยงาน / สถานที่</td>'
      + '<td style="padding:6px 10px;text-align:center;">แผนปีงบประมาณ</td>'
      + '<td style="padding:6px 10px;text-align:center;">Certificate Number</td>'
      + '<td style="padding:6px 10px;text-align:center;">วันที่สอบเทียบ</td>'
      + '<td style="padding:6px 10px;text-align:center;">ผลการตรวจรับรอง</td>'
      + '<td style="padding:6px 10px;text-align:center;">สถานะการอนุมัติ</td>'
      + '<td style="padding:6px 10px;text-align:center;">สถานะเครื่องมือ</td>'
      + '</tr>';

    var rowIdx = 0;
    // ดึงจาก cachedEquipments แทนการ parse HTML (แม่นกว่า)
    var fyFilter     = (document.getElementById('report-fy-filter')     || {}).value || 'all';
    var deptFilter   = (document.getElementById('report-dept-filter')   || {}).value || 'all';
    var typeFilter   = (document.getElementById('report-type-filter')   || {}).value || 'all';
    var statusFilter = (document.getElementById('report-status-filter') || {}).value || 'all';
    var today = new Date(); today.setHours(0,0,0,0);
    var thaiMonths2 = ["ม.ค.","ก.พ.","มี.ค.","เม.ย.","พ.ค.","มิ.ย.","ก.ค.","ส.ค.","ก.ย.","ต.ค.","พ.ย.","ธ.ค."];

    (cachedEquipments||[]).forEach(function(item){
      if (typeFilter !== 'all' && item['ประเภทเครื่องมือ'] !== typeFilter) return;
      if (deptFilter !== 'all' && (item['สถานที่ติดตั้ง']||'') !== deptFilter) return;
      var eqId   = item['ID'] || '';
      var calRec = calSummaryData[eqId] || {};
      var certNo = calRec.certNo || '-';
      var calDate = calRec.calDate || '-';
      var result = calRec.result || 'รอผล';
      var approvalStatus = calRec.approvalStatus || '-';
      var eqCondition    = item['สถานะเครื่องมือ'] || '-';
      var isCal = _needsSchedule(item['สถานะการสอบเทียบ'], item);
      var dueDateRaw  = item['สอบเทียบครั้งถัดไป'] || '';
      var intervalRaw = String(item['ระยะเวลาการสอบเทียบ']||'').trim();
      var planText = '-'; var eqFy = 'all'; var dueDate = null; var dueDiff = null;
      function addIv(d,iv){ var m=parseInt(iv); if(!isNaN(m)) d.setMonth(d.getMonth()+m); return d; }
      if (isCal) {
        if (dueDateRaw) { var t=new Date(dueDateRaw); if(!isNaN(t)) dueDate=t; }
        if (!dueDate && calRec.calDate && intervalRaw) { var t2=new Date(calRec.calDate); if(!isNaN(t2)) dueDate=addIv(t2,intervalRaw); }
        if (!dueDate && item['วันที่สอบเทียบล่าสุด'] && intervalRaw) { var t3=new Date(item['วันที่สอบเทียบล่าสุด']); if(!isNaN(t3)) dueDate=addIv(t3,intervalRaw); }
        if (dueDate) {
          var m=dueDate.getMonth(); var y=dueDate.getFullYear();
          var fy=y; if(m>=9)fy+=1; eqFy=fy.toString();
          planText=thaiMonths2[m]+' '+(fy+543)+(_isMaint(item)?' (บำรุงรักษา)':'');
          dueDiff=Math.ceil((dueDate-today)/(1000*60*60*24));
        }
      }
      if (fyFilter!=='all' && eqFy!==fyFilter) return;
      var isPass=result.includes('ยอมรับ')||result.includes('ผ่าน')||result.includes('Pass');
      var isFail2=result.includes('ปฏิเสธ')||result.includes('ไม่ผ่าน')||result.includes('Fail');
      var isOverdue=dueDiff!==null&&dueDiff<0;
      var isNear60=dueDiff!==null&&dueDiff>=0&&dueDiff<=60;
      if (statusFilter==='pass'    && !isPass)   return;
      if (statusFilter==='fail'    && !isFail2)  return;
      if (statusFilter==='pending' && (isPass||isFail2)) return;
      if (statusFilter==='overdue' && !isOverdue) return;
      if (statusFilter==='near60'  && !isNear60)  return;
      rowIdx++;
      var calDateFmt = calDate && calDate!=='-' ? (function(){ var d=new Date(calDate); return isNaN(d)?calDate:d.toLocaleDateString('th-TH',{year:'numeric',month:'short',day:'numeric'}); })() : '-';
      var rowBg = rowIdx%2===0 ? 'background:#f8fafc;' : '';
      var resultTxt = isPass ? 'ผ่าน / ยอมรับ' : isFail2 ? 'ไม่ผ่าน / ปฏิเสธ' : result;
      var resultColor = isPass ? 'color:#15803d;' : isFail2 ? 'color:#dc2626;' : 'color:#64748b;';
      var overdueNote = isOverdue ? ' (เกิน '+Math.abs(dueDiff)+' วัน)' : isNear60 ? ' (อีก '+dueDiff+' วัน)' : '';
      htmlTable += '<tr style="' + rowBg + '">'
        + '<td style="padding:5px 8px;text-align:center;">' + rowIdx + '</td>'
        + '<td style="padding:5px 8px;font-weight:bold;">' + (item['ชื่อเครื่องมือ']||'-') + '</td>'
        + '<td style="padding:5px 8px;font-size:10pt;color:#475569;">' + (item['รหัสครุภัณฑ์']||'-') + '</td>'
        + '<td style="padding:5px 8px;font-size:10pt;color:#64748b;">' + (item['ประเภทเครื่องมือ']||'-') + '</td>'
        + '<td style="padding:5px 8px;font-size:10pt;">' + (item['สถานที่ติดตั้ง']||'-') + '</td>'
        + '<td style="padding:5px 8px;text-align:center;font-weight:bold;color:#4338ca;">' + planText + overdueNote + '</td>'
        + '<td style="padding:5px 8px;text-align:center;font-family:monospace;">' + certNo + '</td>'
        + '<td style="padding:5px 8px;text-align:center;">' + calDateFmt + '</td>'
        + '<td style="padding:5px 8px;text-align:center;font-weight:bold;' + resultColor + '">' + resultTxt + '</td>'
        + '<td style="padding:5px 8px;text-align:center;">' + approvalStatus + '</td>'
        + '<td style="padding:5px 8px;text-align:center;">' + eqCondition + '</td>'
        + '</tr>';
    });

    if (rowIdx === 0) { showToast('ไม่มีข้อมูลที่ตรงเงื่อนไขสำหรับ Export', 'warning'); return; }

    htmlTable += '<tr style="background:#f1f5f9;font-weight:bold;">'
      + '<td colspan="5" style="padding:5px 10px;text-align:right;">รวมทั้งหมด</td>'
      + '<td colspan="6" style="padding:5px 10px;text-align:left;font-weight:bold;color:#1e3a8a;">' + rowIdx + ' รายการ</td>'
      + '</tr>'
      + '<tr><td colspan="11" style="border:none;height:6px;"></td></tr>'
      + '<tr><td colspan="4" style="border:none;border-top:1px solid #cbd5e1;font-size:9pt;color:#64748b;">' + REPORT_PDF_DOC.code + '</td>'
      +   '<td colspan="7" style="border:none;border-top:1px solid #cbd5e1;font-size:9pt;color:#64748b;text-align:right;">พิมพ์โดย: ' + _xBy + ' | วันที่พิมพ์: ' + _xAt + '</td></tr>'
      + '</table></body></html>';

    var blob = new Blob(['\uFEFF' + htmlTable], { type: 'application/vnd.ms-excel;charset=utf-8;' });
    var url  = URL.createObjectURL(blob);
    var a    = document.createElement('a');
    a.href   = url;
    var fy   = (document.getElementById('report-fy-filter')||{}).value || 'all';
    a.download = 'CalibrationReport_' + (fy !== 'all' ? 'FY'+fy : 'All') + '_' + new Date().toISOString().slice(0,10) + '.xls';
    document.body.appendChild(a); a.click(); document.body.removeChild(a);
    URL.revokeObjectURL(url);
    showToast('📥 Export Excel เรียบร้อย (' + rowIdx + ' รายการ)', 'success');
  }
  // ============================================================
  // หน้าสถานะเครื่องมือ (Equipment Status Page)
  // ============================================================
  var _eqStatusData = [];

  function loadEqStatusPage() {
    showLoader();
    var dept = currentUser ? currentUser.dept : '';
    var role = currentUser ? currentUser.role : '';
    google.script.run
      .withSuccessHandler(function(data) {
        hideLoader();
        cachedEquipments = data || [];
        cachedEquipments.forEach(function(item, i) { item._index = i; item._rowIndex = i + 2; });
        _renderEqStatusPage();
      })
      .withFailureHandler(function(err) {
        hideLoader();
        showToast('โหลดข้อมูลล้มเหลว: ' + err, 'danger');
      })
      .getEquipments(dept, role);
}

  function _renderEqStatusPage() {
    _eqStatusData = cachedEquipments || [];
    _renderEqStatusSummary();
    _renderEqStatusTable();
  }

  function _renderEqStatusSummary() {
    var el = document.getElementById('eq-status-summary');
    if (!el) return;

    var counts = {
      all:       _eqStatusData.length,
      failed:    0,
      broken:    0,
      repair:    0,
      suspended: 0,
      retired:   0,
      ready:     0,
    };

    _eqStatusData.forEach(function(item) {
      var s = item['สถานะเครื่องมือ'] || '';
      if (s === 'สอบเทียบไม่ผ่านเกณฑ์') counts.failed++;
      else if (s === 'เครื่องมือเสีย')   counts.broken++;
      else if (s === 'ส่งซ่อม')          counts.repair++;
      else if (s === 'ระงับการใช้งาน')    counts.suspended++;
      else if (s === 'จำหน่ายออก')       counts.retired++;
      else if (s === 'พร้อมใช้งาน')      counts.ready++;
      else if (s === 'รอทวนสอบก่อนใช้')  counts.verify = (counts.verify || 0) + 1;
    });

    var cards = [
      { label: 'เครื่องมือทั้งหมด',    val: counts.all,       icon: 'inventory_2',     cls: 'text-primary-700 bg-primary-50 border-primary-200',   filter: 'all' },
      { label: 'ไม่ผ่านเกณฑ์',         val: counts.failed,    icon: 'warning',         cls: 'text-red-700 bg-red-50 border-red-200',                filter: 'สอบเทียบไม่ผ่านเกณฑ์' },
      { label: 'เครื่องมือเสีย',        val: counts.broken,    icon: 'build',           cls: 'text-stone-700 bg-stone-50 border-stone-200',          filter: 'เครื่องมือเสีย' },
      { label: 'ส่งซ่อม',              val: counts.repair,    icon: 'home_repair_service', cls: 'text-orange-700 bg-orange-50 border-orange-200',   filter: 'ส่งซ่อม' },
      { label: 'ระงับการใช้งาน',        val: counts.suspended, icon: 'do_not_disturb',  cls: 'text-violet-700 bg-violet-50 border-violet-200',       filter: 'ระงับการใช้งาน' },
      { label: 'พร้อมใช้งาน',          val: counts.ready,     icon: 'check_circle',    cls: 'text-emerald-700 bg-emerald-50 border-emerald-200',    filter: 'พร้อมใช้งาน' },
      { label: 'รอทวนสอบ/อนุมัติ',     val: counts.verify || 0, icon: 'pending_actions', cls: 'text-sky-700 bg-sky-50 border-sky-200',             filter: 'รอทวนสอบก่อนใช้' },
      { label: 'ปลดระวาง',            val: counts.retired,   icon: 'inventory_2',     cls: 'text-slate-600 bg-slate-50 border-slate-200',          filter: 'จำหน่ายออก' },
    ];

    el.innerHTML = cards.map(function(c) {
      return '<div onclick="setEqStatusFilter(\'' + c.filter + '\')" '
        + 'class="rounded-xl border p-3 cursor-pointer hover:shadow-md transition ' + c.cls + '">'
        + '<div class="flex items-center gap-2 mb-1">'
        +   '<span class="material-symbols-outlined text-base">' + c.icon + '</span>'
        +   '<span class="text-[10px] font-bold uppercase tracking-wide">' + c.label + '</span>'
        + '</div>'
        + '<div class="text-2xl font-black">' + c.val + '</div>'
        + '</div>';
    }).join('');
  }

  function setEqStatusFilter(val) {
    var sel = document.getElementById('eq-status-filter');
    if (sel) sel.value = val;
    filterEqStatusPage();
  }

  function _renderEqStatusTable() {
    var tbody = document.getElementById('eq-status-tbody');
    var countEl = document.getElementById('eq-status-count');
    if (!tbody) return;

    var filterVal = (document.getElementById('eq-status-filter') || {}).value || 'all';
    var query     = ((document.getElementById('eq-status-search') || {}).value || '').toLowerCase().trim();

    var filtered = _eqStatusData.filter(function(item) {
      var s = item['สถานะเครื่องมือ'] || '';
      var matchFilter = filterVal === 'all' || s === filterVal;
      var matchQuery  = !query
        || (item['ชื่อเครื่องมือ'] || '').toLowerCase().includes(query)
        || (item['รหัสครุภัณฑ์']  || '').toLowerCase().includes(query);
      return matchFilter && matchQuery;
    });

    // เรียงตามสถานะ (ไม่ผ่าน/เสีย/ซ่อม ขึ้นก่อน)
    var priority = {
      'สอบเทียบไม่ผ่านเกณฑ์': 1,
      'เครื่องมือเสีย':         2,
      'ส่งซ่อม':               3,
      'ระงับการใช้งาน':         4,
      'รอทวนสอบก่อนใช้':       4.5,
      'จำหน่ายออก':            5,
      'พร้อมใช้งาน':           6,
      '':                       7,
    };
    filtered.sort(function(a, b) {
      var pa = priority[a['สถานะเครื่องมือ'] || ''] || 9;
      var pb = priority[b['สถานะเครื่องมือ'] || ''] || 9;
      return pa - pb;
    });

    if (countEl) countEl.textContent = 'แสดง ' + filtered.length + ' จาก ' + _eqStatusData.length + ' รายการ';

    if (filtered.length === 0) {
      tbody.innerHTML = '<tr><td colspan="8" class="px-4 py-12 text-center text-slate-400">'
        + '<span class="material-symbols-outlined text-4xl block mb-2 text-slate-300">check_circle</span>'
        + 'ไม่พบรายการในเงื่อนไขที่เลือก</td></tr>';
      return;
    }

    var statusBadgeMap = {
      'สอบเทียบไม่ผ่านเกณฑ์': '<span class="inline-flex items-center gap-1 px-2 py-1 rounded-full text-[10px] font-bold bg-red-100 text-red-700 border border-red-200">⚠️ ไม่ผ่านเกณฑ์</span>',
      'เครื่องมือเสีย':         '<span class="inline-flex items-center gap-1 px-2 py-1 rounded-full text-[10px] font-bold bg-stone-100 text-stone-700 border border-stone-200">🔧 เครื่องเสีย</span>',
      'ส่งซ่อม':               '<span class="inline-flex items-center gap-1 px-2 py-1 rounded-full text-[10px] font-bold bg-orange-100 text-orange-700 border border-orange-200">🛠️ ส่งซ่อม</span>',
      'ระงับการใช้งาน':         '<span class="inline-flex items-center gap-1 px-2 py-1 rounded-full text-[10px] font-bold bg-violet-100 text-violet-700 border border-violet-200">🚫 ระงับการใช้</span>',
      'จำหน่ายออก':            '<span class="inline-flex items-center gap-1 px-2 py-1 rounded-full text-[10px] font-bold bg-slate-100 text-slate-600 border border-slate-200">📦 ปลดระวาง</span>',
      'พร้อมใช้งาน':           '<span class="inline-flex items-center gap-1 px-2 py-1 rounded-full text-[10px] font-bold bg-emerald-100 text-emerald-700 border border-emerald-200">✅ พร้อมใช้งาน</span>',
      'รอทวนสอบก่อนใช้':       '<span class="inline-flex items-center gap-1 px-2 py-1 rounded-full text-[10px] font-bold bg-sky-100 text-sky-700 border border-sky-200">⏳ รอทวนสอบ/อนุมัติ</span>',
    };

    var sendStatusMap = {
      'รอส่งการสอบเทียบ':     '<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-blue-100 text-blue-700">⏳ รอส่งสอบ</span>',
      'รับเครื่องส่งสอบเทียบ': '<span class="px-2 py-0.5 rounded-full text-[10px] font-bold bg-purple-100 text-purple-700">📦 อยู่ระหว่างสอบ</span>',
    };

    tbody.innerHTML = filtered.map(function(item, idx) {
      var eqStatus  = item['สถานะเครื่องมือ'] || '';
      var sendStatus = item['สถานะการส่งสอบ']  || '';
      var rowBg = idx % 2 === 0 ? '' : 'bg-slate-50/50 dark:bg-slate-700/10';
      var globalIdx = item._index !== undefined ? item._index : cachedEquipments.findIndex(function(e){ return e['ID'] === item['ID']; });

      return '<tr class="' + rowBg + ' hover:bg-primary-50/30 dark:hover:bg-slate-700/30 transition"'
        + ' data-name="' + (item['ชื่อเครื่องมือ']||'').toLowerCase() + '"'
        + ' data-code="' + (item['รหัสครุภัณฑ์']  ||'').toLowerCase() + '"'
        + ' data-status="' + eqStatus + '">'
        + '<td class="px-4 py-3 text-center text-xs text-slate-400">' + (idx + 1) + '</td>'
        + '<td class="px-4 py-3 font-mono text-xs font-bold text-primary-600 dark:text-primary-400 whitespace-nowrap">' + (item['รหัสครุภัณฑ์'] || '-') + '</td>'
        + '<td class="px-4 py-3">'
        +   '<div class="font-semibold text-sm text-slate-800 dark:text-slate-100">' + (item['ชื่อเครื่องมือ'] || '-') + '</div>'
        +   '<div class="text-[11px] text-slate-400 mt-0.5">' + (item['ประเภทเครื่องมือ'] || '') + '</div>'
        + '</td>'
        + '<td class="px-4 py-3 text-xs text-slate-500 whitespace-nowrap">' + (item['สถานที่ติดตั้ง'] || '-') + '</td>'
        + '<td class="px-4 py-3 text-center">'
        +   (statusBadgeMap[eqStatus] || '<span class="text-slate-300 text-xs">—</span>')
        + '</td>'
        + '<td class="px-4 py-3 text-center">'
        +   (sendStatusMap[sendStatus] || '<span class="text-slate-300 text-xs">—</span>')
        + '</td>'
        + '<td class="px-4 py-3 text-xs text-slate-500 max-w-[180px] truncate">'
        +   (item['หมายเหตุสถานะ'] || item['หมายเหตุ'] || '-')
        + '</td>'
        + '<td class="px-4 py-3 text-center">' + _eqStatusActionButtons(item, globalIdx) + '</td>'
        + '</tr>';
    }).join('');
  }

  function filterEqStatusPage() {
    _renderEqStatusTable();
  }
  function _escHtml(s) {
    return String(s).replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
  }

  // ============================================================
  // ✅ แก้ไขผลสอบเทียบ โดย "ย้อนกลับไปที่แบบฟอร์มกรอกข้อมูลเดิม"
  //    - เติมค่าจากแถวในชีตกลับเข้าแบบฟอร์ม (แยกบรรทัด → แถวจุดทดสอบ)
  //    - เรียกฟังก์ชันคำนวณของแต่ละหน้า ให้ผลยอมรับ/ปฏิเสธคำนวณใหม่อัตโนมัติ
  //    - กดบันทึก → เขียนทับแถวเดิม (ไม่เพิ่มแถวใหม่) + รีเซ็ตสถานะอนุมัติ
  // ตำแหน่งคอลัมน์ (index) ตรงกับลำดับที่ saveToSheetWithPdf() บันทึกลงชีต
  // ============================================================
  var CAL_EDIT_MAP = {
    pipette:     { p: 'pi',  box: 'pipette-rows-container',    add: 'addPipetteRow',     calc: 'calcPipetteRow',
                   head: { 0: 'eqSelect', 1: 'serialNo', 2: 'assetBrand', 3: 'calDate', 4: 'certNo', 5: 'capacity' },
                   rows: { 6: 'testVol', 7: 'meanVol', 8: 'uncertainty' }, ctrl: 10 },
    thermo:      { p: 'th',  box: 'thermo-rows-container',     add: 'addThermoRow',      calc: 'calcThermoRow',
                   radio: { 1: 'thermoType' },
                   head: { 0: 'eqSelect', 2: 'serialNo', 3: 'assetBrand', 4: 'calDate', 5: 'certNo' },
                   rows: { 6: 'testPoint', 7: 'stdTemp', 8: 'reading', 9: 'uncertainty' }, ctrl: 11 },
    centrifuge:  { p: 'ce',  box: 'centrifuge-rows-container', add: 'addCentrifugeRow',  calc: 'calcCentrifugeRow',
                   head: { 0: 'eqSelect', 1: 'serialNo', 2: 'assetBrand', 3: 'calDate', 4: 'certNo' },
                   rows: { 5: 'func', 6: 'setting', 7: 'std', 8: 'unc' }, ctrl: 10 },
    balance:     { p: 'ba',  box: 'balance-rows-container',    add: 'addBalanceRow',     calc: 'calcBalanceRow',
                   head: { 0: 'eqSelect', 1: 'serialNo', 2: 'assetBrand', 3: 'calDate', 4: 'certNo' },
                   rows: { 5: 'nom', 6: 'disp', 7: 'unc' }, ctrl: 9 },
    thermohygro: { p: 'thh', box: 'thh-rows-container',        add: 'addThermoHygroRow', calc: 'calcThermoHygroRow',
                   head: { 0: 'eqSelect', 1: 'serialNo', 2: 'assetBrand', 3: 'calDate', 4: 'certNo' },
                   rows: { 5: 'func', 6: 'std', 7: 'inst', 8: 'unc' }, ctrl: 10 },
    phmeter:     { p: 'ph',  box: 'ph-rows-container',         add: 'addPHRow',          calc: 'calcPHRow',
                   head: { 0: 'eqSelect', 1: 'serialNo', 2: 'assetBrand', 3: 'calDate', 4: 'certNo' },
                   rows: { 5: 'function', 6: 'stdRead', 7: 'instRead', 8: 'uncertainty' }, ctrl: 10 },
    oven:        { p: 'ov',  box: 'oven-rows-container',       add: 'addOvenRow',        calc: 'calcOvenRow',
                   radio: { 0: 'ovenType' }, otherText: 'ov-type-other-text',
                   head: { 1: 'eqSelect', 2: 'serialNo', 3: 'assetBrand', 4: 'calDate', 5: 'certNo' },
                   rows: { 6: 'read', 7: 'std', 8: 'unc' }, ctrl: 10 },
    ruler:       { p: 'ru',  box: 'ruler-rows-container',      add: 'addRulerRow',       calc: 'calcRulerRow',
                   head: { 0: 'eqSelect', 1: 'serialNo', 2: 'assetBrand', 3: 'calDate', 4: 'certNo' },
                   rows: { 5: 'nominalVal', 6: 'displayVal', 7: 'uncertainty' }, ctrl: 9 },
    pcr:         { p: 'pcr', box: 'pcr-rows-container',        add: 'addPcrRow',         calc: 'calcPcrRow',
                   head: { 0: 'eqSelect', 1: 'serialNo', 2: 'assetBrand', 3: 'calDate', 4: 'certNo' },
                   rows: { 5: 'tp', 6: 'mean', 7: 'min', 8: 'max' }, ctrl: null }
  };

  // เรียกจากทุก submitXxxForm() หลังสร้าง payload
  function _calEditInject(pageId, payload) {
    var ctx = window._calEditCtx;
    if (!ctx || ctx.pageId !== pageId) return;
    payload.editSheetRow = ctx.sheetRow;
    payload.editUsername = currentUser ? currentUser.username : '';
  }

  function _clearCalEditMode(pageId) {
    if (window._calEditCtx && (!pageId || window._calEditCtx.pageId === pageId)) window._calEditCtx = null;
    var b = document.getElementById('cal-edit-banner-' + pageId);
    if (b && b.parentNode) b.parentNode.removeChild(b);
  }

  function _calToIsoDate(s) {
    s = String(s || '').trim();
    if (!s) return '';
    if (/^\d{4}-\d{1,2}-\d{1,2}/.test(s)) {
      var a = s.slice(0, 10).split('-');
      return a[0] + '-' + ('0' + a[1]).slice(-2) + '-' + ('0' + a[2]).slice(-2);
    }
    var m = s.match(/^(\d{1,2})[\/.\-](\d{1,2})[\/.\-](\d{4})/); // d/m/yyyy (รองรับ พ.ศ.)
    if (m) {
      var y = parseInt(m[3], 10); if (y > 2400) y -= 543;
      return y + '-' + ('0' + m[2]).slice(-2) + '-' + ('0' + m[1]).slice(-2);
    }
    var d = new Date(s);
    return isNaN(d.getTime()) ? '' : d.toISOString().slice(0, 10);
  }

  function _calSetVal(el, v) {
    if (!el) return;
    v = v == null ? '' : String(v).trim();
    if (el.type === 'number') v = v.replace(/,/g, '');
    if (el.type === 'date') v = _calToIsoDate(v);
    el.value = v;
  }

  function _calSplitLines(v) {
    var arr = String(v == null ? '' : v).split(/\r?\n/).map(function (x) { return x.trim(); });
    while (arr.length && arr[arr.length - 1] === '') arr.pop();
    return arr;
  }

  function _editCalRow(pageId, idx) {
    var map  = CAL_EDIT_MAP[pageId];
    var hist = window._calHistoryData && window._calHistoryData[pageId];
    if (!map) { _editCalRowLegacy(pageId, idx); return; }     // หน้าที่ยังไม่มีแบบฟอร์มคำนวณ
    if (!hist) { showToast('ไม่พบข้อมูล กรุณารีเฟรช', 'danger'); return; }

    var headers = hist.headers, row = hist.data[idx];
    var sheetRow = row[headers.indexOf('_row')];
    if (!sheetRow) { showToast('ไม่พบเลขแถว กรุณากดรีเฟรชประวัติ', 'danger'); return; }
    var apIdx = headers.indexOf('สถานะการอนุมัติ');
    if (apIdx >= 0 && String(row[apIdx] || '') === 'อนุมัติแล้ว') { showToast('รายการนี้อนุมัติแล้ว ไม่สามารถแก้ไขได้', 'warning'); return; }

    var sel = document.getElementById(map.p + '-eqSelect');
    if (!sel) { _editCalRowLegacy(pageId, idx); return; }

    // ล้าง option ก่อน เพื่อใช้ตรวจว่า loadEquipmentsForXxx() โหลดเสร็จแล้ว
    sel.innerHTML = '';
    showLoader();
    openCalFormPanel(pageId);   // เปิดฟอร์มเดิม (reset + โหลดรายการเครื่องมือ)
    window._calEditCtx = { pageId: pageId, sheetRow: sheetRow, idx: idx };

    var t0 = Date.now();
    (function waitLoad() {
      if (sel.options.length > 0) { setTimeout(function () { _fillCalEditForm(pageId, map, headers, row, sheetRow); }, 60); return; }
      if (Date.now() - t0 > 20000) {
        hideLoader(); _clearCalEditMode(pageId);
        showToast('โหลดรายการเครื่องมือไม่สำเร็จ กรุณาลองใหม่', 'danger');
        return;
      }
      setTimeout(waitLoad, 150);
    })();
  }

  function _fillCalEditForm(pageId, map, headers, row, sheetRow) {
    try {
      var p = map.p;
      function cell(i) { return row[i] == null ? '' : String(row[i]); }

      // 1) ประเภท (radio) เช่น thermoType / ovenType
      Object.keys(map.radio || {}).forEach(function (ci) {
        var name = map.radio[ci], val = cell(ci).trim();
        if (!val) return;
        var radios = document.getElementsByName(name), hit = null, other = null;
        radios.forEach(function (r) { if (r.value === val) hit = r; if (r.value === 'Other') other = r; });
        var target = hit || other;
        if (!target) return;
        target.checked = true;
        target.dispatchEvent(new Event('change', { bubbles: true }));
        if (!hit && map.otherText) { var ot = document.getElementById(map.otherText); if (ot) ot.value = val; }
      });

      // 2) เลือกเครื่องมือ → trigger autoFill เดิมของหน้า
      var eqCol = Object.keys(map.head).filter(function (k) { return map.head[k] === 'eqSelect'; })[0];
      var eqId  = cell(eqCol).trim();
      var sel   = document.getElementById(p + '-eqSelect');
      var assetCol = Object.keys(map.head).filter(function (k) { return map.head[k] === 'assetBrand'; })[0];
      if (eqId && !Array.prototype.some.call(sel.options, function (o) { return o.value === eqId; })) {
        var op = document.createElement('option');
        op.value = eqId; op.textContent = eqId + ' | ' + (cell(assetCol) || '(จากประวัติ)');
        sel.appendChild(op);
      }
      sel.value = eqId;
      sel.dispatchEvent(new Event('change', { bubbles: true }));

      // 3) ข้อมูลส่วนหัว (ใช้ค่าตามที่บันทึกไว้ในประวัติ)
      Object.keys(map.head).forEach(function (ci) {
        var key = map.head[ci];
        if (key === 'eqSelect') return;
        var el = document.getElementById(p + '-' + key);
        _calSetVal(el, cell(ci));
        if (el) el.dispatchEvent(new Event('change', { bubbles: true }));
      });

      // 3.1) ห้องปฏิบัติการที่ทำการสอบเทียบ
      var labIdx = headers.indexOf('ห้องปฏิบัติการที่สอบเทียบ');
      var labSel = document.getElementById(p + '-cal-dept');
      if (labSel && labIdx >= 0 && cell(labIdx).trim()) {
        var lab = cell(labIdx).trim();
        if (!Array.prototype.some.call(labSel.options, function (o) { return o.value === lab; })) {
          var lo = document.createElement('option'); lo.value = lab; lo.textContent = lab; labSel.appendChild(lo);
        }
        labSel.value = lab;
        labSel.dispatchEvent(new Event('change', { bubbles: true }));
      }

      // 4) แถวจุดทดสอบ: แยกค่าหลายบรรทัด → สร้างแถว → ใส่ค่า → คำนวณใหม่
      var cols = {}, n = 0;
      Object.keys(map.rows).forEach(function (ci) { cols[ci] = _calSplitLines(cell(ci)); n = Math.max(n, cols[ci].length); });
      var ctrlLines = map.ctrl != null ? _calSplitLines(cell(map.ctrl)) : [];
      n = Math.max(n, 1);

      var box = document.getElementById(map.box);
      var addFn = window[map.add], calcFn = window[map.calc];
      if (!box || typeof addFn !== 'function') throw new Error('ไม่พบฟังก์ชันเพิ่มแถว ' + map.add);
      box.innerHTML = '';

      for (var i = 0; i < n; i++) {
        addFn();
        var items = box.querySelectorAll('.' + p + '-row-item');
        var tr = items[items.length - 1];
        if (!tr) continue;
        var rid = tr.id.replace(p + '-row-', '');
        Object.keys(map.rows).forEach(function (ci) {
          _calSetVal(document.getElementById(p + '-' + map.rows[ci] + '-' + rid), cols[ci][i] || '');
        });
        if (map.ctrl != null) {
          // รองรับ "5 - 10", "(5) - (10)", "(-20) - (-10)"
          var nums = String(ctrlLines[i] || '').match(/-?\d+(?:\.\d+)?/g) || [];
          _calSetVal(document.getElementById(p + '-ctrlMin-' + rid), nums[0] || '');
          _calSetVal(document.getElementById(p + '-ctrlMax-' + rid), nums[1] || '');
        }
        if (typeof calcFn === 'function') { try { calcFn(rid); } catch (e) { console.warn(e); } }
      }
      if (typeof _syncPcrPrintFields === 'function' && pageId === 'pcr') { try { _syncPcrPrintFields(); } catch (e) {} }

      // 5) แถบแจ้งโหมดแก้ไข
      var panel = document.getElementById('cal-panel-' + pageId);
      if (panel) {
        var old = document.getElementById('cal-edit-banner-' + pageId);
        if (old) old.parentNode.removeChild(old);
        var b = document.createElement('div');
        b.id = 'cal-edit-banner-' + pageId;
        b.className = 'print:hidden mb-3 flex items-center gap-3 flex-wrap px-4 py-3 rounded-xl border border-amber-300 bg-amber-50 text-amber-800 text-sm dark:bg-amber-900/20 dark:text-amber-300 dark:border-amber-700';
        b.innerHTML = '<span class="material-symbols-outlined">edit_note</span>'
          + '<div class="flex-1"><b>โหมดแก้ไขผลการสอบเทียบ</b> (แถวที่ ' + sheetRow + ' ในชีต) — '
          + 'แก้ค่าในตาราง ระบบจะคำนวณผลใหม่ให้อัตโนมัติ แล้วกดบันทึกเพื่อเขียนทับรายการเดิม'
          + '<div class="text-xs opacity-80 mt-0.5">ไม่แนบไฟล์ PDF ใหม่ = ใช้ไฟล์ใบรับรองเดิม · หลังบันทึกสถานะจะกลับเป็น "รออนุมัติ" และส่งอีเมลแจ้งหัวหน้าอีกครั้ง</div></div>'
          + '<button type="button" class="px-3 py-1.5 rounded-lg border border-amber-400 bg-white hover:bg-amber-100 text-amber-700 text-xs font-bold" '
          + 'onclick="closeCalFormPanel(\'' + pageId + '\')">ยกเลิกการแก้ไข</button>';
        panel.insertBefore(b, panel.firstChild);
        panel.scrollIntoView({ behavior: 'smooth', block: 'start' });
      }
      hideLoader();
    } catch (err) {
      hideLoader();
      console.error('[editCal] ', err);
      showToast('เติมข้อมูลลงแบบฟอร์มไม่สำเร็จ: ' + (err.message || err), 'danger');
    }
  }

  // (เดิม) modal textarea — ใช้เป็น fallback สำหรับหน้าที่ไม่มีแบบฟอร์มคำนวณ เช่น BSC/Fume Hood
  function _editCalRowLegacy(pageId, idx) {
    var hist = window._calHistoryData && window._calHistoryData[pageId];
    if (!hist) { showToast('ไม่พบข้อมูล กรุณารีเฟรช', 'danger'); return; }
    var headers = hist.headers, row = hist.data[idx];
    var sheetRow = row[headers.indexOf('_row')];
    if (!sheetRow) { showToast('ไม่พบเลขแถว กรุณากดรีเฟรชประวัติ', 'danger'); return; }

    var LOCKED = ['ผู้บันทึก','วันที่บันทึก','ไฟล์ใบรับรอง (PDF)','สถานะการอนุมัติ','ผู้อนุมัติ','วันที่อนุมัติ','หมายเหตุการแก้ไข','_row'];
    var fields = headers.map(function (h, i) {
      if (!h || LOCKED.indexOf(h) !== -1) return '';
      var v = row[i] == null ? '' : String(row[i]);
      var rows = Math.min(Math.max(v.split('\n').length, 1), 8);
      return '<div style="margin-bottom:10px;"><label style="display:block;font-size:12px;font-weight:600;color:#475569;margin-bottom:3px;">' + _escHtml(h) + '</label>'
        + '<textarea data-h="' + _escHtml(h) + '" rows="' + rows + '" style="width:100%;padding:6px 8px;border:1.5px solid #cbd5e1;border-radius:6px;font-size:13px;font-family:inherit;box-sizing:border-box;">' + _escHtml(v) + '</textarea></div>';
    }).join('');

    var ov = document.createElement('div');
    ov.style.cssText = 'position:fixed;inset:0;background:rgba(0,0,0,.6);z-index:9999;display:flex;align-items:center;justify-content:center;padding:16px;';
    ov.innerHTML = '<div style="background:#fff;border-radius:16px;width:100%;max-width:640px;max-height:90vh;display:flex;flex-direction:column;box-shadow:0 20px 60px rgba(0,0,0,.3);">'
      + '<div style="background:#b45309;color:#fff;padding:14px 20px;border-radius:16px 16px 0 0;display:flex;justify-content:space-between;align-items:center;">'
      +   '<div><div style="font-size:16px;font-weight:700;">✏️ แก้ไขผลการสอบเทียบ</div><div style="font-size:11px;opacity:.85;">แก้ไขได้จนกว่าหัวหน้าหน่วยจะอนุมัติ · ค่าหลายจุดทดสอบให้แยกบรรทัด</div></div>'
      +   '<button data-x style="background:none;border:none;color:#fff;font-size:22px;cursor:pointer;">✕</button></div>'
      + '<div style="overflow-y:auto;flex:1;padding:16px 20px;">'
      +   '<p style="font-size:11px;color:#b45309;background:#fffbeb;border:1px solid #fde68a;border-radius:6px;padding:6px 10px;margin:0 0 12px;">⚠️ หากแก้ค่าตัวเลข โปรดแก้ช่อง "ผลการตรวจสอบ" ให้สอดคล้องด้วย</p>'
      +   fields + '</div>'
      + '<div style="padding:12px 20px;border-top:1px solid #e2e8f0;display:flex;align-items:center;gap:10px;flex-wrap:wrap;">'
      +   '<label style="flex:1;font-size:12px;color:#475569;display:flex;align-items:center;gap:6px;"><input type="checkbox" data-mail checked> ส่งอีเมลแจ้งหัวหน้าหน่วยให้อนุมัติอีกครั้ง</label>'
      +   '<button data-x style="padding:8px 18px;border:1.5px solid #94a3b8;background:#fff;color:#64748b;border-radius:8px;font-size:13px;cursor:pointer;">ยกเลิก</button>'
      +   '<button data-save style="padding:8px 22px;border:none;background:#b45309;color:#fff;border-radius:8px;font-size:13px;font-weight:700;cursor:pointer;">💾 บันทึกการแก้ไข</button>'
      + '</div></div>';
    document.body.appendChild(ov);

    function close() { if (ov.parentNode) document.body.removeChild(ov); }
    ov.querySelectorAll('[data-x]').forEach(function (b) { b.onclick = close; });

    ov.querySelector('[data-save]').onclick = function () {
      var updates = {};
      ov.querySelectorAll('textarea[data-h]').forEach(function (t) {
        var h = t.getAttribute('data-h');
        var orig = row[headers.indexOf(h)]; orig = orig == null ? '' : String(orig);
        if (t.value !== orig) updates[h] = t.value;
      });
      if (!Object.keys(updates).length) { showToast('ไม่มีข้อมูลที่เปลี่ยนแปลง', 'info'); return; }
      var resend = ov.querySelector('[data-mail]').checked;
      close(); showLoader();
      google.script.run
        .withSuccessHandler(function (res) {
          hideLoader();
          if (!res || !res.success) { showToast('แก้ไขไม่สำเร็จ: ' + (res && res.message || ''), 'danger'); return; }
          var msg = '✅ แก้ไขแล้ว: ' + (res.changed || []).join(', ');
          if (res.mail) msg += res.mail.success ? '<br>📧 ส่งอีเมลถึง ' + (res.mail.recipients || []).join(', ') : '<br>⚠️ ส่งอีเมลไม่สำเร็จ: ' + res.mail.message;
          showToast(msg, res.mail && !res.mail.success ? 'warning' : 'success');
          loadCalHistory(pageId);
        })
        .withFailureHandler(function (err) { hideLoader(); showToast('ขัดข้อง: ' + (err.message || err), 'danger'); })
        .updateCalRecord({
          sheetName: CAL_SHEET_MAP[pageId], sheetRow: sheetRow, updates: updates,
          username: currentUser ? currentUser.username : '', resendEmail: resend
        });
    };
  }
</script>
