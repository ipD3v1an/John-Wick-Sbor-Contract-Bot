import logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, MessageHandler, filters, CallbackContext

# Настройки
TOKEN = "ВАШ_TELEGRAM_BOT_TOKEN"
ADMIN_ID = 123456789  # Ваш ID для админ-уведомлений

# Включим логирование
logging.basicConfig(format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO)
logger = logging.getLogger(__name__)

# Данные квеста (можно заменить на Google Sheets)
QUEST_DATA = {
    "start": {
        "text": "🔫 *Добро пожаловать в Континенталь, наёмник.*\n\nТвой контракт активирован. Найди QR-код на теле курьера в парке.",
        "image": "https://example.com/start.jpg"
    },
    "7-2-9": {
        "text": "💼 *Сейф открыт!*\nВнутри фото Сары. Она ждёт тебя в баре. Спроси у неё про *'любовь'*.",
        "next_qr": "sara"
    },
    "i love you": {
        "text": "🎭 *Сара шепчет:* «Предатель — человек с красным платком. Он сейчас у фонтана...»",
        "next_step": "final"
    }
}

# Команда /start
async def start(update: Update, context: CallbackContext):
    quest = QUEST_DATA["start"]
    await update.message.reply_photo(
        photo=quest["image"],
        caption=quest["text"],
        parse_mode="Markdown"
    )

# Обработка ответов
async def handle_message(update: Update, context: CallbackContext):
    user_input = update.message.text.lower().strip()
    
    if user_input in QUEST_DATA:
        response = QUEST_DATA[user_input]
        await update.message.reply_text(
            response["text"],
            parse_mode="Markdown"
        )
        
        if "next_qr" in response:
            await update.message.reply_text(f"🔍 Следующий QR-код: `{response['next_qr']}`")
    else:
        await update.message.reply_text("❌ *Неверный код!* Совет недоволен...", parse_mode="Markdown")

# Запуск бота
def main():
    application = Application.builder().token(TOKEN).build()
    
    # Обработчики команд
    application.add_handler(CommandHandler("start", start))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    
    # Запускаем бота
    application.run_polling()

if __name__ == "__main__":
    main()
