void setupFingerprint(); 
uint8_t getFingerprintID(); 
void enrollFingerprint(uint8_t id); 
void beepSuccess(); 
void beepFail(); 
void showAttendance(uint8_t id); 

// ----- SETUP -----
void setup() {

  pinMode(BUZZER_PIN, OUTPUT);
  digitalWrite(BUZZER_PIN, LOW);

  Serial.begin(9600);
  while (!Serial);

  lcd.init();
  lcd.backlight();
  lcd.clear();

  lcd.setCursor(0,0);
  lcd.print("Fingerprint Attn");

  lcd.setCursor(0,1);
  lcd.print("Initializing...");
  delay(1000);

  // RTC Initialization
  if (!rtc.begin()) {
    lcd.clear();
    lcd.print("RTC not found!");
    Serial.println("RTC not found");
  }
  else {
    if (rtc.lostPower()) {
      Serial.println("RTC lost power");
      rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
    }
  }

  // Fingerprint sensor setup
  setupFingerprint();

  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Ready to Scan");

  lcd.setCursor(0,1);
  lcd.print("Place finger...");
}


// ----- MAIN LOOP -----
void loop() {

  if (Serial.available()) {

    String s = Serial.readStringUntil('\n');
    s.trim();

    if (s.startsWith("enroll")) {

      int sp = s.indexOf(' ');

      if (sp > 0) {

        uint8_t id = (uint8_t)s.substring(sp+1).toInt();

        if (id > 0) {

          Serial.print("Enrolling ID ");
          Serial.println(id);

          lcd.clear();
          lcd.print("Enroll ID ");
          lcd.print(id);

          enrollFingerprint(id);

          lcd.clear();
          lcd.print("Ready to Scan");
          lcd.setCursor(0,1);
          lcd.print("Place finger...");
        }
      }
    }
  }

  uint8_t id = getFingerprintID();

  if (id != 0) {

    showAttendance(id);

    delay(2000);

    lcd.clear();
    lcd.print("Ready to Scan");

    lcd.setCursor(0,1);
    lcd.print("Place finger...");
  }

  delay(200);
}


// ----- FINGERPRINT SENSOR SETUP -----
void setupFingerprint() {

  fingerSerial.begin(57600);
  finger.begin(57600);

  if (finger.verifyPassword()) {
    Serial.println("Fingerprint sensor found!");
  }
  else {

    Serial.println("Fingerprint sensor not found");

    lcd.clear();
    lcd.print("Sensor Error");
  }
}
