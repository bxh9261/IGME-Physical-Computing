# Stoplight - Digital In/Outs

![](https://i.imgur.com/s2MnQ1F.jpg)

Stoplight is an experiment with digital in/outs where the speed at which the red, yellow, and green lights flash is controlled by the "gas" (white button) and "brake" (yellow button).
<br>

## Original idea
My original plan for this setup was to make a sort of game where the user had to press the brake when the red light was on, and the gas when the green light wason. I had pre-programmed timers with the random() function for the stoplight, but where this idea failed was that I was unable to figure out a way to have multiple threads in the arduino. I couldn't similtaneously run the stoplight timers with the delay() function and check for input from the buttons. I'd be interested to hear if there's a way to do this.
<br>

## Final Product
![](https://i.imgur.com/slEFxaa.mp4)

My final design was a small light show with the 3 LEDs that can be controlled by the "gas" and "brake" buttons. With this, I was able to keep the same setup as my original idea and just change the code. Holding the brake button will slow down the time each light flashes; it is capped at 1.5 seconds. The gas button will slowly speed it up, with a minimum flash time of 0.1 seconds. 
<br>

~~~
// constants won't change. They're used here to set pin numbers:
const int brake = 2;     // the number of the pushbutton pin
const int gas = 12;
const int ledPinG = 5;      // the number of the LED pin
const int ledPinY = 7;
const int ledPinR = 9;


// variables will change:
int brakeState = 0;
int gasState = 0;
int lightSpeed = 500; 

void setup() {
  // initialize the LED pin as an output:
  pinMode(ledPinG, OUTPUT);
  pinMode(ledPinY, OUTPUT);
  pinMode(ledPinR, OUTPUT);
  // initialize the pushbutton pin as an input:
  pinMode(brake, INPUT);
  pinMode(gas, INPUT);
}

void loop() {
  // read the state of the pushbutton value:
  brakeState = digitalRead(brake);
  gasState = digitalRead(gas);
  
  //speed UP, min 0.1 seconds
  if(gasState == HIGH){
    lightSpeed-=200;
    if(lightSpeed < 100){
      lightSpeed = 100;
    }
  }
  //slow down, max 1.5 seconds
  if(brakeState == HIGH){
    lightSpeed+=200;
    if(lightSpeed > 1500){
      lightSpeed = 1500;
    }
  }
  
  stoplight(lightSpeed,lightSpeed,lightSpeed);
  
}

//loop lights
void stoplight(int G, int Y, int R){
      digitalWrite(ledPinR, LOW);
      digitalWrite(ledPinY, LOW);
      digitalWrite(ledPinG, HIGH);   // turn the LED on (HIGH is the voltage level)
      delay(G); 
      digitalWrite(ledPinG, LOW);
      digitalWrite(ledPinY, HIGH);    // turn the LED off by making the voltage LOW
      delay(Y);
      digitalWrite(ledPinY, LOW);
      digitalWrite(ledPinR, HIGH);
      delay(R);
}
~~~

## Future Iterations
I may play around with this more and try to figure out how to make the stoplight game work. If that doesn't work, I could also mess around with longer light strings or perhaps switches to reverse the direction of the light loop. 
