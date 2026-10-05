# SMTP Yapılandırma Hatası ile Arka Plan (Origin) IP Sızıntısı Analizi

Bu çalışmada, bir web uygulaması önünde CDN/WAF (Örn: Cloudflare) koruması bulunmasına rağmen, e-posta sunucusu (SMTP) yapılandırmasındaki bir eksiklik nedeniyle gerçek sunucu IP adresinin nasıl sızabileceği (Information Disclosure) ve bu durumun nasıl çözüleceği analiz edilmiştir.

## 1. Keşif ve Sızıntı Senaryosu
Web uygulaması üzerinden kullanıcıya gönderilen otomatik bir e-postanın (Şifre sıfırlama, aktivasyon vb.) ham başlık bilgileri (Email Headers) incelendiğinde, aracı SMTP servisinin e-postayı teslim aldığı ilk istasyonun IP adresini kayıt altına aldığı gözlemlenmiştir.

Sızıntının gerçekleştiği örnek header satırı (Anonimleştirilmiştir):
`Received: from sunucu_adi [46.4.X.X] (Authenticated sender: no-reply@hedefsite.com) by smtp-servisi.com (Exim);`

Bu durum, saldırganların aradaki CDN koruma katmanını tamamen atlayarak (Bypass) doğrudan ana sunucunun IP adresine (Origin) yönelmesine, port taraması yapmasına veya doğrudan DDoS saldırıları gerçekleştirmesine imkan tanır.

## 2. Doğrulama (Verification)
Sızan IP adresi doğrudan tarayıcıda çağrıldığında, uygulamanın kurulu olduğu web sunucusu (Örn: OpenResty/Nginx) doğrudan yanıt vermekte ve `403 Forbidden` hatası döndürmektedir. Bu durum, IP adresinin koruma arkasında değil, doğrudan erişilebilir bir backend sunucusu olduğunu doğrulamaktadır.

## 3. Çözüm ve Mitigasyon
Bu tür sızıntıların önüne geçmek için şu adımlar uygulanmalıdır:
1. **API Tabanlı Gönderim:** E-postalar SMTP Relay yerine servis sağlayıcının (SendPulse, Mailgun, vb.) Web API'leri (HTTP POST) üzerinden gönderilmelidir. Bu sayede backend IP'si header geçmişine işlenmez.
2. **IP Whitelisting:** Sunucu tarafında sadece güvenilen CDN IP'lerinden gelen isteklere izin verilmeli, doğrudan IP ile gelen tüm dış bağlantılar firewall seviyesinde engellenmelidir.
