/**
 * 小柯攝影工作室｜詢問表單後端（Google Apps Script）
 * ------------------------------------------------------------
 * 功能：
 *   1. 接收網站表單送來的資料，新增一列到 Google 試算表
 *   2. 寄一封「新詢問通知」到你指定的 Email
 *
 * 安裝方式：請看專案 README.md 的「詢問表單串接 Google 試算表」。
 * 修改這支程式後，一定要「管理部署作業 → 編輯 → 版本選『新版本』→ 部署」，
 * 否則網站呼叫到的仍是舊版程式。
 */

// ===== 1. 設定區：只需要改這裡 =====
const CONFIG = {
  // 收通知的信箱，多個信箱用逗號分隔，例如 'a@gmail.com,b@gmail.com'
  NOTIFY_EMAIL: 'your-email@example.com',
  // 試算表中存放資料的工作表名稱（不存在會自動建立）
  SHEET_NAME: '詢問單',
  // 通知信上顯示的寄件者名稱
  SENDER_NAME: '小柯攝影工作室 網站表單',
  // 時區（通知信與試算表時間）
  TIMEZONE: 'Asia/Taipei',
};

// 表單欄位（要和 index.html 中每個欄位的 name 屬性一致）
const FIELDS = [
  { key: 'name',     label: '姓名',     max: 50 },
  { key: 'contact',  label: '聯絡方式', max: 100 },
  { key: 'type',     label: '拍攝類型', max: 30 },
  { key: 'date',     label: '預計日期', max: 20 },
  { key: 'location', label: '拍攝地點', max: 100 },
  { key: 'budget',   label: '預算',     max: 30 },
  { key: 'message',  label: '留言',     max: 2000 },
];
const REQUIRED = ['name', 'contact', 'type'];
const HEADERS = ['送出時間'].concat(FIELDS.map(f => f.label), ['處理狀態', '備註']);


// ===== 2. 網站表單送出時會執行這個函式 =====
function doPost(e) {
  try {
    const p = (e && e.parameter) || {};

    // 防垃圾訊息：隱藏欄位 website 被填寫，代表是機器人 → 假裝成功但不記錄
    if (p.website) return json_({ ok: true });

    // 整理資料：去除前後空白、限制長度
    const data = {};
    FIELDS.forEach(f => { data[f.key] = String(p[f.key] || '').trim().slice(0, f.max); });

    const missing = REQUIRED.filter(k => !data[k]);
    if (missing.length) return json_({ ok: false, error: '缺少必填欄位' });

    // 寫入試算表（加鎖，避免同時多筆送出時互相覆蓋）
    const lock = LockService.getScriptLock();
    lock.waitLock(10000);
    try {
      const row = [new Date()].concat(FIELDS.map(f => safeCell_(data[f.key])), ['未回覆', '']);
      getSheet_().appendRow(row);
    } finally {
      lock.releaseLock();
    }

    // 寄通知信：就算寄信失敗，資料也已經存進試算表，不會遺失
    try {
      sendNotification_(data);
    } catch (mailErr) {
      console.error('通知信寄送失敗：', mailErr);
    }

    return json_({ ok: true });
  } catch (err) {
    console.error(err);
    return json_({ ok: false, error: '系統忙碌，請稍後再試' });
  }
}

// 用瀏覽器直接打開網址時顯示，方便確認部署成功
function doGet() {
  return ContentService.createTextOutput('小柯攝影工作室 詢問表單服務運作中 ✓');
}


// ===== 3. 安裝時手動執行一次：授權 + 建立表頭 + 寄測試信 =====
function setupAndTest() {
  getSheet_();
  sendNotification_({
    name: '測試客戶', contact: 'test@example.com', type: '婚紗攝影',
    date: '2026-12-31', location: '臺中市西屯區', budget: 'NT$25,000 – 40,000',
    message: '這是一封測試信。收到代表通知功能設定成功！',
  });
  console.log('完成！請到信箱確認是否收到測試信：' + CONFIG.NOTIFY_EMAIL);
}


// ===== 以下為內部工具函式 =====

function getSheet_() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  let sheet = ss.getSheetByName(CONFIG.SHEET_NAME);
  if (!sheet) sheet = ss.insertSheet(CONFIG.SHEET_NAME);
  if (sheet.getLastRow() === 0) {
    sheet.appendRow(HEADERS);
    sheet.getRange(1, 1, 1, HEADERS.length)
      .setFontWeight('bold').setBackground('#2b2622').setFontColor('#ffffff');
    sheet.setFrozenRows(1);
    sheet.getRange('A:A').setNumberFormat('yyyy/mm/dd hh:mm');
    sheet.getRange(1, HEADERS.indexOf('預計日期') + 1, sheet.getMaxRows(), 1)
      .setNumberFormat('@');                               // 預計日期以文字保存
    sheet.setColumnWidth(HEADERS.indexOf('留言') + 1, 360);
    // 「處理狀態」做成下拉選單
    const statusCol = HEADERS.indexOf('處理狀態') + 1;
    const rule = SpreadsheetApp.newDataValidation()
      .requireValueInList(['未回覆', '已回覆', '已報價', '已成交', '未成交'], true).build();
    sheet.getRange(2, statusCol, sheet.getMaxRows() - 1, 1).setDataValidation(rule);
  }
  return sheet;
}

function sendNotification_(d) {
  const when = Utilities.formatDate(new Date(), CONFIG.TIMEZONE, 'yyyy/MM/dd HH:mm');
  const subject = `【新詢問】${d.type}｜${d.name}｜${d.date || '日期未定'}`;

  const rows = FIELDS.map(f => {
    const v = d[f.key] ? escape_(d[f.key]).replace(/\n/g, '<br>') : '<span style="color:#999">（未填）</span>';
    return `<tr><th style="text-align:left;padding:8px 12px;background:#f6f2ec;width:90px;vertical-align:top">${f.label}</th>` +
           `<td style="padding:8px 12px;border-bottom:1px solid #eee">${v}</td></tr>`;
  }).join('');

  const sheetUrl = SpreadsheetApp.getActiveSpreadsheet().getUrl();
  const htmlBody =
    `<div style="font-family:sans-serif;color:#2b2622;max-width:560px">` +
    `<h2 style="font-size:18px">網站收到一筆新的詢問</h2>` +
    `<p style="color:#6b625a;font-size:13px">送出時間：${when}</p>` +
    `<table style="border-collapse:collapse;width:100%;font-size:14px">${rows}</table>` +
    `<p style="margin-top:20px"><a href="${sheetUrl}" style="color:#9a6b4f">打開詢問單試算表 →</a></p>` +
    `</div>`;
  const textBody = FIELDS.map(f => `${f.label}：${d[f.key] || '（未填）'}`).join('\n') +
                   `\n\n送出時間：${when}\n試算表：${sheetUrl}`;

  const options = { name: CONFIG.SENDER_NAME, htmlBody: htmlBody };
  // 客戶留的是 Email 時，直接按「回覆」就能回信給客戶
  if (/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(d.contact)) options.replyTo = d.contact;

  MailApp.sendEmail(CONFIG.NOTIFY_EMAIL, subject, textBody, options);
}

// 防止有人輸入 =IMPORTXML(...) 之類的公式攻擊試算表
function safeCell_(v) {
  return /^[=+\-@]/.test(v) ? "'" + v : v;
}

function escape_(s) {
  return String(s).replace(/[&<>"']/g, c =>
    ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c]));
}

function json_(obj) {
  return ContentService.createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}
