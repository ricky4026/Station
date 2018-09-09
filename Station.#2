#include "compiler_defs.h"
#include "C8051F340_defs.h"
#include <communication.h>
//-----------------------------------------------------------------------------
// Global CONSTANTS
//-----------------------------------------------------------------------------

#define SYSCLK      12000000           // SYSCLK frequency in Hz
#define BAUDRATE      9600           // Baud rate of UART in bps


//-----------------------------------------------------------------------------
// Function PROTOTYPES
//-----------------------------------------------------------------------------

void SYSCLK_Init (void);
void UART0_Init (void);
void PORT_Init (void);
void UART1_Init (void);

void Delay(float DTime);
void AccessTablet(ETabletCommand Command);
void Init_Device(void);

void Timer2_Init();
void putchar(unsigned char cData);


xdata TPacker Packer ;
unsigned char RxTimeOutCount=0; // UART 接收超時時間設定
unsigned char RxTimeOutCountTimer3 = 0;// UART 接收超時時間設定
unsigned char *Pointer_Packer = &Packer.UART_Mark;
unsigned char *CasetPointer_Packer = &Packer.UART_Mark;
unsigned char CIndex = 0;
bit	IsRxTimeOut =0;	
bit	IsRxTimeOutTimer3 =0;	
bit IsRxCommand=0;
sbit ss  = P1^2;
sbit Switch = P0^2;
sbit LED    = P0^3;
bit CasseteSEL = 1;

//====test
void Putchar(unsigned char cData);

//-----------------------------------------------------------------------------
// Global Variables
//-----------------------------------------------------------------------------

#define UART_BUFFERSIZE 80
unsigned char TX_Ready =1;
static char Byte;
xdata unsigned int RevTemp=0;
xdata ECasseteNum Tablet;
unsigned char CASInfoTage = 0;
unsigned char Finish = 0,Fail = 0;
unsigned char CassetCount=0;
unsigned char address = 0;
bit IsDrugTimeOut = 0;

unsigned char UART1_Debug[20];
unsigned char STX = 2;
unsigned char ETX = 3;
void UART1_Init (void){
	SBRLL1    = 0x8F;
    SBRLH1    = 0xFD;
    SCON1     = 0x10;		
    SMOD1     = 0x0c;
    SBCON1    = 0x43;  
	EIE2 = 0x02;//開啟UART1中斷
	//EIP2 = 0x02;//提高UART1中斷優先權
}
//-----------------------------------------------------------------------------
void Timer2_Init(){
	TMR2CN = 0x00;
	TMR2L  = 0x00;
	TMR2H  = 0x00;
    ET2    = 1;
}
void Timer3_Init(){
	TMR3CN = 0x00;
	TMR3L  = 0xFF;
	TMR3H  = 0xFF;
    EIE1  |= 0x80;
}
void Putchar(unsigned char cData)
{
	unsigned int i=10000;
	while( TX_Ready == 0) ;
	TX_Ready = 0;
	SBUF0 = cData;
	while(i--);	
}
void UART1_Putchar(unsigned char cData)
{
	unsigned int i=10000;
	while( (SCON1 & 0x20) ==0) ;
	SBUF1 = cData;
	while(i--);	
}

