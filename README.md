#include <QTRSensors.h>
#define SolMotorileri 9
#define SolMotorGeri 10
#define SagMotorileri 11
#define SagMotorGeri 12
#define ena 3
#define enb 5
 
#define KP .2
#define KD 5
#define M1_minumum_hiz 80
#define M2_minumum_hiz 80
#define M1_maksimum_hiz 85
#define M2_maksimum_hiz 85
#define MIDDLE_SENSOR 3
#define NUM_SENSORS 5
#define TIMEOUT 2500
#define EMITTER_PIN 2
#define DEBUG 0
 int lastError = 0;
int last_proportional = 0;
int integral = 0;
QTRSensorsRC qtrrc((unsigned char[]) { A2, A3, A4, A5} ,NUM_SENSORS, TIMEOUT, EMITTER_PIN);
 
unsigned int sensorValues[NUM_SENSORS];
 
void setup() {
 pinMode(SolMotorileri,OUTPUT);
 pinMode(SolMotorGeri,OUTPUT);
 pinMode(SagMotorileri,OUTPUT);
 pinMode(SagMotorGeri,OUTPUT);
 pinMode(ena,OUTPUT);
 pinMode(enb,OUTPUT);
delay(1500);
manual_calibration();
set_motors(0,0);
 
}
 
void loop() {
unsigned int sensors[5];
int position = qtrrc.readLine(sensors);
int error = position - 3000;
 
int motorSpeed = KP * error + KD * (error - lastError);
lastError = error;
 
int leftMotorSpeed = M1_minumum_hiz + motorSpeed;
int rightMotorSpeed = M2_minumum_hiz - motorSpeed;
 
set_motors(leftMotorSpeed, rightMotorSpeed);
 
}
 
void set_motors(int motor1speed, int motor2speed)
{
if (motor1speed > M1_maksimum_hiz ) motor1speed = M1_maksimum_hiz; //MAKSİMUM MOTOR 1 HIZ LİMİTİ
if (motor2speed > M2_maksimum_hiz ) motor2speed = M2_maksimum_hiz; // MAKSİMUM MOTOR 2 HIZ LİMİTİ
if (motor1speed < 0) motor1speed = 30; // MİNIMUMMOTOER 1 HIZ LİMİTİ
if (motor2speed < 0) motor2speed = 30; // MİNİMUM MOTOR 2 HIZ LİMİTİ
analogWrite(ena,motor1speed);
analogWrite(enb,motor2speed);
digitalWrite(SolMotorileri,HIGH);
digitalWrite(SagMotorileri,HIGH);
digitalWrite(SolMotorGeri,LOW);
digitalWrite(SagMotorGeri,LOW);
 
}
 
void manual_calibration() {
 
int i;
for (i = 0; i < 250; i++)
{
qtrrc.calibrate(QTR_EMITTERS_ON);
delay(20);
}
 
if (DEBUG) {
Serial.begin(9600);
for (int i = 0; i < NUM_SENSORS; i++)
{
Serial.print(qtrrc.calibratedMinimumOn[i]);
Serial.print(' ');
}
Serial.println();
 
for (int i = 0; i < NUM_SENSORS; i++)
{
Serial.print(qtrrc.calibratedMaximumOn[i]);
Serial.print(' ');
}
Serial.println();
Serial.println();
}
}
