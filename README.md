📱 iOS Phone Farm Otomasyon Sistemi (TrollStore & SSH)Bu rehber, 18 adet iOS cihazın (iOS 15.8.5) Mac Mini M4 üzerinden merkezi olarak yönetilmesi, uygulama şifrelerinin çözülmesi (decryption) ve toplu kurulum süreçlerini içerir.🛠 Sistem GereksinimleriKontrol Merkezi: Mac Mini M4 (macOS Sequoia)Cihazlar: 18x iPhone (iOS 15.8.5 - Palera1n Jailbreak)Bağlantı: USB üzerinden iproxy köprüsü📥 1. Gerekli Araçlar ve İndirme LinkleriFarmın düzgün çalışması için her cihazda bulunması gereken temel araçlar:AraçKaynak / İndirmeNotTrollStoreTrollStore GitHubAna yükleyiciTrollDecryptorwh1te4ever/TrollDecryptorApp Store uygulamalarını kırmak içinNewTerm 3Sileo (Chariz Repo)Cihaz içi terminalFilza File ManagerTIGI005 RepoDosya yönetimi için🚀 2. Kurulum ve YapılandırmaTrollStore AyarlarıOtomasyonun "Sessiz Kurulum" yapabilmesi için her cihazda şu ayarlar yapılmalıdır:TrollStore > Settings sekmesine girin.URL Scheme: "Enabled" (Aktif) yapın. (Modern sürümlerde apple-magnifier:// maskesini kullanır).Show Install Confirmation Alert: Bu seçeneği "Never" (Asla) olarak ayarlayın. Bu sayede script komut gönderdiğinde telefonda onay kutusu çıkmaz, direkt kurulur.TrollDecryptor ile Uygulama KırmaUygulamaları farm cihazlarına sorunsuz dağıtmak için şifresiz (decrypted) IPA elde etmelisiniz:Yukarıdaki linkten en güncel .tipa veya .ipa dosyasını indirin.Dosyayı TrollStore ile açıp kurun.Kırmak istediğiniz uygulamayı (örn: Mises) App Store'dan indirin ve bir kez açın.TrollDecryptor'ı açın, listeden uygulamayı seçin ve "Decrypt" butonuna basın.Oluşan dosyayı Mac Mini'ye (Desktop/farmapps) transfer edin.🤖 3. 18 Cihaz İçin Toplu Kurulum (Otomasyon)Mac terminalinde iproxy bağlantılarını sağladıktan sonra aşağıdaki scripti kullanın. Bu yöntem, manuel imzalama (zsign) hatalarını tamamen bypass eder.Kurulum Scripti (farm_install.sh)Bash#!/bin/bash

# Yapılandırma
IPA_NAME="mises_decrypted.ipa"
# Farmdaki cihaz portlarını buraya ekleyin (2222, 2223, 2224...)
PORTS=(2222 2223 2224) 

for PORT in "${PORTS[@]}"
do
   echo "-----------------------------------------------"
   echo "Port $PORT: İşlem başlıyor..."
   
   # 1. IPA dosyasını telefona gönder
   scp -P $PORT -o StrictHostKeyChecking=no $IPA_NAME mobile@127.0.0.1:/var/mobile/Documents/
   
   # 2. TrollStore'u 'apple-magnifier' şemasıyla tetikle
   ssh -p $PORT -o StrictHostKeyChecking=no mobile@127.0.0.1 "uiopen 'apple-magnifier://install?url=file:///var/mobile/Documents/$IPA_NAME'"
   
   echo "Port $PORT: Kurulum emri gönderildi."
done
🔍 Sıkça Karşılaşılan SorunlarConnection Refused: iproxy [Port] 22 komutunun arka planda çalıştığından emin olun.Uygulama Kurulmuyor: TrollStore Settings içinden "Rebuild URL Scheme" butonuna basarak tetikleyiciyi yenileyin.SSH Şifre Sorunu: Sürekli şifre girmemek için Mac anahtarınızı ssh-copy-id -p [Port] mobile@127.0.0.1 komutuyla cihazlara tanımlayın.Önemli NotBu farm sistemi, uygulamaların orijinal yetkilerini (Entitlements) TrollStore üzerinden yamaladığı için uygulamaların çökmesini (crash) %100 engeller.
