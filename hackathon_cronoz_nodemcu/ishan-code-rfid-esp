#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <ArduinoJson.h>
#include <WiFiClientSecure.h> 
#include <SPI.h>
#include <MFRC522.h>

#define RST_PIN D1
#define SS_PIN D2

MFRC522 mfrc522(SS_PIN, RST_PIN);   // Create MFRC522 instance.
const char* ssid = "Shadow";
const char* password = "eliot123";
 
String URL = "https://sheetdb.io/api/v1/";
String Sheet_ID = "rgjonmpgew5r5";

const char* host = "sheetdb.io";
const int httpsPort = 443;

char UID[32] = "";
 
void array_to_string();




void setup () {
  
  Serial.begin(115200);

  
   SPI.begin();      // Initiate  SPI bus
   mfrc522.PCD_Init();   // Initiate MFRC522
   Serial.println("Approximate your card to the reader...");
  
  WiFi.begin(ssid, password);
  
  Serial.print("Connecting..");
  while (WiFi.status() != WL_CONNECTED) {
 
    delay(500);
    Serial.print(".");
 
  }

 
}
 
void loop() {
  
  //String UID = "5897";
  
  //String UID = rfid_get();
  

   // Select one of the cards
//   if ( ! mfrc522.PICC_ReadCardSerial()) {
//      return;
//   }

 
  if (WiFi.status() == WL_CONNECTED) { 
    Serial.println("Connected");
    WiFiClientSecure client; 
    client.setInsecure();
    if(!client.connect(host, httpsPort)){
      Serial.println("Connection Failed!");
      return;
    }
    while((!mfrc522.PICC_IsNewCardPresent()) || (!mfrc522.PICC_ReadCardSerial())){
      Serial.println("waiting for card");
      delay(500);
      }
      
//    if (  mfrc522.PICC_IsNewCardPresent()) {
//      if ( mfrc522.PICC_ReadCardSerial()) {
        
        array_to_string(mfrc522.uid.uidByte, 4, UID); //Insert (byte array, length, char array for output)
        Serial.println(UID); //Print the output uid string
        mfrc522.PICC_HaltA();
 //  }
 //  }
    
    
    HTTPClient http;  
    String link = URL + Sheet_ID + "/search?UID=" + UID;
    http.begin(client, link);  
    Serial.print("Link: ");
    Serial.println(link);  
    int httpCode = http.GET();  
   // Serial.println(httpCode);  
    if (httpCode > 0) { //Check the returning code
    
      String payload = http.getString();   //Get the request response payload
      Serial.println(payload);            //Print the response payload
      StaticJsonDocument<200> doc;

      deserializeJson(doc, payload);
     // client.stop();
      http.end();
      
      if (payload == "[]"){
        Serial.println("No data Found");
      }
      else{
        Serial.println(payload); 
        String mark = doc[0]["Mark"];
        Serial.println(mark);
        
        if ((mark =="A") || (mark == "-") || (mark == "")){
         // update_att(UID);


          String link = URL + Sheet_ID + "/UID/" + UID;
      
           // Your Domain name with URL path or IP address with path
           Serial.println(link);
           http.begin(client, link);
           http.addHeader("Content-Type", "application/json"); 
           
           StaticJsonDocument<200> doc;
        
           String att = "P";
        
           JsonArray val = doc.createNestedArray("data");
           JsonObject obj = val.createNestedObject();
        
           obj["Mark"] = String(att);
            /*
             {
              "data":
                  [
                      {   
                          "Mark": "A"
                      }
                  ]
              }
             */
        
          // serializeJsonPretty(val, http);
           char out[128];
           serializeJsonPretty(val, out);
           Serial.println(out);
           
           int httpResponseCode = http.PUT(out);
         
           if(httpResponseCode>0){
         
            String response = http.getString();   
         
            Serial.println(httpResponseCode);
            Serial.println(response);          
         
           }else{
         
            Serial.print("Error on sending PUT Request: ");
            Serial.println(httpResponseCode);
         
           }
           http.end();
           client.stop();
        }
        else{
          Serial.println("Incorrect Sheet Format");
        }
      }
      
        
    }
 
       //Close connection
 
  }
 
  delay(10000);    //Send a request every 30 seconds
}


  void array_to_string(byte array[], unsigned int len, char buffer[])
{
   for (unsigned int i = 0; i < len; i++)
   {
      byte nib1 = (array[i] >> 4) & 0x0F;
      byte nib2 = (array[i] >> 0) & 0x0F;
      buffer[i*2+0] = nib1  < 0xA ? '0' + nib1  : 'A' + nib1  - 0xA;
      buffer[i*2+1] = nib2  < 0xA ? '0' + nib2  : 'A' + nib2  - 0xA;
   }
   buffer[len*2] = '\0';
}


 
 
