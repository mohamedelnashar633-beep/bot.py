# bot.py
import telebot
from telebot import types

TOKEN = "8619078877:AAHRbs6wiM4_93lp-T5ym9bXV_P4XLgaxGk"
ADMIN_ID = 852295587

bot = telebot.TeleBot(TOKEN)

orders = {}
users = set()

# ===== البداية =====

@bot.message_handler(commands=['start'])
def start(message):
    users.add(message.chat.id)

    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)

    markup.add("💳 شحن حساب")
    markup.add("💰 سحب أرباح")

    if message.chat.id == ADMIN_ID:
        markup.add("🔐 لوحة الأدمن")

    bot.send_message(
        message.chat.id,
        "👑 الوكالة الحربية الرسمية 🔰",
        reply_markup=markup
    )

# ===== الشحن =====

@bot.message_handler(func=lambda m: m.text == "💳 شحن حساب")
def deposit(message):
    msg = bot.send_message(message.chat.id, "📌 أرسل ID الخاص بك")
    bot.register_next_step_handler(msg, dep_id)

def dep_id(message):
    orders[message.chat.id] = {"id": message.text}

    msg = bot.send_message(message.chat.id, "💵 أرسل المبلغ")
    bot.register_next_step_handler(msg, dep_amount)

def dep_amount(message):
    orders[message.chat.id]["amount"] = message.text

    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)

    markup.add("📱 فودافون كاش")
    markup.add("💳 إنستا باي")
    markup.add("🏦 بنك مصر")
    markup.add("🏛️ البنك الأهلي")

    msg = bot.send_message(message.chat.id, "💰 اختر طريقة الشحن", reply_markup=markup)
    bot.register_next_step_handler(msg, payment_method)

def payment_method(message):
    data = orders.get(message.chat.id, {})
    data["method"] = message.text

    text = f"""
💀 طلب شحن جديد

🆔 ID: {data.get('id')}
💵 المبلغ: {data.get('amount')}
💳 الطريقة: {data.get('method')}
👤 @{message.from_user.username}

✅ قبول أو رفض الطلب
"""

    markup = types.InlineKeyboardMarkup()
    markup.add(
        types.InlineKeyboardButton("✅ قبول", callback_data=f"accept_{message.chat.id}"),
        types.InlineKeyboardButton("❌ رفض", callback_data=f"reject_{message.chat.id}")
    )

    bot.send_message(ADMIN_ID, text, reply_markup=markup)
    bot.send_message(message.chat.id, "🔥 تم إرسال طلبك وجاري المراجعة")

# ===== لوحة الأدمن =====

@bot.message_handler(func=lambda m: m.text == "🔐 لوحة الأدمن")
def admin_panel(message):
    if message.chat.id != ADMIN_ID:
        return

    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)

    markup.add("📊 الإحصائيات")
    markup.add("📢 رسالة جماعية")

    bot.send_message(message.chat.id, "🔐 لوحة الأدمن", reply_markup=markup)

# ===== الكولباك =====

@bot.callback_query_handler(func=lambda call: True)
def callback(call):
    if call.data.startswith("accept"):
        user_id = int(call.data.split("_")[1])

        bot.send_message(user_id, "✅ تم قبول طلبك وجاري التنفيذ")

        bot.answer_callback_query(call.id, "تم القبول")

    elif call.data.startswith("reject"):
        user_id = int(call.data.split("_")[1])

        bot.send_message(user_id, "❌ تم رفض طلبك")

        bot.answer_callback_query(call.id, "تم الرفض")

# ===== التشغيل =====

bot.polling()
