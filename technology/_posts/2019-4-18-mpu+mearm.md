---
layout: post
title: “电信杯”二等奖作品《体感手套操控的超声波寻物机械臂》（《mearm》）介绍
image: /assets/img/ed/mearm/mearm.jpg
description: >
       本作品分为两大部分，超声波寻物机械臂与体感手套。超声波寻物机械臂中机械臂部分是由四个舵机与机械结构组成，前置超声波模块定位物体位置，然后进入体感手套通过透传蓝牙控制夹取物品模式，此时还会有OLED屏幕显示工作状态。体感手套主板为自制arduino最小系统板配有MPU6050，通过姿势识别发送蓝牙信号控制机械臂，弯曲传感器控制机械爪的开关。
author: author1
---
“电信杯”二等奖作品《体感手套操控的超声波寻物机械臂》（《mearm》）介绍

**原 创 性 声 明**
本团队郑重声明：所呈交的作品及论文，是本团队在指导老师的指导下，独立进行研究所取得的成果。除文中已经注明引用的内容外，本论文不包含任何其他个人或集体已经发表或撰写过的科研成果。对本文的研究做出重要贡献的个人和集体，均已在文中以明确方式标明。本声明的法律责任由本人承担。

---
## 快速目录
{:.no_toc}
1. this unordered seed list will be replaced by toc as unordered list
{:toc}

---

## 整机电路
{:.doc}
![2.1.1](/assets/img/ed/mearm/2.1.1.png)
2.1.1 原理方框图
{:.figure}
![2.1.2](/assets/img/ed/mearm/2.1.2.png)
2.1.2 体感手套主板原理方框图
{:.figure}
![2.1.3](/assets/img/ed/mearm/2.1.3.png)
2.1.3 体感手套主板芯片介绍
{:.figure}
![2.2.1](/assets/img/ed/mearm/2.2.1.png)
 2.2.1 全部系统原理图
{:.figure}
![2.2.2](/assets/img/ed/mearm/2.2.2.png) 
2.2.2 体感手套arduino主板的PCB设计图（正面）
{:.figure}
![2.2.3](/assets/img/ed/mearm/2.2.3.png) 
2.2.3 体感手套arduino主板的PCB设计图（反面）
{:.figure}

---

## 单元电路设计(略网上都有)
+ 舵机
+ OLED
+ 超声波模块
+ 蓝牙模块
+ MPU6050
+ 弯曲传感器
---

##程序代码
机械臂部分代码
~~~c
#include <string.h>
#include <Servo.h>  
#include "U8glib.h"

U8GLIB_SSD1306_128X32 u8g(U8G_I2C_OPT_NONE);

void u8g_prepare(void) {
  u8g.setFont(u8g_font_osb21);
  u8g.setFontRefHeightExtendedText();
  u8g.setDefaultForegroundColor();
  u8g.setFontPosTop(); 
}


uint8_t draw_state = 0;
char serialCmd;

void u8g_disc_circle(uint8_t a) {
  switch (serialCmd)
  {
    case 'f':u8g.drawStr( 0, 0, "fArmdow");break;
    case 'e':u8g.drawStr( 0, 0, "fArmup");break;
    case 'd':u8g.drawStr( 0, 0, "rArmup");break;
    case 'c':u8g.drawStr( 0, 0, "rArmdow");break;
    case 'h':u8g.drawStr( 0, 0, "clawope");break;
    case 'g':u8g.drawStr( 0, 0, "clawclo");break;
    case 'r':u8g.drawStr( 0, 0, "search");break;
    case 'o':u8g.drawStr( 0, 0, "findout");break;
    case 'w':u8g.drawStr( 0, 0, "working");break;
  }
}

 void draw(void) {
  u8g_prepare();
  switch(draw_state >> 3) {
    case 0: u8g_disc_circle(draw_state&7); break;
  }
}


////////////
void out_screen()
{
    u8g.firstPage();  
  do {
    draw();
  } while( u8g.nextPage() );
  
  draw_state++;
  if ( draw_state >= 1*8 )
    draw_state = 0;
    return ;
  }
/////////////

#define TB (fromPos+to*2) 
#define TF (fromPos+to*2) 
#define TR (fromPos+to*2) 
#define TC (fromPos+to*10) 

Servo base, fArm, rArm, claw ;    

const int baseMin = 0;
const int baseMax = 180;
const int rArmMin = 45;
const int rArmMax = 180;
const int fArmMin = 35;
const int fArmMax = 120;
const int clawMin = 0;
const int clawMax = 100;
const int EchoPin = 4;
const int TrigPin = 3;
 
