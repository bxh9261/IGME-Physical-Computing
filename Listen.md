# Goal

My goal was to create some sort of physical prototype designed to invoke the power of music. I took some inspiration from [Wintergatan's 2000 Marble Instrument](https://www.youtube.com/watch?v=IvUU8joBb1Q) and attempted to make a little marble piano of my own. I had initially planned to use the microphone to get the frequency and decide which note to play, but I had trouble implementing it, so the microphone instead adjusts the tempo.

# Final Product

I created a marble xylophone using Unity that drops marbles from somewhere offscreen directly onto a xylophone (in the prototype, thanks to the magic of unity, they disappear, but in actuality they'd probably bounce off and onto the floor, so there would need to be something built to catch them, much like it Wintergatan's machine). The rate at which marbles are dropped is controlled by the microphone. The speed of marbles falling creates emotional changes-- louder input leads to faster/more energetic music. There is a button input which switches the instrument from a major to minor scale. This changes the tone of the music from happy to somber.

LEDs light up similarly to Project 1 to create visual feedback, the sounds level should be apparent from the speed of the marbles but it is also made clear via LEDs.

# Troubles

Due to the limitations of my own PC, the marbles do not fall very smoothly in Unity, on a better computer it'd look a bit more smooth. I was unable to figure out a way to get the frequency of the microphone to affect the note being played (and getting the frequency of each note as well as adjusting for changes in octave would have been incredibly tedious). 

# Photo and Demo Video

[![Marble Xylo](https://img.youtube.com/vi/_VZV-ZmFrZQ/0.jpg)](https://www.youtube.com/watch?v=_VZV-ZmFrZQ "Marble Xylo")

![Unity Screenshot](https://i.imgur.com/dALMq6W.jpg)

![Breadboard with LEDs](https://cdn.discordapp.com/attachments/578323240045248513/826195693369557043/Snapchat-290187232.jpg)

# Code

Arduino: 

     //Marble Xylophone -  Brad "Bee" Hanel

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


         int highest = 0;
         //audio level mode- light up 1 LED for every 25 units of sound (is it dB?)
         for (byte i = 0; i < 7; i = i + 1){
           if(avg > (25*(i+1))){
             digitalWrite(ledPins[i],HIGH);
             highest = i+1;
           }
           else{
             digitalWrite(ledPins[i],LOW);
           }
         }

         Serial.println(highest);

         if(digitalRead(buttonState) == HIGH){
           Serial.println("M");
         }

     }
     
Unity:

      using System.Collections;
      using System.Collections.Generic;
      using UnityEngine;
      using UnityEngine.SceneManagement;
      using System.IO.Ports;

      public class GameManager : MonoBehaviour
      {
          public GameObject platObjMaj;
          public GameObject platObjMin;
          GameObject marb;
          public List<GameObject> platforms;
          List<int> randomNotes;
          float[] xNotes = new float[8];
          int currPlatform = 0;
          public List<AudioSource> audioDataMajor;
          public List<AudioSource> audioDataMinor;
          bool isMajor;
          string value = "0";
          int intValue = 0;

          int randomIndex = 0;

          public float Timer = 2f;

          SerialPort stream = new SerialPort("COM8", 9600);

          // Start is called before the first frame update
          void Start()
          {
              stream.Open();

              isMajor = true;

              randomNotes = new List<int>();

              for (int i = 0; i < 100; i++)
              {
                  randomNotes.Add(Random.Range(0, 8));
              }

              //int counterdebug = 0;
              platforms = new List<GameObject>();
              for (int i = 0; i < 8; i++) {
                  xNotes[i] = -4.75f + (1.5f * i);
              }
          }

          // Update is called once per frame
          void Update()
          {

              for (int i = 0; i < platforms.Count; i++)
              {
                  moveDown(platforms[i]);
              }

              if (Input.GetKeyDown(KeyCode.R))
              {
                  SceneManager.LoadScene("TPW Game");
              }

              for (int i = 0; i < 100; i++)
              {

                  value = stream.ReadLine();
                  if(value == "M")
                  {
                      isMajor = !isMajor;
                  }
                  else
                  {
                      intValue = 0;
                      int.TryParse(value, out intValue);
                  }
              }

              Timer -= Time.deltaTime;
              if (Timer <= 0f)
              {
                  if (isMajor)
                  {
                      marb = Instantiate(platObjMaj, new Vector3(xNotes[randomNotes[randomIndex]], -1, 0), Quaternion.identity);
                  }
                  else
                  {
                      marb = Instantiate(platObjMin, new Vector3(xNotes[randomNotes[randomIndex]], -1, 0), Quaternion.identity);
                  }
                  platforms.Add(marb);
                  if (intValue == 0)
                  {
                      Timer = 2f;
                  }
                  else
                  {
                      Timer = 1.5f/(intValue*1.5f);

                  }

                  randomIndex++;
                  if(randomIndex > 99)
                  {
                      randomIndex = 0;
                  }

              }

          }

          private void moveDown(GameObject go)
          {

              float translation = (Time.deltaTime * 10.0f);
              go.transform.position = new Vector3(go.transform.position.x, go.transform.position.y - translation, go.transform.position.z);

              if (go.transform.position.y < -5)
              {
                  playNoteFor(go);
                  platforms.Remove(go);
                  Destroy(go);
              }
          }

          private void playNoteFor(GameObject go)
          {
              for(int i = 0; i < xNotes.Length; i++)
              {
                  if (go.transform.position.x == xNotes[i])
                  {
                      if (isMajor)
                      {
                          audioDataMajor[i].Play(0);
                      }
                      else
                      {
                          audioDataMinor[i].Play(0);
                      }

                  }
              }
          }
      }
      
# Fritzing

![Fritzing Diagram](https://i.imgur.com/j9OGAi5.jpg)

# Notes/Refs/Sources

[Unity Reference](https://docs.unity3d.com/ScriptReference/AudioSource.Play.html)

Referenced some Unity code from my old [Transnational Paris Workshop class](https://github.com/bxh9261/TPW-Group-3-Game) (a group project, but I worked solely on the Unity part).

