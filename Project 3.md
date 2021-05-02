# Project 3

# Goal

We planned to expand on Nyxara's original project 2 to instead simulate 2 stuffed animals that can communicate with each other over wi-fi. This would create an experience in which two people can communicate through 2 stuffed animals from different places. This is especially relevant to quarantine, as many people have been separated due to COVID restrictions, and this would create a way for them to communicate (beyond, y'know, talking on the phone or something, but I don't know if I'm capable of building a Samsung Galaxy).

# Description

The project has two inputs- banana clips hooked up to tin foil, and an analog microphone. There are also two outputs- the speakers and the LEDs. In its full, non-prototype form, there are two tin foil pieces inside of a stuffed animal, and hugging the stuffed animal will cause the curcuit to complete and the speaker to go off on the other stuffed animal. Similarly, talking to the stuffed animal through the microphone will activate an LED on the other stuffed animal. If the two stuffed animals are far apart, a sound would indicate that the other person is hugging the stuffed animal and an LED would indicate they are talking to it.

# Process

We began by hooking up the speakers Nyxara bought, and tested them by adapting my audio visualizer code to work with the new speakers. Originally, we tested the speakers with a simple button push, but we then messed aroud with the banana clips to create a different input. We realized we could activate the speaker simply by closing the curcuit, so we got some Wendy's and used to tin foil to create an input that works by pushing two pieces of tin foil together. We then hooked up the microphone, and spent a whole getting that working properly with the LED (some of my code from the audio visualizer helped). We then worked on attempting to have our arduinos talk to each other via wi-fi, which is still a real pain on RIT internet.

# Photo and Demo Video

We costumed the speakers to look like flowers :)

![Flowers](https://i.imgur.com/610pYq8.jpg)

[![Demo Video](http://img.youtube.com/vi/CrbzlDGKOp0/0.jpg)](https://www.youtube.com/watch?v=CrbzlDGKOp0)

Board without wire masking
![Board without wire masking](https://i.imgur.com/Z75ywoB.jpg)

# Fritzing

![Fritzing Diagram](https://i.imgur.com/JbKnqp8.png)

# Code

`
 //Bee Hanel and Nyxara Daniels

 // constants won't change. Used here to set a pin number:
 const int speeker = 9;
 const int led = 8;
 const int soundPin = 1;
 const int touchPin = 2;

 // Variables will change:
 int ledState = HIGH;             // ledState used to set the LED
 int soundState = 0;
 int touchState = 0;
 int soundLevel[100];
 int loopSoundAvg = 0; 

 // Generally, you should use "unsigned long" for variables that hold time
 // The value will quickly become too large for an int to store
 unsigned long previousMillis = 0;        // will store last time LED was updated

 long interval = 1000;           // interval at which to blink (milliseconds)

 void setup() {
   // set the digital pin as output:
     pinMode(speeker, OUTPUT);
     pinMode(led, OUTPUT);

   pinMode(soundPin, INPUT);

   digitalWrite(speeker, HIGH);

   Serial.begin(9600);
 }

 void loop() {
   //get the state of the mic and button
   soundState = digitalRead(soundPin);
   touchState = digitalRead(touchPin);
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
   
    digitalWrite(speeker, LOW);
    digitalWrite(led, LOW);
     
    Serial.println(avg);
     //audio level mode- light up 1 LED for every 25 units of sound (is it dB?)
       if(avg > (50)){
         digitalWrite(led,HIGH);
       }
       else{
         digitalWrite(led,LOW);
       }
       
      if(analogRead(touchPin) < 50){
         digitalWrite(speeker,HIGH);
       }
       else{
         digitalWrite(speeker,LOW);
       }

 }
`
