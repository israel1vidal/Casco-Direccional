#include "I2Cdev.h"
#include "MPU6050.h"
#include "Wire.h"

// La dirección del MPU6050 puede ser 0x68 o 0x69, dependiendo
// del estado de AD0. Si no se especifica, 0x68 estará implicito
MPU6050 sensor;

// Valores RAW (sin procesar) del acelerometro y giroscopio en los ejes x,y,z
int ax, ay, az;
int gx, gy, gz;
float x2;
float y;
float temp2;
int toma = 0;
int toma2 = 0;

int boton = 4;  //Boton para controlar la intensidad de brillo de los LEDs

// Variables para mostrar y controlar la direccion
float temp;
float diff;
long tiempo_prev;
float dt;
float ang_x, ang_y;
float x;
float ang_x_prev, ang_y_prev;
char estado = 'A';

#include <Adafruit_NeoPixel.h>
#ifdef __AVR__
#include <avr/power.h>  // Required for 16 MHz Adafruit Trinket
#endif

#define PINIzq 6  // pin al que esta conectado la tira de led1
#define PINDer 3  // pin al que esta conectado la tira de led2

#define NUMPIXELS 5  // cantidad de leds en la tira

Adafruit_NeoPixel pixelsIzq(NUMPIXELS, PINIzq, NEO_GRB + NEO_KHZ800);

Adafruit_NeoPixel pixelsDer(NUMPIXELS, PINDer, NEO_GRB + NEO_KHZ800);

#define ColorAmarillo 200, 200, 0  // rojo, verde, azul (RGB) del 0-255
#define ColorNegro 0, 0, 0
#define DelayEntreLeds 100

unsigned long tiempoDelDelay = 0;
int i1 = 5;
int i2 = 0;
int iA = 5;
int iB = 0;
void GiroIzquierda(void);
void GiroDerecha(void);

void setup() {
  pixelsIzq.begin();           //inicializa la tira con el OBJETO
  pixelsIzq.setBrightness(20);  //modifica el brillo del 0-255

  pixelsDer.begin();           //inicializa la tira con el OBJETO
  pixelsDer.setBrightness(20);  //modifica el brillo del 0-255
  diff = millis();
  Serial.begin(9600);    //Iniciando puerto serial
  Wire.begin();           //Iniciando I2C
  sensor.initialize();    //Iniciando el sensor
  pinMode(boton, INPUT);
  pinMode(12, OUTPUT);
  noTone(12);
  if (sensor.testConnection()) Serial.println("Sensor iniciado correctamente");
  else Serial.println("Error al iniciar el sensor");
}


void loop() {
  switch (estado) {
    ///////////////////////////////////////////////////////////// NO HACER NADA
    case 'A':
              noTone(12);
              Serial.println("estado A");
              temp2 = millis();
              pixelsDer.clear();
              pixelsDer.show();
              pixelsIzq.clear();
              pixelsIzq.show();
              muestreo1();
              muestreo2();
              intensidad();
      break;
    ///////////////////////////////////////////////////////////// GIRO DERECHA
    case 'B':
              tone(12, 1750);
              Serial.println("estado B");
              GiroDerecha();
              calculo();
              interrupcion();
              if (millis() > (temp2 + 10000)) {
                estado = 'A';
              }
      break;
    ///////////////////////////////////////////////////////////// GIRO IZQUIERDA
    case 'C':
              tone(12, 1750);
              Serial.println("estado C");
              GiroIzquierda();
              calculo();
              interrupcion();
              if (millis() > (temp2 + 10000)) {
                estado = 'A';
              }
      break;
      /////////////////////////////////////////////////////////////
  }
}











////////////////////Funciones////////////////////
void calculo() {
  // Leer las aceleraciones y velocidades angulares
  sensor.getAcceleration(&ax, &ay, &az);
  sensor.getRotation(&gx, &gy, &gz);

  dt = (millis() - tiempo_prev) / 1000.0;
  tiempo_prev = millis();

  //Calcular los ángulos con acelerometro
  float accel_ang_x = atan(ay / sqrt(pow(ax, 2) + pow(az, 2))) * (180.0 / 3.14);
  float accel_ang_y = atan(-ax / sqrt(pow(ay, 2) + pow(az, 2))) * (180.0 / 3.14);

  //Calcular angulo de rotación con giroscopio y filtro complemento
  ang_x = 0.98 * (ang_x_prev + (gx / 131) * dt) + 0.02 * accel_ang_x;
  ang_y = 0.98 * (ang_y_prev + (gy / 131) * dt) + 0.02 * accel_ang_y;

  ang_x_prev = ang_x;
  ang_y_prev = ang_y;
  Serial.print("  Rotacion en X:");
  Serial.print(ang_x);
  Serial.print("  Rotacion en Y:");
  Serial.println(ang_y);
  delay(5);

  x = ang_x - 27;
  if (ang_x < ang_y) {
    toma = 1;
  }
  if (ang_x - 65) < ang_y) {
    toma2 = 1;
  }
}

