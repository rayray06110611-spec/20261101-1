# 小黑攝影工作室｜臺中婚紗婚禮紀錄

純靜態網站（單一 `index.html`，HTML + CSS 內嵌），無需建置，可直接部署到 Vercel。

## 專案結構

```
.
├── index.html
├── vercel.json
├── README.md
└── images/
    ├── hero-01.jpg
    ├── about-01.jpg
    ├── wedding-01.jpg ~ wedding-08.jpg      （婚紗 8 張）
    ├── ceremony-01.jpg ~ ceremony-06.jpg    （婚禮紀錄 6 張）
    └── portrait-01.jpg ~ portrait-06.jpg    （形象照 6 張）
```

## 換成真實照片（免費圖庫）

目前 `images/` 內是程式產生的**示意圖**。請從以下免費圖庫下載照片，
**以相同檔名覆蓋**即可（`index.html` 不用改）：

- Unsplash：https://unsplash.com/s/photos/wedding
- Pexels：https://www.pexels.com/search/wedding/
- Pixabay：https://pixabay.com/images/search/wedding/

建議：橫式 3:2、寬 1200–1600px、單張 < 300KB。
若你用自己拍的作品，請注意授權與客戶肖像同意。

## 需要替換的占位資訊

在 `index.html` 搜尋並修改：

| 項目 | 搜尋字串 |
|---|---|
| Email | `hello@example.com`（頁尾、表單，共數處） |
| Instagram | `your_instagram` |
| 網站網址 | `your-domain.vercel.app`（canonical） |
| 價格、案例 | 皆為示意內容 |

## 部署到 GitHub + Vercel

```bash
git init
git add .
git commit -m "init: portfolio site"
git branch -M main
git remote add origin https://github.com/<你的帳號>/<repo>.git
git push -u origin main
```

1. 登入 https://vercel.com → **Add New → Project** → 匯入該 GitHub repo。
2. Framework Preset 選 **Other**，Build Command 與 Output Directory 留空。
3. 按 **Deploy**。之後每次 push 都會自動更新。

## 表單說明

目前表單送出時會以 `mailto:` 開啟訪客的郵件軟體（零後端、可立即使用）。
若想直接收到通知信，可改用 Formspree：

1. 到 https://formspree.io 建立表單，取得 endpoint。
2. 將 `<form>` 的 `action` 改為該 endpoint、`method="post"`，並移除頁尾 `<script>` 內表單攔截的程式碼。
