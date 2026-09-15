# ⏰ Zamanlama Kurulumu — Mesajı 11:00'e Sabitleme

## Neden bu yöntem?

GitHub Actions'ın `schedule` (zamanlanmış) cron'u **"best-effort"** çalışır: GitHub
yoğunken işi kuyruğa atar ve **saatlerce gecikebilir**. Bu yüzden öğle mesajı 05:00,
06:00 ya da 12:30 gibi rastgele saatlerde düşüyordu. Cron dakikasını değiştirmek bu
sorunu çözmez — gecikme GitHub tarafındadır.

Çözüm: iş artık `schedule` ile değil, **dış bir zamanlayıcının** (cron-job.org)
tetiklediği `workflow_dispatch` ile çalışıyor. `workflow_dispatch`, `schedule`'ın
aksine **anında** (saniyeler içinde) çalışır. Böylece mesaj her gün 11:00 ± 1 dakika
içinde düşer.

```
cron-job.org (11:00 TR, hafta içi)
        │  POST + PAT
        ▼
GitHub API: workflow_dispatch  ──►  yemek-bot.yml çalışır  ──►  Slack
        (saniyeler içinde)
```

---

## Adım 1 — GitHub Personal Access Token (PAT) oluştur

Fine-grained token (önerilen):

1. https://github.com/settings/personal-access-tokens/new
2. **Token name**: `yemek-bot-dispatch`
3. **Expiration**: tercihine göre (örn. 1 yıl — yenilemeyi unutma)
4. **Repository access** → **Only select repositories** → `gokberksari/ofis-ogle-yemegi`
5. **Permissions** → **Repository permissions** → **Actions** → **Read and write**
6. **Generate token** ve çıkan `github_pat_...` değerini kopyala (bir daha görünmez).

> Klasik token kullanacaksan `workflow` kapsamı (scope) yeterli.

---

## Adım 2 — cron-job.org'da işi oluştur

1. https://cron-job.org üzerinde ücretsiz hesap aç, **Create cronjob**.
2. **Title**: `Ofis Yemek Botu`
3. **URL**:
   ```
   https://api.github.com/repos/gokberksari/ofis-ogle-yemegi/actions/workflows/yemek-bot.yml/dispatches
   ```
4. **Schedule** (Custom):
   - **Days of week**: Mon, Tue, Wed, Thu, Fri (hafta içi)
   - **Time**: `11:00`
   - **Timezone**: `Europe/Istanbul`  ← bunu seçmeyi unutma
5. **Advanced** ayarları:
   - **Request method**: `POST`
   - **Request headers** (ekle):
     ```
     Accept: application/vnd.github+json
     Authorization: Bearer github_pat_BURAYA_TOKEN
     X-GitHub-Api-Version: 2022-11-28
     Content-Type: application/json
     ```
   - **Request body**:
     ```json
     {"ref":"main"}
     ```
6. **Create** ile kaydet.

> Başarılı istek GitHub'dan **HTTP 204** döner (gövde boş). cron-job.org bunu
> "success" olarak gösterir.

---

## Adım 3 — Test et

- cron-job.org'da işi aç → **Run now** (veya **Test run**).
- GitHub → **Actions** sekmesinde `Ofis Yemek Menüsü Bot` çalışmasının hemen
  başladığını göreceksin. Hafta içi bir günse Slack'e mesaj düşer.
- Hafta sonu tetiklersen bot sessizce geçer (mesaj atmaz) — bu normaldir.

---

## Ek: Birine özel sabah DM'i (örn. 08:00)

Workflow'un `kanal` input'u verilirse mesaj `#ogle-yemegi` yerine oraya gider. Webhook
eski tip "Incoming WebHooks" entegrasyonu olduğu için kişinin **member ID**'si
(`U0...`, profil → ⋮ → Copy member ID) verilince menü o kişiye DM olarak düşer.

cron-job.org'da mevcut işi kopyala, sadece şunları değiştir:

- **Time**: `08:00` (Europe/Istanbul, hafta içi)
- **Request body**:
  ```json
  {"ref":"main","inputs":{"kanal":"U0XXXXXXXXX"}}
  ```

Birden fazla kişi isterse her biri için ayrı bir iş aç. Hafta sonu / tatil kuralları
DM için de aynen geçerli.

---

## Notlar

- **Hafta sonu / tatil**: cron-job.org yalnızca hafta içi tetikliyor; ayrıca botun
  kendisi de hafta sonu ve `RESMİ TATİL` günlerinde mesaj atmıyor. Çift güvenlik.
- **Manuel çalıştırma / test tarihi**: GitHub → Actions → Run workflow ile hâlâ elle
  tetikleyebilir, `test_date` girip geçmiş/gelecek bir günü test edebilirsin.
- **Token süresi dolarsa** mesaj gelmez; cron-job.org geçmişinde 401/403 görürsün.
  Yeni PAT üretip header'daki token'ı güncelle.
- Alternatif zamanlayıcılar da aynı isteği atabilir: EasyCron, Google Cloud Scheduler,
  Cloudflare Worker Cron, hatta her zaman açık bir makinede `crontab` + `curl`.
