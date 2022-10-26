# CAPS-Project

The Community Air Pollution Sensor (CAPS) project aims to bring education, technology and communities together through understanding and tackling local air pollution. With funding from Google, we have developed a sensor that detects particulate matter of three sizes: 1µm, 2.5µm, 10µm. This pollution causes many respiratory issues as well as other health complications, and although there are council-owned air pollution sensors stationed in our areas there is a lack of more granular, road-by-road data. To combat this we teach secondary school students how to make and code our sensor, after which they use their sensors while going to and from school (or otherwise) to map out their walks to school and the wider local area. Using this data we hope to: 1. Map out the local air pollution to better understand the levels, particularly around school start and end times; 2. Talk to our local councils and implement changes based on the data. 3. Engage local communities in the effects of and the fight against air pollution through citizen science and education. 

The code that controls our sensor is written in C++ and is shown below:

## Libraries  

	#include <TinyGPSPlus.h>  
	#include <HardwareSerial.h>  
	#include <PMS.h>  
	#include <LiquidCrystal_I2C.h>  
	#include <DHT.h>  
	#include <SD.h>  
	#include <SPI.h>  
	#include <Arduino.h>  
	#include <WiFi.h>  

## Pin definitions  

	// PMS  
	HardwareSerial PM(1);  
	PMS pms(PM);  
	PMS::DATA data;  
	#define RX1 14  
	#define TX1 13  

	// DHT  
	#define DHTPIN 26    // what pin we're connected to  
	#define DHTTYPE DHT11   // DHT 22  (AM2302)  
	DHT dht(DHTPIN, DHTTYPE);  

	// GPS  
	HardwareSerial GPS(2);  
	TinyGPSPlus gps;  
	#define RX2 17  
	#define TX2 16  

	// LCD  
	#define SDA 33   
	#define SCL 36  
	// set the LCD number of columns and rows  
	const int lcdColumns = 16;  
	const int lcdRows = 2;  
	// set LCD address, number of columns and rows  
	LiquidCrystal_I2C lcd(0x27, lcdColumns, lcdRows);  

	// SD  
	#define SD_CS 5  

	// WiFi  
	#define WIFI_SSID "VM7869624"  
	#define WIFI_PASSWORD "Zg4fgtvyjBwr"  
	
## Data Variables  

	// PMS  
	int pm1; // PM 1  
	int pm2_5; // PM 2.5  
	int pm10; // PM 10  

	// DHT  
	float t; // Temperature  
	float h; // Humidity  

	// GPS  
	// Date  
	uint8_t currentDay; // Day  
	uint8_t currentMonth; // Month  
	uint16_t currentYear; // Year  
	String currentDate; // Full date  

	// Time  
	uint8_t currentHour; // Hour  
	uint8_t currentMinute; // Minute  
	uint8_t currentSecond; // Second  
	String currentTime; // Full time  

	// Location  
	double lat; // Latitude  
	double lon; // Longitude  

## Others  

	// User Details  
	const String username = "Saul DV";  
	const String organisation = "CAPS";  

	int readingID = 1;  

	const String filename = "/data.txt";  
	String message;  

	const unsigned long interval = 5000;  
	unsigned long previousTime;  

## Functions  

	// Waits for the defined interval  
	void waitForInterval (int interval) {  
		previousTime = millis();  
		while (millis() - previousTime <= interval){}}  
<br>

	// Clears the desired line on the LCD display (0 or 1)  
	void clearLCDLine (int line) {   
		lcd.setCursor(0,line);  
		for(int n = 0; n < 16; n++){  
			lcd.print(" ");}  
		lcd.setCursor(0,line);}  
<br>

	// Adds a leading zero to the given number if below 10  
	String addZero(int number) {  
		if (number < 10) {  
			String newNumber = String("0" + String(number));  
			return newNumber;}  
		else {  
			String newNumber = String(number);  
			return newNumber;}}  
