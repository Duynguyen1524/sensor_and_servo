
#include "ti/devices/msp/peripherals/hw_gpio.h"
#include "ti/driverlib/m0p/dl_core.h"
#include "ti_msp_dl_config.h"
#include <sys/_stdint.h>
volatile bool gCheckADC;
#define SAMPLE_PERIOD 32000
#define RESULT_SIZE (64)
volatile uint16_t gAdcResult0[RESULT_SIZE];
volatile uint16_t gAdcResult1[RESULT_SIZE];
volatile uint16_t gAdcResult2[RESULT_SIZE];
volatile uint16_t gAdcResult3[RESULT_SIZE];
#define TempThreshold (30.0)
#define TempMargin (2.5)
#define COVERED (40)
#define SHADED (120)
#define BRIGHT 999
#define DIM 200
#define OFF 5
#define DELAY 960000
#define NUM_servo 40 // servo +90 13% dc and -90 at 3% dc
#define Delta_servo 5 // delta increment pulsewidth
#define DC3 60 // for 3% dc to 13% dc
#define DC13 260
void open_vent(void);
void shut_vent(void);
int main(void)
{
/* Initialize peripherals and enable interrupts */
SYSCFG_DL_init();
NVIC_EnableIRQ(ADC12_0_INST_INT_IRQN);
gCheckADC = false;
uint16_t i = 0;
volatile float adcVolt_TMP;
volatile float temperatureC;
volatile uint32_t ldrValue, tmpValue;
DL_TimerG_startCounter(PWM_SERVO_INST);
DL_TimerG_startCounter(PWM_LED_INST);
volatile uint16_t vent_state = 0; // initial state shut servo
DL_TimerG_setCaptureCompareValue(PWM_SERVO_INST, DC3, DL_TIMER_CC_0_INDEX);
while (1) {
DL_ADC12_startConversion(ADC12_0_INST);
/* Wait until all data channels have been loaded. */
while (gCheckADC == false) {
__WFE();
}
/* Store ADC Results into their respective buffer */
gAdcResult0[i] =DL_ADC12_getMemResult(ADC12_0_INST, DL_ADC12_MEM_IDX_0);
gAdcResult1[i] =DL_ADC12_getMemResult(ADC12_0_INST, DL_ADC12_MEM_IDX_1);
gAdcResult2[i] =DL_ADC12_getMemResult(ADC12_0_INST, DL_ADC12_MEM_IDX_2);
gAdcResult3[i] =DL_ADC12_getMemResult(ADC12_0_INST, DL_ADC12_MEM_IDX_3);
i++;
gCheckADC = false;
/* Sampling done take average and control servo and LED*/
if (i >= RESULT_SIZE) {
//    tmp
    for(volatile int j =0;j<i;j++){tmpValue += gAdcResult0[j];}//take sum of tmpvalue
    tmpValue =  tmpValue/RESULT_SIZE;//take average
    adcVolt_TMP = (float)tmpValue * (2.5/4096);// change the voltage to temp value
     temperatureC = (adcVolt_TMP - 0.5) * 100 ;//convert to Celsius
//    LDR
    for(volatile int j =0;j<i;j++){ldrValue += gAdcResult1[j];}//take sum of ldr value
     ldrValue = ldrValue/RESULT_SIZE;//take average

    if(ldrValue < COVERED){
        DL_TimerG_setCaptureCompareValue(PWM_LED_INST, BRIGHT, DL_TIMER_CC_0_INDEX);
    }
    else if(ldrValue > COVERED && ldrValue < SHADED){DL_TimerG_setCaptureCompareValue(PWM_LED_INST, DIM, DL_TIMER_CC_0_INDEX);}
    else if(ldrValue > SHADED){DL_TimerG_setCaptureCompareValue(PWM_LED_INST,OFF, DL_TIMER_CC_0_INDEX);}

    if(vent_state ==0){
        if(temperatureC > (TempThreshold + TempMargin) && ldrValue > COVERED){
            open_vent();
            vent_state =1;
        }}
    else if (vent_state ==1){
        if(temperatureC < (TempThreshold - TempMargin) || ldrValue < COVERED){
            shut_vent();
            vent_state =0;

        }
    }
// end control for the sample
// __BKPT(0);
    i = 0;

delay_cycles(SAMPLE_PERIOD);
}
else { // conversions not done
;/* No action required */
}
// new conversion back to top of while(1)
DL_ADC12_enableConversions(ADC12_0_INST);
} /* End while(1) */
} /* End Main */
/* ADC Interrupt Service Routine */
void ADC12_0_INST_IRQHandler(void)
{
switch (DL_ADC12_getPendingInterrupt(ADC12_0_INST)) {
case DL_ADC12_IIDX_MEM3_RESULT_LOADED:
gCheckADC = true;
break;
default:
break;
}
}
void open_vent(void){
volatile int i = 0;
volatile int pwm_count = DC3; // dc% = (pwm_count/2000)*100
for (i = 0; i < NUM_servo; i++) {
pwm_count += Delta_servo;
DL_TimerG_setCaptureCompareValue(PWM_SERVO_INST, pwm_count, DL_TIMER_CC_0_INDEX);
delay_cycles(DELAY);
}
}
void shut_vent(void){
volatile int i = 0;
volatile int pwm_count = DC13; // dc% = (pwm_count/2000)*100
for (i = 0; i < NUM_servo; i++) {
pwm_count -= Delta_servo;
DL_TimerG_setCaptureCompareValue(PWM_SERVO_INST, pwm_count, DL_TIMER_CC_0_INDEX);
delay_cycles(DELAY);
}
}
