# Rocking Chair

An interactive rocking chair that uses an Arduino and an ADXL345 accelerometer to control music playback based on movement.

## How It Works

The accelerometer detects the chair's movement and sends the readings to the Arduino, which controls music playback through an Adafruit Music Maker MP3 Shield.

- **When the chair rocks:** Music plays and gets louder as movement increases.
- **When rocking slows down:** The music gradually fades.
- **When the chair stops:** The music fades out and pauses.
- **When rocking resumes:** The music fades back in and continues from where it paused.
- **When the song ends:** It automatically loops.
