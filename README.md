# CUSTOM-POLL-TELEGRAM
Make a custom telegram Poll bot using python where this bot can generate 22 poll, because the limit of poll in telegram only 10

import logging
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import (
    ApplicationBuilder,
    CommandHandler,
    CallbackQueryHandler,
    ContextTypes,
    ConversationHandler,
    MessageHandler,
    filters
)

# List of sites
SITES = [
    "LIST YOU CHOICES HERE"
]

# Define conversation states
ASK_QUESTION, HANDLE_RESPONSE = range(2)

# Group ID (replace with your group's ID)
GROUP_ID =   # Your Telegram group ID

# Start command
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Welcome! Please enter your poll question:")
    return ASK_QUESTION

# Ask for the poll question
async def ask_question(update: Update, context: ContextTypes.DEFAULT_TYPE):
    question = update.message.text
    context.user_data['question'] = question  # Save the question

    # Post the poll in the group
    await post_poll_in_group(context, question)
    await update.message.reply_text("Poll posted in the group!")
    return ConversationHandler.END  # End the conversation

# Post the poll in the group
async def post_poll_in_group(context: ContextTypes.DEFAULT_TYPE, question: str):
    keyboard = []
    for i in range(0, len(SITES), 2):
        site1 = SITES[i]
        site2 = SITES[i + 1]

        row = [
            InlineKeyboardButton(site1, callback_data=f"toggle_{site1}"),
            InlineKeyboardButton(site2, callback_data=f"toggle_{site2}")
        ]
        keyboard.append(row)

    reply_markup = InlineKeyboardMarkup(keyboard)
    await context.bot.send_message(
        chat_id=GROUP_ID,
        text=f"{question}\n\nSelect/Deselect sites:",
        reply_markup=reply_markup
    )

# Handle user selections in the group
async def handle_response(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    # Get the selected site
    site = query.data.split("_")[1]

    # Toggle selection
    if 'selected_sites' not in context.user_data:
        context.user_data['selected_sites'] = set()

    if site in context.user_data['selected_sites']:
        context.user_data['selected_sites'].remove(site)  # Deselect
    else:
        context.user_data['selected_sites'].add(site)  # Select

    # Update the button text to show selection status
    keyboard = []
    for i in range(0, len(SITES), 2):
        site1 = SITES[i]
        site2 = SITES[i + 1]

        # Add checkmark emoji (✅) for selected sites
        site1_text = f"✅ {site1}" if site1 in context.user_data['selected_sites'] else site1
        site2_text = f"✅ {site2}" if site2 in context.user_data['selected_sites'] else site2

        row = [
            InlineKeyboardButton(site1_text, callback_data=f"toggle_{site1}"),
            InlineKeyboardButton(site2_text, callback_data=f"toggle_{site2}")
        ]
        keyboard.append(row)

    reply_markup = InlineKeyboardMarkup(keyboard)
    await query.edit_message_reply_markup(reply_markup=reply_markup)

# Main function
if __name__ == '__main__':
    logging.basicConfig(level=logging.INFO)
    application = ApplicationBuilder().token("YOUR TELEGRAM BOT TOKEN").build()

    # Define conversation handler
    conv_handler = ConversationHandler(
        entry_points=[CommandHandler("start", start)],
        states={
            ASK_QUESTION: [MessageHandler(filters.TEXT & ~filters.COMMAND, ask_question)],
        },
        fallbacks=[]
    )

    # Add conversation handler
    application.add_handler(conv_handler)

    # Add handler for group interactions
    application.add_handler(CallbackQueryHandler(handle_response))

    # Run the bot
    application.run_polling()