void Init_Device(void)
{
	PCA0MD &= ~0x40; 
	PORT_Init();                        // Initialize Port I/O
    SYSCLK_Init ();                     // Initialize Oscillator
    UART0_Init();
	UART1_Init();
	Timer2_Init();
	Timer3_Init();
}
int DropLimit = 0;
unsigned char CasetBCC = 0x00;
void main (void)
{
	Init_Device();
	Packer.UART_Mark = UART_Mark;
	Packer.Cast_Mark = UART_Mark;
	ss = 1;
	LED = 1;                                       // enable)
    EA = 1;

	while(1)
	{
//		Putchar(TabletCommand_Return);
		if(Packer.Tablet.Command != TabletCommand_Idel)
		{
			AccessTablet(Packer.Tablet.Command);//send Command;
			Delay(12500);
			Finish = 0;Fail = 0;
			Packer.Tablet.State = TabletState_Busy;
			while((Finish + Fail) < CassetCount) //&& !IsDrugTimeOut)//wait return; 
			{
				for(address = 0;address < CassetCount; address++)
				{
					if(Packer.CasseteInfo[address].State == TabletState_Sucess)
						continue;
					Packer.Tablet.State = TabletState_Busy;
					Putchar(UART_Mark) ;
					Putchar(TabletCommand_Return);
					CasetBCC = TabletCommand_Return;
					Putchar(Packer.CasseteInfo[address].HNumber);
					CasetBCC ^= Packer.CasseteInfo[address].HNumber;
					Putchar(Packer.CasseteInfo[address].TNumber);
					CasetBCC ^= Packer.CasseteInfo[address].TNumber;
					Putchar(Packer.CasseteInfo[address].SNumber);
					CasetBCC ^= Packer.CasseteInfo[address].SNumber;
					Putchar(ETX);
					CasetBCC ^= ETX;
					Putchar(CasetBCC);
					Delay(1000);
					Packer.CasseteInfo[address].State =Packer.Tablet.State;// TbltState;//Packer.Tablet.State;
			
					if(Packer.CasseteInfo[address].State == TabletState_Sucess)
						Finish ++;	
					else if(Packer.CasseteInfo[address].State == TabletState_Fail)	
						Fail ++;
					else
						Packer.CasseteInfo[address].State = TabletState_Busy;
				}
				DropLimit++;
				if(DropLimit > 0x0100)
					break;
			}			
			if(Finish == CassetCount)
			{
				Packer.Station.State = StationState_Finish;
				Packer.Tablet.Command = TabletCommand_Idel;
			}
			else //if(IsDrugTimeOut)
			{
				for(address = 0;address < CassetCount; address++)
				{
					if(Packer.CasseteInfo[address].State == TabletState_Busy)
						Packer.CasseteInfo[address].State = TabletState_Fail;
				}
				Packer.Tablet.Command = TabletCommand_PackStop;
				Packer.Station.State = StationState_Error;
			}
			DropLimit = 0;
			
		}
	}
}
//----------------------------------------------------------------------------------

void AccessTablet(ETabletCommand Command)
{
	unsigned int i=0;
	Packer.Tablet.CasseteNum = 0x01;					 
	for(i = 0;i<CassetCount;i++){

		Putchar(UART_Mark) ;

		Putchar(Command);	
		CasetBCC = Command;
		Putchar(Packer.CasseteInfo[i].HNumber);
		CasetBCC ^= Packer.CasseteInfo[i].HNumber;
		Putchar(Packer.CasseteInfo[i].TNumber);
		CasetBCC ^= Packer.CasseteInfo[i].TNumber;
		Putchar(Packer.CasseteInfo[i].SNumber);
		CasetBCC ^= Packer.CasseteInfo[i].SNumber;
		Putchar(Packer.CasseteInfo[i].QTYTNum);
		CasetBCC ^= Packer.CasseteInfo[i].QTYTNum;
		Putchar(Packer.CasseteInfo[i].QTYSNum);
		CasetBCC ^= Packer.CasseteInfo[i].QTYSNum;
		Putchar(ETX);
		CasetBCC ^= ETX;
		Putchar(CasetBCC);
		Packer.CasseteInfo[i].State = TabletState_Busy;
	}	

	IsRxCommand =0;
	/*
	while(!IsRxCommand) 
	{
		if (i++ >= 12000)	
		{
			i=0;
			Pointer_Packer = &Packer.UART_Mark;
			Packer.UART_Mark = UART_NoMark ;
			return;
		}
	}*/
	Pointer_Packer = &Packer.UART_Mark ;
	
}
void Delay(float time)
{

	while(time --);

}
//-----------------------------------------------------------------------------

void PORT_Init (void)
{
   P0MDOUT |= 0x10;                    // Enable UTX as push-pull output
   XBR0     = 0x01;                    // Enable UART on P0.4(TX) and P0.5(RX)
   XBR1     = 0x40;                    // Enable crossbar and weak pull-ups
   XBR2     = 0x01;
}
//-----------------------------------------------------------------------------

