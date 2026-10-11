#include <SPI.h>
#include <SD.h>
#include <Wire.h>
#include <Adafruit_VS1053.h>
#include <SparkFun_ADXL345.h>

// Music Maker Shield pins
#define SHIELD_RESET -1
#define SHIELD_CS 7
#define SHIELD_DCS 6
#define CARDCS 4
#define DREQ 3

Adafruit_VS1053_FilePlayer musicPlayer =
  Adafruit_VS1053_FilePlayer(
    SHIELD_RESET, SHIELD_CS,
    SHIELD_DCS, DREQ, CARDCS
  );

ADXL345 adxl = ADXL345();

// Music file
const char* song = "/CAPSTONE/FRERE.MP3";

// Volume settings
const int normalVolume = 0;
const int silentVolume = 100;

// Motion settings
const int threshold = 9;
const int fullMotion = 30;
const unsigned long readInterval = 300UL;

// Fade settings
const float attackRate = 0.50;
const float decayRate = 0.18;
const float volumeSmoothing = 0.06;

// Variables
int lastX, lastY, lastZ;

unsigned long lastRead = 0;
unsigned long lastLoop = 0;

float motionLevel = 1.0;
float currentVolume = normalVolume;

int lastVolumeSent = -1;
bool paused = false;

void setup() {
  Serial.begin(9600);

  // Start accelerometer
  adxl.powerOn();
  adxl.setRangeSetting(2);
  adxl.readAccel(&lastX, &lastY, &lastZ);

  // Start Music Maker
  if (!musicPlayer.begin()) {
    Serial.println("Shield not found!");
    while (1);
  }

  // Start SD card
  if (!SD.begin(CARDCS)) {
    Serial.println("SD card not found!");
    while (1);
  }

  // Check music file
  if (!SD.exists(song)) {
    Serial.println("Music file not found!");
    while (1);
  }

  // Enable background playback
  if (!musicPlayer.useInterrupt(
      VS1053_FILEPLAYER_PIN_INT)) {
    Serial.println("Interrupt failed!");
    while (1);
  }

  // Set starting volume
  musicPlayer.setVolume(normalVolume, normalVolume);

  // Play music
  if (!musicPlayer.startPlayingFile(song)) {
    Serial.println("Playback failed!");
    while (1);
  }

  Serial.println("Music started!");

  lastRead = millis();
  lastLoop = millis();
}

void loop() {
  unsigned long now = millis();
  unsigned long dt = now - lastLoop;
  lastLoop = now;

  // Read movement
  if (now - lastRead >= readInterval) {
    lastRead = now;

    int x, y, z;
    adxl.readAccel(&x, &y, &z);

    int change =
      abs(x - lastX) +
      abs(y - lastY) +
      abs(z - lastZ);

    lastX = x;
    lastY = y;
    lastZ = z;

    // Calculate movement strength
    float strength = 0.0;

    if (change > threshold) {
      strength =
        (float)(change - threshold) /
        (fullMotion - threshold);

      strength = constrain(strength, 0.0, 1.0);
    }

    // Smooth movement
    if (strength > motionLevel) {
      motionLevel +=
        (strength - motionLevel) * attackRate;
    } else {
      motionLevel +=
        (strength - motionLevel) * decayRate;
    }

    Serial.print("Change: ");
    Serial.print(change);
    Serial.print(" | Motion: ");
    Serial.println(motionLevel, 3);
  }

  // Calculate target volume
  float targetVolume =
    normalVolume +
    (silentVolume - normalVolume) *
    (1.0 - motionLevel);

  // Smooth volume changes
  float smoothing =
    1.0 - pow(
      1.0 - volumeSmoothing,
      dt / 10.0
    );

  currentVolume +=
    (targetVolume - currentVolume) *
    smoothing;

  currentVolume = constrain(
    currentVolume,
    (float)normalVolume,
    (float)silentVolume
  );

  // Pause when movement stops
  if (!paused &&
      motionLevel < 0.015 &&
      currentVolume >= silentVolume - 2) {

    musicPlayer.setVolume(254, 254);
    musicPlayer.pausePlaying(true);
    paused = true;

    Serial.println("Music paused");
  }

  // Resume when rocking starts
  if (paused && motionLevel > 0.05) {
    musicPlayer.setVolume(100, 100);
    currentVolume = 100;

    musicPlayer.pausePlaying(false);
    paused = false;

    lastVolumeSent = -1;
    Serial.println("Music resumed");
  }

  // Update speaker volume
  if (!paused) {
    int vol = (int)currentVolume;

    if (vol != lastVolumeSent) {
      musicPlayer.setVolume(vol, vol);
      lastVolumeSent = vol;
    }
  }

  // Loop the music
  if (!paused && musicPlayer.stopped()) {
    Serial.println("Restarting song...");

    if (musicPlayer.startPlayingFile(song)) {
      int vol = (int)currentVolume;
      musicPlayer.setVolume(vol, vol);
      lastVolumeSent = vol;
    } else {
      Serial.println("Could not restart song!");
    }
  }

  delay(10);
}
