# 🍽️ Ofis Yemek Menüsü Bot

GitHub Actions ile çalışan otomatik yemek menüsü bildirimi botu.

## 🎯 Özellikler

- **Otomatik**: Hafta içi her gün saat 11:00'de çalışır
- **Dakika hassas**: Dış zamanlayıcı (cron-job.org) + `workflow_dispatch` ile tam saatinde
- **Akıllı**: Hafta sonu ve resmi tatil günleri sessizce geçilir
- **Zengin içerik**: Çorba, ana yemek, yan yemek, salata, tatlı detayları
- **Kalori bilgisi**: Her menü için kalori hesabı
- **Ücretsiz**: GitHub Actions + cron-job.org ücretsiz katmanı

## ⏰ Çalışma Programı

- **Hafta içi 11:00**: Günlük menü bildirimi
- **Hafta sonu / resmi tatil**: Mesaj atılmaz (sessiz geçer)

> **Neden dış zamanlayıcı?** GitHub Actions'ın `schedule` cron'u best-effort olduğu
> için saatlerce gecikebiliyor (mesaj rastgele saatlerde düşüyordu). Bu yüzden iş artık
> anında çalışan `workflow_dispatch` ile, cron-job.org üzerinden 11:00'de tetikleniyor.
> Kurulum: [`docs/ZAMANLAMA.md`](docs/ZAMANLAMA.md)

## 🔧 Kurulum

1. Bu repository'yi fork edin
2. Slack webhook URL'nizi `SLACK_WEBHOOK_URL` secret'ı olarak ekleyin
3. `#ogle-yemegi` kanalı oluşturun
4. Zamanlayıcıyı kurun: [`docs/ZAMANLAMA.md`](docs/ZAMANLAMA.md)

## 🧪 Test

**Actions** → **Ofis Yemek Menüsü Bot** → **Run workflow**
- Test tarihi: `2025-08-01`
- **Run workflow**

## 📊 Örnek Mesaj

```
🍽️ 01.08.2025 Cuma - Bugünün Menüsü

🍖 Ana Yemekler:
• Terbiyeli Köfte

🥬 Yan Yemekler:
• Su Böreği

🥗 Salatalar:
• Çoban Salata

🍰 Tatlılar:
• Karpuz

⚡ Kalori: 1390 kcal

🤖 Ofis Yemek Bot | Afiyet olsun! 😋
```

## 🔄 Güncelleme

Yeni ay menüsü geldiğinde `yemek_menusu.json` dosyasını güncelleyin.

## 📞 Destek

GitHub Issues ile sorularınızı sorabilirsiniz.
