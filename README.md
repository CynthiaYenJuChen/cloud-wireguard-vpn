# 📡 GCP VM 架設 WireGuard VPN 全流程指南

本教學適用於：

- ✅ Google Cloud Platform（GCP）
- ✅ Ubuntu 20.04 / 22.04 系統
- ✅ 目標：讓手機與筆電透過 VPN 安全連線、取得 GCP 的出口 IP（如翻牆、訪問國外網站）

---

## ✅ 0. 前置準備

- 已註冊 GCP 帳號並啟用計費（可用 Free Tier）
- 已建立專案（Project）
- 已啟用 Compute Engine API

---

## 🚀 1. 建立 VM 虛擬機

1. 進入 GCP Console → Compute Engine → VM instances → Create instance
2. 建議設定：
   - 名稱：`vpn`
   - 區域（Region）：如 `us-central1`（或你選的區域）
   - 機器類型：`e2-micro`（Free Tier）
   - 開機映像：Ubuntu 22.04 LTS
   - 開機磁碟：10GB 標準磁碟即可
3. 勾選「Allow HTTP/HTTPS」暫不需
4. 建立 VM

---

## 🔥 2. SSH 登入 VM 安裝 WireGuard

```bash
sudo apt update && sudo apt install -y curl
curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
chmod +x wireguard-install.sh
sudo ./wireguard-install.sh
```

> 安裝過程中會問一些問題：
>
> - 使用 IPv4
> - Interface 名稱：`wg0`（預設即可）
> - Port：可選 `51820` 或隨機（記下來）
> - DNS：選 `1.1.1.1, 1.0.0.1`

---

## 🌐 3. 開啟 GCP 防火牆 UDP port

1. GCP Console → VPC network → Firewall rules → Create rule
2. 填入以下資訊：
   - 名稱：`allow-wireguard`
   - 網路：`default`
   - 方向：Ingress
   - 來源 IP：`0.0.0.0/0`
   - 協定與埠號：勾選 UDP，輸入剛剛設定的 port，例如 `51820`

---

## 📱 4. 新增手機 / 筆電用戶端設定

每新增一個用戶請再次執行：

```bash
sudo ./wireguard-install.sh
```

→ 選 `Add a new client` → 輸入名稱（如 `phone`、`laptop`）

系統會產生設定檔，例如：

```
/home/你的帳號/wg0-client-phone.conf
```

產生 QR code（給手機用）：

```bash
qrencode -t ansiutf8 < wg0-client-phone.conf
```

---

## 🧩 5. 匯入設定到手機 & 筆電

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

## 🧪 6. 測試是否成功

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

可以跨裝置使用，安全穩定，也能搭配 DuckDNS 做 IP 綁定、自動更新，進一步升級使用體驗 🙌

這能直接貼到readMe嗎 若不行請改成能貼的格式

