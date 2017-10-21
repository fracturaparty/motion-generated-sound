# motion-generated-sound
/**
 * Processing Sound Library, Example 3
 * 
 * This example shows how to make a simple sampler and sequencer 
 * with the Sound library. In this sketch, five different samples are 
 * loaded and played back at different pitches, in this case five 
 * different octaves. The sequencer triggers an event every 200-1000 
 * milliseconds randomly. Each time a sound is played a colored 
 * rect with a random color is displayed.
 */

import processing.sound.*;
import processing.serial.*;
import cc.arduino.*;
Serial port;
//this object class Arduino
//represents guess what? Your board
Arduino arduino;
TriOsc triangle;
SinOsc sine;
Reverb reverb;
LowPass lowPass;
//like in an arduino sketch it's good to
//use variables for pin numbers 

int sensorPin = 2;
int fsrPin =3;
//int sdaPin = 4;
int sclPin = 5;
SoundFile[] files;

// Create an array of values which represent the octaves. 
// 1.0 is playback at normal speed, 0.5 is half and therefore 
// one octave down. 2.0 is double so one octave up.
float[] octave = { 
  0.25, 0.5, 1.0, 2.0, 4.0
};

// The playSound array is defining how many samples will be 
// played at each trigger event
int[] playSound = { 
  1, 1, 1, 1, 1
};

// The trigger is an integer number in milliseconds so we 
// can schedule new events in the draw loop
int trigger=0;

// This array holds the pixel positions of the rectangles 
// that are drawn each event
int[] posx = {
  0, 128, 256, 384, 512
};


void setup() {
  size(640, 360);
  background(255);

  port=new Serial(this, Serial.list()[2],57600);
  port.bufferUntil('\n');
 arduino=new Arduino(this, Arduino.list()[2],57600);
 
  //once it's created I set the pin modes  
arduino.pinMode(fsrPin, Arduino.INPUT); 
  arduino.pinMode(sensorPin, Arduino.INPUT); 
  //arduino.pinMode(sdaPin, Arduino.INPUT);
  arduino.pinMode(sclPin, Arduino.INPUT); 
  
  // Create an array of 5 empty soundfiles
   sine = new SinOsc(this);
  sine.play();
   // Create the triangle oscillator
   triangle = new TriOsc(this); 
   triangle.play();
   reverb = new Reverb(this);
    reverb.process(triangle);
  reverb.process(sine);
  reverb.room(0.5);
  reverb.damp(0.1);
  reverb.wet(0.7);
  files = new SoundFile[5];
  lowPass = new LowPass(this);
  lowPass.process(sine);
  lowPass.process(triangle);
  

  // Load 5 soundfiles from a folder in a for loop. By naming 
  // the files 1., 2., 3., [...], n.aif it is easy to iterate 
  // through the folder and load all files in one line of code.
  for (int i = 0; i < files.length; i++) {
    files[i] = new SoundFile(this, (i+1) + ".aiff");
  }
  
}

void draw() {
  
int analogValue =  arduino.analogRead(fsrPin);
  println(analogValue); //print it for testing purposes
 int analogValueflex = arduino.analogRead(sensorPin);
 println(analogValueflex);
 //int analogValuesda = arduino.analogRead(sdaPin);
 //println(analogValuesda);
 int analogValuescl = arduino.analogRead(sclPin);
 println(analogValuescl);
  // If the determined trigger moment in time matches up with 
  // the computer clock events get triggered.
  if (millis() > trigger && analogValue>0) {

    // Redraw the background every time to erase old rects
    background(255);

    // By iterating through the playSound array we check for 
    // 1 or 0, 1 plays a sound and draws a rect, for 0 nothing happens

  }
    for (int i = 0; i < files.length; i++) {      
      // Check which indexes are 1 and 0.
      if (playSound[i] == 1) {
        float rate;
        // Choose a random color and get set to noStroke()
        fill(int(random(255)), int(random(255)), int(random(255)));
        noStroke();
        // Draw the rect in the positions we defined earlier in posx
        rect(posx[i], 50, 128, 260);
        // Choose a random index of the octave array
        rate = octave[int(random(0, 5))];
        // Play the soundfile from the array with the respective 
        // rate and loop set to false
        files[i].play(rate, 1.0);
      }

      // Renew the indexes of playSound so that at the next event 
      // the order is different and randomized.
      playSound[i] = int(map(analogValue,1,30,0, 2));
    }

    // Create a new triggertime in the future, with a random offset 
    // between 200 and 1000 milliseconds
   
    trigger = millis() + int(random(200, 1000));
    
    if (analogValuescl>0){
  float freq=map(analogValue,0,27,100,2000);
  float amp=0.7;
  float add=0.0;
  float pos=1;
 triangle.set(freq, amp, add, pos);
 float room=0.8;
  float damp=0.7;
  float wet=1;
  reverb.set(room, damp, wet);
    }
     if (analogValuescl>10){
  float freq=map(analogValuescl,10,30,100,2000);
  float amp=0.7;
  float add=0.0;
  float pos=0.5;
  sine.set(freq, amp, add, pos);
  float room=0.8;
  float damp=0.7;
  float wet=1;
  reverb.set(room, damp, wet);
    }
    if (analogValueflex<15){
      float room=0.8;
  float damp=0.7;
  float wet=1;
  reverb.set(room, damp, wet);
  }
 
}
