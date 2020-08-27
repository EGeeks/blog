# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# LSE Ready卡死
使用st-link下载器对板子供电，供电能力不够，导致外部LSE始终不能ready，但LSI可以。换外部电源供电之后，LSE也可以锁定。第二天测试，又卡死，不是电源供电问题，怀疑是LSE不起振，板子上晶振离单片机距离有点远，目前没有安装电池，测试不是没有按照电池的问题。Time_init函数加上如下初始化。
```c
#if 1
  /* Enable LSE */
  RCC_LSEConfig(RCC_LSE_ON);
  /* Wait till LSE is ready */
  while (RCC_GetFlagStatus(RCC_FLAG_LSERDY) == RESET)
  {}

  /* Select LSE as RTC Clock Source */
  RCC_RTCCLKConfig(RCC_RTCCLKSource_LSE);
#else
	RCC_LSICmd(ENABLE);
	
  while (RCC_GetFlagStatus(RCC_FLAG_LSIRDY) == RESET)
  {}		
		
  RCC_RTCCLKConfig(RCC_RTCCLKSource_LSI);
#endif
```

# RTC_WaitForSynchro卡死
首先将标准库例子中RTC_Configuration中的下面三行代码放到上面Time_init中，如果使用LSI则LSI初始化也应放出来，可能是由于没有安装电池的原因。这里和Time_init初始化重复了一部分，是由于BKP_DeInit了（猜测），否则会有异常。
```c
/**
  * @brief  Configures the RTC.
  * @param  None
  * @retval None
  */
void RTC_Configuration(void)
{
#if 0 /*move to top for RTC_WaitForSynchro dead*/
   /* Enable PWR and BKP clocks */
   RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR | RCC_APB1Periph_BKP, ENABLE);

   /* Allow access to BKP Domain */
   PWR_BackupAccessCmd(ENABLE);
#endif
  /* Reset Backup Domain */
  BKP_DeInit();
  
#if 1
#if EN_RTC_LSE
  /* Enable LSE */
  RCC_LSEConfig(RCC_LSE_ON);
  /* Wait till LSE is ready */
  while (RCC_GetFlagStatus(RCC_FLAG_LSERDY) == RESET)
  {}

  /* Select LSE as RTC Clock Source */
  RCC_RTCCLKConfig(RCC_RTCCLKSource_LSE);
#else
  RCC_LSICmd(ENABLE);
	
  while (RCC_GetFlagStatus(RCC_FLAG_LSIRDY) == RESET)
  {}		
		
  RCC_RTCCLKConfig(RCC_RTCCLKSource_LSI);
#endif
#endif
  /* Enable RTC Clock */
  RCC_RTCCLKCmd(ENABLE);

  /* Wait for RTC registers synchronization */
  RTC_WaitForSynchro();

  /* Wait until last write operation on RTC registers has finished */
  RTC_WaitForLastTask();

  /* Enable the RTC Second */
  RTC_ITConfig(RTC_IT_SEC, ENABLE);

  /* Wait until last write operation on RTC registers has finished */
  RTC_WaitForLastTask();

  /* Set RTC prescaler: set RTC period to 1sec */
  RTC_SetPrescaler(32767); /* RTC period = RTCCLK/RTC_PR = (32.768 KHz)/(32767+1) */

  /* Wait until last write operation on RTC registers has finished */
  RTC_WaitForLastTask();

}

void Time_init(void)
{
	/*Enables the clock to Backup and power interface peripherals    */
  RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP | RCC_APB1Periph_PWR,ENABLE);
	/*Allow access to Backup Registers*/
  PWR_BackupAccessCmd(ENABLE);
  
#if EN_RTC_LSE
  /* Enable LSE */
  RCC_LSEConfig(RCC_LSE_ON);
  /* Wait till LSE is ready */
  while (RCC_GetFlagStatus(RCC_FLAG_LSERDY) == RESET)
  {}

  /* Select LSE as RTC Clock Source */
  RCC_RTCCLKConfig(RCC_RTCCLKSource_LSE);
#else
  RCC_LSICmd(ENABLE);
	
  while (RCC_GetFlagStatus(RCC_FLAG_LSIRDY) == RESET)
  {}		
		
  RCC_RTCCLKConfig(RCC_RTCCLKSource_LSI);
#endif
    
	/*delay_us(100);*//*delay because uart print error chars*/
	/*uart_printf("%s line%d\r\n", __FUNCTION__, __LINE__);*/
	if (BKP_ReadBackupRegister(BKP_DR1) != CONFIGURATION_DONE)
  {
    /* Backup data register value is not correct or not yet programmed (when
       the first time the program is executed) */
    /* RTC Configuration */
    uart_printf("RTC configured....\r\n");
    RTC_Configuration();
    uart_printf("Please Set time and date from shell\r\n");		
    BKP_WriteBackupRegister(BKP_DR1, CONFIGURATION_DONE);
  }
  else
  {
		delay_us(2000);
    /* Check if the Power On Reset flag is set */
    if (RCC_GetFlagStatus(RCC_FLAG_PORRST) != RESET)
    {
      uart_printf("Power On Reset occurred....");
    }
    /* Check if the Pin Reset flag is set */
    else if (RCC_GetFlagStatus(RCC_FLAG_PINRST) != RESET)
    {
      uart_printf("External Reset occurred....");
    }
    uart_printf("No need to configure RTC....\r\n");
    /* Wait for RTC registers synchronization */
    RTC_WaitForSynchro();
    /* Enable the RTC Second */
    RTC_ITConfig(RTC_IT_SEC, ENABLE);
    /* Wait until last write operation on RTC registers has finished */
    RTC_WaitForLastTask();
  }
  /* Clear reset flags */
  RCC_ClearFlag();
	/* Check if how many days are elapsed in power down/Low Power Mode-
   Updates Date that many Times*/
  CheckForDaysElapsed();
  s_time_date.Month = BKP_ReadBackupRegister(BKP_DR2);
  s_time_date.Day = BKP_ReadBackupRegister(BKP_DR3);
  s_time_date.Year = BKP_ReadBackupRegister(BKP_DR4);
}
```
但并没有解决我的问题，怀疑是电路进入某种异常状态，虽然没有放后备电池，但总是检测到RTC已经配置，我更改CONFIGURATION_DONE的值，让RTC重新配置后，bug解除！！！