<br>

	// Takes PMS, DHT and GPS readings and saves them to the SD card in CSV format  
	void takeReadings() {  
		// PMS  
		pms.requestRead();  
		if (pms.readUntil(data)){  
			pm1 = data.PM_AE_UG_1_0;  
			pm2_5 = data.PM_AE_UG_2_5;  
			pm10 = data.PM_AE_UG_10_0;}  

		// DHT   
		t = dht.readTemperature();  
		h = dht.readHumidity();    
 
		// GPS  
		while (GPS.available() > 0) {  
			if (gps.encode(GPS.read())) {  
				// Location  
				if (gps.location.isValid()){  
					lat = gps.location.lat();  
					lon = gps.location.lng();}
				
				// Date/Time  
				if (gps.date.isValid()){  
					currentDay = gps.date.day();  
					currentMonth = gps.date.month();  
					currentYear = gps.date.year();  
					currentDate = String(addZero(currentDay) + "/" + addZero(currentMonth) + "/" + currentYear);}
				
				if (gps.time.isValid()) {  
					currentHour = gps.time.hour();   
					currentMinute = gps.time.minute();  
					currentSecond = gps.time.second();  
					currentTime = String(addZero(currentHour) + ":" + addZero(currentMinute) + ":" + addZero(currentSecond));}}}  
					
  	logSDCard(username, organisation, currentTime, currentDate, pm1, pm2_5, pm10, t, h, lat, lon);}  
<br>

	// Print PMS readings to Serial port and LCD  
	void pmsPrint() {  
		pms.requestRead();  

		if (pms.readUntil(data)){  
			pm1 = data.PM_AE_UG_1_0;  
			pm2_5 = data.PM_AE_UG_2_5;  
			pm10 = data.PM_AE_UG_10_0;  
			  
			Serial.println("PMS:");  
			Serial.print("1.0: "); Serial.println(pm1);  
			Serial.print("2.5: "); Serial.println(pm2_5);  
			Serial.print("10.0: "); Serial.println(pm10);  
			Serial.println();  
			
			clearLCDLine(0);  
			lcd.print("PM:"); lcd.print(String(pm1)); lcd.print("  ");  
			lcd.print(String(pm2_5)); lcd.print("  ");  
			lcd.print(String(pm10));} 

		else {  
			Serial.println("No PMS Data.");  
			Serial.println();  
			
			clearLCDLine(0);  
			lcd.print("  No PMS Data.  ");}}  
<br>

	// Print DHT readings to Serial port and LCD  
	void dhtPrint() {  
		t = dht.readTemperature();  
		h = dht.readHumidity();  
		if (isnan(t) || isnan(h)) {  
			Serial.println("No DHT Data");  
			Serial.println();  
			
			clearLCDLine(1);  
			lcd.print("  No DHT Data.  ");}  

		else {  
			Serial.print("Temp: "); Serial.print(t); Serial.println("\*C");   
			Serial.print("Hum: "); Serial.print(h); Serial.println("%");    
			Serial.println();  
		 
			clearLCDLine(1);  
			lcd.print("T:"); lcd.print(String(t));   
			lcd.print(" H:"); lcd.print(String(h));}}    
<br>

	// Print Location readings to Serial port and LCD  
	void locationPrint() {  
		while (GPS.available() > 0) {  
			if (gps.encode(GPS.read())) {   
				if (gps.location.isValid()) {  
					lat = gps.location.lat();  
					lon = gps.location.lng();  

					Serial.print(F("Location: "));  
					Serial.print(lat, 6); Serial.print(F(","));  
					Serial.println(lon, 6);  
					Serial.println();  

					clearLCDLine(0);  
					lcd.print("Lat: "); lcd.print(String(lat, 6));  
					clearLCDLine(1); 
					lcd.print("Long: "); lcd.print(String(lon, 6));}
					
				else {
					Serial.println("No Location Data");
					Serial.println();

					clearLCDLine(0);
					lcd.print("  No Location   ");
					clearLCDLine(1);
					lcd.print("     Data.      ");}
				return;}}}
<br>

	// Print Date/Time readings to Serial port and LCD  
	void dateTimePrint() {  
		while (GPS.available() > 0) {  
			if (gps.encode(GPS.read())) {        
				if (gps.date.isValid()) {  
					currentDay = gps.date.day();  
					currentMonth = gps.date.month();  
					currentYear = gps.date.year();  
					currentDate = String(addZero(currentDay) + "/" + addZero(currentMonth) + "/" + currentYear);  
					
					Serial.print("Date: ");    
					Serial.print(currentDate);  
					Serial.println();  

					clearLCDLine(0);  
					lcd.print("Date: ");  
					lcd.print(currentDate);}  
					
				else {  
					Serial.println("No Date Data");  
					Serial.println();  

					clearLCDLine(0);  
					lcd.print("  No Date Data  ");}  
					
				if (gps.time.isValid()) {  
					currentHour = gps.time.hour();  
					currentMinute = gps.time.minute();  
					currentSecond = gps.time.second();  
					currentTime = String(addZero(currentHour) + ":" + addZero(currentMinute) + ":" + addZero(currentSecond));  

					Serial.print("Time: ");  
					Serial.println(currentTime);  
					Serial.println();  

					clearLCDLine(1);  
					lcd.print("Time: ");  
					lcd.print(currentTime);}  
					
				else {  
					Serial.println("No Time Data");  
					Serial.println();  

					clearLCDLine(1);  
					lcd.print("  No Time Data  ");}  
				return;}}}  
