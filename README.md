import os
import logging
import asyncio
import tempfile
import traceback
from datetime import datetime
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes, CallbackQueryHandler
import yt_dlp as youtube_dl
import requests
from urllib.parse import urlparse, parse_qs

# إعدادات التسجيل
logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO
)
logger = logging.getLogger(__name__)

# توكن البوت
BOT_TOKEN = "8445933881:AAHNLDF121Dwqp_PaQlIzB-HJhb6XJXkXC8"

# مجلد التحميلات
DOWNLOAD_FOLDER = "yt_downloads"
if not os.path.exists(DOWNLOAD_FOLDER):
    os.makedirs(DOWNLOAD_FOLDER)

class YouTubeDownloader:
    def __init__(self):
        self.supported_formats = {
            'video': ['mp4', 'webm', 'mkv'],
            'audio': ['mp3', 'm4a', 'wav']
        }
    
    def get_video_info(self, url):
        """الحصول على معلومات الفيديو"""
        ydl_opts = {
            'quiet': True,
            'no_warnings': True,
            'socket_timeout': 30,
        }
        
        try:
            with youtube_dl.YoutubeDL(ydl_opts) as ydl:
                info = ydl.extract_info(url, download=False)
                return {
                    'success': True,
                    'title': info.get('title', 'بدون عنوان'),
                    'duration': info.get('duration', 0),
                    'uploader': info.get('uploader', 'غير معروف'),
                    'view_count': info.get('view_count', 0),
                    'thumbnail': info.get('thumbnail', ''),
                    'formats': info.get('formats', [])
                }
        except Exception as e:
            logger.error(f"Error getting video info: {e}")
            return {'success': False, 'error': '❌ لا يمكن الوصول إلى هذا الفيديو'}

    def format_duration(self, seconds):
        """تنسيق المدة"""
        if not seconds:
            return "غير معروف"
        try:
            minutes, seconds = divmod(seconds, 60)
            hours, minutes = divmod(minutes, 60)
            if hours > 0:
                return f"{hours:02d}:{minutes:02d}:{seconds:02d}"
            return f"{minutes:02d}:{seconds:02d}"
        except:
            return "غير معروف"
    
    def format_file_size(self, size_bytes):
        """تنسيق حجم الملف"""
        if not size_bytes:
            return "غير معروف"
        try:
            for unit in ['B', 'KB', 'MB', 'GB']:
                if size_bytes < 1024.0:
                    return f"{size_bytes:.1f} {unit}"
                size_bytes /= 1024.0
            return f"{size_bytes:.1f} GB"
        except:
            return "غير معروف"
    
    def download_video(self, url, quality='720', format_type='video'):
        """تحميل الفيديو أو الصوت"""
        try:
            if format_type == 'video':
                ydl_opts = {
                    'format': f'best[height<={quality}]/best',
                    'outtmpl': os.path.join(DOWNLOAD_FOLDER, '%(title).100s.%(ext)s'),
                    'quiet': True,
                    'no_warnings': True,
                }
            else:
                ydl_opts = {
                    'format': 'bestaudio/best',
                    'outtmpl': os.path.join(DOWNLOAD_FOLDER, '%(title).100s.%(ext)s'),
                    'postprocessors': [{
                        'key': 'FFmpegExtractAudio',
                        'preferredcodec': 'mp3',
                        'preferredquality': '192',
                    }],
                    'quiet': True,
                    'no_warnings': True,
                }
            
            with youtube_dl.YoutubeDL(ydl_opts) as ydl:
                info = ydl.extract_info(url, download=True)
                final_path = ydl.prepare_filename(info)
                
                if format_type == 'audio':
                    final_path = final_path.replace('.webm', '.mp3').replace('.m4a', '.mp3')
                
                if not os.path.exists(final_path):
                    return {'success': False, 'error': '❌ لم يتم إنشاء الملف'}
                
                file_size = os.path.getsize(final_path)
                
                return {
                    'success': True,
                    'path': final_path,
                    'title': info.get('title', 'بدون عنوان'),
                    'duration': info.get('duration', 0),
                    'file_size': file_size
                }
                
        except Exception as e:
            logger.error(f"Download error: {e}")
            return {'success': False, 'error': f'❌ خطأ في التحميل: {str(e)}'}

