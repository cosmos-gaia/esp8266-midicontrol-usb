 # ESP8266 Midi Controller via USB

<img src='https://i.imgur.com/fmmPZHt.png' alt='midi control logo' width='200' />

> [!WARNING]
> This code are writted by *Gemini 2.5 Flash*. Also this is an experimental prototype and Im just a newbie in Arduino's and ESP8266.   

### *Wiring and code you can see below.*     
### *This project just need an ESP8266, one button and one 10K potentiometer.*      


## About this DIY
This is just a simple project that controls Midi and CC (Control Change) in your DAW, completely **WITHOUT *MIDIUSB*** library.   
All `Serial.w&p`  are translated by [Hairless](https://projectgus.github.io/hairless-midiserial/) to MIDI and forwarded to virtual midi port, you can use [loopMidi](https://www.tobias-erichsen.de/software/loopmidi.html) as *virtual port* and select it on your DAW.   
And make sure to **NOT** uncomment the `Serial.printIn` line, if you do, that's will also translated by *Hairless* and mess up the MIDI message, random notes will be occur.    
Don't forget to reinstall your CH341/CH340 driver. Just reinstall. Why not?    
If it works without update, it works.

<sub> *`Serial.w&p` mean `Serial.write` and `Serial.print`

## Parts

• Breadboard    
• ESP8266 (im using Wemos D1 Mini)    
• Male to Male jumper cable   
• MicroUSB cable   
• 1x 10KΩ Potentiometer    
• 1x Button/tactile switch   

## Wiring diagram 

<img src='https://i.imgur.com/rXheDXD.png' alt='Wiring diagram' width='300' height='300'/>      

I know my diagram isn't neat as you expect, im not using proper software (im too lazy to turn on my laptop bruh)    
If you looking for wiring photo on breadboard, here : 
### [Breadboard photo](https://i.imgur.com/4xvGKTc.jpeg)



## Before you go..
> [!CAUTION]
> **NEVER EVER** to uncomment the `Serial.printIn` line, it'll absolutely MESS UP the MIDI message. **DONT. EVER. DO. THAT.**
  And also..

> [!IMPORTANT]
> The comments are Indonesian. Maybe a little dictionary may help you.    

## Basic words
Pengaturan = configuration, settings   
Tombol = buttons      
Variabel = variable      
Untuk = for     
Potensio = potentio      
Umum = general     
Konfigurasi = configuration, settings. Same as 'pengaturan'.     
Logika = logic        
Menyimpan = saving     
Nilai = value      
Awal = beginning     
Jika = if     
Ditekan = pressed     
Cek = check      
Apakah = is      
Sedikit = a little      
Mencegah = preventing, prevent     
Kirim = send      
Waktu = time       
Dan = and     
Baca = read      
Berikutnya = next     

I think you're good to go!

## Code

```
// --- PENGATURAN TOMBOL ---
const int BUTTON_PIN = D1; // Pin D1 pada WeMos D1 Mini (GPIO 5) untuk tombol
const byte MIDI_NOTE = 60; // MIDI Note Number (C4)
const byte NOTE_VELOCITY = 100; // Velocity untuk Note On
const int DEBOUNCE_DELAY = 50; // Durasi debounce dalam milidetik

// Variabel untuk debouncing tombol
unsigned long lastDebounceTime = 0;
bool buttonState = HIGH;      // Status tombol yang sudah didebounce (HIGH = tidak ditekan)
bool lastButtonState = HIGH;  // Status tombol terakhir yang dibaca (raw)
bool noteIsOn = false;        // True jika not sedang berbunyi

// --- PENGATURAN POTENSIOMETER ---
const int POT_PIN = A0;      // Pin analog A0 pada WeMos D1 Mini untuk potensiometer
const byte CC_NUMBER = 7;    // MIDI Control Change Number (misal: Volume)
const int CHANGE_THRESHOLD = 2; // Threshold perubahan nilai potensiometer

// Variabel untuk menyimpan nilai potensiometer
int previousPotValue = 0;
int currentPotValue = 0;

// --- PENGATURAN UMUM MIDI ---
const byte MIDI_CHANNEL = 0; // MIDI Channel 0-15 (berarti Channel 1-16 di DAW)

void setup() {
  Serial.begin(115200); // Inisialisasi Serial untuk Hairless MIDI

  // Konfigurasi pin tombol
  pinMode(BUTTON_PIN, INPUT_PULLUP); // Gunakan INPUT_PULLUP internal WeMos

  // Inisialisasi nilai potensiometer awal
  previousPotValue = analogRead(POT_PIN);
}

void loop() {
  // --- LOGIKA TOMBOL ---
  int reading = digitalRead(BUTTON_PIN); // Baca status tombol (raw)

  // Cek apakah status tombol (raw) berubah
  if (reading != lastButtonState) {
    lastDebounceTime = millis(); // Reset timer debounce
  }

  // Jika waktu debounce sudah lewat DAN status tombol stabil berbeda dari status yang tersimpan
  if ((millis() - lastDebounceTime) > DEBOUNCE_DELAY) {
    if (reading != buttonState) {
      buttonState = reading; // Update status tombol stabil

      // Jika tombol ditekan (LOW) dan not belum berbunyi
      if (buttonState == LOW && !noteIsOn) {
        // Kirim pesan Note On
        Serial.write(0x90 | MIDI_CHANNEL); // Status Byte: Note On + Channel
        Serial.write(MIDI_NOTE);           // MIDI Note Number
        Serial.write(NOTE_VELOCITY);       // Velocity
        // Serial.println("Note On"); // Komentari untuk Hairless MIDI
        noteIsOn = true;                   // Set status not menjadi ON
      }
      // Jika tombol dilepas (HIGH) dan not sedang berbunyi
      else if (buttonState == HIGH && noteIsOn) {
        // Kirim pesan Note Off
        Serial.write(0x80 | MIDI_CHANNEL); // Status Byte: Note Off + Channel
        Serial.write(MIDI_NOTE);           // MIDI Note Number
        Serial.write(0);                   // Velocity (biasanya 0 untuk Note Off)
        // Serial.println("Note Off"); // Komentari untuk Hairless MIDI
        noteIsOn = false;                  // Set status not menjadi OFF
      }
    }
  }
  lastButtonState = reading; // Simpan status tombol (raw) untuk iterasi berikutnya

  // --- LOGIKA POTENSIOMETER ---
  currentPotValue = analogRead(POT_PIN); // Baca nilai analog potensiometer

  // Cek apakah ada perubahan signifikan pada potensiometer
  if (abs(currentPotValue - previousPotValue) > CHANGE_THRESHOLD) {
    // Petakan nilai analog (0-1023) ke rentang MIDI (0-127)
    int midiValue = map(currentPotValue, 0, 1023, 0, 127);

    // Kirim pesan MIDI Control Change
    Serial.write(0xB0 | MIDI_CHANNEL); // Status Byte: Control Change + Channel
    Serial.write(CC_NUMBER);           // CC Number
    Serial.write(midiValue);           // CC Value

    // Serial.print("CC "); // Komentari untuk Hairless MIDI
    // Serial.print(CC_NUMBER);
    // Serial.print(": ");
    // Serial.println(midiValue); // Komentari untuk Hairless MIDI

    previousPotValue = currentPotValue; // Update nilai sebelumnya
  }
  
  // Sedikit delay untuk mencegah WeMos terlalu cepat memproses loop
  // Ini penting agar Serial buffer tidak penuh dan memberikan waktu untuk sistem lain.
  delay(1);
}

```