<br>

	// Write to the SD card  
	void writeFile(fs::FS &fs, const char * path, const char * message) {  
		Serial.printf("Writing file: %s\n", path);  

		File file = fs.open(path, FILE_WRITE);  
		if (!file) {  
			Serial.println("Failed to open file for writing");  
			return;}  
			
		if (file.print(message)) {  
			Serial.println("File written");}  
			
		else {  
			Serial.println("Write failed");}   
			
		file.close();}  
<br>

	// Compile a CSV data message and append it to the SD file  
	void logSDCard(String username, String organisation, String currentTime, String currentDate, int pm1, int pm2_5, int pm10, float t, float h,  
								 double lat, double lng) {  
		message = username + "," + organisation + "," + String(readingID) + "," + currentTime + "," + currentDate + ","   
									+ String(pm1) + "," + String(pm2_5) + "," + String(pm10) +  
									"," + String(t) + "," + String(h) + "," + String(lat,6) + "," + String(lng,6) + "\r\n";  
		Serial.print("Save data: ");  
		Serial.println(message);  
		appendFile(SD, "/data.txt", message.c_str());  
	}  
<br>

	// Append data to the SD file  
	void appendFile(fs::FS &fs, const char * path, const char * message) {  
		Serial.printf("Appending to file: %s\n", path);  

		File file = fs.open(path, FILE_APPEND);  
		if(!file) {  
			Serial.println("Failed to open file for appending");  
			return;}    
			
		if(file.print(message)) {  
			Serial.println("Message appended");  
			Serial.println();}   
			
		else {
			Serial.println("Append failed");  
			Serial.println();}  
			
		file.close();  
	}


## Setup

	void setup() {

		// Begin Serial communication
		Serial.begin(9600);

		// PMS
		PM.begin(9600, SERIAL_8N1, RX1, TX1);
		pms.activeMode();

		// DHT
		dht.begin();

		// GPS
		GPS.begin(9600, SERIAL_8N1, RX2, TX2);

		// LCD
		lcd.init(); // initialize LCD              
		lcd.backlight(); // turn on LCD backlight
		lcd.clear(); // clear LCD screen
		lcd.print("     Hello!     ");
		waitForInterval(2000);

		// SD
		SD.begin(SD_CS);  
		if (!SD.begin(SD_CS)) {
			Serial.println("No SD Card.");
			Serial.println();

			clearLCDLine(0);
			lcd.print("   No SD Card.  ");
			waitForInterval(2000);}

		// WiFi
		WiFi.begin(WIFI_SSID, WIFI_PASSWORD);
		Serial.print("Connecting to Wi-Fi");
		clearLCDLine(0);
		lcd.print("Trying the Wifi");
		lcd.setCursor(0,1);

		int count = 0;
		while (WiFi.status() != WL_CONNECTED && count < 16) {
			Serial.print(".");

			lcd.print(".");
			waitForInterval(200);
			count++;}
			
		if (WiFi.status() != WL_CONNECTED) {
			Serial.println("Wifi Connection failed");
			Serial.println();

			lcd.clear();
			lcd.print("    No WiFi.    ");}
			
		else if (WiFi.status() == WL_CONNECTED) {
			Serial.println();
			Serial.print("Connected with IP: ");
			Serial.println(WiFi.localIP());
			Serial.println();

			lcd.clear();
			lcd.print("WiFi Connected!");}
		waitForInterval(2000);}

## Loop

	void loop() {
		takeReadings(); // Take readings and save to SD card

		pmsPrint(); // Print PMS readings on first row of LCD
		dhtPrint(); // Print DHT readings on second row of LCD
		waitForInterval(interval); // Wait 5 seconds (when interval = 5000)

		locationPrint(); // Print location readings
		waitForInterval(interval); // Wait 5 seconds

		dateTimePrint(); // Print Date/Time readings
		waitForInterval(interval); // Wait 5 seconds

		readingID++; // Increase reading number by 1}