# إنشاء كائن التحميل
downloader = YouTubeDownloader()

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """أمر البدء - تم إصلاحه"""
    try:
        user = update.effective_user
        first_name = user.first_name if user else "صديقي"
        
        welcome_text = f"""
🎬 **مرحباً {first_name}!**

أنا بوت متقدم لتحميل مقاطع اليوتيوب 🚀

**المميزات:**
✅ تحميل فيديو بجودات متعددة
✅ تحميل الصوت فقط (MP3)
✅ دعم جميع روابط اليوتيوب
✅ واجهة مستخدم تفاعلية
✅ سرعة تحميل عالية

**طريقة الاستخدام:**
1. أرسل رابط اليوتيوب
2. اختر نوع التحميل
3. انتظر حتى يكتمل التحميل

⚡ **استمتع بالتجربة!**
        """
        
        # استخدام زر بسيط بدون رابط لتجنب الأخطاء
        keyboard = [
            [InlineKeyboardButton("📖 طريقة الاستخدام", callback_data="help")],
            [InlineKeyboardButton("🎬 تحميل فيديو", callback_data="download_direct")]
        ]
        reply_markup = InlineKeyboardMarkup(keyboard)
        
        await update.message.reply_text(
            welcome_text, 
            reply_markup=reply_markup, 
            parse_mode='Markdown'
        )
        logger.info(f"تم إرسال رسالة الترحيب للمستخدم: {user.id if user else 'Unknown'}")
        
    except Exception as e:
        logger.error(f"Error in start command: {e}")
        logger.error(traceback.format_exc())
        
        # رسالة بديلة بسيطة في حالة الخطأ
        try:
            await update.message.reply_text(
                "🎬 مرحباً! أنا بوت تحميل اليوتيوب\n\n"
                "أرسل لي رابط يوتيوب وسأحاول تحميله لك 😊\n\n"
                "استخدم /help للمساعدة"
            )
        except Exception as e2:
            logger.error(f"Even fallback failed: {e2}")

