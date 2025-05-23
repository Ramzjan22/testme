import logging
import pandas as pd
from telegram import Update
from telegram.ext import (
    Application,
    CommandHandler,
    MessageHandler,
    filters,
    ContextTypes,
)
from datetime import datetime
import os

# Enable logging
logging.basicConfig(
    format="%(asctime)s - %(name)s - %(levelname)s - %(message)s", level=logging.INFO
)
logger = logging.getLogger(__name__)

# Correct answers for 16 questions (modify as needed)
CORRECT_ANSWERS = ['a', 'b', 'c', 'a', 'b', 'c', 'a', 'b', 'c', 'a', 'b', 'c', 'a', 'b', 'c', 'a']
CSV_FILE = "results.csv"

# Initialize CSV file if it doesn't exist
def init_csv():
    if not os.path.exists(CSV_FILE):
        df = pd.DataFrame(columns=["user_id", "username", "score", "submission_time"])
        df.to_csv(CSV_FILE, index=False)

# Calculate score and save to CSV
def process_answers(user_id, username, answers):
    # Validate input
    if len(answers) != len(CORRECT_ANSWERS):
        return f"Xato: {len(CORRECT_ANSWERS)} ta savol uchun {len(CORRECT_ANSWERS)} ta javob yuborishingiz kerak!"
    if not all(c.lower() in 'abc' for c in answers):
        return "Xato: Javoblar faqat 'a', 'b' yoki 'c' bo'lishi kerak!"
    
    # Calculate score
    score = sum(1 for user_ans, correct_ans in zip(answers.lower(), CORRECT_ANSWERS) if user_ans == correct_ans)
    
    # Save to CSV
    submission_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    new_data = {
        "user_id": user_id,
        "username": username or "Unknown",
        "score": score,
        "submission_time": submission_time
    }
    df = pd.read_csv(CSV_FILE) if os.path.exists(CSV_FILE) else pd.DataFrame(columns=["user_id", "username", "score", "submission_time"])
    df = pd.concat([df, pd.DataFrame([new_data])], ignore_index=True)
    df.to_csv(CSV_FILE, index=False)
    
    # Calculate rank
    df['submission_time'] = pd.to_datetime(df['submission_time'])
    df = df.sort_values(by=['score', 'submission_time'], ascending=[False, True]).reset_index(drop=True)
    rank = df.index[df['user_id'] == user_id].tolist()[0] + 1
    
    return f"Sizning natijangiz: {score}/{len(CORRECT_ANSWERS)}\nSiz {rank}-o'rindasiz!"

# Start command handler
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    await update.message.reply_text(
        f"Assalomu alaykum, {update.effective_user.first_name}!\n"
        f"Test javoblarini {len(CORRECT_ANSWERS)} ta harf (a, b, c) shaklida yuboring, masalan: abcabcabcabcabca"
    )

# Message handler for answers
async def handle_answers(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    user_id = update.effective_user.id
    username = update.effective_user.username or update.effective_user.first_name
    answers = update.message.text.strip()
    
    result = process_answers(user_id, username, answers)
    await update.message.reply_text(result)

# Error handler
async def error_handler(update: Update, context: ContextTypes.DEFAULT_TYPE) -> None:
    logger.error(f"Update {update} caused error {context.error}")

def main() -> None:
    # Replace 'YOUR_BOT_TOKEN' with the token from BotFather
    application = Application.builder().token("8189191443:AAG2JVIYUssa-yZKdVONSacrNJF5EYJK_Y8").build()
    
    # Initialize CSV
    init_csv()
    
    # Add handlers
    application.add_handler(CommandHandler("start", start))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_answers))
    application.add_error_handler(error_handler)
    
    # Start the bot
    application.run_polling(allowed_updates=Update.ALL_TYPES)

if __name__ == "__main__":
    main()
