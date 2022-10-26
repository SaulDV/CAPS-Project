# CAPS-Project

The Community Air Pollution Sensor (CAPS) project aims to bring education, technology and communities together through understanding and tackling local air pollution. With funding from Google, we have developed a sensor that detects particulate matter of three sizes: 1µm, 2.5µm, 10µm. This pollution causes many respiratory issues as well as other health complications, and although there are council-owned air pollution sensors stationed in our areas there is a lack of more granular, road-by-road data. To combat this we teach secondary school students how to make and code our sensor, after which they use their sensors while going to and from school (or otherwise) to map out their walks to school and the wider local area. Using this data we hope to: 1. Map out the local air pollution to better understand the levels, particularly around school start and end times; 2. Talk to our local councils and implement changes based on the data. 3. Engage local communities in the effects of and the fight against air pollution through citizen science and education. 

The code that controls our sensor is written in C++ for the Arduino IDE and is split into sections explained below:

## Libraries  

Here we outline the different libraries that are required for the different components to interact with the ESP32 properly.

## Pin definitions  

Here we specify which component is connected to which pin of the ESP32. This is by no means the only way to configure these components, just the way we have decided for this project. 	
	
## Data Variables  

Here we name the variables that we will be using throughout the rest of the code, named here to avoid repetition.	

## Others  

Here we give any other information that is useful for the running of the code or needed to label the data. 

## Functions  

This section contains all of the functions we will use throughout the Setup and Loop functions, mostly made to avoid repetition. 

## Setup

Here we initialise and turn on any components that need it and connect to Wifi if available. 

## Loop

This code is repeated after the Setup is complete, until the sensor is turned off. We collect readings from the components and save them to the SD card.
