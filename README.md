# duallocking
#include <ESP8266WiFi.h>
int touch_status=0; 
const char* ssid = "nopassword";
const char* password = "nopassword" ;
int ledPin =D7; // GPIO13---D7 of NodeMCU
WiFiServer server(80);
 //int b=0,l=0,a=0;
 int returnValue=0;
void setup() {
  Serial.begin(9600);
  delay(10);
//one led to d7,other to d6,touch sensor to d1
  pinMode(ledPin, OUTPUT);
  digitalWrite(ledPin, LOW);
  pinMode(D1,INPUT);
 pinMode(D6,OUTPUT);
 // Connect to WiFi network
  Serial.println();
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);
 
  WiFi.begin(ssid, password);
 
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("");
  Serial.println("WiFi connected");
  // Start the server
  server.begin();
  Serial.println("Server started");
  // Print the IP address
  Serial.print("Use this URL to connect: ");
  Serial.print("http://");
  Serial.print(WiFi.localIP());
  Serial.println("/");
  }
 
void loop() {
  // Check if a client has connected
  int touch_sense=digitalRead(D1);
  
  Serial.println(touch_sense);
  Serial.println(touch_status);

  
  WiFiClient client = server.available();
  if (!client) {
    return;
  }
 
  // Wait until the client sends some data
  Serial.println("new client");
  while(!client.available()){
    delay(1);
  }
 
  // Read the first line of the request
  String request = client.readStringUntil('\r');
  Serial.println(request);
  client.flush();
 // Match the request
  int value = LOW;
  if (request.indexOf("/LED=ON") != -1)  {
    digitalWrite(ledPin, HIGH);
  value = HIGH;
   touch_status=1;
  }
  if (request.indexOf("/LED=OFF") != -1)  {
    digitalWrite(ledPin, LOW);
    value = LOW;
    touch_status=0;
  }
  delay(1000);
Serial.println(touch_status);
  Serial.println(touch_sense);
  if(touch_status==1 && touch_sense==1)
{ 
  digitalWrite(D6,HIGH);
  
}
if(touch_sense==0)
{
  digitalWrite(D6,LOW);
  }

  
 
// Set ledPin according to the request
//digitalWrite(ledPin, value);
 
  // Return the response
  client.println("HTTP/1.1 200 OK");
  client.println("Content-Type: text/html");
  client.println(""); //  do not forget this one
  client.println("<!DOCTYPE HTML>");
  client.println("<html>");
 
  client.print("Led is now: ");
 
  if(value == HIGH) {
    client.print("On");
  } else {
    client.print("Off");
  }
  client.println("<br><br>");
  client.println("<a href=\"/LED=ON\"\"><button>On </button></a>");
  client.println("<a href=\"/LED=OFF\"\"><button>Off </button></a><br />");  
  client.println("</html>");
 
  delay(1);
  Serial.println("Client disonnected");
  Serial.println("");
 
}
