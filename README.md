#define BUZZER 32  
#define BUTTON 35  

#define NOTE_C4  262
#define NOTE_D4  294
#define NOTE_E4  330
#define NOTE_F4  349
#define NOTE_G4  392
#define NOTE_A4  440
#define NOTE_AS4 466
#define NOTE_C5  523


int melody[] = {
  NOTE_C4, NOTE_C4, 
  NOTE_D4, NOTE_C4, NOTE_F4,
  NOTE_E4, NOTE_C4, NOTE_C4, 
  NOTE_D4, NOTE_C4, NOTE_G4,
  NOTE_F4, NOTE_C4, NOTE_C4,
  
  NOTE_C5, NOTE_A4, NOTE_F4, 
  NOTE_E4, NOTE_D4, NOTE_AS4, NOTE_AS4,
  NOTE_A4, NOTE_F4, NOTE_G4,
  NOTE_F4
};

int durations[] = {
  4, 8, 
  4, 4, 4,
  2, 4, 8, 
  4, 4, 4,
  2, 4, 8,
  
  4, 4, 4, 
  4, 4, 4, 8,
  4, 4, 4,
  2
};

int tempo = 134;
int maintempo = 134;

int wholenote = (60000 * 4) / tempo;

int divider = 0, noteDuration = 0;

// Variables for timer interrupt
hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

// Variable to store the tempo of the song


// Variable to store the current speed level
int speed = 1;

void IRAM_ATTR onTimer() {
    // Toggle the state of the buzzer pin
    digitalWrite(BUZZER, !digitalRead(BUZZER));
}

void IRAM_ATTR onButton() {
    // Check the state of the button
    if (digitalRead(BUTTON) == LOW) {
        // Increase the speed level
        if(speed <= 5) speed++;
        else speed = 1;
    }
    // Change the tempo based on the current speed level
    switch (speed) {
        case 1:
            tempo = maintempo / 2;
            break;
        case 2:
            tempo = maintempo / 1.5;
            break;
        case 3:
            tempo = maintempo / 1;
            break;
        case 4:
            tempo = maintempo * 1.5;
            break;
        case 5:
            tempo = maintempo * 2;
            break;
    }
}

void setup() {
    pinMode(BUZZER, OUTPUT);
    pinMode(BUTTON, INPUT);
  
    // Set up the timer interrupt
    timer = timerBegin(0, 80, true);
    timerAttachInterrupt(timer, &onTimer, true);

    // Set up the I/O interrupt
    attachInterrupt(BUTTON, onButton, FALLING);
    play();
}
void loop() {
  //blank
}

void play(){
  for (int i = 0; i < sizeof(melody) / sizeof(int); i++) {
    
    wholenote = (60000 * 4) / tempo;
    
    divider = durations[i];
    if (divider > 0) {
      
      noteDuration = (wholenote) / divider;
    } else if (divider < 0) {
      
      noteDuration = (wholenote) / abs(divider);
      noteDuration *= 1.5; 
    }
    
    
    tone(BUZZER, melody[i], noteDuration * 0.9); 
    delay(noteDuration * 0.9); 
    noTone(BUZZER); 
    delay(noteDuration * 0.1); 
  }
}