int DSD = 15; 
int moveStep = 3;

void setup()
{
  base.attach(5);    
  delay(200);         
  rArm.attach(10);    
  delay(200);         
  fArm.attach(9);     
  delay(200);         
  claw.attach(6);     
  delay(200);          

  serialCmd = 'w'; 
  out_screen();
  
  base.write(90); 
  delay(10);
  fArm.write(90); 
  delay(10);
  rArm.write(90); 
  delay(10);
  claw.write(90);  
  delay(10); 


  u8g_prepare();
  Serial.begin(38400); 
/////////////////
  pinMode(TrigPin, OUTPUT); 
  pinMode(EchoPin, INPUT);
  pinMode(2, OUTPUT);
  digitalWrite(2,HIGH); 
/////////////////
}


void loop()
{
   

  if (Serial.available()>0) 
  {  
    
    serialCmd = Serial.read();
    out_screen();
    armDataCmd(serialCmd); 
    
////////////////////////

//////////////////////////
  }

}
 
void armDataCmd(char serialCmd)
{ 
    switch(serialCmd)
    {    
      case 'a': 
        ser('b',1);
        break;
      case 'b': 
        ser('b',-1);
        break;

      case 'c': 
        ser('r',1);
        break;
      case 'd': 
        ser('r',-1);
        break;

      case 'e': 
        ser('f',1);
        break;
      case 'f': 
        ser('f',-1);
        break;

      case 'g': 
        claw.write(clawMin);
        break;
      case 'h': 
       claw.write(clawMax);
        break;

      case 'r':
        armIniPos();  
        break;
    }
    reportStatus();
}

void ser(char servoName, int to)
{    
  int fromPos;

  switch(servoName){
    case 'b':
            fromPos=base.read();
            if(TB>= baseMin && TB<= baseMax)
                base.write(TB);
            break;
    
    case 'r':
            fromPos=rArm.read();
            if(TR>= rArmMin && TR<= rArmMax)
                rArm.write(TR);
            break;
    
    case 'f':
            fromPos=fArm.read();
            if(TF>= fArmMin && TF<= fArmMax)
                fArm.write(TF);
            break;
        
    case 'c':
            fromPos=claw.read();
            if(TC>= clawMin && TC<= clawMax)
                claw.write(TC);
            break;
  }
    
}


 
void reportStatus(){ 

}


//////
 
void servoCmd(char servoName, int toPos, int servoDelay){  
  Servo servo2go;  
  int fromPos; 
  switch(servoName){
    case 'b':
      if(toPos >= baseMin && toPos <= baseMax){
        servo2go = base;
        fromPos = base.read();  
      } 
      break;
  
    case 'c':
      if(toPos >= clawMin && toPos <= clawMax){    
        servo2go = claw;
        fromPos = claw.read();      
      }
        break; 

    case 'f':
      if(toPos >= fArmMin && toPos <= fArmMax){
        servo2go = fArm;
        fromPos = fArm.read();  
      }
       break;   

    case 'r':
      if(toPos >= rArmMin && toPos <= rArmMax){
        servo2go = rArm;
        fromPos = rArm.read(); 
      }  
      break; 
  }

  if (fromPos <= toPos){  
    for (int i=fromPos; i<=toPos; i++){
      servo2go.write(i);
      delay (servoDelay);
    }
  }  else { 
    for (int i=fromPos; i>=toPos; i--){
      servo2go.write(i);
      delay (servoDelay);
    }
  }
}


int findit()
{
   digitalWrite(TrigPin, LOW);
  delayMicroseconds(2); 
  digitalWrite(TrigPin, HIGH); 
  delayMicroseconds(10); 
  digitalWrite(TrigPin, LOW); 
  
  return ((int)(pulseIn(EchoPin, HIGH) / 58.0));
  }

void armIniPos(){
  int robotIniPosArray[4][3] = {
    {'b', 90, DSD},
    {'r', 90, DSD},
    {'f', 90, DSD},
    {'c', 90, DSD} 
  };   

  for (int i = 0; i < 4; i++){
    servoCmd(robotIniPosArray[i][0], robotIniPosArray[i][1], robotIniPosArray[i][2]);
  }


int fin,ste; 
fin = 90; ste = 1;
while(1)
{
  if(fin == baseMax)
    ste = -1;

  if(fin == baseMin)
    ste = 1;
  
  if(findit()<=12)
    {
      delay(500);
      if(findit()<=12)
      {base.write(fin+8*ste);
      serialCmd = 'o';
      out_screen();
        break;}
      }
 /////////
  base.write(fin);
  fin += ste;
  delay(10);
  }
  //////////
  for(int i = 0 ; i <= 100 ; i++)
  {
      while(Serial.available()>0)char cle = Serial.read();
  } 
///////////
}
~~~


