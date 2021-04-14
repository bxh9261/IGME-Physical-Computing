# Network Communication

![](https://i.imgur.com/WCErLC2.jpg)

Like many on the RIT wi-fi, I was unable to get the device, completely connected, but I can walk through what I was able to get done.
<br>

## Network Scan
I used ScanNetworks.ino to get the MAC Address from the nano and other information about the wi-fi networks, the results are below (MAC Address omitted)

~~~
** Scan Networks **
number of available networks:10
0) RIT	Signal: -67 dBm	Encryption: Unknown
1) RIT-Legacy	Signal: -68 dBm	Encryption: None
2) eduroam	Signal: -69 dBm	Encryption: Unknown
3) RIT-Guest	Signal: -69 dBm	Encryption: None
4) RIT-Legacy	Signal: -73 dBm	Encryption: None
5) eduroam	Signal: -73 dBm	Encryption: Unknown
6) RIT-Guest	Signal: -73 dBm	Encryption: None
7) RIT	Signal: -74 dBm	Encryption: Unknown
8) DIRECT-UEML-2160	Signal: -83 dBm	Encryption: WPA2
9) eduroam	Signal: -86 dBm	Encryption: Unknown
~~~
<br>



## Connecting to RIT-Legacy
Unfortunately, I had some problems connecting to RIT-Legacy.
<br>

![](https://i.imgur.com/11saB3m.jpg)
This seems like a catch-22, since I can't connect to the RIT network *until* I do this.

~~~
Attempting to connect to Network named: RIT-Legacy
Attempting to connect to Network named: RIT-Legacy
Attempting to connect to Network named: RIT-Legacy
~~~

## Future Iterations
Perhaps when I am on my home wi-fi I will be able to connect, unfortunately I was unable to connect to the RIT-Legacy wi-fi to complete the assignment.

## Sources

WifiFadeServerClient code from W. Michelle Harris:
~~~
/*
   Fade_WifiServer

   Example of using an Arduino wifi to trade sensor readings with another Arduino
   Adaapted by W Michelle Harris, April 2021

   Carlos Castellanos
   August 17, 2020

   -----------------------------------------------------------------------------------

   Things that you need:
   - (2) wifi-enabled Arduinos (e.g. Arduino Uno Wifi Rev 2)
   - (2) analog sensors or potentiometer

   The Arduinos run as servers and reecive the sensor data from a client (the other Arduino)
   They then use one another's sensor data to change the brightness of the LED using PWM

*/

#include <SPI.h>
#include <WiFiNINA.h>

/*
   please enter your sensitive data in the arduino_secrets.h file
*/
char ssid[] = "RIT-Legacy";       // Name of WPA2 enterprise network (unused for WPA)
char user[] = SECRET_SSID;        // WPA2 enterprise username or WPA network name
const char pass[] = SECRET_PASS;  // WPA2 enterprise user password or WPA network password
const unsigned int keyIndex = 0;  // your network key Index number (needed only for WEP)

int status = WL_IDLE_STATUS;

int wifiPort = 2390;
WiFiServer server(wifiPort);         // create a server on port wifiPort

WiFiClient clientSelf;               // will be a web client
IPAddress thatIP(192, 168, 2, 2);    //ip address of other Nano to connect to (same port)

boolean alreadyConnected = false;    // whether or not the client was connected previously

const unsigned int ledPin = 9;       // the PWM pin the LED is attached to
const unsigned int sensorPin = 0;    // analog pin 0

void setup() {

  
  pinMode(ledPin, OUTPUT);
  Serial.begin(9600);

  // check the status of the wifi module
  if (WiFi.status() == WL_NO_MODULE) {
    Serial.println(F("Communication with WiFi module failed!"));
    // don't continue
    while (true);
  }

  String fv = WiFi.firmwareVersion();
  if (fv < WIFI_FIRMWARE_LATEST_VERSION) {
    Serial.println(F("Please upgrade the firmware"));
  }

  // attempt to connect to the wifi network
  while (status != WL_CONNECTED) {
    Serial.print(F("Attempting to connect to Network named: "));
    Serial.println(ssid);
    // Connect to WPA/WPA2 network.
    // If using a WEP network use WiFi.begin(ssid, keyIndex, key);
    // If using an open network use WiFi.begin(ssid);
    status = WiFi.begin(user, pass);
    //status = WiFi.beginEnterprise(ssid, user, pass);

    // wait 10 seconds for connection
    delay(10000);
  }

  Serial.println("Connected to wifi");
  Serial.print("SSID: ");
  Serial.println(WiFi.SSID()); // the SSID of the network you're connected to
  IPAddress ip = WiFi.localIP(); // this board's IP address
  Serial.print("My IP Address: ");  // Share this with your networking partner!
  Serial.println(ip);               //double check in browser as http://ip
  //You can use https://ipchicken.com to see your computer's public IP and infer it for the Nano

  // start the server
  Serial.println(F("\nStarting server..."));
  server.begin();

  // start the client
  httpRequest();
}

void loop() {
  /** Server code **/
  // wait for a new client
  WiFiClient clientOutside = server.available();

  // when the client sends data, reply with data from the server
  if (clientOutside) {
    if (!alreadyConnected) {
      // clean out the input buffer
      clientOutside.flush();
      Serial.println(F("We have a new client"));
      alreadyConnected = true;
    }

    // read the analog sensor value and send to client
    int outVal = analogRead(sensorPin);
    server.println(outVal);
    Serial.print("To client: ");
    Serial.println(outVal);
    delay(5); // slight delay to keep things stable
  }

  /*** Client code **/
  // if there are incoming bytes available
  // from the outside server, read them and print them:
  String inString = "";
  while (clientSelf.available() > 0) {
    char c = clientSelf.read();
    if (c != 10) { // look for a new line ('\n' or ASCII 10)
      inString += c;
    } else {
      break; //end of line, so break out of while
    }
  }
  //If received useful value, use it to set LED value
  if (inString.length() > 0) {
    Serial.print("Received as client: ");
    Serial.println(inString);
    int pwmVal = inString.toInt() / 4; //scale down
    analogWrite(ledPin, pwmVal);
  }

  if (!clientSelf.connected()) { //try to connect again
    delay(1000);
    httpRequest();
  }
}

// this method makes a HTTP client connection to the other server:
//via https://www.arduino.cc/en/Tutorial/LibraryExamples/WebClientRepeating
void httpRequest() {
  // close any connection before send a new request.
  clientSelf.stop();

  Serial.println("\nStarting connection to outside server...");
  // if you get a connection, report back via serial:
  //alternately if (clientSelf.connect("HarrisNano.wireless.rit.edu", wifiPort)) {
  if (clientSelf.connect(thatIP, wifiPort)) {
    Serial.println("connected to server");
    // Make a HTTP request:
    clientSelf.println("GET / HTTP/1.1");
    clientSelf.print("Host: ");
    clientSelf.println(thatIP);

  } else {
    Serial.println("Connection to outside server failed");
  }
}
~~~
