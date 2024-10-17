# Sunshade manuel 
This is a manuel for combining information about the temperature inside your home and comming weather to make a sun shade automatic

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


Then generate a new API Key, and note the key: 


After you have created your account, we can use the code Emmanuel Odunlade wrote [here](https://randomnerdtutorials.com/esp8266-weather-forecaster/) as a start.
But what you will see is that this code still uses ArduinoJson 5, what needs to be changed to ArduinoJson 6+. So you do that with the following steps: 

This is where i personaly got stuck, 


installeren van de json library


## Combining 
It won't work since the API isn't working, but you can still get far without it needing to work. 

So first we want it to make automatic decision, where it takes the temperature from inside and the upcomming weather into consideration. So using the code that we have from the OpenWeather API, we can add the code from the BMP280 and the logic needed for this to work: 
```
void diffDataAction(String nowT, String later, String weatherType, ) {
  int indexNow = nowT.indexOf(weatherType);
  int indexLater = later.indexOf(weatherType);
  // if weather type = rain, if the current weather does not contain the weather type and the later message does, send notification
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