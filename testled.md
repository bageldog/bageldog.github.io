Last year, for Christmas, I recieved a reel of LED strips. I decided that I was going to put them to use as ambient lighting in my room.
My plan was not just to make them regular lights with a switch, but to use an IoT device to control them, create custom animations, and be able to toggle them from my PC.
After some thought, I decided to use a Particle Photon for this task, as it was well suited and also was what I had lying around already.
I set up my photon, and with a bit of help got this bit of code running. 
```
/**
 * This is a minimal example, see extra-examples.cpp for a version
 * with more explantory documentation, example routines, how to
 * hook up your pixels and all of the pixel types that are supported.
 *
 * On Photon, Electron, P1, Core and Duo, any pin can be used for Neopixel.
 *
 * On the Argon, Boron and Xenon, only these pins can be used for Neopixel:
 * - D2, D3, A4, A5
 * - D4, D6, D7, D8
 * - A0, A1, A2, A3
 *
 * In addition on the Argon/Boron/Xenon, only one pin per group can be used at a time.
 * So it's OK to have one Adafruit_NeoPixel instance on pin D2 and another one on pin
 * A2, but it's not possible to have one on pin A0 and another one on pin A1.
 */

#include "Particle.h"
#include "neopixel.h"

SYSTEM_MODE(AUTOMATIC);

// IMPORTANT: Set pixel COUNT, PIN and TYPE
#define PIXEL_PIN D0
#define PIXEL_COUNT 90
#define PIXEL_TYPE WS2812B

Adafruit_NeoPixel strip(PIXEL_COUNT, PIXEL_PIN, PIXEL_TYPE);


int LED = D0;

int cycleCounter = 0;

int greenValue = 0;
int redValues[PIXEL_COUNT];
int greenValues[PIXEL_COUNT];
int blueValues[PIXEL_COUNT];
bool rising[PIXEL_COUNT];

bool animate = true;
int animationId = 2;
bool switchedOff = false;



#define MAX_INTENSITY 128
#define STEP_VALUE 1
#define SKIP_CYCLES 1
// SKIP_CYCLES = 1 for the matrix




void setup()
{

    initTheMatrix(0);
    //initTwinkleColor(32,32,0);

    strip.begin();
    strip.show(); // Initialize all pixels to 'off'
    pinMode(LED, OUTPUT);
    
    bool success = Particle.function("Pause", funcPause);
    bool success2 = Particle.function("ChangeAnimation", funcChangeAnimation);
    bool success3 = Particle.function("Off", funcOff);
}


int funcPause(String extra) {
    if (animate == true) {
        animate = false;
    }else {
        animate = true;
    }
  return 0;
}


int funcChangeAnimation(String extra) {
    
    // animationId +=1;
    // if (animationId == 3) { animationId == 0;}
    
    animate = false;
    
    switch (animationId) {
        case 0: // Matrix
            initTwinkleColor(32, 32, 0);
            animationId = 1;
            break;
        case 1: // Twinkle
            animationId=2;
            break;
        case 2: // AllRGB
            initTheMatrix(0);
            animationId=0;
            break;
    }
    
    animate = true;
    
    return 0;
}

int funcOff(String extra) {
    if (switchedOff == true) {
        switchedOff = false;
        initAnimation();
    }else {
        switchedOff = true;
        turnOffAll();
    }
  return 0;
}

void turnOffAll() {
    
        int loop;
        for(loop=0;loop<PIXEL_COUNT;loop++) {
            
            redValues[loop] = 0;   
            greenValues[loop] = 0;
            blueValues[loop] = 0;
            
        }
}

void initAnimation() {
    switch (animationId) {
        case 0: // Matrix
            initTheMatrix(0);
            animationId=0;
            break;
        case 1: // Twinkle
            initTwinkleColor(32, 32, 0);
            animationId = 1;
            break;
        case 2: // AllRGB
            animationId=2;
            break;
            
    }
}

void allRGB(int r, int g, int b) {
    
    if (cycleCounter % 100 == 0) {
        int loop;
        for(loop=0;loop<PIXEL_COUNT;loop++) {
            
            redValues[loop] = 255;   
            greenValues[loop] = 40;
            blueValues[loop] = 0;
            
        }
    }

}

void initTheMatrix(int rgb) {
    int i;
    for(i=0;i<PIXEL_COUNT;i++) {
        
        switch (rgb){
            case 0:
                 redValues[i] = random(0, MAX_INTENSITY);
                 greenValues[i] = 0;
                 blueValues[i] = 0;
                 break;
             case 1:
                redValues[i];
                greenValues[i] = random(0, MAX_INTENSITY);
                blueValues[i] = 0;
                break;
            case 2:
                redValues[i] = 0;
                greenValues[i] = 0;
                 blueValues[i] = random(0, MAX_INTENSITY);
                 break;
            
        }
        
        int risingValue = random(0,2);
        if (risingValue == 0) {
            rising[i] = true;
        }
        else {
            rising[i] = false;
        }
        
    }
}


void theMatrix(int rgb)
{
    int f;
    
    if (cycleCounter % 2 == 0) {
   
            for(f=0; f<PIXEL_COUNT; f++) {
                if (rising[f] == true) {
                    if (hasReachedValue(rgb, f, MAX_INTENSITY))  {
                        rising[f] = false;
                    }
                    else {
                        incrementOneValue(rgb, f, STEP_VALUE);
                    }
                } else {
                    if (redValues[f] == STEP_VALUE || greenValues[f] == STEP_VALUE || blueValues[f] == STEP_VALUE) {
                        rising[f] = true;
                    }
                    else {
                        incrementOneValue(rgb, f, STEP_VALUE * -1);
                    }
                }
            }
            

    }

}


bool hasReachedValue(int rgb, int index, int value) {
    
    switch (rgb){
        case 0:
             return (redValues[index] == value) ? true : false;
             break;
         case 1:
            return (greenValues[index] == value) ? true : false;
            break;
        case 2:
            return (blueValues[index] == value) ? true : false;
            break;
    }
    
}

void incrementOneValue(int rgb, int index, int incrementValue) {
    
    switch (rgb){
        case 0:
             redValues[index] += incrementValue ;
             break;
         case 1:
            greenValues[index] += incrementValue;
            break;
        case 2:
             blueValues[index] += incrementValue;
             break;
    }
    
}


void initTwinkleColor(int r, int g, int b) {
    int i;
    for(i=0;i<PIXEL_COUNT;i++) {
        redValues[i] = random(0, MAX_INTENSITY);
        greenValues[i] = random(0, MAX_INTENSITY);
        blueValues[i] = random(0, MAX_INTENSITY);
        
        int risingValue = random(0,2);
        if (risingValue == 0) {
            rising[i] = true;
        }
        else {
            rising[i] = false;
        }
    }
}


void twinkleColor()
{
    int f;
    
    if (cycleCounter % 2 == 0) {
        for(f=0; f<PIXEL_COUNT; f++) {
            if (rising[f] == true) {
                if (redValues[f] == MAX_INTENSITY || greenValues[f] == MAX_INTENSITY || blueValues[f] == MAX_INTENSITY) {
                    rising[f] = false;
                }
                else {
                    redValues[f] = redValues[f] + STEP_VALUE;
                    greenValues[f] = greenValues[f] + STEP_VALUE;
                    blueValues[f] = blueValues[f] + STEP_VALUE;
                }
            } else {
                if (redValues[f] == STEP_VALUE || greenValues[f] == STEP_VALUE || blueValues[f] == STEP_VALUE) {
                    rising[f] = true;
                }
                else {
                    redValues[f] = redValues[f] - STEP_VALUE;
                    greenValues[f] = greenValues[f] - STEP_VALUE;
                    blueValues[f] = blueValues[f] - STEP_VALUE;
                }
            }
        }
    }
  
}

    


void loop()
{
    
    uint16_t f;
    
    if (switchedOff == false && animate == true && animationId == 0) {
        theMatrix(0);
    }
    
    if (switchedOff == false && animate == true && animationId == 1){
        twinkleColor();
    }
    
    if (switchedOff == false && animate == true && animationId == 2){
        allRGB(32, 0, 32); 
    }
        
    cycleCounter++;
  
    // Update strip with pixel values
    for(f=0; f<PIXEL_COUNT; f++) {
        strip.setColor(f, redValues[f], greenValues[f], blueValues[f]);    
    }
     strip.show();

}
```
This code allowed me to make API calls to the particle API in order to do things like turn on the lights, change their animation and pause the animation.
After I did this, I installed the particle CLI so that I could make these calls from the command line. I did a test and voila, working web-enabled lighting! In my excitement, I made some simple batch files, stuck them on my desktop and had some lights I could be proud of. Featuring 3 animations, a red scrolling effect that reminded me of The Matrix, hence the name, as well as a fairy lights effect, and a simple static orange for when I'm not wanting them to look so flashy.
*Batch file below*
```
@echo off
echo "Lights toggled."
particle function call lights Off
```
And here they are working!
[![](http://img.youtube.com/vi/peOa4qs7SJ4/0.jpg)](http://www.youtube.com/watch?v=peOa4qs7SJ4 "ledtest")



