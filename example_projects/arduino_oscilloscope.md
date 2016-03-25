# THIS WORKS!!!

Arduino runs this.

```C
#define ANALOG_IN 0

void setup() {
  Serial.begin(9600); 
  //Serial.begin(115200); 
}

void loop() {
  int val = analogRead(ANALOG_IN);                                              
  Serial.write( 0xff );                                                         
  Serial.write( (val >> 8) & 0xff );                                            
  Serial.write( val & 0xff );
}
```

Processing IDE runs this:

```processing
 /*
 * Oscilloscope
 * Gives a visual rendering of analog pin 0 in realtime.
 * 
 * This project is part of Accrochages
 * See http://accrochages.drone.ws
 * 
 * (c) 2008 Sofian Audry (info@sofianaudry.com)
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation, either version 3 of the License, or
 * (at your option) any later version.
 * 
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 * 
 * You should have received a copy of the GNU General Public License
 * along with this program.  If not, see <http://www.gnu.org/licenses/>.
 */ 
import processing.serial.*;

Serial port;  // Create object from Serial class
int val;      // Data received from the serial port
int[] values;
float zoom;

void setup() 
{
  size(1280, 480);
  // Open the port that the board is connected to and use the same speed (9600 bps)
  port = new Serial(this, Serial.list()[0], 9600);
  values = new int[width];
  zoom = 1.0f;
  smooth();
}

int getY(int val) {
  return (int)(height - val / 1023.0f * (height - 1));
}

int getValue() {
  int value = -1;
  while (port.available() >= 3) {
    if (port.read() == 0xff) {
      value = (port.read() << 8) | (port.read());
    }
  }
  return value;
}

void pushValue(int value) {
  for (int i=0; i<width-1; i++)
    values[i] = values[i+1];
  values[width-1] = value;
}

void drawLines() {
  stroke(255);
  
  int displayWidth = (int) (width / zoom);
  
  int k = values.length - displayWidth;
  
  int x0 = 0;
  int y0 = getY(values[k]);
  for (int i=1; i<displayWidth; i++) {
    k++;
    int x1 = (int) (i * (width-1) / (displayWidth-1));
    int y1 = getY(values[k]);
    line(x0, y0, x1, y1);
    x0 = x1;
    y0 = y1;
  }
}

void drawGrid() {
  stroke(255, 0, 0);
  line(0, height/2, width, height/2);
}

void keyReleased() {
  switch (key) {
    case '+':
      zoom *= 2.0f;
      println(zoom);
      if ( (int) (width / zoom) <= 1 )
        zoom /= 2.0f;
      break;
    case '-':
      zoom /= 2.0f;
      if (zoom < 1.0f)
        zoom *= 2.0f;
      break;
  }
}

void draw()
{
  background(0);
  drawGrid();
  val = getValue();
  if (val != -1) {
    pushValue(val);
  }
  drawLines();
}
```

Need to try this:

```java
  /*
 * Oscilloscope
 * Gives a visual rendering of analog pin 0 in realtime.
 * 
 * This project is part of Accrochages
 * See http://accrochages.drone.ws
 * 
 * (c) 2008 Sofian Audry (info@sofianaudry.com)
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation, either version 3 of the License, or
 * (at your option) any later version.
 * 
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 * 
 * You should have received a copy of the GNU General Public License
 * along with this program.  If not, see <http://www.gnu.org/licenses/>.
 */ 
import processing.serial.*;
 
Serial port;  // Create object from Serial class
int val;      // Data received from the serial port
int[] values;
float zoom;
float center;
float scale = 1;
 
void setup() 
{
  size(1280, 480);
  // Open the port that the board is connected to and use the same speed (9600 bps)
  port = new Serial(this, Serial.list()[0], 9600);
  values = new int[width];
  zoom = 1.0f;
  smooth();
}
 
int getY(int val) {
  return (int)(height/2  -(val-512+center)*scale / 1023.0f * (height - 1));
}
 
int getValue() {
  int value = -1;
  while (port.available() >= 3) {
    if (port.read() == 0xff) {
      value = (port.read() << 8) | (port.read());
    }
  }
  return value;
}
 
void pushValue(int value) {
  for (int i=0; i<width-1; i++)
    values[i] = values[i+1];
  values[width-1] = value;
}
 
void drawLines() {
  stroke(255);
  
  int displayWidth = (int) (width / zoom);
  
  int k = values.length - displayWidth;
  
  int x0 = 0;
  int y0 = getY(values[k]);
  for (int i=1; i<displayWidth; i++) {
    k++;
    int x1 = (int) (i * (width-1) / (displayWidth-1));
    int y1 = getY(values[k]);
    line(x0, y0, x1, y1);
    x0 = x1;
    y0 = y1;
  }
}
 
void drawGrid() {
  stroke(255, 0, 0);
  line(0, height/2, width, height/2);
}
 
void keyReleased() {
  println(key+": "+(int)key);
  switch (key) {
    case '4':
    case '*':
      zoom *= 2.0f;
      println(zoom);
      if ( (int) (width / zoom) <= 1 )
        zoom /= 2.0f;
      break;
    case '6':
    case '/':
      zoom /= 2.0f;
      if (zoom < 1.0f)
        zoom *= 2.0f;
      break;
     case '+':     scale*=2;  break;
     case '-':   scale /= 2;  break;
     
     case '8': center += 10/scale; break;
     case '2': center -= 10/scale; break;
     case '5': center = 0; scale = 1; break;  
  }
}
 
void draw()
{
  background(0);
  drawGrid();
  val = getValue();
  if (val != -1) {
    pushValue(val);
  }
  drawLines();
}
```

Path to Sources:

http://www.instructables.com/id/Arduino-Oscilloscope-poor-mans-Oscilloscope/
https://gist.github.com/chrismeyersfsu/3270358#file-gistfile1-c
https://gist.githubusercontent.com/chrismeyersfsu/3270358/raw/360a60c8756aa57cbc9e22abc557fb4320a87028/gistfile1.c
https://gist.github.com/chrismeyersfsu/3270419#file-gistfile1-c
https://gist.githubusercontent.com/chrismeyersfsu/3270419/raw/6ea4f17834b2ea4ef38f0f2c8e1a9f63104147f8/oscilloscope.c
http://www.instructables.com/file/FHKX0Z9HET1AI5C/
https://www.google.com/search?q=simple+led+circuit&espv=2&tbm=isch&imgil=xAh8uaZb6Tn20M%253A%253BOsPHmVE9A5eu8M%253Bhttp%25253A%25252F%25252Fwww.circuitdiagram.org%25252Fsimple-basic-led-circuit.html&source=iu&pf=m&fir=xAh8uaZb6Tn20M%253A%252COsPHmVE9A5eu8M%252C_&usg=__q-_zB_YgDcAaEV-CHQniZmd1ikA%3D&biw=1138&bih=896&ved=0ahUKEwjPnua90MnLAhWiFqYKHbl9AaEQyjcIKA&ei=_KLrVs-wH6KtmAW5-4WICg#imgrc=xAh8uaZb6Tn20M%3A
http://www.electroschematics.com/2573/led-circuit/
https://www.google.com/search?sourceid=chrome-psyapi2&ion=1&espv=2&ie=UTF-8&q=led%205v%20circuit&oq=led%205v%20circuit&aqs=chrome..69i57j0l5.5699j0j8
