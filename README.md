# Automatic sunshade manual 
This is a manuel for combining information about the temperature inside your home and comming weather to make a sun shade automatic

Requirements: 
- Arduino IDE
- NodeMCU ESP8266
- BMP280
- Wifi / Hotspot 

## 1. Measuring the tempeture with BMP280
Connect the BMP280 to the NodeMCU board. Start by connecting the VCC(BMP280) pin to the 3.3V(ESP8266) output and the GND(BMP280) to the ground(ESP8266). 
Than connect the SCL(BMP280) to the D1(ESP8266) pin and the SDA(BMP280) to D2(ESP8266). 

Open your library manager with Sketch > include library > manage libraries, or the library icon in the left side menu bar as seen in the screenshot. Then install the following libraries: `Adafruit BMP280 Library` and `Adafruit Unified Sensor` both by Adafruit. 
![Group 19](https://github.com/user-attachments/assets/24df21c7-1c9f-4dfc-9f8f-3b57b513164d)
![Group 20](https://github.com/user-attachments/assets/437e4520-281c-4ace-b215-981c2b7f3216)

When installing the libraries pay attention that you don't exedently download the library for the BME280, this is a very simular device but it works just a bit different where the code wont work. 

![Schermafbeelding 2024-10-18 091911](https://github.com/user-attachments/assets/713a8f97-328a-40b2-a1c0-2837c12eccca)

If you have the wrong library you will probebly get stuck when the ESP8266
is trying to connect with the BMP280 but won't find it, and get the following error / message: 

![Schermafbeelding 2024-10-17 214524](https://github.com/user-attachments/assets/06d4eb79-5631-4476-ae34-a467e8bb8985)

Then you can test your BMP280 with the example code, that you can find by file > examples > Adafruit BMP280 Library (might need to scroll down) > **bmp280test**. Where you only have to change line 37 from `status = bmp.begin();` to `status = bmp.begin(0x76);`. 
![Group 18](https://github.com/user-attachments/assets/0218cd4a-c74f-4a3b-b37d-0ce3ce2f48c6)

Check if your BAUD matches te one in the code. 
![Group 21](https://github.com/user-attachments/assets/302cb65a-3493-444d-85c6-c60b97e4521e)

Witch if every thing works correct you will get the following message in your serialmonitor:
![screenshot19239101](https://github.com/user-attachments/assets/f946d4df-d4d1-43c8-a856-0c8692acd287)

## Weather forecaster with openweathermap API 
To predict the upcomming weather you can use [OpenWeather](https://openweather.co.uk/). Once there start by creating (or loggin in) an account: 

Then navigate to the "my API keys" tab, like this screenshot: 
![Group 23](https://github.com/user-attachments/assets/03ec4838-ad4e-465e-a3b8-dd01b509046a)

Then generate a new API Key, and save this key for later: 
![Group 22](https://github.com/user-attachments/assets/890b2bb1-924e-4366-9fa3-a4340ea446b3)

After you have created your account, you need to instal another library, `ArduinoJson` by Benoit Blanchon. With that we can use the code Emmanuel Odunlade wrote [here](https://randomnerdtutorials.com/esp8266-weather-forecaster/) as a start, but have to change a bit to make it work. 


But what you will see is that this code still uses ArduinoJson 5, what needs to be changed to ArduinoJson 6+. Wich i couldn't really figure out. I tried to do the following things: 
1. I started by adjusting the buffer size. Initially, it was set to 2500, but changing it didn’t resolve the issue.
2. I used [this guide](https://arduinojson.org/v6/doc/upgrade/) to modify the JsonBuffer, in this article they explain the changes from ArduinoJson 5 and 6 resulting in the following code:
```
void parseJson(const char * jsonString) {
  // Define the size of the JSON document based on the expected size of the JSON object.
  const size_t bufferSize = 2 * JSON_ARRAY_SIZE(1) + JSON_ARRAY_SIZE(2) + 4 * JSON_OBJECT_SIZE(1) + 3 * JSON_OBJECT_SIZE(2) + 3 * JSON_OBJECT_SIZE(4) + JSON_OBJECT_SIZE(5) + 2 * JSON_OBJECT_SIZE(7) + 2 * JSON_OBJECT_SIZE(8) + 720;
  DynamicJsonDocument jsonBuffer(bufferSize);

  // Deserialize the JSON document
  DeserializationError error = deserializeJson(jsonBuffer, jsonString);

  // Check if deserialization was successful
  if (error) {
    Serial.print("deserializeJson() failed: ");
    Serial.println(error.f_str());
    return;
  }

  // Extract values from the JSON object
  JsonArray list = jsonBuffer["list"];
  JsonObject nowT = list[0];
  JsonObject later = list[1];

  String city = jsonBuffer["city"]["name"];
  
  float tempNow = nowT["main"]["temp"];
  float humidityNow = nowT["main"]["humidity"];
  String weatherNow = nowT["weather"][0]["description"];

  float tempLater = later["main"]["temp"];
  float humidityLater = later["main"]["humidity"];
  String weatherLater = later["weather"][0]["description"];

  // Checking for four main weather possibilities
  diffDataAction(weatherNow, weatherLater, "clear");
  diffDataAction(weatherNow, weatherLater, "rain");
  diffDataAction(weatherNow, weatherLater, "snow");
  diffDataAction(weatherNow, weatherLater, "hail");

  Serial.println();
}
```

3. I also tried debugging the response using this code:
``` 
Serial.println("Response:");
  while (client.available()) {
    char c = client.read();
    Serial.print(c);  // Print each character of the response
  }
```

However, I still encountered buffer issues, which meant the data wasn't being processed correctly.

Although the code connects to the board and WiFi, it doesn't yet retrieve weather information.

## Combining the temperature and API
Although the weather API isn't fully functional, you can still work on the logic. 

You can use temperature data from inside the house and upcoming weather predictions to automate the sunshade. Using the OpenWeather API code, you can add BMP280 sensor data and the necessary logic for automatic operation:
```
void diffDataAction(String nowT, String later, String weatherType, ) {
  int indexNow = nowT.indexOf(weatherType);
  int indexLater = later.indexOf(weatherType);
  if (bme.readTemperature() > 25) {
    if (weatherType == "rain" || weatherType == "snow" || weahterType == "hail") {
      //Sunshade doesn't extend, because it is / is going to rain
    } else {
      //Sunshade does extend, because it is hot and isn't / isn't going to rain
    }
  } else if (bme.readTemperature() < 15) {
    //Sunshade does retracts, because it is cold inside and the weather doesn't matter
  } else {
    if (weatherType == "rain" || weatherType == "snow" || weahterType == "hail") {
      //Sunshade doesn't extend because of the weather
    } else {
      //Sunshade does extend, because the tempeture inside is avarage and it isn't going to rain
    }
  }
}
```

This allows the sunshade to function almost automatically. The user only needs to set temperature thresholds for when it's considered too hot or too cold. After that, the sunshade can monitor the inside temperature and current/upcoming weather to ensure it doesn't extend unnecessarily during rain or other adverse weather conditions.

## Sources
- https://github.com/msandholz/ESP8266-with-BME280
- https://forum.arduino.cc/t/connecting-bmp280-esp8266/1020675
- https://randomnerdtutorials.com/esp8266-weather-forecaster/
- https://arduinojson.org/v6/doc/upgrade/
- ChatGPT
