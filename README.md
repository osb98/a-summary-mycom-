# **a-summary-of-mycom (atmega128)**  
###들어가기전 딜레이에 대하여   
1) 1초 지연시킬 때 delay값  
시스템의 클록 주파수가 14.7456MHz이므로  
주기(머신사이클)와 16 (16비트 처리함으로)입니다. 16비트 이므로   
0.0678 x 10의 마이너스 육승 x 16 = 1.0848 x 10의 마이너스 육승  
그리고 
~~
void delay(unsigned long del)  
{  
while(del--);  
}  
~~
delay문에서는 unsigned long int범위는 0∼4,294,967,295입니다.(p45참조)  
동작원리는 del은 1씩 빼갑니다. 즉 del에 4,294,967,296×0.0678×10-6×16=4,659.2초 해서 921,828,0.0678×10-6×16=1초  
즉 위의 함수문에서 del에 921828입력하면 1초가 됨  
(실측값은 오차가 있을수 있음)  


###1. 외부 인터럽트  
* 핵심 포인트  
공간의 활용  
상승 하강 신호  

