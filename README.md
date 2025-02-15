# Telegram-bot
import telebot
import threading
import time

TOKEN = "7845865963:AAG3yGXIXOkNZkaG5Clo4tdv6keNXk688zI"
OWNER_ID = 6270876509  # آیدی عددی سازنده
CHANNEL_USERNAME = "Justice_died"  # نام کاربری کانال بدون @
VIDEOS = []  # لیست فیلم‌های ذخیره‌شده

bot = telebot.TeleBot(TOKEN)

# بررسی عضویت کاربر در کانال
def is_user_subscribed(user_id):
    try:
        chat_member = bot.get_chat_member(f"@{CHANNEL_USERNAME}", user_id)
        return chat_member.status in ["member", "administrator", "creator"]
    except:
        return False

# شروع ربات
@bot.message_handler(commands=['start'])
def send_welcome(message):
    user_id = message.chat.id

    if user_id == OWNER_ID:
        bot.send_message(user_id, "سلام سازنده عزیز! ویدیوهای موردنظر را ارسال کنید.")
    else:
        markup = telebot.types.InlineKeyboardMarkup()
        btn_channel = telebot.types.InlineKeyboardButton("🔗 Channel", url=f"https://t.me/{CHANNEL_USERNAME}")
        btn_continue = telebot.types.InlineKeyboardButton("✅ Continue", callback_data="check_subscription")
        markup.add(btn_channel)
        markup.add(btn_continue)

        bot.send_message(user_id, "🔹 جهت دریافت فیلم‌ها در چنل زیر عضو شوید:", reply_markup=markup)

# ذخیره ویدیوها (فقط برای سازنده)
@bot.message_handler(content_types=['video'])
def save_video(message):
    user_id = message.chat.id

    if user_id == OWNER_ID:
        video_id = message.video.file_id
        VIDEOS.append(video_id)
        bot.send_message(user_id, "✅ ویدیو ذخیره شد!")
    else:
        bot.send_message(user_id, "⛔ شما اجازه ارسال ویدیو ندارید!")

# حذف پیام‌ها بعد از 30 ثانیه
def delete_messages(user_id, messages):
    time.sleep(15)
    for msg in messages:
        try:
            bot.delete_message(user_id, msg)
        except:
            pass

# بررسی عضویت و ارسال ویدیوها
@bot.callback_query_handler(func=lambda call: call.data == "check_subscription")
def check_subscription(call):
    user_id = call.message.chat.id

    if is_user_subscribed(user_id):
        bot.send_message(user_id, "✅ عضویت تایید شد! در حال ارسال ویدیوها...")

        sent_messages = []
        for video in VIDEOS:
            msg = bot.send_video(user_id, video)
            sent_messages.append(msg.message_id)

        warning_msg = bot.send_message(
            user_id,
            "💠 فیلم ها تا 15 ثانیه دیگر پاک خواهند شد..!  لطفاً در سیو مسیج یا جایی دیگر ذخیره کنید."
        )
        sent_messages.append(warning_msg.message_id)

        # استارت تایمر برای حذف پیام‌ها
        threading.Thread(target=delete_messages, args=(user_id, sent_messages)).start()

    else:
        bot.answer_callback_query(call.id, "⛔ هنوز عضو کانال نشده‌اید!")
        send_welcome(call.message)

bot.polling()
