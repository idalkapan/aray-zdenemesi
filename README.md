# aray-zdenemesi
import tkinter as tk
from tkinter import filedialog
import string
import sqlite3
from datetime import datetime


# Veritabanı oluşturma ve tablo oluşturma fonksiyonu
def veritabani_olustur():
    conn = sqlite3.connect('metin_analizi.db')
    c = conn.cursor()
    c.execute('''CREATE TABLE IF NOT EXISTS analiz_sonuclari
                 (id INTEGER PRIMARY KEY AUTOINCREMENT,
                  dosya1 TEXT,
                  dosya2 TEXT,
                  analiz_tarihi TEXT,
                  sonuc TEXT)''')
    conn.commit()
    conn.close()

class MetinKarsilastir:
    def _init_(self, dosya1="", dosya2=""):
        self.dosya1 = dosya1
        self.dosya2 = dosya2

    def kelime_say(self, dosya):
        harf_sayisi = 0
        kelime_sayisi = 0
        etkisiz_kelime_sayisi = 0
        kelime_sayilari = {}
        etkisiz_kelimeler = ["bu", "şu", "ve", "ama", "veya"]  # Örnek etkisiz kelimeler listesi

        metin = dosya.lower().translate(str.maketrans("", "", string.punctuation))
        kelimeler = metin.split()
        harf_sayisi = len(metin.replace(" ", ""))
        kelime_sayisi = len(kelimeler)

        for kelime in kelimeler:
            if kelime in kelime_sayilari:
                kelime_sayilari[kelime] += 1
            else:
                kelime_sayilari[kelime] = 1

            if kelime in etkisiz_kelimeler:
                etkisiz_kelime_sayisi += 1

        return harf_sayisi, kelime_sayisi, etkisiz_kelime_sayisi, kelime_sayilari

    def karsilastir(self):
        harf_sayisi1, kelime_sayisi1, etkisiz_kelime_sayisi1, kelime_sayilari1 = self.kelime_say(self.dosya1)
        harf_sayisi2, kelime_sayisi2, etkisiz_kelime_sayisi2, kelime_sayilari2 = self.kelime_say(self.dosya2)

        sonuc = f"Dosya 1:\n"
        sonuc += f"Toplam harf sayısı: {harf_sayisi1}\n"
        sonuc += f"Toplam kelime sayısı: {kelime_sayisi1}\n"
        sonuc += f"Etkisiz kelime sayısı: {etkisiz_kelime_sayisi1}\n"
        

        sonuc += f"\nDosya 2:\n"
        sonuc += f"Toplam harf sayısı: {harf_sayisi2}\n"
        sonuc += f"Toplam kelime sayısı: {kelime_sayisi2}\n"
        sonuc += f"Etkisiz kelime sayısı: {etkisiz_kelime_sayisi2}\n"
        

        most_used1 = sorted(kelime_sayilari1.items(), key=lambda x: x[1], reverse=True)[:5]
        least_used1 = sorted(kelime_sayilari1.items(), key=lambda x: x[1])[:5]

        sonuc += f"\nDosya 1'de En Çok Kullanılan 5 Kelime:\n"

        for word, count in most_used1:
            sonuc += f"{word}: {count}\n"

        sonuc += f"\nDosya 1'de En Az Kullanılan 5 Kelime:\n"

        for word, count in least_used1:
            sonuc += f"{word}: {count}\n"

        most_used2 = sorted(kelime_sayilari2.items(), key=lambda x: x[1], reverse=True)[:5]
        least_used2 = sorted(kelime_sayilari2.items(), key=lambda x: x[1])[:5]

        sonuc += f"\nDosya 2'de En Çok Kullanılan 5 Kelime:\n"

        for word, count in most_used2:
            sonuc += f"{word}: {count}\n"

        sonuc += f"\nDosya 2'de En Az Kullanılan 5 Kelime:\n"

        for word, count in least_used2:
            sonuc += f"{word}: {count}\n"

        sonuc += f"\nİki dosyada ortak kullanılan kelimeler:\n"
        benzer_kelimeler = set(kelime_sayilari1.keys()) & set(kelime_sayilari2.keys())
        for kelime in benzer_kelimeler:
            sonuc += f"{kelime}\n"

        return sonuc

