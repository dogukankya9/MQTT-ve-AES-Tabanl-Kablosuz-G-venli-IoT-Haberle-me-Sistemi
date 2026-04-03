"""
========================================================
 MQTT + AES Güvenli IoT Alıcı Programı
 Bu kod ne yapar?
 1. MQTT broker'a bağlanır
 2. ESP8266'dan gelen şifreli veriyi dinler
 3. Veriyi AES ile çözer
 4. Ekrana yazdırır
 5. Kullanıcı isterse şifreli ON/OFF komutu gönderir
========================================================
"""

from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad, pad
import base64
import paho.mqtt.client as mqtt

/****************************************************
 Python tarafında yorumlarda // değil # kullanılır
****************************************************/

# =========================
# MQTT ayarları
# =========================
BROKER = "broker.hivemq.com"
PORT = 1883

TOPIC_DATA = "dogukankaya/mqttaes/data"   # ESP'nin veri gönderdiği topic
TOPIC_CMD = "dogukankaya/mqttaes/cmd"     # ESP'nin komut aldığı topic

# =========================
# AES ayarları
# ESP tarafı ile aynı olmak zorunda
# =========================
AES_KEY = b"1234567890ABCDEF"
AES_IV  = b"ABCDEF1234567890"

# --------------------------------------------------
# decrypt_message()
# Amaç: Gelen base64 şifreli veriyi çözmek
# --------------------------------------------------
def decrypt_message(cipher_b64: str) -> str:
    # Önce base64 çözülür
    raw = base64.b64decode(cipher_b64)

    # AES nesnesi oluşturulur
    cipher = AES.new(AES_KEY, AES.MODE_CBC, AES_IV)

    # Veri çözülür, padding kaldırılır
    decrypted = unpad(cipher.decrypt(raw), AES.block_size)

    # Byte verisi metne çevrilir
    return decrypted.decode("utf-8")

# --------------------------------------------------
# encrypt_message()
# Amaç: ON/OFF gibi komutları şifrelemek
# --------------------------------------------------
def encrypt_message(plain_text: str) -> str:
    # AES nesnesi oluşturulur
    cipher = AES.new(AES_KEY, AES.MODE_CBC, AES_IV)

    # Metin byte dizisine çevrilir ve blok boyutuna tamamlanır
    encrypted = cipher.encrypt(pad(plain_text.encode("utf-8"), AES.block_size))

    # MQTT ile rahat göndermek için base64'e çevrilir
    return base64.b64encode(encrypted).decode("utf-8")

# --------------------------------------------------
# on_connect()
# MQTT bağlantısı kurulduğunda otomatik çalışır
# --------------------------------------------------
def on_connect(client, userdata, flags, rc):
    print("MQTT baglandi. Kod:", rc)

    # Veri topic'ine abone ol
    client.subscribe(TOPIC_DATA)
    print("Subscribe olunan topic:", TOPIC_DATA)

# --------------------------------------------------
# on_message()
# MQTT üzerinden mesaj geldiğinde otomatik çalışır
# --------------------------------------------------
def on_message(client, userdata, msg):
    # Gelen veri byte olarak gelir, string'e çevir
    cipher_text = msg.payload.decode("utf-8")

    print("\n=================================")
    print("[MQTT] Gelen sifreli veri:")
    print(cipher_text)

    try:
        # Veriyi çöz
        plain_text = decrypt_message(cipher_text)

        print("[AES] Cozulmus veri:")
        print(plain_text)
    except Exception as e:
        print("[HATA] Veri cozulurken sorun oldu:", e)

# --------------------------------------------------
# MQTT istemcisi oluşturulur
# --------------------------------------------------
client = mqtt.Client()

# Callback fonksiyonları atanır
client.on_connect = on_connect
client.on_message = on_message

# Broker'a bağlan
client.connect(BROKER, PORT, 60)

# Arka planda MQTT dinleme döngüsü başlat
client.loop_start()

print("\nKomut gondermek icin:")
print("1 yaz -> LED ON")
print("0 yaz -> LED OFF")
print("q yaz -> cikis")

# --------------------------------------------------
# Kullanıcıdan komut al ve gönder
# --------------------------------------------------
while True:
    command = input("\nSecim: ").strip().lower()

    if command == "1":
        # ON komutunu şifrele
        encrypted_cmd = encrypt_message("ON")

        # Şifreli komutu MQTT ile gönder
        client.publish(TOPIC_CMD, encrypted_cmd)

        print("Sifreli ON komutu gonderildi.")

    elif command == "0":
        # OFF komutunu şifrele
        encrypted_cmd = encrypt_message("OFF")

        # Şifreli komutu MQTT ile gönder
        client.publish(TOPIC_CMD, encrypted_cmd)

        print("Sifreli OFF komutu gonderildi.")

    elif command == "q":
        # Programdan çık
        break

    else:
        print("Gecersiz secim. 1, 0 veya q gir.")

# MQTT bağlantısını düzgün kapat
client.loop_stop()
client.disconnect()

print("Program kapatildi.")
