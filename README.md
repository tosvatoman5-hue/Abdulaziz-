import logging
from datetime import datetime
from telegram import Update, ChatMember
from telegram.ext import ApplicationBuilder, ChatMemberHandler, ContextTypes, MessageHandler, filters

# Admin Telegram User ID (sizning profilingiz ID’si)
ADMIN_USER_ID = 7600474681  # <-- Sizning Telegram ID’ingiz

# Enable logging
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

async def notify_admin(context: ContextTypes.DEFAULT_TYPE, text: str):
    """Send notification to the admin in a private chat."""
    await context.bot.send_message(chat_id=ADMIN_USER_ID, text=text)

def extract_status_change(old: ChatMember, new: ChatMember):
    """Detect user join or leave events."""
    if old.status in ['left', 'kicked'] and new.status in ['member', 'administrator']:
        return 'joined'
    elif old.status in ['member', 'administrator'] and new.status in ['left', 'kicked']:
        return 'left'
    return None

async def chat_member_update(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Handle chat member updates (join/leave events)."""
    old_member = update.chat_member.old_chat_member
    new_member = update.chat_member.new_chat_member
    user = new_member.user

    event_type = extract_status_change(old_member, new_member)
    if event_type:
        timestamp = datetime.utcnow().strftime('%Y-%m-%d %H:%M:%S UTC')
        username = user.username or "(no username)"
        log_text = (
            f"User {username} (ID: {user.id}) {event_type} "
            f"{'group' if update.effective_chat.type in ['group', 'supergroup'] else 'channel'} "
            f"at {timestamp}."
        )
        logger.info(log_text)
        await notify_admin(context, log_text)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Respond with instructions."""
    await update.message.reply_text("I'm monitoring group join/leave events and will notify the admin.")

def main():
    # Siz bergan token
    bot_token = "7600474681:AAG6REUWhUr4S6Or0EykK5PHKbByJ4PLwhw"
    app = ApplicationBuilder().token(bot_token).build()

    # Handler for /start command
    app.add_handler(MessageHandler(filters.Command("start"), start))

    # Handler for chat member updates (joins/leaves)
    app.add_handler(ChatMemberHandler(chat_member_update, chat_member_types=ChatMemberHandler.CHAT_MEMBER))

    app.run_polling()

if __name__ == "__main__":
    main()
