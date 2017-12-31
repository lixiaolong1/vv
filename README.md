#include <IRremote.h> #include <Adafruit_NeoPixel.h> #include <Wire.h> #include "I2Cdev.h" #include <SHT2x.h> #include "U8glib.h"

#define val_max 255 #define val_min 0

#define PIN 6 #define PIXEL_PIN A0 #define PIXEL_COUNT 6 #define temp1 28//#define定义常量 #define temp2 30 #define temp3 32

U8GLIB_SSD1306_128X64 u8g(U8G_I2C_OPT_NONE); #define setFont_L u8g.setFont(u8g_font_7x13) #define setFont_M u8g.setFont(u8g_font_fixed_v0r) #define setFont_S u8g.setFont(u8g_font_fixed_v0r) #define setFont_SS u8g.setFont(u8g_font_fub25n)

#define INCREASE 0x1FEF807 //增加+ #define DECREASE 0x1FE708F //减少- #define NUM_0 0X1FED827 //数字0 #define NUM_1 0X1FE807F //数字1 #define NUM_2 0X1FE40BF //数字2 #define NUM_3 0X1FEC03F //数字3 #define NUM_4 0X1FE20DF //数字4 #define NUM_5 0X1FEA05F //数字5 #define NUM_6 0X1FE609F //数字6 #define NUM_7 0X1FEE01F //数字7 #define NUM_8 0X1FE10EF //数字8 #define NUM_9 0X1FE906F //数字9 #define NUM_10 0X1FE48B7 //switch

Adafruit_NeoPixel strip = Adafruit_NeoPixel(6, PIN, NEO_GRB + NEO_KHZ800);

int RECV_PIN = 10; //红外线接收器OUTPUT端接在pin 10 IRrecv irrecv(RECV_PIN); //定义IRrecv对象来接收红外线信号 decode_results results; //解码结果放在decode_results构造的对象results里

int pos = 8, color = 100; float sensor_tem;//

uint32_t color_n[9] = { strip.Color(255, 0, 0), strip.Color(248, 141, 30), strip.Color(255, 255, 0), strip.Color(0, 255, 0), strip.Color(0, 127, 255), strip.Color(0, 0, 255), strip.Color(139, 0, 255), strip.Color(255, 255, 255), strip.Color(0, 0, 0) }; uint32_t color_m[9][3] = { {0, 255, 255}, {255, 0, 0}, {248, 141, 30}, {255, 255, 0}, {0, 255, 0}, {0, 127, 255}, {0, 0, 255}, {139, 0, 255}, {255, 255, 255} };

#include <Microduino_Motor.h> #define Pin_X A1 #define Pin_Y A2

//D6，D8控制1A，1B的电机 #define OUT1A 6 #define OUT1B 8 //D5，D7控制2A，2B的电机 #define OUT2A 5 #define OUT2B 7

int value, data;

void setup() { // put your setup code here, to run once: Serial.begin(9600); //串口初始化 pinMode(Pin_X, INPUT); pinMode(Pin_Y, INPUT); pinMode(OUT1A, OUTPUT); pinMode(OUT1B, OUTPUT); pinMode(OUT2A, OUTPUT); pinMode(OUT2B, OUTPUT);

Wire.begin(); strip.begin(); strip.show(); irrecv.enableIRIn(); for (int i = 0; i < 9; i++) { colorSetall(color_n[i]); delay(1000); } } // 启动红外解码} void motor_sta(int num, int fadeValue) { if (num == 0) { analogWrite(OUT1A, fadeValue); digitalWrite(OUT1B, LOW); analogWrite(OUT2A, fadeValue); digitalWrite(OUT2B, LOW); } else if (num == 1) { digitalWrite(OUT1A, LOW); analogWrite(OUT1B, fadeValue); digitalWrite(OUT2A, LOW); analogWrite(OUT2B, fadeValue); } }

void loop() { read(); do { draw(); } while ( u8g.nextPage() );

if (irrecv.decode(&results)) {

//解码成功，收到一组红外线信号 Serial.println(results.value, HEX);//// 输出红外线解码结果（十六进制） if (results.value != 0&&results.value !=0X1FE807F) { int value=255; int n=1; motor_sta(n,value);} if (results.value ==0X1FE807F) { int value=0; int n=0;

motor_sta(n,value);} switch (results.value) { case INCREASE: color += 50; if (color > 250) color = 250; break; case DECREASE: color -= 50; if (color < 0) color = 0; break; case NUM_1: color = 100; pos = 0; break; case NUM_2: pos = 1; color = 100; break; case NUM_3: pos = 2; color = 100; break; case NUM_4: pos = 3; color = 100; break; case NUM_5: pos = 4; color = 100; break; case NUM_6: pos = 5; color = 100; break; case NUM_7: pos = 6; color = 100; break; case NUM_8: pos = 7; color = 100; break; case NUM_9: pos = 8; color = 100; break; case NUM_10: color=0; break;}

} irrecv.resume(); colorSetall(strip.Color(map(color, val_min, val_max, 0, color_m[pos][0]), map(color, val_min, val_max, 0, color_m[pos][1]), map(color, val_min, val_max, 0, color_m[pos][2]))); } void colorSetall(uint32_t c) { for (uint16_t i = 0; i < strip.numPixels(); i++) { strip.setPixelColor(i, c); } strip.show(); } void read() { sensor_tem = SHT2x.GetTemperature() ;//把获得的温度值赋给变量sensor_tem Serial.print(sensor_tem); Serial.println("tem"); Serial.print(SHT2x.GetHumidity()); Serial.println("%"); delay(1000); }

void draw() { setFont_L; u8g.setPrintPos(1, 64); u8g.print(" "); u8g.print(sensor_tem); u8g.print("tem "); u8g.print(SHT2x.GetHumidity()); u8g.print("%"); } void colorSet(uint32_t c) { for (uint16_t i = 0; i < strip.numPixels(); i++) //从0自增到LED灯个数减1 { strip.setPixelColor(i, c); //此函数表示将第i个LED点亮 } strip.show(); //LED灯显示 }
