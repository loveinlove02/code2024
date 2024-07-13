```cpp
#include <LiquidCrystal_I2C.h>
#include <Wire.h> 
#include<Servo.h>   

// LiquidCrystal_I2C lcd(0x3F,16,2);
LiquidCrystal_I2C lcd(0x27,16,2);

int v;       //속도 (현재 속도를 저장하는 변수)
int vmax;    //최대 속도 (지금까지 측정된 최대 속도를 저장하는 변수)
int Sigpin =  11 ;  // 신호 입력 핀

Servo servo;  // 서보 모터 객체

void  setup ()
{
  Serial.begin ( 9600 );        // 컴퓨터와의 통신을 시작합니다. 
  pinMode (Sigpin , INPUT);     // 신호 입력 핀을 입력으로 설정

  lcd.init();                   // lcd 초기화
  lcd.backlight();              // 백라이트 켜기                

  servo.attach(7);              // 서보모터 7번에 연결
  servo.write(0);               // 각도 0도
}

void  loop ()
{ 
  unsigned long T;              // 주기
  double f;                     // 주파수 
  char s[20];                   // 배열
  vmax = 0 ;
  while (digitalRead (Sigpin));        // 신호가 `HIGH`일 때까지 기다립니다.
  while ( ! digitalRead (Sigpin));     // 신호가 `LOW`로 바뀔 때까지 기다립니다.
 
  T = pulseIn(Sigpin , HIGH) + pulseIn(Sigpin , LOW);    // 주기 측정
  f = 1 / ( double ) T;                                  // 주파수 측정(주기의 역수) 주기(T)가 2초라면, 주파수(f)는 1/2 = 0.5Hz입니다

  // 공식 : v = (f * 1000000) / 44.0
  // f * 1e6: 주파수를 마이크로초 단위로 변환합니다. 1초는 1,000,000 마이크로초입니다.
  // 44.0: 센서의 감도에 따라 속도를 변환하는 숫자
  v = int ((f * 1e6 ) /44.0 );      // 현재 속도 측정   
  vmax = max (v, vmax);             // 속도의 Max값 측정 (현재 속도(v)와 이전 최대 속도(vmax)를 비교하여 더 큰 값을 vmax에 저장)
  sprintf (s, "% 3d km / h" , vmax); 
  Serial.println (s);               

  lcd.clear();         // LCD에 적힌 글씨 지우기 
  lcd.setCursor(0,0);  // 커서 설정
  lcd.print(String(vmax) + "km/h");    // LCD에 속도 출력 
  delay ( 100 );                       // 기다리기

  if(vmax>=30) {                       // 속도가 30km 이상   
    Serial.println("각도를 바꾼다");
    servo.write(45);
    delay(5000);

    Serial.println("각도를 원래대로");
    servo.write(0);
    delay(1000);    
  }
}
```
