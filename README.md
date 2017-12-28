#include "audio.h"
#include <SoftwareSerial.h>
#include <IRremote.h>
#include <Adafruit_NeoPixel.h>

#include <Servo.h>
#define val_max 4
#define val_min 1
#define INTERVAL_Time 10

#define humanHotSensor 4

#define PIN 6//彩灯
#define SERVOPIN SDA//舵机

Adafruit_NeoPixel strip = Adafruit_NeoPixel(2, PIN, NEO_GRB + NEO_KHZ800);

Servo myservo;
bool isUP = true;

//unsigned long lcd_time = millis(); //刷新时间计时器
int val = 0;
int music_vol = 26; //初始音量0~30
bool humanHotState = false;
boolean on_off;
boolean statusChange;
bool playing = false;

int pos = 0;
int soundNum = 1;

unsigned long Time_millis = millis();
int RECV_PIN = 10;   //红外线接收器OUTPUT端接在pin 10
IRrecv irrecv(RECV_PIN);  //定义IRrecv对象来接收红外线信号
decode_results results;   //解码结果放在decode_results构造的对象results里
struct config_type
{
  int EEPROM_music_num;       //歌曲的数目
  int EEPROM_music_vol;       //歌曲的音量
};


void setup() {
  Serial.begin(9600);
irrecv.enableIRIn(); // 启动红外解码
  pinMode(humanHotSensor, INPUT);
  audio_init(DEVICE_Flash,4,music_vol);    //初始化mp3模块
  //audio_init(DEVICE_TF, MODE_loopOne, music_vol);

  strip.begin();
  strip.show(); // Initialize all pixels to 'off'
  myservo.attach(SERVOPIN);
}

void loop() {
  humanHotState = digitalRead(humanHotSensor);
  // print out the state of the button:
  //Serial.println(humanHotState);
  //delay(1);        // delay in between reads for stability

  if (irrecv.decode(&results)) {      //解码成功，收到一组红外线信号
    if(results.value==33456255)
       audio_play();              //音频工作
    else if(results.value==33439935)
       audio_pause(); 

    else if(results.value==33472575)
       {
       music_num++;  //歌曲序号加
      if (music_num > music_num_MAX)  //限制歌曲序号范围，如果歌曲序号大于30
      {
        music_num = 1; //歌曲序号返回1
      }
      audio_choose(music_num);
      audio_play();}
    else if(results.value==33431775)
       Serial.println("D\n");
    else if(results.value==33448095)
       Serial.println("E\n");


      Serial.println("play sound.....");
      audio_play();
     audio_choose(1);
      playing = true;
    
  
  
}

void updateServo() {
  if (Time_millis > millis()) Time_millis = millis();
  if (millis() - Time_millis > INTERVAL_Time) {

    if (pos == 30) {
      isUP = true;
    } else if (pos == 130) {
      isUP = false;
    }

    if (isUP) {
      pos++;
    } else {
      pos--;
    }
    //Serial.println(pos);
    myservo.write(pos);
    Time_millis = millis();
  }
}

// Fill the dots one after the other with a color
void colorWipe(uint32_t c) {
  for (uint16_t i = 0; i < strip.numPixels(); i++) {
    strip.setPixelColor(i, c);
    strip.show();
    //delay(wait);
  }
}
