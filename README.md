//8-bit-song
#include <Arduino.h>

#define BUZZER_PIN 32
#define BUTTON_PIN 18

#define C6  1047
#define D6  1175
#define E6  1319
#define F6  1397
#define G6  1568
#define A6  1760
#define B6  1976
#define C7  2093 
#define REST     0

int melody[] = {
  E6, D6, C6, D6, E6, E6, E6,
  D6, D6, D6,
  E6, G6, G6,
  E6, D6, C6, D6, E6, E6, E6, E6,
  D6, D6, E6, D6, C6
};

int noteDurations[] = {
  4, 4, 4, 8, 8, 4, 
  4, 8, 8,
  4, 4, 4, 8, 8, 4,
  4, 8, 8,
  4, 4, 4, 4, 
  4, 4, 4, 4,
  4, 8, 8, 4,
  4, 8, 8, 4,
  2, 2
};

int totalNotes = sizeof(melody) / sizeof(melody[0]);

float speedLevels[] = {2.0, 1.5, 1.0, 0.75, 0.5}; 
volatile int speedIndex = 2; 
volatile bool buttonPressed = false;

hw_timer_t * timer = NULL;
volatile bool toggleState = false;

void IRAM_ATTR onTimer() {
  toggleState = !toggleState;
  digitalWrite(BUZZER_PIN, toggleState);
}

void IRAM_ATTR onButtonPress() {
  static unsigned long lastInterruptTime = 0;
  unsigned long interruptTime = millis();
  
  if (interruptTime - lastInterruptTime > 200) { 
    speedIndex++;
    if (speedIndex >= 5) {
      speedIndex = 0; 
    }
  }
  lastInterruptTime = interruptTime;
}

void setup() {
  Serial.begin(115200);
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(BUTTON_PIN, INPUT_PULLUP);

  attachInterrupt(BUTTON_PIN, onButtonPress, FALLING);

  timer = timerBegin(1000000); 

  timerAttachInterrupt(timer, &onTimer);
  
}

void playNote(int frequency, int durationRaw) {
  if (frequency == 0) {
    timerAlarm(timer, 0, false, 0); 
    digitalWrite(BUZZER_PIN, LOW);
  } else {
    uint64_t ticks = 1000000 / (frequency * 2);
    timerAlarm(timer, ticks, true, 0);
  }
  int durationMs = (1000 / durationRaw) * speedLevels[speedIndex];
  
  delay(durationMs);

  timerAlarm(timer, 0, false, 0); 
  digitalWrite(BUZZER_PIN, LOW);
  delay(durationMs * 0.1); 
}

void loop() {
  for (int i = 0; i < totalNotes; i++) {
    playNote(melody[i], noteDurations[i]);
  }
  delay(1000); 
}