def yukleDosya1():
    dosya1 = filedialog.askopenfilename(initialdir="/", title="Dosya Seç", filetypes=(("Metin Dosyaları", ".txt"), ("Tüm Dosyalar", ".*")))
    inputtxt1.delete(1.0, tk.END)
    inputtxt1.insert(tk.END, open(dosya1, encoding='utf-8').read())

def yukleDosya2():
    dosya2 = filedialog.askopenfilename(initialdir="/", title="Dosya Seç", filetypes=(("Metin Dosyaları", ".txt"), ("Tüm Dosyalar", ".*")))
    inputtxt2.delete(1.0, tk.END)
    inputtxt2.insert(tk.END, open(dosya2, encoding='utf-8').read())

def printInput():
    inp1 = inputtxt1.get(1.0, "end-1c")
    inp2 = inputtxt2.get(1.0, "end-1c")

    newobject = MetinKarsilastir()
    newobject.dosya1 = inp1
    newobject.dosya2 = inp2
    sonuc = newobject.karsilastir()
    sonucText.config(state=tk.NORMAL)
    sonucText.delete(1.0, tk.END)
    sonucText.insert(tk.END, sonuc)
    sonucText.config(state=tk.DISABLED)

    conn = sqlite3.connect('metin_analizi.db')
    c = conn.cursor()
    c.execute("INSERT INTO analiz_sonuclari (dosya1, dosya2, analiz_tarihi, sonuc) VALUES (?, ?, ?, ?)",
              (inp1, inp2, datetime.now().strftime("%Y-%m-%d %H:%M:%S"), sonuc))
    conn.commit()
    conn.close()

def veritabani_goruntule():
    conn = sqlite3.connect('metin_analizi.db')
    c = conn.cursor()
    c.execute("SELECT analiz_tarihi, substr(dosya1, 1, 50), substr(dosya2, 1, 50), substr(sonuc, 1, 50) FROM analiz_sonuclari")
    kayitlar = c.fetchall()
    conn.close()

    goruntuleme_penceresi = tk.Toplevel(frame)
    goruntuleme_penceresi.title("Veritabanı Kayıtları")

    listbox = tk.Listbox(goruntuleme_penceresi, width=100, height=10)
    listbox.pack()

    for kayit in kayitlar:
        listbox.insert(tk.END, f"Tarih: {kayit[0]}, Dosya 1: {kayit[1]}..., Dosya 2: {kayit[2]}..., Sonuç: {kayit[3]}...")

