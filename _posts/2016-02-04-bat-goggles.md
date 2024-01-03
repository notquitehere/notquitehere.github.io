---
title: "Bat goggles"
date: 2016-02-04
modified: 2022-02-18
published: true
excerpt:
cover: /assets/2016-02-04-becky-bat.jpg
tags: maker arduino
---


These were developed to mimic as closely as possible bio-sonar using an Arduino, HC-SR04 Ultrasonic distance sensor and a piezo buzzer as part of a workshop to introduce school children to the basics of electronics and coding.

{:.gallery.two-by-one}
![](/assets/2016-02-04-bat_goggles_closeup.jpg)
![](/assets/2016-02-04-bat_goggles_overview.jpg)


## Electronics

The circuit is very simple for this particular project. It proved cheaper to import Arduino's from China than to use a stripped down circuit such as the [shrimp](http://start.shrimping.it/kit/shrimp.html).

## Code

This code is based on Suneth Attygalle's [batgoggles code](http://suneth.com/2008/05/ultrasonic-batgoggles/), and the [Arduino Ping Tutorial](https://www.arduino.cc/en/Tutorial/Ping)

```cpp
// declare the pins

const int trigPin = 5;
const int echoPin = 6;
const int buzzerPin = 4;

// set up constants

const long scalingfactor = 10;

// set global variables

int cm;
unsigned long time1, time2, i, duration;
unsigned long maxtime = 1000;

void setup() {
  // initialise all the pins
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(buzzerPin, OUTPUT);

  // uncomment this if you need to check why things aren't
  // working
  // Serial.begin(9600);
}

void loop() {
  // beep

  tone(buzzerPin, 262); // sets the note to middle c (ish)
  delay(90);
  check_distance();

  // if the sensor doesn't time out remove the time it took for
  // the sensor to be polled from the remaining delay so that
  // the beeps remain even

  if (time2-time1&lt;30) {
    delay(60-(time2-time1));
  }
  noTone(buzzerPin);

  // delay

  i = 0; // resets the counter
  while (i &lt; maxtime){ // do this until maxtime is reached
    if (i % 60 == 0){
      check_distance();
    }
    delay(1);
    ++i; // increment counter
  }

}

void check_distance(){

  // note the time before checking the sensor

  time1 = millis();

  // The sensor is triggered by a HIGH pulse of 10 or more
  // microseconds. Giving a short LOW pulse beforehand ensures a
  // clean HIGH pulse.

  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  // read the signal from the sensor: a HIGH pulse whose duration
  // is the time (in microseconds) from the sending of the Ping
  // to the reception of its echo from an object. This will time
  // out after 30ms.

  duration = pulseIn(echoPin, HIGH);

  // note the time after checking the sensor

  time2 = millis();

  // translate the pulse duration into centimeters
  // The speed of sound is 340 m/s or 29 microseconds per
  // centimeter. The ping travels out and back, so to find the
  // distance of the object we take half the distance travelled.

  cm = duration / 29 / 2;
  maxtime = cm * scalingfactor;

  // Not sure why it's not working? Uncomment this to check that
  // your sensor is giving the correct output

  // Serial.print(duration);
  // Serial.print(\" millis\");
  // Serial.println();
  // Serial.print(cm);
  // Serial.print(\" cm\");
  // Serial.println();
}
```

Because of the age of the children involved in this workshop we didn't use the Arduino IDE to create the code, instead we used a graphical interface ([Ardublockly](https://github.com/carlosperate/ardublockly)). I added a few new blocks to make it work for what I wanted but unfortunately ran out of time to get my code merged into the main codebase.

