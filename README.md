# CambienRemvaCua
# ESP8266



# Arduino


#include <IRremote.h>
#include <Arduino_FreeRTOS.h>
#include <SerialCommand.h>
SerialCommand SCmd;

const int receiverPin = 7;        //chan nhan tin hieu remote
int cbmua = 8;                    //chan nhan tin hieu cambienmua
int in1 = 9;                      //chan nhan tin hieu motor1
int in2 = 10;                     //chan nhan tin hieu motor1
const int in3 = 11;               //chan nhan tin hieu motor2
const int in4 = 12;               //chan nhan tin hieu motor2

int LightValue;
int tinhieumua;
const int LIGHT_SENSOR_PIN = A0;  // chan nhan tin hieu Light sensor
const int ANALOG_THRESHOLD = 400; // muc nhan tin hieu Light sensor

IRrecv irrecv(receiverPin);       // tạo đối tượng IRrecv mới
decode_results results;           // lưu giữ kết quả giải mã tín hiệu

char vitri = 'A';                 // vi tri rem
char vitriCuaLight = 'A';         // vi tri cua

boolean flag = false;             // mark of rain sensor
boolean flag_2 = false;           // mark of light sensor

int trangThaiHeThong = 0;


void setup() {
  Serial.begin(9600);
  xTaskCreate(rainTest, "rain", 200, NULL, 1, NULL);
  xTaskCreate(lightTest, "rain", 200, NULL, 1, NULL);

}
void tienMotor1(){
  digitalWrite(in1, LOW);
  digitalWrite(in2, HIGH);
}
void dungMotor1(){
  digitalWrite(in1, LOW);
  digitalWrite(in2, LOW);
}
void luiMotor1() {
  digitalWrite(in1, HIGH);
  digitalWrite(in2, LOW);
}
void tienMotor2() {
  digitalWrite(in3, LOW);
  digitalWrite(in4, HIGH);
}
void dungMotor2() {
  digitalWrite(in3, LOW);
  digitalWrite(in4, LOW);
}
void luiMotor2() {
  digitalWrite(in3, HIGH);
  digitalWrite(in4, LOW);
}

void rainTest(void *pvParameters){
  
  //if(trangThaiHeThong == 0){
  Serial.begin(9600);
  pinMode(in1, OUTPUT);
  pinMode(in2, OUTPUT);
  SCmd.addCommand("rem",rem);
  SCmd.addCommand("cua", cua);
  SCmd.addCommand("trangThai", trangThai);
  irrecv.enableIRIn(); // start the IR receiver
  

  while(1){
    SCmd.readSerial();
    if(trangThaiHeThong == 0){
    tinhieumua=digitalRead(cbmua);

    //rain sensor
    // located at A
    if(vitri=='A'){
      int i = 0;
      if(irrecv.decode(&results)){
          Serial.println(results.value, HEX);
          if(results.value == 0xFF18E7) {
            i = 1;
          }
          irrecv.resume();
      }
      if(tinhieumua==LOW) {
        flag = true;
        i = 1;
      }
      // Do
      if(i == 1) {
        tienMotor1();
          vTaskDelay(800/portTICK_PERIOD_MS); // thoi gian chay la 800s
        dungMotor1();
        vitri='B'; 
      }
    }
    // located B
    if(vitri=='B'){
      int i = 0;

      //signal of remote
      if(irrecv.decode(&results)){
        Serial.println(results.value, HEX);
        if(results.value == 0xFF4AB5) {
          i = 1;
        }
        irrecv.resume();
      }

      //signal of rainsensor
      if(tinhieumua==HIGH && flag == true) {
        i = 1;
      }
      // Do
      if(i == 1) {
        luiMotor1();
        vTaskDelay(500/portTICK_PERIOD_MS);  // thoi gian la 500
        dungMotor1();
        vitri='A'; 
        flag = false;
      }
    }
  }
  }
}


void lightTest(void *pvParameters){
 
  Serial.begin(9600);
  
  pinMode(in3, OUTPUT);
  pinMode(in4, OUTPUT);

  while(1){
    //Light sensor
    // Located A
    if(trangThaiHeThong == 0){
    LightValue= analogRead(LIGHT_SENSOR_PIN);
    if(vitriCuaLight=='A') {
      int j = 0;

      // signal of remote
      if(irrecv.decode(&results)){
        Serial.println(results.value, HEX);
        if(results.value == 0xFF10EF) {
          j = 1;
        }
        irrecv.resume();
      }

      // signal of light sensor
      if(LightValue < ANALOG_THRESHOLD) {
        j = 1;
        flag_2 = true;
      }
      // Do
      if( j == 1) {
        tienMotor2();
        vTaskDelay(800/portTICK_PERIOD_MS);
        dungMotor2();
        vitriCuaLight='B';
      } 
    }

    // Located at B
    if(vitriCuaLight=='B') {
      int j = 0;
      // signal of remote
      if(irrecv.decode(&results)){
        Serial.println(results.value, HEX);
        if(results.value == 0xFF5AA5) {
          j = 1;
        }
        irrecv.resume();
      }

      // signal of light sensor
      if(LightValue > ANALOG_THRESHOLD  && flag_2 == true) {
        j = 1;
      }
      // DO
      if( j == 1) {
        luiMotor2();
        vTaskDelay(800/portTICK_PERIOD_MS);
        dungMotor2();
        vitriCuaLight='A';
        flag_2 = false;      
      } 
    }
  }
  }
}

void loop(){
  SCmd.readSerial();
}


void trangThai(){
  int val;
  char *arg;
  arg = SCmd.next();
  val = atoi(arg);
  if(val == 1){
    trangThaiHeThong = 1;
    Serial.println("Da nhan tin hieu");
    Serial.println(trangThaiHeThong);
  }

  else{
    trangThaiHeThong = 0;
    Serial.println("Da nhan tin hieu");
    Serial.println(trangThaiHeThong);
  }
}



void rem(){
  int val;
  char *arg;
  arg = SCmd.next();
  val = atoi(arg);
  if (val == 1  && trangThaiHeThong == 1){
    Serial.println("Da nhan tin hieu");
    tienMotor2();
    vTaskDelay(800/portTICK_PERIOD_MS);
    dungMotor2();
  }
  else if (val == 0 && trangThaiHeThong == 1){
    Serial.println("Da nhan tin hieu");
    luiMotor2();
    vTaskDelay(800/portTICK_PERIOD_MS);
    dungMotor2();
  }
}

void cua(){
  int val;
  char *arg;
  arg = SCmd.next();
  val = atoi(arg);
  if (val == 1 && trangThaiHeThong == 1){
    Serial.println("Da nhan tin hieu");
    tienMotor1();
          vTaskDelay(800/portTICK_PERIOD_MS); // thoi gian chay la 800s
        dungMotor1();
  }
  else if (val == 0 && trangThaiHeThong == 1){
    Serial.println("Da nhan tin hieu");
    luiMotor1();
        vTaskDelay(500/portTICK_PERIOD_MS);  // thoi gian la 500
        dungMotor1();
  }
}
