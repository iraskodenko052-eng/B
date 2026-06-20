import telebot
import requests
import random
import string
import urllib.parse

# Твій токен
TOKEN = "8702281914:AAGkp2R5vdXhzQIBaSm4oDsTPtrxR4eKOZs"
bot = telebot.TeleBot(TOKEN)

# Команди
@bot.message_handler(commands=['start', 'help'])
def help_cmd(message):
    bot.reply_to(message, "Команди: /weather (погода), /pic (фото), /gif (гіф), /pass (пароль), або просто пиши питання.")

@bot.message_handler(commands=['weather'])
def weather(m):
    bot.send_chat_action(m.chat.id, 'typing')
    q = m.text.replace('/weather', '').strip()
    # Якщо місто не вказано, покаже погоду по IP (або помилку)
    res = requests.get(f"https://wttr.in/{urllib.parse.quote(q)}?format=3").text
    bot.reply_to(m, f"🌤 {res}")

@bot.message_handler(commands=['pic'])
def pic(m):
    bot.send_chat_action(m.chat.id, 'upload_photo')
    q = m.text.replace('/pic', '').strip()
    bot.send_photo(m.chat.id, f"https://loremflickr.com/640/480/{urllib.parse.quote(q)}")

@bot.message_handler(commands=['gif'])
def gif(m):
    bot.send_chat_action(m.chat.id, 'upload_document')
    q = m.text.replace('/gif', '').strip()
    url = f"https://tenor.googleapis.com/v2/search?q={urllib.parse.quote(q or 'fun')}&key=LIVDSRZULELA&limit=1"
    try:
        data = requests.get(url).json()
        bot.send_animation(m.chat.id, data['results'][0]['media_formats']['gif']['url'])
    except:
        bot.reply_to(m, "❌ Нічого не знайдено.")

@bot.message_handler(commands=['pass'])
def password(m):
    bot.reply_to(m, f"🔑 `{''.join(random.choices(string.ascii_letters + string.digits, k=12))}`", parse_mode='Markdown')

@bot.message_handler(func=lambda m: True)
def chat(m):
    bot.send_chat_action(m.chat.id, 'typing')
    # Відповідь ШІ
    res = requests.get(f"https://text.pollinations.ai/{urllib.parse.quote(m.text)}")
    bot.reply_to(m, res.text)

if __name__ == "__main__":
    # infinity_polling дозволяє боту працювати без перерви на серверах
    bot.infinity_polling()

