

`
//Audio Visualizer - Brad Hanel

// constants won't change. Used here to set a pin number:
const int ledPins[7] = {4,5,6,7,8,9,10};
const int soundPin = 1;
const int buttonPin = 11;

// Variables will change:
int ledState = HIGH;             // ledState used to set the LED
int soundState = 0;
int buttonState = 0;
int soundLevel[100];
int loopSoundAvg = 0; 
int mode = 0;

// Generally, you should use "unsigned long" for variables that hold time
// The value will quickly become too large for an int to store
unsigned long previousMillis = 0;        // will store last time LED was updated

long interval = 1000;           // interval at which to blink (milliseconds)

void setup() {
  // set the digital pin as output:
  for (byte i = 0; i < 7; i = i + 1) {
    pinMode(ledPins[i], OUTPUT);
  }
  
  pinMode(soundPin, INPUT);
  pinMode(buttonPin, INPUT);
  
  digitalWrite(ledPins[0], HIGH);
  
  Serial.begin(9600);
}

void loop() {
  //get the state of the mic and button
  soundState = digitalRead(soundPin);
  buttonState = digitalRead(buttonPin);
  // check to see if it's time to blink the LED; that is, if the difference
  // between the current time and last time you blinked the LED is bigger than
  // the interval at which you want to blink the LED.
  unsigned long currentMillis = millis();
  
  //this gets the average of the last 100 sounds inputs. makes for a smoother visualization
  soundLevel[loopSoundAvg] = analogRead(soundPin);
  loopSoundAvg = loopSoundAvg + 1;
  if(loopSoundAvg > 99){
    loopSoundAvg = 0;
  }
  
  //calculating the average of the last 100 inputs
  int sum = 0;
  for (byte i=0; i<100; i+=1){
     sum += soundLevel[i];
  }
  int avg = sum/100;
  
  //switches between 2 modes of audio visualization
  if(buttonState == HIGH){
    if(mode == 0){
      mode = 1;
      digitalWrite(ledPins[0],HIGH);
      for(int i = 1; i < 7; i+=1){
        digitalWrite(ledPins[i], LOW);
      }
    }else{
      mode = 0;
    }
  }

  
  if(mode == 0){
    //audio level mode- light up 1 LED for every 25 units of sound (is it dB?)
    for (byte i = 0; i < 7; i = i + 1){
      if(avg > (25*(i+1))){
        digitalWrite(ledPins[i],HIGH);
      }
      else{
        digitalWrite(ledPins[i],LOW);
      }
    }
  }else{
    //light path mode- cyccle through LEDs faster as sound level increases
    interval = 1000 - (avg * 7);
    if(interval < 20){
      interval = 20;
    }
    
    if (currentMillis - previousMillis >= interval) {
      // save the last time you blinked the LED
      previousMillis = currentMillis;
    
      // if the LED is on, turn on the next one
      for (byte i = 0; i < 7; i = i + 1) {
        if(digitalRead(ledPins[i]) == HIGH){
          digitalWrite(ledPins[i], LOW);
          if(i == 6){
            digitalWrite(ledPins[0],HIGH);
          }
          else{
            digitalWrite(ledPins[i+1],HIGH);
          }
          break;
        }
      }
    }
  }
  
}
`