void muestreo1() {
  calculo();
  intensidad()
  if (estado == 'A') {
    while (x > ang_y) {
      while (x > ang_y) {
        calculo();
        intensidad()
      }
      cont2 = 0;
      if (toma == 0) {
        estado = 'B';
      } else {
        toma = 0;
      }
    }
  }
}


void muestreo2() {
  calculo();
  if ((ang_x > -11) && (ang_x < 15)) {
    y = ang_y + 3;
    x2 = ang_x + 3;
  }
  if (estado == 'A') {
    while (ang_x > x2 && ang_y > y) {
      while (ang_x > x2 && ang_y > y) {
        calculo();
        intensidad()
      }
      if (toma2 == 1) {
        estado = 'C';
      } else {
        toma2 = 0;
      }
    }
  }
}


void interrupcion() {
  if (estado == 'B') {
    while (x > ang_y) {
      while (x > ang_y)estado = 'A';
      GiroDerecha();
      calculo();
      intensidad()
    }
    cont2 = 0;
    if (toma == 0) {
      estado = 'A';
    } else {
      toma = 0;
    }
  }

  if ((ang_x > -15) && (ang_x < 15)) {
    y = ang_y + 3;
    x2 = ang_x + 3;
  }
  if (estado == 'C') {
    while (ang_x > x2 && ang_y > y) {
      while (ang_x > x2 && ang_y > y) {
        calculo();
        GiroIzquierda();
        intensidad()
      }
      if (toma2 == 1) {
        estado = 'A';
      } else {
        toma2 = 0;
      }
    }
  }
}


////////////////////SENTIDOS DE GIROS////////////////////
void GiroIzquierda() {
  pixelsDer.clear();
  pixelsIzq.clear();

  if (millis() >= tiempoDelDelay + DelayEntreLeds) {

    if (i1 >= 0 ) {
      pixelsIzq.setPixelColor(i1, pixelsIzq.Color(ColorAmarillo));
      pixelsIzq.show();
      i1 --;
    }
    else {
      pixelsIzq.setPixelColor(i1, pixelsIzq.Color(ColorNegro));
      pixelsIzq.show();
      pixelsDer.setPixelColor(i2, pixelsDer.Color(ColorAmarillo));
      pixelsDer.show();
      i2++;

      if (i2 > NUMPIXELS ) {
        i1 = 5;
        i2 = -1;
      }
    }
    tiempoDelDelay = millis();
  }

}


void GiroDerecha() {
  pixelsDer.clear();
  pixelsIzq.clear();

  if (millis() >= tiempoDelDelay + DelayEntreLeds) {
    if (iA >= 0) {
      pixelsDer.setPixelColor(iA, pixelsDer.Color(ColorAmarillo));
      pixelsDer.show();
      iA--;
    } else {
      pixelsDer.setPixelColor(iA, pixelsDer.Color(ColorNegro));
      pixelsDer.show();
      pixelsIzq.setPixelColor(iB, pixelsIzq.Color(ColorAmarillo));
      pixelsIzq.show();
      iB++;
      if (iB > NUMPIXELS) {
        iA = 5;
        iB = 0;
      }
    }
    tiempoDelDelay = millis();
  }
}


void intensidad() {
  if (boton == HIGH) {
    while (boton == HIGH);
    boton = boton + 1;
    if (boton == 0) {
      pixelsIzq.setBrightness(20);
      pixelsDer.setBrightness(20);
    }
    if (boton == 1) {
      pixelsIzq.setBrightness(100);
      pixelsDer.setBrightness(100);
    }
    if (boton == 2) {
      pixelsIzq.setBrightness(240);
      pixelsDer.setBrightness(240);
    }
    if (boton == 3) {
      boton = 0;
    }
  }
}
