sudo apt update && sudo apt upgrade
sudo apt install python3-pip python3-opencv espeak ffmpeg
pip install opencv-python
pip install SpeechRecognition
pip install pyttsx3
pip install escpos-python
pip install pyaudio
sudo apt-get install portaudio19-dev python3-pyaudio
sudo apt-get install portaudio19-dev python3-pyaudio
nano salom_ai.py
import cv2
import speech_recognition as sr
import pyttsx3
from escpos.printer import Usb
import time

# 1. Ovozli javob funksiyasi
def speak(text):
    engine = pyttsx3.init()
    engine.setProperty('rate', 150)
    engine.say(text)
    engine.runAndWait()

# 2. Mijozning gapini eshitish
def listen():
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("👂 Tinglanmoqda...")
        audio = r.listen(source)
        try:
            text = r.recognize_google(audio, language='uz-UZ')
            print("✅ Tushunildi:", text)
            return text
        except sr.UnknownValueError:
            print("⚠️ Tushunilmadi")
            return "Tushunilmadi"
        except sr.RequestError:
            print("🌐 Internet xatolik")
            return "Xatolik"

# 3. Chek chiqarish
def print_receipt(xizmat_nomi):
    try:
        printer = Usb(0x0416, 0x5011)  # PRINTER USB ID (lsusb bilan tekshiring)
        printer.text("Xizmat nomi: {}\n".format(xizmat_nomi))
        printer.text("Sana: {}\n".format(time.strftime("%Y-%m-%d %H:%M:%S")))
        printer.text("Rahmat!\n")
        printer.cut()
        print("🖨️ Chek chiqarildi.")
    except Exception as e:
        print("❌ Printer xatolik:", e)

# 4. Kamera orqali harakatni aniqlash
cap = cv2.VideoCapture(0)
print("📷 Kamera ishga tushdi. Mijozni kutmoqda...")

motion_detected = False
_, frame1 = cap.read()
gray1 = cv2.cvtColor(frame1, cv2.COLOR_BGR2GRAY)

while True:
    _, frame2 = cap.read()
    gray2 = cv2.cvtColor(frame2, cv2.COLOR_BGR2GRAY)
    diff = cv2.absdiff(gray1, gray2)
    _, thresh = cv2.threshold(diff, 25, 255, cv2.THRESH_BINARY)
    motion = thresh.sum()

    if motion > 2000000 and not motion_detected:
        motion_detected = True
        print("🧍 Mijoz aniqlandi.")
        speak("Assalomu alaykum! Qanday xizmat kerak?")
        xizmat = listen()
        print_receipt(xizmat)
        speak("Rahmat! Yaxshi kun tilaymiz.")
        break

    gray1 = gray2

cap.release()
cv2.destroyAllWindows()
