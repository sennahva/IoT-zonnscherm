# Sunshade manuel 
This is a manuel for combining information about the temperature inside your home and comming weather to make a sun shade automatic

Requirements: 
- Arduino IDE
- NodeMCU ESP8266
- BMP280
- Wifi / Hotspot 

## 1. Measuring the tempeture 
Connect the BMP280 to the NodeMCU board. Start by connecting the VCC(BMP280) pin to the 3.3V(ESP8266) output and the GND(BMP280) to the ground(ESP8266). 
Than connect the SCL(BMP280) to the D1(ESP8266) pin and the SDA(BMP280) to D2(ESP8266). 

Open your library manager with Sketch > include library > manage libraries. Then install the following libraries: `Adafruit BMP280 Library` and `Adafruit Unified Sensor` both by Adafruit. 

When installing the libraries pay attention that you don't exedently download the library for the BME280, this is a very simular device but it works just a bit different where the code wont work. If you have the wrong library you will probebly get stuck when the ESP8266 is trying to connect with the BMP280 but won't find it, and get the following error / message: 

Then you can test your BMP280 with the example code, that you can find by file > examples > Adafruit BMP280 Library (might need to scroll down) > **bmp280test**. Where you only have to change line 37 from `status = bmp.begin();` to `status = bmp.begin(0x76);`. 

Check if your BAUD rate matches te one in the code. 

Witch if every thing works correct you will get the following message in your serialmonitor:


## Weather API 
To predict the upcomming weather you can use [OpenWeather](https://openweather.co.uk/). Once there start by creating (or loggin in) an account: 


Then navigate to the "my API keys" tab, like this screenshot: 


Then generate a new API Key, and note this key for later: 


After you have created your account, you need to instal another library, `ArduinoJson` by Benoit Blanchon. With that we can use the code Emmanuel Odunlade wrote [here](https://randomnerdtutorials.com/esp8266-weather-forecaster/) as a start, but have to change a bit to make it work. 


But what you will see is that this code still uses ArduinoJson 5, what needs to be changed to ArduinoJson 6+. Wich i couldn't really figure out. I tried to do the following things: 
1. First i started with chancing the buffer size, originally it is on 2500 but no matter what it was it didn't work. 
2. Using [Arduinojson](https://arduinojson.org/v6/doc/upgrade/) to change the Jsonbuffer. This is an artical where they explain the changes from ArduinoJson 5 and 6, wich i made the following with: 
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

3. Last i tried to debug the response, with te following: 
``` 
Serial.println("Response:");
  while (client.available()) {
    char c = client.read();
    Serial.print(c);  // Print each character of the response
  }
```

But this gave the following output, wich still means the buffer isn't working correct: 



But after all than it still would work, it connected with my board and wifi, but wouldn't give the weather information. 

## Combining 
It won't work since the API isn't working, but you can still get far without it needing to work. 

So first we want it to make automatic decision, where it takes the temperature from inside and the upcomming weather into consideration. So using the code that we have from the OpenWeather API, we can add the code from the BMP280 and the logic needed for this to work: 
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

In this way the Sunshade can work almost automatic, the only thing the user needs to do is set temperatures on what he finds to hot and cold. But after that the sunshade can check the temperature and the current and upcomming weather to make sure it wont extend the sunshade when it is raining or not needed, and vise versa. 

## Sources
- https://github.com/msandholz/ESP8266-with-BME280
- https://forum.arduino.cc/t/connecting-bmp280-esp8266/1020675
- https://randomnerdtutorials.com/esp8266-weather-forecaster/
- https://arduinojson.org/v6/doc/upgrade/
- ChatGPT