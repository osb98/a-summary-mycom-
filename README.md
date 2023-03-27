# **a-summary-of-mycom (atmega128)**  
  
  
## 들어가기전 간단히 알아야 할 것에 대하여  
  
  
1) 출력과 입력  
만약 DDRB = 0xff 일 경우 b포트 출력으로 설정한다는 이야기  
반대로 만약 DDRA = 0x00 일경우 a 포트 입력으로 설정한다는 이야기  
PORTA = 0xff 일때 포트 A에 출력값이 1111 1111 로 설정된다는 이야기이다.  
  
  
2) 1초 지연시킬 때 delay값  
시스템의 클록 주파수가 14.7456MHz이므로  
주기(머신사이클)와 16 (16비트 처리함으로)입니다. 16비트 이므로   
0.0678 x 10의 마이너스 육승 x 16 = 1.0848 x 10의 마이너스 육승  
그리고  
~~~
void delay(unsigned long del)  
{  
while(del--);  
}  
~~~
delay문에서는 unsigned long int범위는 0∼4,294,967,295
동작원리는 del은 1씩 빼가기 때문에. 즉 del에 4,294,967,296×0.0678×10-6×16=4,659.2초 해서 921,828,0.0678×10-6×16=1초  
즉 위의 함수문에서 del에 921828입력하면 1초가 됨  
(실측값은 오차가 있을수 있음)  
  
  
~~~
// 출력과 입력 딜레이 사용 간단한 LED 0~7 ON OFF 예제 
#include <iom128.h>  
void delay (unsigned long del) //언사인드 롱델 -와 + 부분에서 -부분을 플러스 쪽으로 넘겨 +부분 두배로 사용 선언   
{  
while(del--);  
}  

void main (void)  
{  
DDRB = Oxff; //b포트 출력설정  
  do{
      PORTB=0xff; // 포트b 초기 출력값 1111 1111  
      delay(921828); // 딜레이 1초  
      PORTB=0xff; // 포트b 출력값 0000 0000  
      delay(921828); // 딜레이 1초  
      }  
      while(1);  //반복선언
}
~~~
## 1. 외부 인터럽트  
* 핵심 포인트  
공간의 활용  
상승 하강 신호  

## 2. 타이머 인터럽트  
* 핵심 포인트  
공간의 활용 
시간개념 추가  
시간계산 방법  

## 3. step motor
* 핵심 포인트  
1) 1상여자 방식  
A B a바 b바 순서로 내부 회로 구성   
편의 성을 위해 b바는 b a바는 a로 설명   
4가지 상중에서 전진일 경우  
b B  A  A  
0   0  0   1 >> 0x01  
0   1  0   0 >> 0x04  
0   0  1   0 >> 0x02  
1   0  0   0 >> 0x08 과 같은 순서로 동작한다 후진일 경우 반대이다.  
   
   
**정방향 (전진)  0x01, 0x04, 0x02, 0x08  
역방향 (후진은) 전진 반대로**  
  
  


2) 2상여자 방식  
A B a바 b바 순서로 내부 회로 구성   
편의 성을 위해 b바는 b a바는 a로 설명  
4가지 상중에서 전진일 경우  
b B  A  A  
0   1  0   1 >> 0x05  
0   1  1   0 >> 0x06  
1   0  1   0 >> 0x0a  
1   0  0   1 >> 0x09 과 같은 순서로 동작한다 후진일 경우 반대이다.  
  
**정방향 (전진)  0x05, 0x06, 0x0a, 0x09  
역방향 (후진은) 전진 반대로**  
  
  
2) 1,2상여자 방식  
A B a바 b바 순서로 내부 회로 구성   
편의 성을 위해 b바는 b a바는 a로 설명   
4가지 상중에서 전진일 경우   
b B  A  A  
0  0  0 1 >> 0x01  
0  1  0 1 >> 0x05  
0  0  1 0 >> 0x04  
0  1  1 0 >> 0x06  
0  0  1 0 >> 0x02  
1  0  1 0 >> 0x0a  
1  0  0 0 >> 0x08  
1  0  0 1 >> 0x09 과 같은 순서로 동작한다 후진일 경우 반대이다.  
  
**정방향 (전진)  0x01, 0x05, 0x04, 0x06, 0x02, 0x0a, 0x08, 0x09  
역방향 (후진은) 전진 반대로**  
  
**1,2상 여자 모두 사용한 방식의 특징  
1상, 2상 여자 방식 보다 정확하지만 한번 움직일때 반 움직인다  
(1.8도/step 인 경우  
1상, 2상 여자방식은 200펄스로 1바퀴 움직이지만  
모두 사용한 방식은 200펄스로 반바퀴 움직인다)**
  
  
  
### step motor 예제  
~~~
//1상 여자 방식 사용 step motor 전진  
#include<iom128.h>
unsigned char data[4]={0x01, 0x04, 0x02, 0x08};
void delay(unsigned int del)
{
while(del--);
}

void main(void)
{
unsigned char i; // index
DDRA=0x0f;

do{
PORTA=data[i];
i++;
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);

if(i==4) i=0;
} while(1);
}
~~~
~~~
//2상 여자 방식 사용 step motor 전진 
#include<iom128.h>
unsigned char data[4]={0x05, 0x06, 0x0a, 0x09};
void delay(unsigned int del)
{
while(del--);
}

void main(void)
{
unsigned char i; // index
DDRA=0x0f;

do{
PORTA=data[i];
i++;
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);
delay(50000);

if(i==4) i=0;
} while(1);
}
~~
~~
//1,2상 여자 방식 모두 사용한 step motor 전진
#include<iom128.h>
unsigned char data[4]={0x01, 0x05, 0x04, 0x06, 0x02, 0x0a, 0x08, 0x09};
void delay(unsigned int del)
{
while(del--);
}

void main(void)
{
unsigned char i; // index
unsigned char k; // pulse counter
DDRA=0x0f;

for{k=0;k<50;k++}
for{i=0;i<8;i++}{
PORTA=data[i];

delay(50000);
delay(50000);

} 
}
~~~
~~~
