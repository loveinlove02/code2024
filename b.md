```c++
#include <LiquidCrystal_I2C.h>
#include <Wire.h> 
#include <SimpleTimer.h>
#include<Servo.h>

LiquidCrystal_I2C lcd(0x27,16,2);
Servo servo;

int v;                    //속도
int vmax;                 //최대 속도
int Sigpin =  11 ;        // 신호 입력 핀

unsigned  long T;         // 주기
double f;                 // 주파수 
char s[ 20 ];             // Serial 출력 Length
  
SimpleTimer t1;
SimpleTimer t2;

void setup() {
  Serial.begin ( 9600 );        // 시리얼모니터
  pinMode (Sigpin , INPUT);     // 속도 측정 모듈 핀모드 설정

  // LCD 설정
  lcd.init();
  lcd.backlight();

  // 스레드 설정
  t1.setInterval(100,fn1);
  t2.setInterval(100,fn2);

  // 서보 모터 설정
  servo.attach(7);
  servo.write(0);
}

void loop() {
  t1.run();
  t2.run();
}

void fn1(void) {
vmax = 0 ;
  while (digitalRead (Sigpin)); 
  while ( ! digitalRead (Sigpin));
 
  T =pulseIn (Sigpin , HIGH) + pulseIn (Sigpin , LOW);  // 주기 측정
  f = 1 / ( double ) T;             // 주파수 측정
  v = int ((f * 1e6 ) /44.0 );      // 속도 측정   
  vmax = max (v, vmax);             // 속도의 Max값 측정

  lcd.clear(); 
  lcd.setCursor(0,0);
  lcd.print(String(vmax) + "km/h"); 
}

void fn2(void) {
  sprintf (s, "% 3d km / h" , vmax);  // Serial 출력
  Serial.print("vmax: ");
  Serial.println(s);

  if (vmax>=30) {                     // 30km 이상이면 작동 
    servo.write(45);
    delay(5000);
    
    servo.write(0);
    delay(1000);
  }
}
```
