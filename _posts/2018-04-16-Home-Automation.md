---
layout: post
title: Home Automation using Arduino and Bluetooth module 
description : In this post I will be discussing about how to develop a home automation system using an Arduino board with Bluetooth module being remotely controlled by any Android OS smartphone. In order to achieve this, a Bluetooth module is interfaced to the Arduino board at the receiver end while on the transmitter end, a GUI application on the cell phone sends ON/OFF commands to the receiver where loads are connected. By touching the specified location on the GUI, the loads can be turned ON/OFF remotely through this technology.
author : Ayush Agarwal
tags: [Home-automation, Arduino, Bluetooth-module]
time : 15 minutes read
---

[![]({{ site.baseurl }}/assets/img/post/home_automation.png)](#) 

The main objective of this project is to develop a home automation system using an Arduino board with Bluetooth being remotely controlled by any Android OS smartphone. As technology is advancing so houses are also getting smarter. Modern houses are gradually shifting from conventional switches to centralized control system, involving remote controlled switches. Presently, conventional wall switches located in different parts of the house makes it difficult for the user to go near them to operate. Even more it becomes more difficult for the elderly or physically handicapped people to do so. Remote controlled home automation system provides a most modern solution with smartphones. Moreover home automation systems in today's market costs more than Rs. 50,000 so also wanted to make a low budget home automation system.

>  In order to achieve this, a Bluetooth module is interfaced to the Arduino board at the receiver end while on the transmitter end, a GUI application on the cell phone sends ON/OFF commands to the receiver where loads are connected. By touching the specified location on the GUI, the loads can be turned ON/OFF remotely through this technology.

You can the demostration video in the following link :

<iframe src="https://www.youtube.com/embed/1ieyT6df8ec?ecver=1" allow="autoplay; encrypted-media" allowfullscreen="" width="640" height="360" frameborder="0"></iframe>

## Description of the Project

This project is one of the important [Arduino Projects](https://www.projectsof8051.com/arduino-projects/). Arduino based home automation using Bluetooth project helps the user to control any electronic device using Device Control app on their Android Smartphone. The android app sends commands to the controller – Arduino, through wireless communication, namely, Bluetooth. The Arduino is connected to the main PCB which has five relays as shown in the block diagram. These relays can be connected to different electronic devices like lights, television, fan, etc.

When the user presses on the ‘On’ button displayed on the app for the device 1, the Buzzer is switched on. This Buzzer can be switched off, by pressing the same button again.

Similarly, when the user presses on the ‘On’ button displayed on the app for the device 2, the fan is switched on. The fan can be switched off, by pressing the same button again.

This project of home automation using Bluetooth and Arduino can be used for controlling any AC or DC devices.

Basic flow of the project is given below :

[![]({{ site.baseurl }}/assets/img/post/diagram.jpg)](#) 

## Hardware Requirement

The list of components mentioned here are specifically for controlling 4 different loads.

*   Arduino UNO       ([Click Here to Buy!](https://amzn.to/2qDZMmy))
*   HC – 05 Bluetooth Module        ([Click Here to Buy!](https://amzn.to/2JVPYfq))
*   5 V Relay X 4      ([Click Here to Buy!](https://amzn.to/2qIf60z))
*   Bread board      ([Click Here to Buy!]( https://amzn.to/2qG8Veg))
*   Connecting wires      ([Click Here to Buy!](https://amzn.to/2HG8lEp))
*   Bluetooth enabled smartphone or tablet
*   5V Power Source

## Software Requirement

*   Arduino 1.8.5 compiler
*   Android application

## Design And Implementation

As we are not able to get a 4 channel relay board so we made it by ourselves. Components required are as follows :

*   One Pref. board
*   Four 5 volt DC Relay
*   Four leds for indicating purpose
*   Four 220 and 100 ohm resistor
*   Four diodes of 1N4007
*   Four 2N2222 npn transistor
*   Four two-pin block connector
*   Some jumper wires

Circuit diagram for the one relay module is as follows :

[![]({{ site.baseurl }}/assets/img/post/relay_cicuit.jpg)](#) 

And what we made is as follows but you can use the pre made relay board from market.

[![]({{ site.baseurl }}/assets/img/post/relay_board.jpg)](#)   

#### Final Circuit

For the sake of simplicity, we replaced the the relay module made by us with the pre-built 4 channel relay in the final circuit diagram.

[![]({{ site.baseurl }}/assets/img/post/circuit.png)](#)   

#### Arduino Code

Given below is the Arduino code that you can compile and program in your Arduino UNO.

```groovy
//using ports 10, 11, 12, 13
int relay1 = 10;
int relay2 = 11;
int relay3 = 12;
int relay4 = 13;
int val;

void setup() {

  Serial.begin(9600);
  pinMode(relay1,OUTPUT);
  pinMode(relay2,OUTPUT);
  pinMode(relay3,OUTPUT);
  pinMode(relay4,OUTPUT);
  digitalWrite(relay1,HIGH);
  digitalWrite(relay2,HIGH);
  digitalWrite(relay3,HIGH);
  digitalWrite(relay4,HIGH);

}
void loop() {

  //check data serial from bluetooth android App
  while (Serial.available() > 0){
    val = Serial.read();
    Serial.println(val);
  }

  //Relay is on
  if( val == 1 ) {
    digitalWrite(relay1,HIGH); }
  else if( val == 2 ) {
    digitalWrite(relay2,HIGH); }  else if( val == 3 ) {
    digitalWrite(relay3,HIGH); }
  else if( val == 4 ) {
    digitalWrite(relay4,HIGH); }

  //relay all on
  else if( val == 0 ) {
    digitalWrite(relay1,HIGH);
    digitalWrite(relay2,HIGH);
    digitalWrite(relay3,HIGH);
    digitalWrite(relay4,HIGH);
  }
  //relay is off
  else if( val == 5 ) {
    digitalWrite(relay1,LOW); }
  else if( val == 6 ) {
    digitalWrite(relay2,LOW); }
  else if( val == 7 ) {
    digitalWrite(relay3,LOW); }
  else if( val == 8 ) {
    digitalWrite(relay4,LOW); }

  //relay all off
  else if( val == 10 ) {
    digitalWrite(relay1,LOW);
    digitalWrite(relay2,LOW);
    digitalWrite(relay3,LOW);
    digitalWrite(relay4,LOW);
  }
}
```

#### Android Application

I had built a basic bluetooth based app that just sends the information to the bluetooth controler. Here is the GitHub [link](https://github.com/aagarwal1012/Home-Automation).

You can clone my app using git using the command in git bash:

```groovy
git clone https://github.com/aagarwal1012/Home-Automation.git
```

Android apk can be downloaded form [**Here!**](https://drive.google.com/file/d/1-G7GkjjILzkK2z8o3YBheVA99TwepST5/view?usp=sharing)

Here are the Screenshot of the app.

<img src="{{ site.baseurl }}/assets/img/post/app_screenshot.png" alt="" height="40%" width="50%">  

## Future Development

*   Arduino based device control using Bluetooth on Smartphone project can be enhanced to control the speed of the fan or volume of the buzzer etc.
*   Home automation and Device controlling can be done using Internet of Things – IOT technology.
*   We can replace Bluetooth by GSM modem so that we can achieve device controlling by sending SMS using GSM modem.

In case, if you find any difficulty in the project or any problem or error in installation and validation do not hesitate to post a comment below.