void SYSCLK_Init (void)
{
   OSCICN |= 0x03;                     // Configure internal oscillator for
                                       // its maximum frequency
   RSTSRC  = 0x04;                     // Enable missing clock detector
}

void UART0_Init (void)
{
   SCON0 = 0x10;                       // SCON0: 8-bit variable bit rate
                                       //        level of STOP bit is ignored
                                       //        RX enabled
                                       //        ninth bits are zeros
   TH1 = -(SYSCLK/BAUDRATE/2/4);
   CKCON &= ~0x0B;                  // T1M = 0; SCA1:0 = 01
   CKCON |=  0x01;

   TL1 = TH1;                          // init Timer1
   TMOD &= ~0xf0;                      // TMOD: timer 1 in 8-bit autoreload
   TMOD |=  0x20;
   TR1 = 1;                            // START Timer1
   TX_Ready = 1;                       // Flag showing that UART can transmit
   IP |= 0x10;                         // Make UART high priority
   ES0 = 1;                            // Enable UART0 interrupts

}

//-----------------------------------------------------------------------------
void Timer2_ISR (void) interrupt INTERRUPT_TIMER2 
{
	TF2H=0;
	if (RxTimeOutCount ++ >=3)
	{
		TR2 =0;
		TMR2H = 0x00;
		TMR2L = 0x00;
		RxTimeOutCount=0;
		Pointer_Packer =&Packer.Station.Command;
		IsRxTimeOut =1;
	}
}
//-----------------------------------------------------------------------------

void CommandANAY()
{	
	unsigned char cstnp = 0;
	CassetCount = 0;
	for(cstnp = 1;cstnp < CIndex;cstnp+=5 )
	{
		Tablet.HNumber = Packer.RecevieData[cstnp];
		Tablet.TNumber = Packer.RecevieData[cstnp+1];
		Tablet.SNumber = Packer.RecevieData[cstnp+2];
		Tablet.QTYTNum = Packer.RecevieData[cstnp+3];
		Tablet.QTYSNum = Packer.RecevieData[cstnp+4];
		RevTemp = (Tablet.HNumber & 0x0F) * 100 +
				  (Tablet.TNumber & 0x0F) *  10 +
				  (Tablet.SNumber & 0x0F) ;
		if((RevTemp / 20) == P4 || RevTemp >= 100)
		{
			Packer.CasseteInfo[CassetCount].HNumber =  Tablet.HNumber;
			Packer.CasseteInfo[CassetCount].TNumber =  Tablet.TNumber;
			Packer.CasseteInfo[CassetCount].SNumber =  Tablet.SNumber;
			Packer.CasseteInfo[CassetCount].QTYTNum =  Tablet.QTYTNum;
			Packer.CasseteInfo[CassetCount].QTYSNum =  Tablet.QTYSNum;
			Packer.CasseteInfo[CassetCount].State = TabletState_Busy;
			CassetCount++;
		}

	}
}

