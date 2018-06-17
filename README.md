1.main.c

#include<lpc214x.h>
#include"lcd.c"
#include"ultrasonic.c"
#include"timer.h"
#define delay for(i=0;i<65000;i++);	
unsigned int range=0,i;
int main()
{
  VPBDIV=0x01;                 // PCLK = 60MHz
  IO1DIR=0xffffffff;
  IODIR0=datalines;
  IODIR1=0X01C00000;
  ultrasonic_init();
  lcd_init();
  show("DISTANCE: ");
  while(1)
  {
        lcd_cmd(0x8b);
        range=get_range();
        lcd_data((range/100)+48);
        lcd_data(((range/10)%10)+48);
        lcd_data((range%10)+48);
        show("cm");
        delay;
        delay;
   }
}



2.lcd.c


void lcd_init(void);
void lcd_cmd(unsigned int cmd);
void lcd_data(unsigned int data);
void show(unsigned char *data);
void delay(int count);
/*********************************************************************************/
#define RS  (1<<24)		   //p1.24
#define RW  (1<<23)		   //p1.23
#define EN  (1<<22)		   //p1.22

#define datalines 0X00003C00	//p0.10 to p0.13

/*********************************************************************************************/
void lcd_init(void)
{
   lcd_cmd(0X28);	                     // enable 5x7 mode for chars
   lcd_cmd(0X01);                           //clear  display
   lcd_cmd(0x02);		           //cursor home
   lcd_cmd(0X06);	           	  //cursor move direction  
   lcd_cmd(0x0C);		         // display on
   lcd_cmd(0X01);	        	//clear display
   lcd_cmd(0x80);                      //forcebly cursor position in line number 1
}

void lcd_cmd(unsigned int cmd)
{
 unsigned char temp=0;
 unsigned int temp1=0;
 IOCLR1= RS;		         //rs=0
 IOCLR1=EN;			//en=0
 IOCLR1=RW;			//rw =0
 temp=cmd;
 temp= (temp>>4)&0x0F;
 temp1=(temp<<10)&datalines;
 IOSET1=EN;
  delay(30);			//en =1
 IOCLR0=datalines;
 IOSET0=temp1;
 delay(1000);
 IOCLR1=EN;				//en =0
 temp=cmd;
 temp&=0x0F;
 temp1=(temp<<10)&datalines;
 IOCLR1=  RS;  //rs =0
 IOSET1 = EN; // EN = 1
  delay(30);  
 IOCLR0 = datalines;
 IOSET0=temp1;
 delay(1000);
 IOCLR1=EN;	    //en=0
}

void lcd_data(unsigned int data)
{
 unsigned char temp=0;
 unsigned int temp1=0;
 IOCLR1= RS;			// rs =0
 IOCLR1=EN;			    //en = 0
 IOCLR1=RW;				//rw =0
 temp=data;
 temp= (temp>>4)&0x0F;
 temp1=(temp<<10)&datalines;
 IOSET1=EN;				//en =1
 IOSET1=RS;
 delay(1000);				//rs=1
 IOCLR0=datalines;
 IOSET0=temp1;
 delay(1000);
 IOCLR1=EN;			   // en =0
 temp=data;
 temp&=0x0F;
 temp1=(temp<<10)&datalines;
 IOSET1 = EN; // EN = 1 
 IOSET1 = RS; //  rs =1
 delay(1000);
 IOCLR0 = datalines;
 IOSET0=temp1;
 delay(1000);
 IOCLR1=EN;	  // en =0
}

void show(unsigned char *data)
{
 while(*data)
 {
  lcd_data(*data++);
 } 
}

void delay(int count)
{
 int i,j;
 for(j=0;j<count;j++)
 {
  for(i=0;i<35;i++);
 }
}


3.ultrasonic.c

#include<lpc214x.h>
/******************************************/
void ultrasonic_init(void);
void send_pulse(void);
unsigned int get_range(void);
void timer1delay(unsigned int b);
/***************************************************************************/
#define trig (1<<0)               //P0.0  trigger 
#define echo (IO0PIN&(1<<1))         //P0.1 as EINT0   echo

void ultrasonic_init()
{
    IO0DIR|=(1<<0);
    T0CTCR=0;
    T0PR=59;
}
 
void send_pulse()
{
    T0TC=T0PC=0;
    IO0SET=trig;                            //trig=1
    timer1delay(10);                        //10us delay
    IO0CLR=trig;                            //trig=0
}
 
unsigned int get_range()
{
    unsigned int get=0;
    send_pulse();
    while(!echo);
    T0TCR=0x01;
    while(echo);
    T0TCR=0;
    get=T0TC;
     if(get<38000)
        get=get/59;
     else
        get=0;
      return get;
}

4.timer.c

void timer0delay(unsigned int a);
//void timer1delay(unsigned int b);
 
void timer0delay(unsigned int a)    //1ms
{
    T0CTCR=0X0000;
    T0PR=59999;
    T0MR0=a;
    T0MCR=0x00000004;
    T0TCR=0X02;
    T0TCR=0X01;
    while(T0TC!=T0MR0);
    T0TC=0;
}
 
void timer1delay(unsigned int b)   //1us
{
    T1CTCR=0X0000;
    T1PR=59;
    T1MR0=b;
    T1MCR=0x00000004;
    T1TCR=0X02;
    T1TCR=0X01;
    while(T1TC!=T1MR0);
    T1TC=0;
}

