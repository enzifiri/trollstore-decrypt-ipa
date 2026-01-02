# 📱 iOS Phone Farm Otomasyon Sistemi (TrollStore & SSH)

Bu depo, 18 adet iOS cihazın (iOS 15.8.5) Mac Mini M4 üzerinden merkezi olarak yönetilmesi, uygulama şifrelerinin çözülmesi (decryption) ve toplu kurulum süreçlerini içeren teknik rehberi barındırır.

## 🛠 Donanım ve Yazılım Altyapısı
* **Kontrol Merkezi:** Mac Mini M4 (macOS Sequoia)
* **Cihazlar:** 18x iPhone (iOS 15.8.5 - Palera1n Jailbreak)
* **Bağlantı:** USB üzerinden `iproxy` köprüsü (Port 2222-2240)

## 📥 1. Temel Araçlar ve Kaynaklar

Farm cihazlarının kurulumu için gerekli temel yazılımlar:

| Araç | Kaynak / Link | Amaç |
| :--- | :--- | :--- |
| **TrollStore** | [GitHub Repo](https://github.com/opa334/TrollStore) | Kalıcı uygulama yükleyici |
| **TrollDecryptor** | [GitHub Releases](https://github.com/wh1te4ever/TrollDecryptor/releases) | App Store uygulamalarını kırmak için |
| **iproxy** | `brew install libimobiledevice` | USB üzerinden SSH bağlantısı |
| **NewTerm 3** | Sileo (Chariz Repo) | Cihaz içi terminal erişimi |

---

## ⚙️ 2. Kritik Cihaz Ayarları

Otomasyonun "Sessiz Kurulum" (Silent Install) yapabilmesi için her cihazda şu ayarlar yapılmalıdır:

1.  **TrollStore Konfigürasyonu:**
    * **URL Scheme:** "Enabled" (Aktif) konuma getirilmelidir. 
    * **Maske:** Modern sürümlerde tetikleyici olarak `apple-magnifier://` kullanılır.
    * **Show Install Confirmation Alert:** Otomatik onay için bu ayar **"Never"** (Asla) olarak seçilmelidir.

2.  **SSH Erişimi:**
    * Cihazlara `mobile` kullanıcısı ve belirlenen şifre (örn: `asd123asd`) ile erişim sağlanır.
    * Sürekli şifre girmemek için: `ssh-copy-id -p [PORT] mobile@127.0.0.1`

---

## 🔓 3. Uygulama Hazırlama (Decryption)

Uygulamaları farm cihazlarına dağıtmak için "Decrypted" hale getirilmesi şarttır:
1.  **TrollDecryptor**'ı kurun ve hedef uygulamayı App Store'dan indirin.
2.  Uygulamayı bir kez açıp kapatın.
3.  TrollDecryptor içinden uygulamayı seçip **"Decrypt"** butonuna basın.
4.  Oluşan `.ipa` dosyasını Mac Mini'ye (Airdrop veya SCP ile) aktarın.

---

## 🤖 4. Toplu Kurulum Otomasyonu (Bash Script)

Aşağıdaki script, Mac Mini üzerindeki IPA dosyasını 18 cihaza sırayla gönderir ve TrollStore üzerinden kurulumu tetikler. Bu yöntem `zsign` hatalarını tamamen bypass eder.

### `farm_install.sh`
```bash
#!/bin/bash

# Dosya ismi
IPA_NAME="mises_decrypted.ipa"

# Farm cihazlarının port listesi (2222'den başlayarak)
PORTS=(2222 2223 2224 2225) 

for PORT in "${PORTS[@]}"
do
   echo "-----------------------------------------------"
   echo "Port $PORT: İşlem başlatılıyor..."
   
   # 1. Dosyayı Mac'ten iPhone'a gönder
   scp -P $PORT -o StrictHostKeyChecking=no $IPA_NAME mobile@127.0.0.1:/var/mobile/Documents/
   
   # 2. TrollStore'u tetikleyerek kurulum emri ver
   # apple-magnifier maskesi sessiz kurulum sağlar
   ssh -p $PORT -o StrictHostKeyChecking=no mobile@127.0.0.1 "uiopen 'apple-magnifier://install?url=file:///var/mobile/Documents/$IPA_NAME'"
   
   echo "Port $PORT: Kurulum başarıyla tetiklendi."
done