void RunCommand()
{
	unsigned char cstnp=0;	

	if(Packer.RecevieData[0] == EStationCommand_Return)
	{
		for(cstnp = 0;cstnp<CassetCount;cstnp++)
		{
			if(Packer.CasseteInfo[cstnp].HNumber ==  Packer.RecevieData[1] &&
			   Packer.CasseteInfo[cstnp].TNumber ==  Packer.RecevieData[2] &&
		   	Packer.CasseteInfo[cstnp].SNumber ==  Packer.RecevieData[3] )
			{
				UART1_Putchar(UART_Mark);
				UART1_Putchar(Packer.CasseteInfo[cstnp].State);
			}
		}
	}
	else if(Packer.RecevieData[0] != EStationCommand_Idel)
	{
		CommandANAY();
		if(CassetCount == 0)
			return;
		Packer.Station.Command = Packer.RecevieData[0];
		Packer.Tablet.Command = Packer.RecevieData[0];
		Packer.Station.State = 	StationState_Busy;
	}

}
byte BCCTemp,BCC,Debug_Index =0;
void UART1_ISR(void)   interrupt 16 //接收BOARD的資料
{
	EIE2 &= ~0x02;
	if (SCON1 & 0x02)		// TI1
	{
		SCON1 &= (~0x02);
	}

	if (SCON1 & 0x01)		// RI1
	{
		SCON1 &= (~0x01);
		*(Pointer_Packer) = SBUF1;
		UART1_Debug[Debug_Index] = SBUF1;
		if(++Debug_Index >= 20)
			Debug_Index=0;
		BCCTemp ^= SBUF1;
		if(Pointer_Packer == &Packer.UART_Mark)
		{
			if (Packer.UART_Mark == UART_Mark)
			{
				Packer.UART_Mark = UART_NoMark ;
				Pointer_Packer = &Packer.RecevieData[CIndex];
			}
			else
			{	
				Pointer_Packer = &Packer.UART_Mark;
			}
		}
		else if(Pointer_Packer == &Packer.RecevieData[CIndex])
		{
			if(CIndex == 0)
				BCCTemp = SBUF1;
			if(Packer.RecevieData[CIndex] == ETX)
			{	
				Pointer_Packer = &Packer.Station.BCC;
				BCC = BCCTemp;						
			}
			else
			{
				CIndex++;
				Pointer_Packer = &Packer.RecevieData[CIndex];
				
			}
		}
		else if(Pointer_Packer == &Packer.Station.BCC)	
		{
			Pointer_Packer = &Packer.UART_Mark;
			
   			if(BCC ==Packer.Station.BCC )
				RunCommand();
			CIndex = 0;
		}
		else 
		{	
			Pointer_Packer = &Packer.UART_Mark;
		}		
	}
	EIE2 |= 0x02;
}

//-----------------------------------------------------------------------------
void Timer3_ISR (void) interrupt INTERRUPT_TIMER3 
{
	TMR3CN &= 0x7F;
	if (RxTimeOutCountTimer3 ++ >=200)
	{
		TMR3CN &= 0xFB;
		TMR3H = 0xFF;
		TMR3L = 0xFF;
		RxTimeOutCountTimer3 = 0;
		IsDrugTimeOut =1;
	}
}
//----------------------------------------------------------------------------
void CassetMessageRev()
{
	Packer.Tablet.State = Packer.CassetRev[0];
}
unsigned char BCC = 0x00,CassetIndex = 0x00;
void UART0_Interrupt (void) interrupt 4  //接收CASSETE的資料
{
   if (TI0 == 1)                   // Check if transmit flag is set
   {
      TI0 = 0;                           // Clear interrupt flag
      TX_Ready = 1;                    // Indicate transmission complete

   }
   else if (RI0 == 1 )
   {
 	    RI0 = 0;                           // Clear interrupt flag
		*(CasetPointer_Packer) = SBUF0;
   		if (CasetPointer_Packer == &Packer.Cast_Mark)
		{
			if (Packer.Cast_Mark == UART_Mark)
			{
				Packer.Cast_Mark = UART_NoMark ;
				CasetPointer_Packer = &Packer.CassetRev[CassetIndex];// &Packer.Tablet.State;
			}
			else
			{	
				CassetIndex = 0;
				CasetPointer_Packer = &Packer.Cast_Mark;
			}
		}	
		else if (CasetPointer_Packer == &Packer.CassetRev[CassetIndex])
		{					
			if(Packer.CassetRev[CassetIndex] == ETX)
			{	
				CasetPointer_Packer = &Packer.Cast_Mark;			
				CassetMessageRev();
				CassetIndex = 0;							
			}
			else
			{
				CassetIndex++;
				CasetPointer_Packer = &Packer.CassetRev[CassetIndex];
				
			}	
			
		}
		else 
		{
			CassetIndex = 0;
			CasetPointer_Packer = &Packer.Cast_Mark;
		}
		
   }
}

//-----------------------------------------------------------------------------
// End Of File
//-----------------------------------------------------------------------------