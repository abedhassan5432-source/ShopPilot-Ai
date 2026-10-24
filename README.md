# ShopPilot AI — Omnichannel AI Salesman (Next.js + n8n)

> **ShopPilot AI** — Facebook Messenger + Instagram DM + WhatsApp + Website চ্যাট — **সব এক ইনবক্সে**।
> AI নিজে একজন সেলসম্যানের মতো কথা বলে, প্রোডাক্ট ছবি থেকে চিনে সাজেস্ট করে,
> অর্ডার নেয়, পার্সেল ট্র্যাক করে আর পোস্টের কমেন্টের অটো রিপ্লাই দেয়।

---

## ১. n8n দিয়ে কি পুরোটা করা যায়?

**সৎ উত্তর: ৮০% যায়, ২০% যায় না।**

| কাজ | n8n একা পারে? | কারণ |
|---|---|---|
| FB / IG / WhatsApp message ধরা | ✅ হ্যাঁ | Webhook + Trigger node |
| মেসেজ পাঠানো (reply) | ✅ হ্যাঁ | HTTP Request → Graph API |
| AI দিয়ে উত্তর লেখা | ✅ হ্যাঁ | AI Agent / LLM node |
| ভয়েস → টেক্সট | ✅ হ্যাঁ | Whisper API node |
| Auto comment reply | ✅ হ্যাঁ | Webhook + HTTP |
| **Unified Inbox UI (এজেন্ট বসে দেখবে)** | ❌ | n8n এ রেডি ইনবক্স নেই |
| **অর্ডার ডেস্ক + স্ট্যাটাস পাইপলাইন** | ❌ | Data table খুব দুর্বল |
| **প্রোডাক্ট ক্যাটালগ ম্যানেজমেন্ট** | ❌ | ছবি/স্টক/দাম ম্যানেজ করা কষ্ট |
| **চ্যাট হিস্ট্রি + কাস্টমার প্রোফাইল + LTV** | ❌ | SQL-less, স্কেল করে না |
| **পার্সেল ট্র্যাকিং পেজ (কাস্টমারের জন্য)** | ❌ | পাবলিক পেজ বানাতে হয় |

### তাই সেরা আর্কিটেকচার = Hybrid

```
Facebook ─┐
Instagram ─┤
WhatsApp ──┼──► n8n (channel connector)  ──► POST /api/agent/chat
Website  ──┘                                      │
                                            ┌─────▼──────────────┐
                                            │  Insaf AI Brain    │
                                            │  (this Next.js app)│
                                            │  • intent detect   │
                                            │  • catalog search  │
                                            │  • vision          │
                                            │  • voice → text    │
                                            │  • order create    │
                                            │  • tracking        │
                                            │  • Claude/Gemini   │
                                            └─────┬──────────────┘
                                                  ▼
                                     PostgreSQL + Admin Dashboard
```

**n8n = হাত-পা। এই অ্যাপ = মগজ।** এতে সুবিধা — AI লজিক এক জায়গায়, চ্যানেল বদলালেও কোড বদলায় না।

---

## ২. এই প্যাকেজে যা যা আছে (Ready to Use)

| পেজ | কাজ |
|---|---|
| `/` | ড্যাশবোর্ড — KPI, চ্যানেল স্ট্যাটাস, অর্ডার পাইপলাইন, বেস্ট সেলার |
| `/inbox` | **Unified Inbox** — ৪ চ্যানেলের সব চ্যাট এক জায়গায়, AI ড্রাফট, হিউম্যান হ্যান্ডওভার, কাস্টমার প্রোফাইল + অর্ডার |
| `/catalog` | প্রোডাক্ট ক্যাটালগ — যোগ করলেই AI "মুখস্থ" করে ফেলে |
| `/orders` | অর্ডার ডেস্ক — স্ট্যাটাস অ্যাডভান্স, কুরিয়ার টাইমলাইন |
| `/automation` | **৬টা ready-to-import n8n workflow** + চেকলিস্ট + curl টেস্ট |
| `/settings` | Claude / Gemini / Built-in সুইচ, system prompt, কমেন্ট রুল |
| `/chat` | লাইভ ওয়েব উইজেট ডেমো (আসল এজেন্ট টেস্ট) |
| `/track` | পাবলিক পার্সেল ট্র্যাকিং পেজ |

