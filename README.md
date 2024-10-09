# STM32WB0x-Demo-Workshop
Discover how the [STM32WB0 series](https://community.st.com/t5/developer-news/new-stm32wb0-products-facilitate-bluetooth-low-energy-5-4/ba-p/695356) can streamline your Bluetooth® Low Energy 5.4 integration.


# Overview
This repository integrates the PDF material used during [STM32WB0 Demo Workshop event](https://content.st.com/stm32wb0x-workshop-emea.html)

Thanks to material you will be able to replicate:
1. How to build basic beacon over [NUCLEO-WB05KZ](https://www.st.com/en/evaluation-tools/nucleo-wb05kz.html)
2. How to create a basic peripheral profile (P2PServer) over [NUCLEO-WB09KE](https://www.st.com/en/evaluation-tools/nucleo-wb09ke.html)
3. How to evaluate and play (ESL like) with Periodic Advertising with Response feature over [NUCLEO-WB09KE](https://www.st.com/en/evaluation-tools/nucleo-wb09ke.html)
4. How to easily add BLE to you existing application using Zephyr ecosystem thanks to [X-NUCLEO-WB05KN1](https://www.st.com/en/evaluation-tools/x-nucleo-wb05kn1.html)
5. Understand how to secure your WB0 hardware design, how to test it and move to certification

All samples examples used during the session are part of [STM32CubeWB0](https://www.st.com/en/embedded-software/stm32cubewb0.html#get-software)

***Installation requirements***
- [STM32CubeWB0](https://www.st.com/en/embedded-software/stm32cubewb0.html#get-software)
- [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)


## 1. Build Beacon application over NUCLEO-WB05KZ
In 03-STM32WB0-WS-Handson Introduction and Beacon.pdf you will understand BLE Beacon application concepts and material required to build application over NUCLEO-WB05KZ.

**How to replicate:** From [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html), in file menu > select new STM32 Project > in Example Selector import BLE_Beacon example code. 
                                                                                                                                                                                              

						   
## 2. Build a peripheral profile (P2PServer) over NUCLEO-WB09KE
In 04-STM32WB0-WS-Handson_P2PServer.pdf file you will understand BLE profile concepts, trough service and characteristic demystification.

**How to replicate:** Please follow the steps by steps explanations part of the pdf. You will find the needed BLE_WS_WB0_HandsOn.ioc here **[link](https://github.com/stm32-hotspot/STM32WB0x-Demo-Workshop/tree/main/BLE_WS_WB0_HandsOn%20P2PServer/BLE_WS_WB0_HandsOn.ioc )**

To ease Profile configuration (UUIDs) and application code to be added in user section, you can copy code from bellow lines.

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**_BLE_P2PSever UUIDs_**

Service UUID
```c
8F E5 B3 D5 2E 7F 4A 98 2A 48 7A CC 40 FE 00 00
```
LED UUID
```c
19 ED 82 AE ED 21 4C 9D 41 45 22 8E 41 FE 00 00
```
Switch UUID
```c
19 ED 82 AE ED 21 4C 9D 41 45 22 8E 42 FE 00 00
```

-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**_Add application code to move to discoverable_**

1.code needs to be added in **STM32_WPAN/App/app_ble.c** inside the function App_BLE_Init ~line 558 in **/*USER CODE BEGIN APP_BLE_Init_2*/**

```c
APP_BLE_Procedure_Gap_Peripheral(PROC_GAP_PERIPH_ADVERTISE_START_FAST);
```
2.still in **STM32_WPAN/App/app_ble.c** inside SVCCTL_App_Notification function
~line 626 between tags **/*USER CODE BEGIN EVT_DISCONN_COMPLETE*/**

```c
APP_BLE_Procedure_Gap_Peripheral(PROC_GAP_PERIPH_ADVERTISE_START_FAST);
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**_Add application code to toggle LED from client_**

code needs to be added in **STM32_WPAN/App/p2p_server_app.c** inside the function P2P_SERVER_Notification() ~line 112 in **/*USER CODE BEGIN Service1Char1_WRITE_NO_RESP_EVT*/**

```c
HAL_GPIO_TogglePin(GPIOB, LED_GREEN_Pin|LED_BLUE_Pin|LED_RED_Pin);
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**_Add application code to rise an alarm from device to Smartphone_**

0. Enable RCC_SYSCFG clock in **main.c**  ~line 279  in **/*SER CODE BEGIN MX_GPIO_Init_1*/**
This section will not be needed for coming version of STM32CubeIDE

```c
__HAL_RCC_SYSCFG_CLK_ENABLE();
```  
1. code needs to be added in **Core/Inc/app_conf.h** ~line 436  in **/*USER CODE BEGIN CFG_Task_Id_t*/**

```c
TASK_BUTTON_1,
```

2. in **STM32_WPAN/App/p2p_server_app.c** inside the function P2P_SERVER_APP_Init() ~line 183 add the below code in 
**/*USER CODE BEGIN Service1_APP_Init*/**

```c
UTIL_SEQ_RegTask( 1U << TASK_BUTTON_1, UTIL_SEQ_RFU, P2P_SERVER_Switch_c_SendNotification);
```

3. add the below code in **Core/Src/app_entry.c** ~line 513 inside **/*USER CODE BEGIN FD_WRAP_FUNCTIONS*/** 

```c
void HAL_GPIO_EXTI_Callback(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin)
{
	  if (GPIO_Pin == B1_Pin)
	  {
	    UTIL_SEQ_SetTask(1U << TASK_BUTTON_1, CFG_SEQ_PRIO_0);
	  }

	  return;
}
```
4.  in **STM32_WPAN/App/p2p_server_app.c** in the functionP2P_SERVER_Switch_c_SendNotification() ~line 205 inside **/*USER CODE BEGIN Service1Char2_NS_1*/** add:

```c
a_P2P_SERVER_UpdateCharData[0] = 0x01; /* Device Led selection */
a_P2P_SERVER_UpdateCharData[1] = 0x00;
/* Update notification data length */
p2p_server_notification_data.Length = (p2p_server_notification_data.Length) + 2;
notification_on_off = Switch_c_NOTIFICATION_ON;
```

## 3. Evaluate and Play with PAwR over NUCLEO-WB09KE
This hands on allows to play with BLE 5.4 PAwR feature. In 05-STM32WB0-WS-PAwR.pdf file you will understand PAwR concepts and applications targetted.

**How to replicate:** the demo ([ESL like](https://github.com/stm32-hotspot/STM32WB0-BLE-PAwR-ESL)) is based on samples code part of [STM32CubeWB0](https://www.st.com/en/embedded-software/stm32cubewb0.html#get-software) 

1. Build and flash STM32Cube_FW_WB0_V1.0.0\Projects\NUCLEO-WB09KE\Applications\BLE\BLE_PAwR_Broadcaster to NUCLEO-WB09KE
   - or flash the image part of PAwR binaries repository
3. Build and flash STM32Cube_FW_WB0_V1.0.0\Projects\NUCLEO-WB09KE\Applications\BLE\BLE_PAwR_Observer to NUCLEO-WB09KE
   - or flash the image part of PAwR binaries repository
5. Connect BLE_PAwR_Broadcaster NUCLEO-WB09KE to Docklight or Tera Term
   - Baudrate 115200 bps
   - Data bits: 8
   - Parity: none
   - Stop bits: 1.
6. Connect BLE_PAwR_Observer NUCLEO-WB09KE to Docklight or Tera Term
   - Baudrate 921600 bps
   - Data bits: 8
   - Parity: none
   - Stop bits: 1.
7. Once board associated (PAwR train exhanged) insert AT command on the BLE_PAwR_Broadcaster terminal to control LED of the BLE_PAwR_Observer.Type ATE to enable local echo.

- Turn on an LED on BLE_PAwR_Observer:
- - AT+LED=0,0,1
- Turn off the LED on BLE_PAwR_Observer with:
- - AT+LED=0,0,0

The demo is based on ESL profile implemenatation. 

I would strongly recommand to follow **https://github.com/stm32-hotspot/STM32WB0-BLE-PAwR-ESL** to get more details how to replicate and how to understand demo.

## 4. Add BLE to you existing application using Zephyr ecosystem thanks to X-NUCLEO-WB05KN1
First 06-STM32WB0-WS-Handson_BLE_AddOn.pdf will give you clear understanding about network processor topology and advantages. 
Thanks to X-NUCLEO-WB05KN1 expansion board you will then able to easily add BLE to an existing application.

**How to replicate:** The demo is based on Zephyr ecosytem and associated shield https://docs.zephyrproject.org/latest/boards/shields/x_nucleo_wb05kn1/doc/index.html. 

Here we do recommand to follow up [Zephyr instructions](https://docs.zephyrproject.org/latest/develop/getting_started/index.html) for setup installation and build instruction.

## 5. Understand how to design your HW over WB0, how to test it and move to certification
In this part you will first understand how to secure your schematics and layout over STM32WB0, and how to tests it thanks to [STM32CubeMonRF](https://www.st.com/en/development-tools/stm32cubemonrf.html)

**How to replicate:** The sample code required to test your HW is part of [STM32CubeWB0](https://www.st.com/en/embedded-software/stm32cubewb0.html#get-software) 
1. Build and flash STM32Cube_FW_WB0_V1.0.0\Projects\NUCLEO-WB09KE\Applications\BLE\BLE_TransparentMode to NUCLEO-WB09KE
2. Open STM32CubeMonRF
3. Set Baudrate to 115200 bps
4. From RF tests panel set a tone and access any RF tests
5. see results on spectrum analyzer
   