体感手套部分代码
~~~c++
#include <MPU6050.h>
#include <I2Cdev.h>
#include <Wire.h>

MPU6050 accelgyro;

unsigned long now, lastTime =0;
float dt;

int16_t ax, ay, az, gx, gy, gz;
float aax=0, aay=0, aaz=0, agx=0, agy=0, agz=0;
long axo = 0, ayo = 0, azo = 0;             //加速度计偏移量
long gxo = 0, gyo = 0, gzo = 0;             //陀螺仪偏移量
 
float pi = 3.14;
float AcceRatio = 16384.0;                  //加速度计比例系数
float GyroRatio = 131.0;                    //陀螺仪比例系数
 
uint8_t n_sample = 8;                       //加速度计滤波算法采样个数
float aaxs[8] = {0}, aays[8] = {0}, aazs[8] = {0};         //x,y轴采样队列
long aax_sum, aay_sum,aaz_sum;                      //x,y轴采样和
 
float a_x[10]={0}, a_y[10]={0},a_z[10]={0} ,g_x[10]={0} ,g_y[10]={0},g_z[10]={0}; //加速度计协方差计算队列
float Px=1, Rx, Kx, Sx, Vx, Qx;             //x轴卡尔曼变量
float Py=1, Ry, Ky, Sy, Vy, Qy;             //y轴卡尔曼变量
float Pz=1, Rz, Kz, Sz, Vz, Qz;             //z轴卡尔曼变量

int a,b,c,d,e,f,k,temp_up,temp_down,aa,bb,cc,dd,ee,ff,kk,wanquu=0,wanqqu=0;
long long i=2;
#define LED   10

void setup()
{
    Wire.begin();
    Serial.begin(38400);
    pinMode(LED,OUTPUT);
    Serial.print('r');
    pinMode(3,OUTPUT);
    digitalWrite(3,HIGH);
    delay(8000);
    accelgyro.initialize();                 //初始化
 
    unsigned short times = 100;             //采样次数
    for(int i=0;i<times;i++)
    {
        accelgyro.getMotion6(&ax, &ay, &az, &gx, &gy, &gz); //读取六轴原始数值
        axo += ax; ayo += ay; azo += az;      //采样和
        gxo += gx; gyo += gy; gzo += gz;
    
    }
    
    axo /= times; ayo /= times; azo /= times; //计算加速度计偏移
    gxo /= times; gyo /= times; gzo /= times; //计算陀螺仪偏移
}
 
