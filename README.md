# Codex Quota Dashboard Kit

OpenClaw + Codex OAuth profillerini görsel dashboard'da gösterir.

## Özellikler
- Profil bazlı quota görünümü
- Manual `Refresh` butonu (cron ile sürekli sorgu yok)
- `flock` kilidi ile güvenli collector (çakışan refresh'ler engellenir)
- Ayrı agent ile güvenli auth-order probing (önerilir)
- Tailscale üzerinden uzaktan erişim

## Gereksinimler
- `openclaw`
- `jq`
- `python3`
- (Opsiyonel ama önerilir) `tailscale`

## Kurulum
```bash
git clone https://github.com/roskva000/openclaw-authorder-quotastate.git
cd openclaw-authorder-quotastate
bash install.sh
```

Kurulum sonrası çıktıdaki linki aç.

## Kullanım
- Dashboard: `http://<tailscale-ip>:8787`
- Refresh: UI'dan `🔄 Refresh`

## Komutlar
```bash
bash scripts/start.sh
bash scripts/stop.sh
bash scripts/codex-quota-collector.sh
bash scripts/codex-quota-report.sh --json | jq .
```

## Konfigürasyon
`.env` dosyasından:
- `AGENT_ID=quota`
- `CREATE_AGENT=1`
- `PROVIDER=openai-codex`
- `PORT=8787`
- `BIND_HOST=auto`
- `REFRESH_TIMEOUT=180`

## Sık Karşılaşılan Durumlar / Sorun Giderme

### 1) Tailscale yoksa link açılmıyor
- Varsayılan `BIND_HOST=auto`, Tailscale IP bulamazsa `127.0.0.1`'e düşer.
- Çözüm:
  - Aynı makineden aç: `http://127.0.0.1:8787`
  - veya `.env` içinde `BIND_HOST=0.0.0.0` yapıp `bash scripts/stop.sh && bash scripts/start.sh`

### 2) `openclaw/jq/python3` eksik hatası
- `install.sh` dependency kontrolü yapar ve eksikte durur.
- Eksik binary'i kurup tekrar `bash install.sh` çalıştır.

### 3) `quota` agent yok / özel agent kullanılacak
- Varsayılan `AGENT_ID=quota`.
- Otomatik oluşturma açık: `CREATE_AGENT=1`.
- Mevcut agent kullanacaksan `.env` içinde `AGENT_ID=<agent-id>` ve gerekirse `CREATE_AGENT=0`.

### 4) Profillerin hepsi aynı usage görünüyor
- Bu script bug'ı da olabilir, provider davranışı da olabilir.
- Doğrulama için:
  ```bash
  bash scripts/codex-quota-report.sh --json | jq '.profiles[] | {profile, windows}'
  ```
- Eğer profile göre fark yoksa provider aynı usage penceresi dönüyor olabilir.

### 5) Refresh'e basınca hata (500/timeout)
- Log kontrolü:
  ```bash
  tail -n 200 dashboard.log
  ```
- Manuel test:
  ```bash
  bash scripts/codex-quota-collector.sh
  ```
- Gerekirse timeout arttır: `.env` -> `REFRESH_TIMEOUT=240`, sonra server restart.

### 6) Port dolu hatası
- `.env` içinde `PORT=8788` gibi değiştir.
- Ardından:
  ```bash
  bash scripts/stop.sh
  bash scripts/start.sh
  ```

### 7) Dashboard eski veri gösteriyor
- Hard refresh yap (`Ctrl+F5` / mobilde tam yenile)
- Son data dosyasını kontrol et:
  ```bash
  cat data/latest.json | jq '.generatedAt'
  ```

### 8) Auth order canlı trafiği etkiler mi?
- Önerilen kullanım: ayrı agent (`AGENT_ID=quota`), böylece main agent akışı etkilenmez.
- Bu yüzden varsayılan kurulum zaten izole agent modelini kullanır.

## Güvenlik Notu
- Repo `.env`, `data/`, `*.log`, `*.pid` dosyalarını git'e almaz.
- Yine de paylaşmadan önce `git status` ile kontrol etmen önerilir.

## Kaldırma
```bash
bash uninstall.sh
```

Bu sadece dashboard process + cron satırını kaldırır; dosyaları silmez.