### API Endpoints (n8n এগুলো কল করে)

```
POST /api/agent/chat        ← মূল দরজা (সব চ্যানেল)
POST /api/agent/comment     ← FB/IG অটো কমেন্ট রিপ্লাই
POST /api/webhooks/facebook ← Meta webhook (Messenger + IG)
POST /api/webhooks/whatsapp ← WhatsApp Cloud API webhook
GET  /api/tracking/{code}   ← পার্সেল ট্র্যাকিং
GET  /api/stats             ← KPI + দৈনিক ডাইজেস্ট
GET  /api/n8n/workflows     ← ওয়ার্কফ্লো JSON ডাউনলোড
```

---

## ৩. ৫ মিনিটে চালানো

```bash
# 1. DB স্কিমা পুশ
npx drizzle-kit push

# 2. ডেভ সার্ভার
npm run dev

# 3. ব্রাউজারে
http://localhost:3000/chat      # AI টেস্ট করুন
http://localhost:3000/inbox     # চ্যাট হিস্ট্রি দেখুন
```

ডেমো ডেটা (১৪ প্রোডাক্ট, ৮ কাস্টমার, চ্যাট, অর্ডার, ট্র্যাকিং) প্রথম রিকোয়েস্টেই অটো সিড হয়।

---

## ৪. AI মডেল চালু করা (Claude বা Gemini)

`.env` বা ডিপ্লয় environment এ যেকোনো একটা দিন:

```bash
ANTHROPIC_API_KEY=sk-ant-...      # Claude চাইলে
GEMINI_API_KEY=AIza...            # Gemini চাইলে
INSAF_WEBHOOK_SECRET=my-secret    # n8n থেকে কল সিকিউর করতে
META_VERIFY_TOKEN=insaf-ai-verify # Meta webhook verification
```

তারপর `/settings` পেজে provider বদলে দিন — ব্যাস।

**Key না দিলেও অ্যাপ চলবে** — Built-in Bangla Sales Engine
(intent detection + catalog keyword search + rule-based reply) ফ্রিতে কাজ করবে।
কোনো API bill আসবে না, তাই ডেমো/প্রোডাক্ট সেল করাও সহজ।

---

## ৫. n8n সেটআপ (স্টেপ বাই স্টেপ)

1. **n8n ইনস্টল** — n8n.cloud ট্রায়াল, অথবা
   `docker run -it --rm -p 5678:5678 -e GENERIC_TIMEZONE=Asia/Dhaka n8nio/n8n`
2. **ওয়ার্কফ্লো import** — `/automation` পেজে যান → প্রতিটা কার্ডে **📋 JSON কপি** →
   n8n → Workflows → Import from JSON → Paste
3. **`https://YOUR-APP-URL`** খুঁজে আপনার আসল URL বসান
4. **n8n Credentials / Env vars** দিন:
   - `INSAF_WEBHOOK_SECRET`
   - `FB_PAGE_ACCESS_TOKEN` (Messenger + IG + Comments)
   - `WHATSAPP_TOKEN`, `WHATSAPP_PHONE_NUMBER_ID`
   - `OPENAI_API_KEY` (ভয়েস ট্রান্সক্রিপশন)
   - `TELEGRAM_BOT_TOKEN`, `TELEGRAM_CHAT_ID` (ডাইজেস্ট)
5. **Meta App** — developers.facebook.com → App → Messenger + Instagram →
   Webhook URL = `https://<n8n>/webhook/insaf-facebook`, Verify token = `insaf-ai-verify`
6. **WhatsApp Cloud API** — Meta Business Manager → WhatsApp → phone number ID + permanent token
7. **Activate** — সব ওয়ার্কফ্লো ON করুন। এখন সব মেসেজ `/inbox` তে আসবে।

### ৬টা Ready Workflow

