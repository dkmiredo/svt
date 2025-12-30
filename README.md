<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SVT 追星生活手帳 💎</title>
    
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Kiwi+Maru:wght@400;500&family=Noto+Sans+TC:wght@500&display=swap" rel="stylesheet">
    <link href="https://cdn.jsdelivr.net/npm/fullcalendar@6.1.8/index.global.min.css" rel="stylesheet">
    
    <style>
        :root { --rose-quartz: #F7CAC9; --serenity: #92A8D1; }
        body {
            background: linear-gradient(180deg, var(--rose-quartz) 0%, var(--serenity) 100%);
            background-attachment: fixed; min-height: 100vh;
            font-family: 'Kiwi Maru', 'Noto Sans TC', sans-serif;
            color: #555; overflow-x: hidden;
        }
        .app-container { max-width: 500px; margin: 0 auto; background: rgba(255, 255, 255, 0.4); backdrop-filter: blur(10px); min-height: 100vh; position: relative; }
        
        /* 頁面基本樣式 */
        .page-section { display: none; padding: 20px; animation: fadeIn 0.3s ease; padding-bottom: 120px; }
        .page-section.active { display: block; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }

        /* 公用組件 */
        .section-header { background-color: var(--serenity); color: white; padding: 6px 18px; border-radius: 50px; display: inline-block; font-size: 0.9rem; font-weight: bold; margin-bottom: 10px; box-shadow: 0 4px 10px rgba(146, 168, 209, 0.3); }
        .svt-card { background: rgba(255, 255, 255, 0.9); border-radius: 15px; padding: 15px; margin-bottom: 12px; border: 1px solid rgba(146, 168, 209, 0.2); position: relative; }
        .btn-svt { background: var(--serenity); color: white; border-radius: 50px; border: none; padding: 8px 20px; font-weight: bold; width: 100%; }
        .text-serenity { color: var(--serenity) !important; }

        /* 編輯與重整按鈕 */
        .edit-icon { position: absolute; top: 12px; right: 12px; color: var(--serenity); cursor: pointer; font-size: 0.9rem; opacity: 0.6; }
        .reset-btn { font-size: 0.75rem; color: #ddd; cursor: pointer; margin-left: 8px; transition: 0.3s; }
        .reset-btn:hover { color: var(--rose-quartz); transform: rotate(90deg); }

        /* 記帳多選成員 */
        .member-checkbox-group { display: flex; flex-wrap: wrap; gap: 8px; background: #fdfdfd; padding: 12px; border-radius: 12px; border: 1px solid #eee; max-height: 160px; overflow-y: auto; }
        .member-tag { font-size: 0.75rem; padding: 4px 12px; border-radius: 20px; border: 1px solid var(--serenity); color: var(--serenity); cursor: pointer; }
        .member-tag.active { background: var(--serenity); color: white; }

        /* 導覽列 */
        .bottom-nav { position: fixed; bottom: 0; left: 50%; transform: translateX(-50%); width: 100%; max-width: 500px; background: white; display: flex; justify-content: space-around; align-items: center; padding: 8px 0; border-top: 1px solid #eee; z-index: 1000; height: 75px; }
        .nav-item { color: #ccc; text-align: center; flex: 1; cursor: pointer; font-size: 0.75rem; }
        .nav-item i { font-size: 1.2rem; display: block; }
        .nav-item.active { color: var(--serenity); }
        .nav-avatar-btn { width: 55px; height: 55px; border-radius: 50%; border: 3px solid white; background: var(--serenity); display: flex; align-items: center; justify-content: center; color: white; font-weight: bold; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }

        /* 主頁頭像與評分 */
        .profile-header { height: 160px; background: #eee center/cover; position: relative; border-bottom: 4px solid white; cursor: pointer; }
        .profile-avatar { width: 90px; height: 90px; border-radius: 50%; border: 4px solid white; position: absolute; bottom: -45px; left: 20px; background: white; object-fit: cover; z-index: 10; }
        .member-avatar-sm { width: 38px; height: 38px; border-radius: 50%; margin-right: 12px; border: 1.5px solid white; background: linear-gradient(135deg, var(--rose-quartz), var(--serenity)); display: flex; align-items: center; justify-content: center; flex-shrink: 0; overflow: hidden; cursor: pointer; }
        .member-avatar-sm img { width: 100%; height: 100%; object-fit: cover; }
        .hand-drawn-diamond { width: 22px; height: 22px; stroke: white; fill: none; stroke-width: 1.8; }
        .gem-rating { font-size: 0.85rem; color: #eee; cursor: pointer; transition: 0.2s; }
        .gem-rating.active { color: var(--serenity); }

        /* 彈窗樣式 */
        .member-detail-modal { position: fixed; top: 0; left: 50%; transform: translateX(-50%); width: 100%; max-width: 500px; height: 100%; background: #fff; z-index: 2000; display: none; overflow-y: auto; padding: 25px; }
        .mini-pc { width: 80px; height: 120px; object-fit: cover; border-radius: 8px; margin: 4px; border: 1px solid #eee; }
        .sync-fin-item { font-size: 0.8rem; background: #f0f4f8; padding: 5px 10px; border-left: 3px solid var(--serenity); margin-top: 5px; border-radius: 4px; }
    </style>
</head>
<body>

<div class="app-container">
    <input type="file" id="member-file-input" style="display:none;" accept="image/*">
    <input type="file" id="bg-trigger" style="display:none;" accept="image/*" onchange="handleImageUpload('bg')">
    <input type="file" id="av-trigger" style="display:none;" accept="image/*" onchange="handleImageUpload('avatar')">

    <section id="page-finance" class="page-section">
        <div class="section-header">記帳</div>
        <div class="svt-card" id="finance-form">
            <input type="hidden" id="edit-fin-id" value="">
            <label class="small text-muted mb-1">日期</label>
            <input type="date" id="fin-date" class="form-control mb-3">
            <label class="small text-muted mb-1">成員 (多選)</label>
            <div id="fin-member-multi" class="member-checkbox-group mb-3"></div>
            <label class="small text-muted mb-1">類別</label>
            <select id="fin-category" class="form-select mb-3">
                <option value="專輯">專輯</option><option value="小卡">小卡</option><option value="周邊">周邊</option><option value="其他">其他</option>
            </select>
            <label class="small text-muted mb-1">金額</label>
            <div class="input-group mb-3">
                <input type="number" id="fin-amt" class="form-control" placeholder="輸入金額">
                <select id="fin-currency" class="form-select" style="max-width: 90px;">
                    <option value="TWD">台幣</option><option value="KRW">韓元</option><option value="JPY">日幣</option>
                </select>
            </div>
            <button class="btn btn-svt" id="btn-save-finance" onclick="addFinance()">儲存</button>
        </div>
        <div id="list-finance"></div>
    </section>

    <section id="page-diary" class="page-section">
        <div class="section-header">追星日記</div>
        <div id="calendar" class="bg-white p-2 rounded mb-3"></div>
        <div class="svt-card">
            <input type="date" id="diary-date" class="form-control mb-2">
            <textarea id="diary-content" class="form-control mb-2" rows="3" placeholder="今天發生了什麼事？"></textarea>
            <button class="btn btn-svt" onclick="addDiary()">儲存日記</button>
        </div>
        <div id="day-detail-area">
            <div class="section-header">當日紀錄預覽</div>
            <div id="selected-day-content" class="svt-card">請點擊日曆日期查看</div>
        </div>
    </section>

    <section id="page-profile" class="page-section active" style="padding:0;">
        <div id="prof-bg" class="profile-header" onclick="document.getElementById('bg-trigger').click()">
            <img id="prof-avatar-img" src="" class="profile-avatar" onclick="event.stopPropagation(); document.getElementById('av-trigger').click()">
        </div>
        <div class="p-4 bg-white shadow-sm mb-3" style="padding-top: 50px !important;">
            <div class="d-flex justify-content-between align-items-center">
                <div><h4 id="display-name" class="fw-bold m-0 text-serenity">Carat</h4><p id="display-status" class="text-muted small mb-0">Say the name!</p></div>
                <button class="btn btn-sm btn-outline-secondary rounded-pill px-3" onclick="toggleEdit()">編輯</button>
            </div>
            <div id="edit-fields" style="display:none;" class="mt-3">
                <input type="text" id="input-name" class="form-control mb-2" placeholder="暱稱">
                <input type="text" id="input-status" class="form-control mb-2" placeholder="狀態消息">
                <button class="btn btn-svt btn-sm" onclick="saveProfileText()">儲存資料</button>
            </div>
        </div>
        <div class="px-3">
            <div class="section-header">成員列表</div>
            <div id="member-ratings"></div>
            <div style="height: 50px;"></div>
        </div>
    </section>

    <section id="page-translate" class="page-section"><div class="section-header">翻譯</div><iframe src="https://translate.google.com/m?sl=ko&tl=zh-TW" style="width:100%; height:500px; border-radius:15px; border:none;"></iframe></section>
    <section id="page-album" class="page-section">
        <div class="section-header">卡冊</div>
        <div class="svt-card">
            <select id="pc-member-select" class="form-select mb-2"></select>
            <input type="file" id="pc-file" class="form-control mb-2" accept="image/*">
            <button class="btn btn-svt" onclick="addPC()">加入收藏</button>
        </div>
        <div id="list-pc" class="row g-2"></div>
    </section>

    <div id="memberModal" class="member-detail-modal">
        <span class="float-end fs-2" onclick="closeMemberModal()" style="cursor:pointer;">&times;</span>
        <h3 id="modal-member-name" class="fw-bold text-serenity mt-4"></h3>
        <hr>
        <div id="modal-content"></div>
    </div>

    <nav class="bottom-nav">
        <div class="nav-item" onclick="showPage('finance', this)"><i class="fas fa-wallet"></i><span>記帳</span></div>
        <div class="nav-item" onclick="showPage('diary', this)"><i class="fas fa-book"></i><span>日記</span></div>
        <div class="nav-item active" onclick="showPage('profile', this)" style="margin-top:-25px;"><div id="nav-avatar-container" class="nav-avatar-btn">主頁</div></div>
        <div class="nav-item" onclick="showPage('translate', this)"><i class="fas fa-language"></i><span>翻譯</span></div>
        <div class="nav-item" onclick="showPage('album', this)"><i class="fas fa-images"></i><span>卡冊</span></div>
    </nav>
</div>

<script src="https://cdn.jsdelivr.net/npm/fullcalendar@6.1.8/index.global.min.js"></script>
<script>
    const members = ["S.COUPS", "JEONGHAN", "JOSHUA", "JUN", "HOSHI", "WONWOO", "WOOZI", "THE 8", "MINGYU", "DK", "SEUNGKWAN", "VERNON", "DINO"];
    const KEY = 'svt_v72_';
    const getL = k => JSON.parse(localStorage.getItem(KEY+k)) || (k.includes('list')?[]:{});
    const setL = (k, v) => localStorage.setItem(KEY+k, JSON.stringify(v));

    let calendar;
    let targetMember = "";

    // --- 記帳功能 ---
    function initFinanceMembers() {
        document.getElementById('fin-member-multi').innerHTML = members.map(m => `<span class="member-tag" onclick="this.classList.toggle('active')">${m}</span>`).join('');
        document.getElementById('pc-member-select').innerHTML = members.map(m => `<option value="${m}">${m}</option>`).join('');
    }

    function addFinance() {
        const editId = document.getElementById('edit-fin-id').value;
        const date = document.getElementById('fin-date').value;
        const amt = document.getElementById('fin-amt').value;
        const selected = Array.from(document.querySelectorAll('#fin-member-multi .member-tag.active')).map(el => el.innerText);
        if(!date || !amt || selected.length === 0) return alert('請填寫完整資訊！');
        
        let list = getL('fin_list');
        const data = { id: editId ? Number(editId) : Date.now(), date, members: selected, category: document.getElementById('fin-category').value, amt, currency: document.getElementById('fin-currency').value };
        
        if (editId) list = list.map(i => i.id == editId ? data : i);
        else list.unshift(data);
        
        setL('fin_list', list);
        renderFinance();
        alert('已儲存！');
        resetFinanceForm();
    }

    function resetFinanceForm() {
        document.getElementById('edit-fin-id').value = "";
        document.getElementById('fin-amt').value = '';
        document.querySelectorAll('#fin-member-multi .member-tag').forEach(el => el.classList.remove('active'));
        document.getElementById('btn-save-finance').innerText = "儲存";
    }

    function renderFinance() {
        const list = getL('fin_list');
        document.getElementById('list-finance').innerHTML = list.map(i => `
            <div class="svt-card d-flex justify-content-between align-items-center">
                <i class="fas fa-pencil-alt edit-icon" onclick="editFinance(${i.id})"></i>
                <div>
                    <div class="small text-muted">${i.date} · ${i.category}</div>
                    <div class="fw-bold text-serenity">${i.members.join(', ')}</div>
                </div>
                <div class="fw-bold text-dark text-end">${i.currency} ${Number(i.amt).toLocaleString()}</div>
            </div>
        `).join('');
    }

    function editFinance(id) {
        const item = getL('fin_list').find(i => i.id == id);
        if(!item) return;
        document.getElementById('edit-fin-id').value = item.id;
        document.getElementById('fin-date').value = item.date;
        document.getElementById('fin-category').value = item.category;
        document.getElementById('fin-amt').value = item.amt;
        document.getElementById('fin-currency').value = item.currency;
        document.querySelectorAll('#fin-member-multi .member-tag').forEach(tag => {
            if(item.members.includes(tag.innerText)) tag.classList.add('active');
            else tag.classList.remove('active');
        });
        document.getElementById('btn-save-finance').innerText = "更新紀錄";
        showPage('finance', document.querySelector('.nav-item'));
    }

    // --- 日記功能 ---
    function addDiary() {
        const date = document.getElementById('diary-date').value;
        const content = document.getElementById('diary-content').value;
        if(!date || !content) return alert('請輸入內容！');
        const list = getL('diary_list');
        const idx = list.findIndex(d => d.date === date);
        if(idx > -1) list[idx].content = content; else list.push({ date, content });
        setL('diary_list', list);
        alert('日記儲存成功！');
        refreshDayDetail(date);
    }

    function refreshDayDetail(date) {
        const diary = getL('diary_list').find(d => d.date === date);
        const fins = getL('fin_list').filter(f => f.date === date);
        let html = `<div class="fw-bold text-serenity mb-2">${date}</div>`;
        if(diary) html += `<i class="fas fa-pencil-alt edit-icon" onclick="editDiaryFromPreview('${date}')"></i>`;
        html += `<div class="mb-3">${diary ? diary.content : '<span class="text-muted">本日無日記內容</span>'}</div>`;
        if(fins.length > 0) {
            html += `<hr><div class="small fw-bold text-muted mb-1">💰 本日開銷：</div>`;
            fins.forEach(f => html += `<div class="sync-fin-item" onclick="editFinance(${f.id})">[${f.category}] ${f.members.join(', ')}：${f.currency} ${f.amt}</div>`);
        }
        document.getElementById('selected-day-content').innerHTML = html;
    }

    function editDiaryFromPreview(date) {
        const d = getL('diary_list').find(i => i.date === date);
        document.getElementById('diary-date').value = d.date;
        document.getElementById('diary-content').value = d.content;
    }

    // --- 卡冊功能 ---
    async function addPC() {
        const file = document.getElementById('pc-file').files[0];
        if(!file) return alert('請選擇照片');
        const base64 = await compressImage(file, 600, 0.6);
        const list = getL('pc_list');
        list.unshift({ id: Date.now(), mem: document.getElementById('pc-member-select').value, img: base64 });
        setL('pc_list', list); renderPC();
    }
    function renderPC() {
        document.getElementById('list-pc').innerHTML = getL('pc_list').map(i => `<div class="col-4"><img src="${i.img}" class="img-fluid rounded shadow-sm"></div>`).join('');
    }

    // --- 主頁詳細資料彈窗邏輯 ---
    function openMemberModal(name) {
        const fin = getL('fin_list').filter(i => i.members.includes(name));
        const pcs = getL('pc_list').filter(i => i.mem === name);
        const diaries = getL('diary_list').filter(i => i.content.includes(name));
        
        document.getElementById('modal-member-name').innerText = name;
        let html = `<p class="small text-muted">與 ${name} 相關的所有紀錄：</p>`;
        
        html += `<div class="fw-bold mt-3 text-serenity">💰 支出</div>` + (fin.length ? fin.map(f => `<div class="small py-1 border-bottom d-flex justify-content-between"><span>${f.date} ${f.category}</span><span>${f.currency} ${f.amt}</span></div>`).join('') : '<p class="small text-muted">暫無支出紀錄</p>');
        html += `<div class="fw-bold mt-3 text-serenity">📷 小卡</div><div class="d-flex flex-wrap">` + (pcs.length ? pcs.map(p => `<img src="${p.img}" class="mini-pc">`).join('') : '<p class="small text-muted">暫無小卡</p>') + `</div>`;
        html += `<div class="fw-bold mt-3 text-serenity">✍️ 日記提及</div>` + (diaries.length ? diaries.map(d => `<div class="p-2 bg-light rounded mb-2 small"><b>${d.date}</b>: ${d.content}</div>`).join('') : '<p class="small text-muted">尚未在日記提到他</p>');
        
        document.getElementById('modal-content').innerHTML = html;
        document.getElementById('memberModal').style.display = 'block';
    }
    function closeMemberModal() { document.getElementById('memberModal').style.display = 'none'; }

    // --- 主頁評分與頭像渲染 ---
    function renderProfile() {
        const info = getL('profile'), pics = getL('member_pics'), ratings = getL('ratings');
        document.getElementById('display-name').innerText = info.name || "Carat";
        document.getElementById('display-status').innerText = info.status || "Say the name!";
        const avImg = document.getElementById('prof-avatar-img'), navC = document.getElementById('nav-avatar-container');
        
        if(info.avatar) {
            avImg.src = info.avatar;
            navC.innerHTML = `<img src="${info.avatar}" style="width:100%; height:100%; border-radius:50%; object-fit:cover;">`;
        } else {
            navC.innerText = "主頁";
        }
        if(info.bg) document.getElementById('prof-bg').style.backgroundImage = `url(${info.bg})`;

        document.getElementById('member-ratings').innerHTML = members.map(m => {
            const r = ratings[m] || 0;
            return `
            <div class="svt-card d-flex align-items-center justify-content-between py-2">
                <div class="d-flex align-items-center">
                    <div class="member-avatar-sm" onclick="triggerMemberPic('${m}')">
                        ${pics[m] ? `<img src="${pics[m]}">` : `<svg class="hand-drawn-diamond" viewBox="0 0 24 24"><path d="M12 3 L18 8 L12 21 L6 8 Z M6 8 L18 8 M9 3 L9 8 M15 3 L15 8 M9 8 L12 21 L15 8" /></svg>`}
                    </div>
                    <span class="fw-bold text-serenity small" onclick="openMemberModal('${m}')" style="cursor:pointer;">${m}</span>
                </div>
                <div class="d-flex align-items-center">
                    ${[1,2,3,4,5].map(n => `<i class="fas fa-gem gem-rating ${r >= n ? 'active' : ''} mx-1" onclick="rate('${m}', ${n})"></i>`).join('')}
                    <i class="fas fa-sync-alt reset-btn" onclick="rate('${m}', 0)"></i>
                </div>
            </div>`;
        }).join('');
    }

    // --- 圖片上傳邏輯 ---
    async function handleImageUpload(type) {
        const file = document.getElementById(type === 'bg' ? 'bg-trigger' : 'av-trigger').files[0];
        if (!file) return;
        const base64 = await compressImage(file, type === 'bg' ? 800 : 300, 0.5);
        const info = getL('profile');
        info[type] = base64; setL('profile', info); renderProfile();
    }
    function triggerMemberPic(m) { targetMember = m; document.getElementById('member-file-input').click(); }
    document.getElementById('member-file-input').onchange = async function(e) {
        const base64 = await compressImage(e.target.files[0], 200, 0.5);
        const pics = getL('member_pics');
        pics[targetMember] = base64; setL('member_pics', pics); renderProfile();
    };

    function compressImage(file, maxWidth, quality) {
        return new Promise(res => {
            const reader = new FileReader(); reader.readAsDataURL(file);
            reader.onload = e => {
                const img = new Image(); img.src = e.target.result;
                img.onload = () => {
                    const canvas = document.createElement('canvas');
                    let w = img.width, h = img.height;
                    if (w > maxWidth) { h = (maxWidth / w) * h; w = maxWidth; }
                    canvas.width = w; canvas.height = h;
                    canvas.getContext('2d').drawImage(img, 0, 0, w, h);
                    res(canvas.toDataURL('image/jpeg', quality));
                };
            };
        });
    }

    // --- 換頁邏輯 ---
    function showPage(p, el) {
        document.querySelectorAll('.page-section').forEach(s => s.classList.remove('active'));
        document.getElementById('page-'+p).classList.add('active');
        document.querySelectorAll('.nav-item').forEach(n => n.classList.remove('active'));
        el.classList.add('active');
        window.scrollTo(0,0);
        if(p==='finance') renderFinance();
        if(p==='album') renderPC();
    }

    function rate(m, n) { const r = getL('ratings'); r[m]=n; setL('ratings', r); renderProfile(); }
    function toggleEdit() { const d = document.getElementById('edit-fields'); d.style.display = d.style.display==='none'?'block':'none'; }
    function saveProfileText() {
        const info = getL('profile');
        info.name = document.getElementById('input-name').value || info.name;
        info.status = document.getElementById('input-status').value || info.status;
        setL('profile', info); renderProfile(); toggleEdit();
    }

    window.onload = () => {
        initFinanceMembers();
        renderProfile();
        const today = new Date().toISOString().split('T')[0];
        document.getElementById('fin-date').value = today;
        document.getElementById('diary-date').value = today;
        calendar = new FullCalendar.Calendar(document.getElementById('calendar'), {
            initialView: 'dayGridMonth', height: 'auto',
            dateClick: (info) => { document.getElementById('diary-date').value = info.dateStr; refreshDayDetail(info.dateStr); }
        });
        calendar.render();
    };
</script>
</body>
</html>
