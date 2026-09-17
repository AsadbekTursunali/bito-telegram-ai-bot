Bito ERP Telegram Bot
Do'kon sotuvchilari Telegram orqali ovozli xabar yuborib, Bito ERP tizimiga kirim, chiqim, sotuv va xarid tranzaksiyalarini avtomatik kiritadigan bot.

📄 To'liq texnik spetsifikatsiya: bito-erp-telegram-bot-spec.md 📄 Bito API to'liq hujjati: bito-openapi.json

Loyiha nima qiladi
Sotuvchi Telegram guruhiga yoki botga ovozli xabar yuboradi ("Kassaga 200 ming tushdi" kabi)
Bot ovozni matnga aylantiradi (STT)
Matndan tranzaksiya ma'lumotini (turi, summa, kontragent) ajratib oladi (LLM)
Sotuvchidan tasdiq so'raydi (✅/✏️/❌)
Tasdiqlangach, Bito ERP API'ga yozadi
Har bir amal log qilinadi
Hozirgi bosqich (MVP): faqat kirim va chiqim. Sotuv va xarid keyingi bosqichda qo'shiladi (sababi spetsifikatsiyaning 4-bo'limida yozilgan).

Talab qilinadigan narsalar
Narsa	Qayerdan olinadi
Bito API key	Bito panel → Integratsiya bo'limi
Telegram Bot Token	@BotFather orqali /newbot
STT xizmati kaliti	Whisper API yoki Yandex SpeechKit (hali tanlanmagan — spetsifikatsiyaning ochiq savollariga qarang)
LLM API kaliti	Claude yoki OpenAI
Python 3.11+ (yoki Node.js 20+, agar shu yo'l tanlansa)	—
Loyiha strukturasi (rejalashtirilgan)
bito-erp-bot/
├── README.md
├── bito-erp-telegram-bot-spec.md
├── bito-openapi.json
├── .env.example
├── requirements.txt
├── app/
│   ├── main.py                 # Bot kirish nuqtasi (webhook handler)
│   ├── config.py                # stores/employees mapping yuklovchi
│   ├── stt/
│   │   └── transcribe.py        # Ovozni matnga aylantirish
│   ├── parser/
│   │   └── llm_parser.py        # Matndan tranzaksiya JSON chiqarish
│   ├── bito/
│   │   ├── client.py            # Bito API bilan ishlovchi asosiy klass
│   │   ├── income_expense.py    # /transaction/income, /transaction/expense
│   │   ├── trade.py              # /trade/create (2-bosqich)
│   │   └── purchase.py           # /purchase/create (3-bosqich)
│   ├── telegram/
│   │   ├── handlers.py           # Xabarlarni qabul qilish, tugmalar
│   │   └── messages.py           # Foydalanuvchiga ko'rsatiladigan matnlar
│   └── db/
│       ├── models.py             # stores, employees, transactions_log
│       └── repository.py
└── data/
    └── config.yaml               # store_id, warehouse_id, employee mapping
Muhit o'zgaruvchilari (.env.example)
# Bito
BITO_API_KEY=
BITO_BASE_URL=https://api.systematicdev.uz/integration-api/integration/api/v2/

# Telegram
TELEGRAM_BOT_TOKEN=

# STT
STT_PROVIDER=whisper   # yoki yandex
STT_API_KEY=

# LLM
LLM_PROVIDER=claude    # yoki openai
LLM_API_KEY=

# Ma'lumotlar bazasi
DATABASE_URL=sqlite:///./bot.db
⚠️ .env faylini hech qachon git repozitoriyga qo'shmang — .gitignorega qo'shilishi shart.

Ishga tushirish (rejalashtirilgan, MVP tayyor bo'lgach)
# 1. Muhitni sozlash
cp .env.example .env
# .env faylini o'z kalitlaringiz bilan to'ldiring

# 2. Kutubxonalarni o'rnatish
pip install -r requirements.txt

# 3. do'kon/xodim/ombor ID'larini Bito API'dan olib, data/config.yaml ga yozish

# 4. Botni ishga tushirish
python -m app.main
Rivojlantirish bosqichlari
Batafsil reja bito-erp-telegram-bot-spec.md faylining 7-bo'limida. Qisqacha:

✅ Bito API bilan bog'lanish (test so'rov)
⬜ Config sozlash (do'kon/xodim/ombor ID'lari)
⬜ Telegram bot skeleton
⬜ STT integratsiyasi
⬜ LLM parser — kirim/chiqim
⬜ Tasdiqlash UI
⬜ Kirim/chiqim yozish — pilot sinov
⬜ Sotuv qo'shish
⬜ Xarid qo'shish
Litsenziya
Shaxsiy loyiha — ochiq litsenziya belgilanmagan.