| # | নাম | কাজ |
|---|---|---|
| 1 | Facebook Messenger Agent | FB মেসেজ → AI → রিপ্লাই |
| 2 | Instagram DM Agent | IG DM → AI → রিপ্লাই |
| 3 | WhatsApp Cloud Agent | WA মেসেজ/ছবি/ভয়েস → AI → রিপ্লাই |
| 4 | Voice Note Understanding | Whisper → টেক্সট → AI |
| 5 | Auto Comment Reply | পোস্টের কমেন্টে দাম/স্টক রিপ্লাই + DM |
| 6 | Daily Sales Digest | রাত ১০টায় Telegram রিপোর্ট |

---

## ৬. AI কীভাবে কাজ করে (Pipeline)

```
message
  │
  ├─► intent detection
  │     greeting / product_search / price / order / contact /
  │     track / delivery / complaint / other
  │
  ├─► attachment handling
  │     image  → vision labels → catalog match
  │     voice  → transcript → সাধারণ টেক্সট ফ্লো
  │
  ├─► catalog search (বাংলা + বাংলিশ + English synonym)
  │
  ├─► side effects
  │     order intent → order + order_items + tracking event + stock decrement
  │     track intent → latest order + timeline
  │     complaint    → human handoff
  │
  └─► reply composition
        Claude / Gemini  → natural language
        fallback         → rule-based Bangla sales copy
```

---

## ৭. বিক্রি করার প্ল্যান (আপনি যেমন চাইছেন)

| প্ল্যান | দাম | যা থাকবে |
|---|---|---|
| Starter | ৳৫,০০০/মাস | ১ চ্যানেল, ৫০০ মেসেজ, Built-in AI |
| Growth | ৳১২,০০০/মাস | ৪ চ্যানেল, Claude/Gemini, অর্ডার ডেস্ক |
| Business | ৳২৫,০০০/মাস | সব + কমেন্ট অটো-রিপ্লাই + ভয়েস + ডাইজেস্ট |

**Setup fee:** ৳১৫,০০০–৩০,০০০ (one-time, n8n + Meta + WhatsApp setup করে দেওয়া)
**Add-on:** ৳৩,০০০/মাস per extra brand, ৳২,০০০/মাস maintenance

### টার্গেট কাস্টমার
ফেসবুক পেজ ভিত্তিক ফ্যাশন/গ্যাজেট/বিউটি সেলার, যাদের দিনে ১০০+ ইনবক্স আসে কিন্তু
রিপ্লাই দেওয়ার লোক নেই — বাংলাদেশে হাজার হাজার এমন পেজ আছে।

---

## ৮. Tech Stack

- **Next.js 16** (App Router, React 19, TypeScript)
- **PostgreSQL + Drizzle ORM** (14 টেবিল: channels, customers, conversations, messages, products, orders, order_items, tracking_events, comment_rules, webhook_events, agent_settings)
- **Tailwind CSS 4**
- **AI**: Anthropic Claude / Google Gemini / Built-in rule engine (zero-cost fallback)
- **n8n**: channel connector + scheduler

---

## ৯. রোডম্যাপ (পরের ধাপ)

- [ ] Steadfast / Pathao / RedX API দিয়ে আসল কুরিয়ার ট্র্যাকিং
- [ ] pgvector দিয়ে semantic catalog search (এখন keyword-based)
- [ ] Multi-tenant (একাধিক ব্র্যান্ড এক ইনস্টলে)
- [ ] bKash / Nagad payment link অটো-পাঠানো
- [ ] Abandoned cart recovery (n8n schedule + template message)
- [ ] Instagram comment + Story reply automation
- [ ] Live human takeover with Socket.io

---

## ১০. লিগ্যাল চেকলিস্ট

- WhatsApp ২৪-ঘণ্টা সেশন উইন্ডো ও template approval মেনে চলুন
- Meta Platform Terms — অটোমেশন নিয়ে স্পষ্ট জানান ("AI assistant")
- কাস্টমার ডেটা (ফোন/ঠিকানা) এনক্রিপ্ট ও ব্যাকআপ
- রিফান্ড/রিটার্ন পলিসি এজেন্টকে শেখান (system prompt এ যোগ করুন)
