# Telegram-_bot
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes
import requests

# Token
TOKEN = "7880321029:AAFg3I6t-WqeWmIqE1peCA-ySL90-AbXAU8"
CHANNEL_ID = '-1002383820713'

# المتغير لتخزين الرصيد
all_mony = 0
all_links = []

url_tiktok = "https://www.pythonanywhere.com/user/KAAPA/shares/301fa53a18764561a77349e15b6a80ba/Chek/chek_tiktok"
url_facebook = "https://www.pythonanywhere.com/user/KAAPA/shares/301fa53a18764561a77349e15b6a80ba/Chek/chek_custom"
# دالة المساعدة
async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_name = update.message.from_user.first_name
    chat_id = update.effective_chat.id
    await context.bot.send_message(
        chat_id=chat_id,
        text=f'اهلاً {user_name}، هذه منصة التسويق بالعمولة السودانية. لتبدأ العمل معنا هناك بعض الشروط:\n'
             '١- أدنى مبلغ للسحب (٢٠٠٠ جنيه سوداني), لا يمكنك سحب مبلغ أقل من الحد الأدنى.\n'
             '٢- إذا حدثت مشكلة خارجة عن إرادتنا ولم نستطع التعرف عليك أو الوصول إليك فنحن أبرياء من أموالك يوم القيامة.\n'
             'إذا كنت موافقاً على هذه الشروط اضغط على /start.'
    )

# دالة البدء
async def start_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text_message = update.message.text
    chat_id = update.effective_chat.id
    await context.bot.send_message(
        chat_id=chat_id,
        text="بما أنك موافق، يمكنك البدء في إرسال روابط المنشورات المطلوبة. "
             "هذا هو رابط المجموعة التي يتم فيها عرض المنشورات: https://t.me/KAPPASHOP7/."
    )

    if text_message.startswith('http') and text_message not in all_links:
    all_links.append(text_message)
    if 'tiktok' in text_message:
        await process_tiktok_link(update, context)
    elif 'facebook.com' in text_message:
        await process_custom_link(update, context)
    else:
        await context.bot.send_message(
            chat_id=chat_id,
            text='هذا الرابط مستخدم من قبل.'
        )

# معالجة إدخال روابط تيك توك
async def process_tiktok_link(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global all_mony

    text = update.message.text
    respons = await requests.post(url=url_tiktok,date=text)

    if respons == 'done':
        all_mony += 100
        await update.message.reply_text(f'جيد، لقد حصلت على ١٠٠ جنيه، الآن أنت تملك {all_mony} جنيه سوداني.')
    elif respons is None:
        await update.message.reply_text('عذراً، هذا الرابط اكتملت حملته بنجاح.')
    else:
        await update.message.reply_text('هذا ليس رابط تيك توك صحيح او هذا الرابط ليس صالح .')

# معالجة إدخال الروابط المخصصة
async def process_custom_link(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global all_mony  # للوصول إلى المتغير العالمي

    text = update.message.text
    response = await requests.post(url=url_facebook,date=text)

    if response == 'good':
        all_mony += 100  # إضافة 100 إلى الرصيد
        await update.message.reply_text(f'جيد، الآن أنت تملك {all_mony} جنيه سوداني.')
    elif response == 'no more':
        await update.message.reply_text('أنا آسف، لقد وصلنا للعدد المطلوب في هذه الحملة.')
else:
await update.message.reply_text('هذا الرابط غير صالح ')

# دالة عرض الرصيد
async def my_mony_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global all_mony
    chat_id = update.effective_chat.id
    await context.bot.send_message(
        chat_id=chat_id,
        text=f'أنت تملك {all_mony} جنيه سوداني.'
    )

# دالة سحب الأموال
async def get_yoru_mony_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global all_mony
    chat_id = update.effective_chat.id
    current_balance = all_mony

    await context.bot.send_message(
        chat_id=chat_id,
        text=f'أنت تملك {current_balance} جنيه سوداني. أدخل المبلغ الذي تريد سحبه:'
    )

# معالجة مبلغ السحب
async def handle_withdraw_amount(update: Update, context: ContextTypes.DEFAULT_TYPE):
    global all_mony
    chat_id = update.effective_chat.id
    text = update.message.text
    current_balance = all_mony

    try:
        amount = int(text)

        if amount > current_balance or amount < 2000:
            await context.bot.send_message(
                chat_id=chat_id,
                text=f'يجب أن يكون المبلغ مساويًا أو أكبر من الحد الأدنى (٢٠٠٠ جنيه سوداني) وألا يكون أكبر من {current_balance}.'
            )
        else:
            all_mony -= amount
            await context.bot.send_message(
                chat_id=chat_id,
                text=f"تم خصم {amount} جنيه سوداني. الآن أنت تملك {all_mony} جنيه سوداني."
            )

            await context.bot.send_message(
                chat_id=chat_id,
                text="الرجاء إدخال بيانات حساب بنكك كالتالي:\nالاسم رباعي:\nرقم الحساب:"
            )
    except ValueError:
        await context.bot.send_message(
            chat_id=chat_id,
            text="الرجاء إدخال مبلغ صحيح."
        )

# استقبال معلومات الحساب البنكي
async def forward_to_channel(update: Update, context: ContextTypes.DEFAULT_TYPE):
    chat_id = update.effective_chat.id
    message_text = update.message.text

    # إرسال المعلومات إلى القناة
    await context.bot.send_message(
        chat_id=CHANNEL_ID,
        text=f"قام المسوق {chat_id} بسحب مبلغ. معلومات الحساب البنكي:\n{message_text}"
    )

    # تأكيد الإرسال للمستخدم
    await context.bot.send_message(
        chat_id=chat_id,
        text="تم إرسال معلومات حسابك البنكي بنجاح إلى الإدارة."
    )

# الوظيفة الرئيسية لتشغيل البوت
async def main():
    app = Application.builder().token(TOKEN).build()

    help_handler = CommandHandler('help', help_command)
    start_handler = CommandHandler('start', start_command)
    my_mony_handler = CommandHandler('my_mony', my_mony_command)
    get_your_mony_handler = CommandHandler('get_your_mony', get_yoru_mony_command)

    app.add_handler(help_handler)
    app.add_handler(start_handler)
    app.add_handler(my_mony_handler)
    app.add_handler(get_your_mony_handler)
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND & filters.Regex(r'https?://(www\.)?tiktok\.com/.+'), process_tiktok_link))

app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND & filters.Regex(r'https?://(www\.)?facebook\.com/.+'), process_custom_link))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_withdraw_amount))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, forward_to_channel))

    if __name__ == '__main__':
        asyncio.run(main())