async def help_command(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """أمر المساعدة"""
    try:
        help_text = """
📖 **طريقة الاستخدام:**

1. **أرسل رابط اليوتيوب** الذي تريد تحميله
2. **اختر نوع التحميل:** فيديو أو صوت
3. **للفيديو:** اختر الجودة المطلوبة
4. **انتظر** حتى يكتمل التحميل

**ملاحظات:**
- يدعم الروابط القصيرة (youtu.be)
- يدعم قوائم التشغيل (سيحمل أول فيديو)
- الحد الأقصى للفيديو: 50 دقيقة
- الجودة التلقائية: 720p

❓ للمساعدة اضغط /start
        """
        await update.message.reply_text(help_text, parse_mode='Markdown')
    
    except Exception as e:
        logger.error(f"Error in help command: {e}")
        await update.message.reply_text("❌ حدث خطأ في عرض المساعدة")

def is_valid_youtube_url(url):
    """التحقق من صحة رابط اليوتيوب"""
    youtube_domains = [
        'youtube.com',
        'www.youtube.com', 
        'm.youtube.com',
        'youtu.be',
        'www.youtu.be'
    ]
    
    try:
        parsed = urlparse(url)
        return any(domain in parsed.netloc for domain in youtube_domains)
    except:
        return False

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """معالجة الرسائل النصية"""
    try:
        text = update.message.text.strip()
        
        # التحقق من أن النص هو رابط يوتيوب
        if not is_valid_youtube_url(text):
            await update.message.reply_text("""
❌ **رابط غير مدعوم**
يرجى إرسال رابط يوتيوب صحيح مثل:
- https://www.youtube.com/watch?v=...
- https://youtu.be/...
            """.strip())
            return
        
        # إظهار رسالة الانتظار
        wait_msg = await update.message.reply_text("🔍 جاري التحقق من الرابط...")
        
        # الحصول على معلومات الفيديو
        video_info = downloader.get_video_info(text)
        
        if not video_info or not video_info.get('success'):
            error_msg = video_info.get('error', '❌ لم أتمكن من الوصول إلى هذا الفيديو') if video_info else '❌ خطأ في التحقق من الرابط'
            await wait_msg.edit_text(error_msg)
            return
        
        # التحقق من مدة الفيديو
        if video_info.get('duration', 0) > 3600:
            await wait_msg.edit_text("""
❌ **الفيديو طويل جداً**
لا يمكن تحميل مقاطع longer من 60 دقيقة
يرجى اختيار مقطع أقصر
            """.strip())
            return
        
        # حفظ معلومات الفيديو
        context.user_data['video_url'] = text
        context.user_data['video_info'] = video_info
        
        # تنسيق معلومات الفيديو
        duration = downloader.format_duration(video_info['duration'])
        title = video_info['title']
        if len(title) > 50:
            title = title[:50] + "..."
        
        info_text = f"""
📹 **تم العثور على الفيديو:**

🎬 **العنوان:** {title}
⏰ **المدة:** {duration}
👤 **القناة:** {video_info['uploader']}
👁️ **المشاهدات:** {video_info['view_count']:,}

**اختر نوع التحميل:**
        """
        
        # أزرار الاختيار
        keyboard = [
            [
                InlineKeyboardButton("🎥 تحميل فيديو", callback_data="choose_video"),
                InlineKeyboardButton("🎵 تحميل صوت", callback_data="download_audio")
            ]
        ]
        reply_markup = InlineKeyboardMarkup(keyboard)
        
        await wait_msg.edit_text(info_text, reply_markup=reply_markup, parse_mode='Markdown')
    
    except Exception as e:
        logger.error(f"Error in handle_message: {e}")
        await update.message.reply_text("❌ حدث خطأ في معالجة الرابط")

async def button_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """معالجة ضغط الأزرار"""
    try:
        query = update.callback_query
        await query.answer()
        
        user_data = context.user_data
        video_url = user_data.get('video_url')
        
        if query.data == "help":
            await help_command(update, context)
            return
        
        elif query.data == "download_direct":
            await query.edit_message_text("💫 أرسل لي رابط اليوتيوب مباشرة لبدأ التحميل!")
            return
        
        if not video_url:
            await query.edit_message_text("❌ انتهت الجلسة، يرجى إرسال الرابط مرة أخرى")
            return
        
        if query.data == "choose_video":
            # عرض خيارات جودة الفيديو
            keyboard = [
                [
                    InlineKeyboardButton("📹 720p", callback_data="quality_720"),
                    InlineKeyboardButton("📹 480p", callback_data="quality_480")
                ],
                [
                    InlineKeyboardButton("📹 360p", callback_data="quality_360"),
                    InlineKeyboardButton("🎵 صوت فقط", callback_data="download_audio")
                ],
                [InlineKeyboardButton("🔙 رجوع", callback_data="back_to_main")]
            ]
            reply_markup = InlineKeyboardMarkup(keyboard)
            
            await query.edit_message_text(
                "🎥 **اختر جودة التحميل:**\n\nملاحظة: الجودة الأعلى تحتاج وقت أطول",
                reply_markup=reply_markup,
                parse_mode='Markdown'
            )
        
        elif query.data == "download_audio":
            await query.edit_message_text("🎵 جاري تحميل الصوت...")
            await download_media(video_url, 'audio', '192', query, context)
        
        elif query.data.startswith("quality_"):
            quality = query.data.replace("quality_", "")
            await query.edit_message_text(f"📹 جاري تحميل الفيديو بجودة {quality}p...")
            await download_media(video_url, 'video', quality, query, context)
        
        elif query.data == "back_to_main":
            # إعادة عرض الخيارات الرئيسية
            info_text = """
📹 **اختر نوع التحميل:**
            """
            keyboard = [
                [
                    InlineKeyboardButton("🎥 تحميل فيديو", callback_data="choose_video"),
                    InlineKeyboardButton("🎵 تحميل صوت", callback_data="download_audio")
                ]
            ]
            reply_markup = InlineKeyboardMarkup(keyboard)
            await query.edit_message_text(info_text, reply_markup=reply_markup, parse_mode='Markdown')
    
    except Exception as e:
        logger.error(f"Error in button_handler: {e}")
        try:
            await query.edit_message_text("❌ حدث خطأ في معالجة الطلب")
        except:
            pass

async def download_media(url, media_type, quality, query, context):
    """تحميل الوسائط"""
    try:
        # التحميل
        result = downloader.download_video(url, quality, media_type)
        
        if result['success']:
            # إعداد البيانات للإرسال
            file_size = downloader.format_file_size(result['file_size'])
            duration = downloader.format_duration(result['duration'])
            
            caption = f"""
🎬 **{result['title']}**
⏰ **المدة:** {duration}
📊 **الحجم:** {file_size}
✅ **تم التحميل بنجاح!**
            """.strip()
            
            # إرسال الملف
            with open(result['path'], 'rb') as media_file:
                if media_type == 'video':
                    await context.bot.send_video(
                        chat_id=query.message.chat_id,
                        video=media_file,
                        caption=caption,
                        parse_mode='Markdown'
                    )
                else:
                    await context.bot.send_audio(
                        chat_id=query.message.chat_id,
                        audio=media_file,
                        caption=caption,
                        parse_mode='Markdown',
                        title=result['title'][:64]
                    )
            
            # تنظيف الملف
            try:
                os.remove(result['path'])
            except:
                pass
            
            await query.edit_message_text("✅ تم إرسال الملف بنجاح!")
            
        else:
            error_msg = result.get('error', '❌ حدث خطأ غير معروف أثناء التحميل')
            await query.edit_message_text(error_msg)
            
    except Exception as e:
        logger.error(f"Error in download_media: {e}")
        error_msg = "❌ حدث خطأ أثناء إرسال الملف. يرجى المحاولة مرة أخرى."
        try:
            await query.edit_message_text(error_msg)
        except:
            pass
        
        # تنظيف الملف في حالة الخطأ
        if 'result' in locals() and result.get('success') and os.path.exists(result.get('path', '')):
            try:
                os.remove(result['path'])
            except:
                pass

async def error_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """معالجة الأخطاء العامة"""
    try:
        logger.error(f"Exception while handling an update: {context.error}")
        
        # إرسال رسالة خطأ للمستخدم
        if update and update.effective_message:
            error_message = """
❌ **حدث خطأ غير متوقع**

السبب المحتمل:
• مشكلة في الاتصال بالإنترنت
• الفيديو غير متوفر أو محمي
• ضغط على الخادم

يرجى المحاولة مرة أخرى بعد قليل 🕐
            """.strip()
            
            try:
                await update.effective_message.reply_text(error_message)
            except:
                pass
    except Exception as e:
        logger.error(f"Error in error_handler: {e}")

def main():
    """الدالة الرئيسية"""
    try:
        # إنشاء التطبيق
        application = Application.builder().token(BOT_TOKEN).build()
        
        # إضافة المعالجات
        application.add_handler(CommandHandler("start", start))
        application.add_handler(CommandHandler("help", help_command))
        application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
        application.add_handler(CallbackQueryHandler(button_handler))
        application.add_error_handler(error_handler)
        
        # بدء البوت
        print("🚀 البوت يعمل الآن...")
        print("✅ تم إصلاح مشكلة رسالة الترحيب")
        print("📞 البوت جاهز لاستقبال الطلبات")
        
        # تشغيل البوت
        application.run_polling()
        
    except Exception as e:
        logger.error(f"Fatal error in main: {e}")

if __name__ == '__main__':
    main()