def veritabani_duzenle():
    conn = sqlite3.connect('metin_analizi.db')
    c = conn.cursor()
    c.execute("SELECT id, analiz_tarihi, substr(dosya1, 1, 50), substr(dosya2, 1, 50) FROM analiz_sonuclari")
    kayitlar = c.fetchall()
    conn.close()

    duzenleme_penceresi = tk.Toplevel(frame)
    duzenleme_penceresi.title("Veritabanı Kayıtlarını Düzenle")

    listbox = tk.Listbox(duzenleme_penceresi, width=100, height=10)
    listbox.pack()

    for kayit in kayitlar:
        listbox.insert(tk.END, f"ID: {kayit[0]}, Tarih: {kayit[1]}, Dosya 1: {kayit[2]}..., Dosya 2: {kayit[3]}...")

    def kayit_sec():
        selected = listbox.curselection()
        if selected:
            secilen_kayit = kayitlar[selected[0]]
            id = secilen_kayit[0]

            conn = sqlite3.connect('metin_analizi.db')
            c = conn.cursor()
            c.execute("SELECT dosya1, dosya2 FROM analiz_sonuclari WHERE id=?", (id,))
            kayit = c.fetchone()
            conn.close()

            dosya1_text.delete(1.0, tk.END)
            dosya1_text.insert(tk.END, kayit[0])
            dosya2_text.delete(1.0, tk.END)
            dosya2_text.insert(tk.END, kayit[1])

            def kayit_guncelle():
                yeni_dosya1 = dosya1_text.get(1.0, "end-1c")
                yeni_dosya2 = dosya2_text.get(1.0, "end-1c")

                newobject = MetinKarsilastir(yeni_dosya1, yeni_dosya2)
                yeni_sonuc = newobject.karsilastir()

                conn = sqlite3.connect('metin_analizi.db')
                c = conn.cursor()
                c.execute("UPDATE analiz_sonuclari SET dosya1=?, dosya2=?, sonuc=?, analiz_tarihi=? WHERE id=?",
                          (yeni_dosya1, yeni_dosya2, yeni_sonuc, datetime.now().strftime("%Y-%m-%d %H:%M:%S"), id))
                conn.commit()
                conn.close()
                duzenleme_penceresi.destroy()
                sonucText.config(state=tk.NORMAL)
                sonucText.delete(1.0, tk.END)
                sonucText.insert(tk.END, "Kayıt başarıyla güncellendi.")
                sonucText.config(state=tk.DISABLED)

            def kayit_sil():
                conn = sqlite3.connect('metin_analizi.db')
                c = conn.cursor()
                c.execute("DELETE FROM analiz_sonuclari WHERE id=?", (id,))
                conn.commit()
                conn.close()
                duzenleme_penceresi.destroy()
                sonucText.config(state=tk.NORMAL)
                sonucText.delete(1.0, tk.END)
                sonucText.insert(tk.END, "Kayıt başarıyla silindi.")
                sonucText.config(state=tk.DISABLED)

            guncelle_button.config(command=kayit_guncelle)
            sil_button.config(command=kayit_sil)
        else:
            sonucText.config(state=tk.NORMAL)
            sonucText.delete(1.0, tk.END)
            sonucText.insert(tk.END, "Lütfen düzenlemek veya silmek için bir kayıt seçin.")
            sonucText.config(state=tk.DISABLED)

    listbox.bind('<<ListboxSelect>>', lambda event: kayit_sec())

    dosya1_label = tk.Label(duzenleme_penceresi, text="Dosya 1:")
    dosya1_label.pack()
    dosya1_text = tk.Text(duzenleme_penceresi, height=10, width=80)
    dosya1_text.pack()

    dosya2_label = tk.Label(duzenleme_penceresi, text="Dosya 2:")
    dosya2_label.pack()
    dosya2_text = tk.Text(duzenleme_penceresi, height=10, width=80)
    dosya2_text.pack()

    guncelle_button = tk.Button(duzenleme_penceresi, text="Güncelle")
    guncelle_button.pack()

    sil_button = tk.Button(duzenleme_penceresi, text="Sil")
    sil_button.pack()

# Veritabanını oluştur
veritabani_olustur()

# Tkinter arayüzü
frame = tk.Tk()
frame.title("TextBox Input")

inputtxt1 = tk.Text(frame, height=15, width=80)
inputtxt2 = tk.Text(frame, height=15, width=80)

inputtxt1.grid(row=0, column=1)
inputtxt2.grid(row=1, column=1)

yukleButton1 = tk.Button(frame, text="Dosya 1 Yükle", command=yukleDosya1)
yukleButton1.grid(row=0, column=2)

yukleButton2 = tk.Button(frame, text="Dosya 2 Yükle", command=yukleDosya2)
yukleButton2.grid(row=1, column=2)

printButton = tk.Button(frame, text="Onayla", command=printInput)
printButton.grid(row=2, column=1)

veritabaniButton = tk.Button(frame, text="Veritabanını Görüntüle", command=veritabani_goruntule)
veritabaniButton.grid(row=2, column=2)

duzenleButton = tk.Button(frame, text="Veritabanını Düzenle", command=veritabani_duzenle)
duzenleButton.grid(row=2, column=3)

sonucText = tk.Text(frame, height=15, width=80)
sonucText.grid(row=3, column=1)
sonucText.config(state=tk.DISABLED)

frame.mainloop()