void loop()
{
    unsigned long now = millis();             //当前时间(ms)
    dt = (now - lastTime) / 1000.0;           //微分时间(s)
    lastTime = now;                           //上一次采样时间(ms)
 
    accelgyro.getMotion6(&ax, &ay, &az, &gx, &gy, &gz); //读取六轴原始数值
 
    float accx = ax / AcceRatio;              //x轴加速度
    float accy = ay / AcceRatio;              //y轴加速度
    float accz = az / AcceRatio;              //z轴加速度
 
    aax = atan(accy / accz) * (-180) / pi;    //y轴对于z轴的夹角
    aay = atan(accx / accz) * 180 / pi;       //x轴对于z轴的夹角
    aaz = atan(accz / accy) * 180 / pi;       //z轴对于y轴的夹角
 
    aax_sum = 0;                              // 对于加速度计原始数据的滑动加权滤波算法
    aay_sum = 0;
    aaz_sum = 0;
  
    for(int i=1;i<n_sample;i++)
    {
        aaxs[i-1] = aaxs[i];
        aax_sum += aaxs[i] * i;
        aays[i-1] = aays[i];
        aay_sum += aays[i] * i;
        aazs[i-1] = aazs[i];
        aaz_sum += aazs[i] * i;
    
    }
    
    aaxs[n_sample-1] = aax;
    aax_sum += aax * n_sample;
    aax = (aax_sum / (11*n_sample/2.0)) * 9 / 7.0; //角度调幅至0-90°
    aays[n_sample-1] = aay;                        //此处应用实验法取得合适的系数
    aay_sum += aay * n_sample;                     //本例系数为9/7
    aay = (aay_sum / (11*n_sample/2.0)) * 9 / 7.0;
    aazs[n_sample-1] = aaz; 
    aaz_sum += aaz * n_sample;
    aaz = (aaz_sum / (11*n_sample/2.0)) * 9 / 7.0;
 
    float gyrox = - (gx-gxo) / GyroRatio * dt; //x轴角速度
    float gyroy = - (gy-gyo) / GyroRatio * dt; //y轴角速度
    float gyroz = - (gz-gzo) / GyroRatio * dt; //z轴角速度
    agx += gyrox;                             //x轴角速度积分
    agy += gyroy;                             //x轴角速度积分
    agz += gyroz;
    
    /* kalman start */
    Sx = 0; Rx = 0;
    Sy = 0; Ry = 0;
    Sz = 0; Rz = 0;
    
    for(int i=1;i<10;i++)
    {                 //测量值平均值运算
        a_x[i-1] = a_x[i];                      //即加速度平均值
        Sx += a_x[i];
        a_y[i-1] = a_y[i];
        Sy += a_y[i];
        a_z[i-1] = a_z[i];
        Sz += a_z[i];
    
    }
    
    a_x[9] = aax;
    Sx += aax;
    Sx /= 10;                                 //x轴加速度平均值
    a_y[9] = aay;
    Sy += aay;
    Sy /= 10;                                 //y轴加速度平均值
    a_z[9] = aaz;
    Sz += aaz;
    Sz /= 10;
 
    for(int i=0;i<10;i++)
    {
        Rx += sq(a_x[i] - Sx);
        Ry += sq(a_y[i] - Sy);
        Rz += sq(a_z[i] - Sz);
    
    }
    
    Rx = Rx / 9;                              //得到方差
    Ry = Ry / 9;                        
    Rz = Rz / 9;
  
    Px = Px + 0.0025;                         // 0.0025在下面有说明...
    Kx = Px / (Px + Rx);                      //计算卡尔曼增益
    agx = agx + Kx * (aax - agx);             //陀螺仪角度与加速度计速度叠加
    Px = (1 - Kx) * Px;                       //更新p值
 
    Py = Py + 0.0025;
    Ky = Py / (Py + Ry);
    agy = agy + Ky * (aay - agy); 
    Py = (1 - Ky) * Py;
  
    Pz = Pz + 0.0025;
    Kz = Pz / (Pz + Rz);
    agz = agz + Kz * (aaz - agz); 
    Pz = (1 - Kz) * Pz;
 
    /* kalman end */

    
    if((int)agx>45 ) cc+=2; //大臂向下
    else if((int)agx<-45)dd+=2;  //大臂向上

    else if((int)agy>40 ) ee+=2; //前臂向左
    else if((int)agy<-30)ff+=2;  //前臂向右

    else if((int)agx>30 ) cc++; //大臂向下
    else if((int)agx<-30)dd++;  //大臂向上

    else if((int)agy>25 ) ee++; //前臂向左
    else if((int)agy<-20) ff++;  //前臂向右
    
    if(cc>5) 
    {Serial.print('c'); cc=0;}
    if(dd>5) 
    {Serial.print('d'); dd=0;}

    if(ee>5) 
    {Serial.print('e'); ee=0;}
    if(ff>5) 
    {Serial.print('f'); ff=0;}
//    
   // if((int)agz>45 ) aa++; //大臂向下
  //  else if((int)agz<-45)bb++;  //大臂向上
 //   if(aa>30) 
 //   {Serial.print('a'); aa=0;}
 //   if(bb>30) 
  //  {Serial.print('b'); bb=0;}

    

    
//    else if((int)agy>45)Serial.print('e');//前臂向左
//    else if((int)agy<-45)Serial.print('f');//前臂向右

   // temp_up=agz+10;
   // temp_down=agz-10;
//    else if((int)agz<-45)Serial.print('a');//base向左
//    else if((int)agz>45)Serial.print('b');//base向右
    
    
//    else if(agx>-30&&agx<30&&agy>-30&&agy<30) digitalWrite(LED, HIGH);
//    else digitalWrite(LED, LOW);
   if(i%10==0)
{
  if(analogRead(A0)<750) 
  {   Serial.print('h'); i++; delay(200);}
  i-=10;
}
  if(i%10==1)
  {
    if(analogRead(A0)>980) 
  {   Serial.print('g'); i++;delay(200);  }
  i-=10;
  }
  

//  else if(wanqu>00)
  {
    //45度
  }
//  else //四十五度以下
 
    delay(20);
    i+=2;
}

~~~