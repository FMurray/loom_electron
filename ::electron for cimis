//electron for cimis
//using the ADS1115 (pyranometer, anemometer) and SHT15 currently

// This #include statement was automatically added by the Particle IDE.
#include "SparkFunPhant/SparkFunPhant.h"

// This #include statement was automatically added by the Particle IDE.
#include "Adafruit_ADS1X15/Adafruit_ADS1X15.h"
#include "SHT1x/SHT1x.h"


// Specify data and clock connections and instantiate SHT1x object
#define dataPin  C0
#define clockPin C1
SHT1x sht1x(dataPin, clockPin);
Adafruit_ADS1115 ads;  /* Use this for the 16-bit version */
FuelGauge fuel; 

//phant stuff
const char server[] = "data.sparkfun.com"; // Phant destination server
const char publicKey[] = "QGq126n0ZNiMWVMzL7AN"; // Phant public key
const char privateKey[] = "JqkVwJE2YRF9bk91YPJZ"; // Phant private key
Phant phant(server, publicKey, privateKey); // Create a Phant object

const int POST_RATE = 30000; // Time between posts, in ms.
unsigned long lastPost = 0; // global variable to keep track of last post time

float temp_c;
float temp_f;
float humidity;
float light; 
float windspeed = 0.00; //not hooked up yet
float battery_level;
    
float multiplier = 0.1875F; //milli Volts per bit for ADS1115
 
float adc0, adc1, adc2, adc3;
float av0, av1, av2, av3;


void setup()
{
	Serial.begin(9600); 
	Serial.print("Ok!");
	ads.setGain(GAIN_TWOTHIRDS);  // 2/3x gain +/- 6.144V  1 bit =       3mV       0.1875mV (DEFAULT)
  // ads.setGain(GAIN_ONE);        // 1x gain   +/- 4.096V  1 bit =     2mV       0.125mV
  // ads.setGain(GAIN_TWO);        // 2x gain   +/- 2.048V  1 bit =     1mV       0.0625mV
  // ads.setGain(GAIN_FOUR);       // 4x gain   +/- 1.024V  1 bit =     0.5mV     0.03125mV
  // ads.setGain(GAIN_EIGHT);      // 8x gain   +/- 0.512V  1 bit =     0.25mV    0.015625mV
  // ads.setGain(GAIN_SIXTEEN);    // 16x gain  +/- 0.256V  1 bit =     0.125mV   0.0078125mV  
    ads.begin();
}

void loop()
{   
    if (postToPhant() > 0)
        {
            System.sleep(D0,RISING,900);
        }
}

void updateSensorVals() {
    adc0 = ads.readADC_SingleEnded(0);
    av0 = adc0 * multiplier;
    
    light = av0 * .025;  // conversion factor to w m^-2

    temp_f = sht1x.readTemperatureF();
    humidity = sht1x.readHumidity();
    battery_level = fuel.getSoC();
}

int postToPhant()
{
	// Use phant.add(<field>, <value>) to add data to each field.
	// Phant requires you to update each and every field before posting,
	// make sure all fields defined in the stream are added here.
	updateSensorVals();
	
    phant.add("battery_level", battery_level);
    phant.add("humidity", humidity);
    phant.add("light", light);
    phant.add("temp", temp_f);
    phant.add("windspeed", windspeed);
	
    TCPClient client;
    char response[512];
    int i = 0;
    int retVal = 0;
    
    if (client.connect(server, 80)) // Connect to the server
    {
		// Post message to indicate connect success
        Serial.println("Posting!"); 
		
		// phant.post() will return a string formatted as an HTTP POST.
		// It'll include all of the field/data values we added before.
		// Use client.print() to send that string to the server.
        client.print(phant.post());
        delay(1000);
		// Now we'll do some simple checking to see what (if any) response
		// the server gives us.
        while (client.available())
        {
            char c = client.read();
            Serial.print(c);	// Print the response for debugging help.
            if (i < 512)
                response[i++] = c; // Add character to response string
        }
		// Search the response string for "200 OK", if that's found the post
		// succeeded.
        if (strstr(response, "200 OK"))
        {
            Serial.println("Post success!");
            retVal = 1;
        }
        else if (strstr(response, "400 Bad Request"))
        {	// "400 Bad Request" means the Phant POST was formatted incorrectly.
			// This most commonly ocurrs because a field is either missing,
			// duplicated, or misspelled.
            Serial.println("Bad request");
            retVal = -1;
        }
        else
        {
			// Otherwise we got a response we weren't looking for.
            retVal = -2;
        }
    }
    else
    {	// If the connection failed, print a message:
        Serial.println("connection failed");
        retVal = -3;
    }
    client.stop();	// Close the connection to server.
    return retVal;	// Return error (or success) code.
}