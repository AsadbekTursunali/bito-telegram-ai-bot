Bito ERP Telegram Bot — Texnik Spetsifikatsiya
Versiya: 0.2 (Bito'ning haqiqiy OpenAPI hujjati asosida yangilangan) Ko'lam: Shaxsiy foydalanish — bir yoki bir nechta tanish do'kon(lar) uchun. Bito marketplace mahsuloti emas.

1. Maqsad
Do'kon sotuvchilari Telegram guruhida yoki bot bilan shaxsiy chatda ovozli xabar yuborib, quyidagi tranzaksiyalarni Bito ERP tizimiga avtomatik kiritishlari:

Kirim (naqd pul kirishi)
Chiqim (xarajat)
Sotuv (mijozga mahsulot sotish)
Xarid (ta'minotchidan mahsulot olish)
Bot ovozni tushunadi, tuzilgan tranzaksiyaga aylantiradi, sotuvchidan tasdiq oladi, so'ng Bito API orqali yozadi.

2. Ko'lam va cheklovlar
Faqat oldindan ma'lum bo'lgan do'kon(lar) uchun ishlaydi — dinamik onboarding UI yo'q
Do'kon/xodim moslashtirish (mapping) config faylda yoki oddiy jadvalda qo'lda kiritiladi
Marketplace review, ko'p mijozli (multi-tenant) infratuzilma, iframe vidjet — ushbu bosqichda kerak emas
Til: o'zbek tili (lotin/kirill aralash bo'lishi mumkin)
3. Bito API — tasdiqlangan texnik tafsilotlar
Manba: foydalanuvchi tomonidan yuklangan bito-openapi.json (OpenAPI 3.0, 464 endpoint, Bito 2.0 Integration API v1.0.0)

3.1 Auth — muhim tuzatish
Avvalgi taxmindan farqli o'laroq, Bito OAuth 2.0 emas, oddiy API key ishlatadi:

Sarlavha (header): api-key: <sizning kalitingiz>
Kalit Bito panelidagi Integratsiya bo'limidan olinadi
Bu shaxsiy loyiha uchun ancha soddalashtiradi: token refresh, OAuth redirect flow — hech biri kerak emas, faqat bitta statik kalitni xavfsiz saqlash yetarli
3.2 Server manzillari
Muhit	URL
Dev/Test	https://api.systematicdev.uz/integration-api/integration/api/v2/
Production	https://api.bito.uz/integration-api/integration/api/v2/
3.3 Asosiy endpoint'lar (tranzaksiya turlari bo'yicha)
Amal	Endpoint	Murakkablik
Kirim	POST /transaction/income	Oddiy — amount, payment_type_id, employee_id, currency_id
Chiqim	POST /transaction/expense	Oddiy — kirim bilan bir xil sxema
Sotuv	POST /trade/create	Murakkab — mahsulot(lar), ombor, narx, valyuta, to'lov usuli talab qiladi
Xarid	POST /purchase/create	Murakkab — ta'minotchi, ombor, mahsulot(lar) talab qiladi
Yordamchi (ma'lumot qidirish) endpoint'lari:

Maqsad	Endpoint
Mahsulotni nomi bo'yicha topish	POST /product/get-by-name ({"name": "..."})
Mahsulotni shtrix-kod bo'yicha topish	POST /product/get-by-barcode
Mijozni telefon raqami bo'yicha topish	GET /customer/get-by-phone-number
Ta'minotchini telefon bo'yicha topish	GET /supplier/get-by-phone/{phone_number}
Xodimni telefon bo'yicha topish	POST /employee/get-by-phone-number
Omborlar ro'yxati	POST /warehouse/get-all
To'lov usullari ro'yxati	GET /payment-method/get-all
Asosiy valyuta	GET /currency/get-main
3.4 /transaction/income va /transaction/expense — so'rov strukturasi
Ikkalasi bir xil sxemani ishlatadi:

{
  "payment_type_id": "...",
  "employee_id": "...",
  "user_type": "customer | supplier | employee | person | other",
  "customer_id": "... (user_type ga qarab)",
  "amount": 500000,
  "paid": 500000,
  "payment_method_id": "...",
  "currency_id": "...",
  "atachments": []
}
Eslatma: user_type ga qarab mos *_id maydon (customer_id, supplier_id, person_id va h.k.) talab qilinadi — botning LLM-parser qatlami buni aniqlashi kerak (masalan, "Aziz akadan pul keldi" → user_type: customer, mijoz ID /customer/get-by-phone-number yoki nomi bo'yicha qidiruv orqali topiladi).

3.5 /trade/create (sotuv) — muhim murakkablik
Bu endpoint MVP uchun eng qiyin qism, chunki u to'liq buxgalteriya darajasidagi ma'lumotni talab qiladi:

Majburiy maydonlar: organization_id, state, responsible_id, currency, by_balance, payments, applied_cashbacks, changes, products, date, discounts, installment_plan_interval, additional_costs, currency_values

Har bir mahsulot qatorida majburiy: product_id, amount (miqdor), price, discounts (bo'sh massiv bo'lsa ham), marks, extra_costs, warehouse_id

Demak, ovozli xabardan sotuv yozish uchun bot avval:

Aytilgan mahsulot nomini /product/get-by-name orqali product_idga aylantirishi,
Do'konning standart warehouse_idsini (config'dan) qo'shishi,
Narxni yo aytilgan summadan, yo mahsulotning standart narxidan olishi,
To'lov usulini (naqd/karta) payment_method_idga bog'lashi kerak.
3.6 /purchase/create (xarid) — o'xshash murakkablik
Majburiy maydonlar: organization_id, state, date, responsible_id, supliaer_id (diqqat: hujjatda shu xato yozilgan — kod yozishda ham aynan shu nom ishlatilishi kerak), warehouse_id, currency_id, orders, additional_costs

4. MVP ko'lamini qayta ko'rib chiqish
Yuqoridagi murakkablik farqi sabab, bosqichma-bosqich yondashuv tavsiya etiladi:

1-bosqich (MVP): Faqat kirim va chiqim — bular oddiy, tez ishlab chiqiladi, sotuvchilar uchun eng ko'p ishlatiladigan amal ("kassaga 200 ming tushdi", "500 ming ijaraga to'landi" kabi)

2-bosqich: Sotuv — mahsulot-nomi-qidiruv va ombor/narx moslashtirish logikasi qo'shiladi. Bu yerda ovozli xabarda mahsulot noaniq aytilsa (masalan, qisqartma nom), botning tasdiqlash oynasi orqali to'g'ri mahsulotni tanlash imkoniyati muhim.

3-bosqich: Xarid — ta'minotchi bilan bog'liq bo'lgani uchun, ta'minotchilar ro'yxati oldindan Bito'da mavjud bo'lishi kerak.

5. Arxitektura komponentlari
[Sotuvchi] --ovoz--> [Telegram Bot API]
                          |
                          v
                  [Bot Backend Service]
                    |    |         |
                    v    v         v
              [STT]  [Config]   [LLM Parser]
                    \    |         /
                     v   v        v
                [Tasdiqlash logikasi]
                   (kerak bo'lsa: mahsulot/mijoz
                    qidiruv API'lari chaqiriladi)
                          |
                     (✅ tasdiqlandi)
                          |
                          v
              [Bito API] (api-key header bilan)
                          |
                          v
                     [Audit log DB]
5.1 Telegram Bot qatlami
Webhook orqali ishlaydi
Ovozli xabar faylini yuklab oladi
Inline tugmalar bilan tasdiqlash so'raydi (✅ Tasdiqlash / ✏️ Tuzatish / ❌ Bekor qilish)
5.2 Speech-to-Text (STT)
O'zbek tilidagi ovozni matnga aylantiradi (Whisper, Yandex SpeechKit va h.k. amaliy sinovdan o'tkazilishi kerak)
Tasdiqlash qadami majburiy — STT xatosi moliyaviy xatoga aylanmasligi uchun
5.3 LLM Parser
Matnni tranzaksiya turiga mos JSON'ga aylantiradi. Endi ikki xil chiqish formati kerak:

Kirim/chiqim uchun: oddiy — summa, to'lov turi, kontragent turi
Sotuv/xarid uchun: murakkabroq — mahsulot nomi(lari), miqdor, narx (agar aytilgan bo'lsa)
5.4 Config / mapping ma'lumotlari
Jadval	Maydonlar
stores	store_id, bito_api_key, telegram_chat_id, default_warehouse_id
employees	telegram_user_id, bito_employee_id, store_id
transactions_log	id, store_id, employee_id, raw_text, parsed_json, bito_response, status, created_at
5.5 Audit / Log
Har bir tranzaksiya (muvaffaqiyatli yoki xato) log qilinadi; xatolik bo'lsa retry navbatiga qo'yiladi.

6. Texnologiya taklifi
Qism	Taklif
Bot framework	Python — aiogram (yoki Node.js — telegraf)
Backend	FastAPI (Python) yoki Express (Node.js)
STT	Whisper API / lokal Whisper / Yandex SpeechKit
LLM parser	Claude yoki GPT (structured output/JSON mode)
Ma'lumotlar bazasi	PostgreSQL yoki SQLite
Hosting	VPS
7. Rivojlantirish bosqichlari
Bito API bilan bog'lanish — api-key header bilan sandbox/dev serverga (api.systematicdev.uz) test so'rovi (masalan, /warehouse/get-all)
Config sozlash — do'kon, xodim, ombor, to'lov usuli ID'larini oldindan yig'ib olish (/employee/get-paging, /warehouse/get-all, /payment-method/get-all, /currency/get-main)
Telegram bot skeleton — ovozli xabarni qabul qilish
STT integratsiyasi
LLM parser — kirim/chiqim uchun (MVP)
Tasdiqlash UI
/transaction/income va /transaction/expense yozish — pilot sinov
Sotuv (/trade/create) qo'shish — mahsulot qidiruv logikasi bilan
Xarid (/purchase/create) qo'shish
Kengaytirish — boshqa do'konlarga
8. Xavf va ularni yumshatish
Xavf	Yumshatish
STT xato tushunishi	Majburiy tasdiqlash, past ishonchda qayta so'rash
/trade/create, /purchase/create murakkabligi	MVP'da faqat kirim/chiqim, sotuv/xarid keyingi bosqichda
Mahsulot nomi noaniq aytilsa	/product/get-by-name bir nechta natija qaytarsa, tasdiqlash oynasida tanlov beriladi
Bito API vaqtincha ishlamasligi	Retry navbati, xatolikni sotuvchiga bildirish
API key oshkor bo'lishi	.env/secret manager'da saqlash, kodga yozilmasligi kerak
Moliyaviy xato (noto'g'ri summa)	Katta summalarda qo'shimcha ogohlantirish
9. Ochiq savollar
Har bir do'konda nechta ombor (warehouse) bor — bittami, ko'pmi? (Config murakkabligiga ta'sir qiladi)
Sotuvda narx har doim mahsulotning standart narxidanmi, yoki sotuvchi og'zaki boshqa narx aytishi mumkinmi?
To'lov turi (naqd/karta) har doim ovozda aytiladimi, yoki standart qiymat (masalan, "naqd") qo'yish mumkinmi?
Qaysi STT xizmati o'zbek tilida eng yaxshi natija beradi — amaliy test talab qiladi# bito-telegram-ai-bot
