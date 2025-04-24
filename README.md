# 📡 GCP VM 架設 WireGuard VPN 全流程指南

本教學適用於：

- ✅ Google Cloud Platform（GCP）
- ✅ Ubuntu / Debian（GCP預設） 系統
- ✅ 目標：讓手機與筆電透過 VPN 安全連線、取得 GCP 的出口 IP（如翻牆、訪問國外網站）

---

## ✅ 0. 前置準備

- 已註冊 GCP 帳號並啟用計費（可用 Free Tier）
- 已建立專案（Project）
- 啟用 Compute Engine API

---

## 🚀 1. 建立 VM 虛擬機

1. 進入 GCP Console → Compute Engine → VM instances → Create instance
2. 低成本建議設定：
   - 名稱：`vpn`
   - 區域（Region）：如 `us-central1`（或你選的區域）
   - 機器類型：下面Machine type選`e2-micro`（Free Tier）（預估價格不會即時反映 Free Tier 折扣（200GB免費），待我適用，不過新戶有免費額度可用）
   - 開機磁碟：10GB 標準磁碟即可（免費版需<=30GB)
     左欄OS and storage -> 按change按鈕 -> Boot disk type 改成 `standard` persistent disk
   - 關掉右側計費欄中的Logging、Monitoring
   - 點左欄Data protection區域：Back up your data中選No backups以關掉快照
   - IP要用Ephemeral（臨時IP），預設即是，在左欄Networking區域：Network interfaces裡Primary internal IPv4 address可查看
4. 建立 VM

---

## 🔥 2. SSH 登入 VM 安裝 WireGuard
`可點建好的VM SSH路徑 呼叫GVP terminal`
優勢：有一鍵腳本，穩定高速（很省資源），加密等級高（使用現代加密技術 Curve25519）
📋 腳本特色（作者：Angristan，社群公認穩定）
✅ 自動安裝 WireGuard
✅ 自動生成用戶設定檔（含 QR Code）
✅ 自動開放防火牆（UDP 51820）
✅ 支援多用戶帳號新增
✅ 輕量、超穩定、適合手機筆電一起用

```bash
curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
chmod +x wireguard-install.sh
sudo ./wireguard-install.sh
```

> 安裝過程中會問一些問題：
>
> - Client WireGuard IPv4要改成外部IP
> - Interface 名稱：`wg0`（預設即可）
> - Port：可選 `51820` 或隨機（記下來）
> - DNS：選 `1.1.1.1, 1.0.0.1`
> - 等腳本跑完後它會問你「VPN 使用者名稱」→ 自己取個名稱

完成後你就可以：
手機安裝 WireGuard App ➜ 掃描 QR Code
或把設定檔下載後丟到筆電上用：
1. ls查看`.conf`檔命名
2. ```bash cat ~/laptop.conf```
3. 使用vim語法或其他方法複製上面取得的資訊於筆電上
```bash
vim laptop.conf #可新建及進入編輯
#䩞上
#儲存：按esc到最下面輸入
:wq
```


🔧 加新用戶指令：
之後要再新增一個 VPN 使用者，只要執行：
```bash
sudo ./wireguard-install.sh
```
選擇「Add a new client」，它會幫你生成第 2 組 .conf 檔和 QR Code！
---

## 🌐 3. 開啟 GCP 防火牆 UDP port 才能使用

1. GCP Console → 左欄選VPC network → Firewall 裡點選 Create firewall rule （不是policy）
2. 填入以下資訊：
   - 名稱：`allow-wireguard`
   - 網路：`default`（預設）
   - 方向：Ingress （預設）
   - IPv4 ranges: 來源 IP：`0.0.0.0/0`
   - 協定與埠號：勾選 UDP，輸入剛剛設定的 port，例如 `51820`
---

## 🧩 4. 匯入設定到手機 & 筆電

### 📱 手機（iOS / Android）

1. 安裝 WireGuard App
2. 點「+」→「掃描 QR Code」
3. 匯入 → 點啟用 ✅

### 💻 筆電（macOS / Windows）

1. 安裝 WireGuard App
2. 點「Import tunnel from file」
3. 匯入 `.conf` 檔
4. 點啟用即可

---

## 🧪 5. 測試是否成功

連上 VPN 後，開瀏覽器輸入：

```
whatismyip.com
```

- ✅ IP 若為 GCP 公網 IP（如 `35.xx.xx.xx`），表示成功！

---

## 📌 附加設定（建議）

### ✅ 啟用 WireGuard 開機自啟

```bash
sudo systemctl enable wg-quick@wg0
```

### ✅ 手動啟動 / 停用

```bash
sudo systemctl start wg-quick@wg0
sudo systemctl stop wg-quick@wg0
```

---

## 🎉 完成！你已擁有專屬 GCP VPN
未來預計搬到Oracle免費版vm，以加快連線速度

可以跨裝置使用，安全穩定，也能搭配 DuckDNS 做 IP 綁定、自動更新，進一步升級使用體驗 🙌

這能直接貼到readMe嗎 若不行請改成能貼的格式

