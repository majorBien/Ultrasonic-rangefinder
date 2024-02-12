
#include "main.h"
#include "adc.h"
#include "dma.h"
#include "i2c.h"
#include "tim.h"
#include "usart.h"
#include "gpio.h"
 
/* Private includes ----------------------------------------------------------*/
/* USER CODE BEGIN Includes */
#include <stdio.h>
#include <string.h>
#include "GFX.h"
/* USER CODE END Includes */
 
/* Private typedef -----------------------------------------------------------*/
/* USER CODE BEGIN PTD */
 
/* USER CODE END PTD */
 
/* Private define ------------------------------------------------------------*/
/* USER CODE BEGIN PD */
/* USER CODE END PD */
 
/* Private macro -------------------------------------------------------------*/
/* USER CODE BEGIN PM */
 
/* USER CODE END PM */
 
/* Private variables ---------------------------------------------------------*/
TIM_HandleTypeDef htim1;
 
UART_HandleTypeDef huart1;
UART_HandleTypeDef huart2;
 
/* USER CODE BEGIN PV */
 
/* USER CODE END PV */
 
/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
/* USER CODE BEGIN PFP */
 
/* USER CODE END PFP */
 
/* Private user code ---------------------------------------------------------*/
/* USER CODE BEGIN 0 */
void delay (uint16_t time)
{
    __HAL_TIM_SET_COUNTER(&htim1, 0);
    while (__HAL_TIM_GET_COUNTER (&htim1) < time);
}
 
 
uint32_t IC_Val1 = 0;
uint32_t IC_Val2 = 0;
uint32_t Difference = 0;
uint8_t Is_First_Captured = 0;  // is the first value captured ?
uint8_t Distance  = 0;
 
#define TRIG_PIN GPIO_PIN_4
#define TRIG_PORT GPIOB
 
void HAL_TIM_IC_CaptureCallback(TIM_HandleTypeDef *htim)
{
    if (htim->Channel == HAL_TIM_ACTIVE_CHANNEL_1)  // if the interrupt source is channel1
    {
        if (Is_First_Captured==0) // if the first value is not captured
        {
            IC_Val1 = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_1); // read the first value
            Is_First_Captured = 1;  // set the first captured as true
            // Now change the polarity to falling edge
            __HAL_TIM_SET_CAPTUREPOLARITY(htim, TIM_CHANNEL_1, TIM_INPUTCHANNELPOLARITY_FALLING);
        }
 
        else if (Is_First_Captured==1)   // if the first is already captured
        {
            IC_Val2 = HAL_TIM_ReadCapturedValue(htim, TIM_CHANNEL_1);  // read second value
            __HAL_TIM_SET_COUNTER(htim, 0);  // reset the counter
 
            if (IC_Val2 > IC_Val1)
            {
                Difference = IC_Val2-IC_Val1;
            }
 
            else if (IC_Val1 > IC_Val2)
            {
                Difference = (0xffff - IC_Val1) + IC_Val2;
            }
 
            Distance = Difference * .034/2;
            Is_First_Captured = 0; // set it back to false
 
            // set polarity to rising edge
            __HAL_TIM_SET_CAPTUREPOLARITY(htim, TIM_CHANNEL_1, TIM_INPUTCHANNELPOLARITY_RISING);
            __HAL_TIM_DISABLE_IT(&htim1, TIM_IT_CC1);
        }
    }
}
 
 
void HCSR04_Read (void)
{
    HAL_GPIO_WritePin(TRIG_PORT, TRIG_PIN, GPIO_PIN_SET);  // pull the TRIG pin HIGH
    delay(10);  // wait for 10 us
    HAL_GPIO_WritePin(TRIG_PORT, TRIG_PIN, GPIO_PIN_RESET);  // pull the TRIG pin low
 
    __HAL_TIM_ENABLE_IT(&htim1, TIM_IT_CC1);
}
/* USER CODE END 0 */
 
/**
  * @brief  The application entry point.
  * @retval int
  */
int main(void)
{
  /* USER CODE BEGIN 1 */
    uint16_t raw;
    char msg[8];
 
    int x = 3;
    int y = 10;
    int x2 = 12;
    int y2 = 10;
    int x3 = 21;
    int y3 = 10;
    int x4 = 30;
    int y4 = 10;
    int x5 = 39;
    int y5 = 10;
    int x6 = 48;
    int y6 = 10;
    int x7 = 57;
    int y7 = 10;
    int x8 = 66;
    int y8 = 10;
    int x9 = 75;
    int y9 = 10;
    int x10 = 84;
    int y10 = 10;
    int x11 = 93;
    int y11 = 10;
 
 
    int rozx = 1;
    int rozy = 1;
    int rozmiar = 0;
  /* USER CODE END 1 */
 
  /* MCU Configuration--------------------------------------------------------*/
 
  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
  HAL_Init();
 
  /* USER CODE BEGIN Init */
 
  /* USER CODE END Init */
 
  /* Configure the system clock */
  SystemClock_Config();
 
  /* USER CODE BEGIN SysInit */
 
  /* USER CODE END SysInit */
 
  /* Initialize all configured peripherals */
  MX_GPIO_Init();
  MX_DMA_Init();
  MX_USART2_UART_Init();
  MX_I2C1_Init();
  MX_ADC1_Init();
  MX_TIM1_Init();
  /* USER CODE BEGIN 2 */
  SSD1306_init();
  HAL_TIM_IC_Start_IT(&htim1, TIM_CHANNEL_1);
  /* USER CODE END 2 */
 
  /* Infinite loop */
  /* USER CODE BEGIN WHILE */
  while (1)
  {
    /* USER CODE END WHILE */
      /* USER CODE END WHILE */
      HCSR04_Read();
          HAL_GPIO_WritePin(GPIOA, GPIO_PIN_11, GPIO_PIN_SET);
          HAL_ADC_Start(&hadc1);
          HAL_ADC_PollForConversion(&hadc1, HAL_MAX_DELAY);
          raw = HAL_ADC_GetValue(&hadc1);
          HAL_GPIO_WritePin(GPIOA, GPIO_PIN_11, GPIO_PIN_RESET);
          sprintf(msg, "%hu\r\n", Distance);
        HAL_UART_Transmit(&huart2, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);
        //  SSD1306_display_clear();
        //  HAL_Delay(100);
 
 
          if(raw<50)
          {
              HAL_Delay(10);
              y--;
              y2--;
              y3--;
              y4--;
              y5--;
              y6--;
              y7--;
              y8--;
              y9--;
              y10--;
              y11--;
              HAL_Delay(10);
          }
 
 
          if(raw>1100&&raw<1700)
              {
                  HAL_Delay(10);
                 y++;
                 y2++;
                 y3++;
                 y4++;
                 y5++;
                 y6++;
                 y7++;
                 y8++;
                 y9++;
                 y10++;
                 y11++;
                  HAL_Delay(10);
              }
 
          if(raw>1800&&raw<3100)
                  {
                      HAL_Delay(10);
                    x++;
                    x2++;
                    x3++;
                    x4++;
                    x5++;
                    x6++;
                    x7++;
                    x8++;
                    x9++;
                    x10++;
                    x11++;
 
                      HAL_Delay(10);
                  }
 
          if(raw>680&&raw<900)
                  {
 
                      HAL_Delay(10);
                    x--;
                    x2--;
                    x3--;
                    x4--;
                    x5--;
                    x6--;
                    x7--;
                    x8--;
                    x9--;
                    x10--;
                    x11--;
                      HAL_Delay(10);
                  }
 
          if(raw>2060&&raw<2110)
                      {
              HAL_Delay(100);
      rozmiar++;
      HAL_Delay(10);
 
 
 
                      }
 
          if(rozmiar>2)
          {
              rozmiar = 0;
          }
 
          if(rozmiar == 0)
          {
                 rozx = 1;
                 rozy = 1;
          }
          if(rozmiar == 1)
          {
                 rozx = 2;
                 rozy = 1;
          }
 
          if(rozmiar == 2)
          {
                 rozx = 1;
                 rozy = 2;
          }
 
          if(x>128)
          {
            x = 3;
          }
          if(x2>128)
          {
            x2 = 3;
          }
          if(x3>128)
          {
            x3 = 3;
          }
          if(x4>128)
          {
            x4 = 3;
          }
          if(x5>128)
          {
            x5 = 3;
          }
          if(x6>128)
          {
            x6 = 3;
          }
          if(x7>128)
          {
            x7 = 3;
          }
          if(x8>128)
          {
            x8 = 3;
          }
          if(x9>128)
          {
            x9 = 3;
          }
          if(x10>128)
          {
            x10 = 3;
          }
          if(x11>128)
              {
                x11 = 3;
              }
 
          if(x<0)
          {
            x = 125;
          }
 
          if(x2<0)
          {
            x2 = 125;
          }
 
          if(x3<0)
          {
            x3 = 125;
          }
 
          if(x4<0)
          {
            x4 = 125;
          }
 
          if(x5<0)
          {
            x5 = 125;
          }
 
          if(x6<0)
          {
            x6 = 125;
          }
 
          if(x7<0)
          {
            x7 = 125;
          }
 
          if(x8<0)
          {
            x8 = 125;
          }
 
          if(x9<0)
          {
            x9 = 125;
          }
 
          if(x10<0)
          {
            x10 = 125;
          }
          if(x11<0)
                  {
                    x11 = 125;
                  }
 
 
 
 
          GFX_draw_string(x, y, "O", WHITE, BLACK, rozx, rozy);
          GFX_draw_string(x2, y2, "d", WHITE, BLACK, rozx, rozy);
          GFX_draw_string(x3, y3, "l", WHITE, BLACK, rozx, rozy);
          GFX_draw_string(x4, y4, "e", WHITE, BLACK, rozx, rozy);
          GFX_draw_string(x5, y5, "g", WHITE, BLACK, rozx, rozy);
          GFX_draw_string(x6, y6, "l", WHITE, BLACK, rozx, rozy);
          GFX_draw_string(x7, y7, "o", WHITE, BLACK, rozx, rozy);
          GFX_draw_string(x8, y8, "s", WHITE, BLACK, rozx, rozy);
          GFX_draw_string(x9, y9, "c", WHITE, BLACK, rozx, rozy);
          GFX_draw_string(x10, y10, ":", WHITE, BLACK, rozx, rozy);
          GFX_draw_string(x11, y10, msg, WHITE, BLACK, rozx, rozy);
 
          HAL_Delay(10);
          SSD1306_display_clear();
 
          SSD1306_display_repaint();
 
 
 
 
 
          HAL_Delay(10);
    /* USER CODE BEGIN 3 */
  }
  /* USER CODE END 3 */
}
 
/**
  * @brief System Clock Configuration
  * @retval None
  */
void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};
  RCC_PeriphCLKInitTypeDef PeriphClkInit = {0};
 
  /** Configure LSE Drive Capability
  */
  HAL_PWR_EnableBkUpAccess();
  __HAL_RCC_LSEDRIVE_CONFIG(RCC_LSEDRIVE_LOW);
  /** Initializes the RCC Oscillators according to the specified parameters
  * in the RCC_OscInitTypeDef structure.
  */
  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_LSE|RCC_OSCILLATORTYPE_MSI;
  RCC_OscInitStruct.LSEState = RCC_LSE_ON;
  RCC_OscInitStruct.MSIState = RCC_MSI_ON;
  RCC_OscInitStruct.MSICalibrationValue = 0;
  RCC_OscInitStruct.MSIClockRange = RCC_MSIRANGE_6;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_ON;
  RCC_OscInitStruct.PLL.PLLSource = RCC_PLLSOURCE_MSI;
  RCC_OscInitStruct.PLL.PLLM = 1;
  RCC_OscInitStruct.PLL.PLLN = 36;
  RCC_OscInitStruct.PLL.PLLP = RCC_PLLP_DIV7;
  RCC_OscInitStruct.PLL.PLLQ = RCC_PLLQ_DIV2;
  RCC_OscInitStruct.PLL.PLLR = RCC_PLLR_DIV2;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }
  /** Initializes the CPU, AHB and APB buses clocks
  */
  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_HCLK|RCC_CLOCKTYPE_SYSCLK
                              |RCC_CLOCKTYPE_PCLK1|RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_PLLCLK;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;
 
  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_4) != HAL_OK)
  {
    Error_Handler();
  }
  PeriphClkInit.PeriphClockSelection = RCC_PERIPHCLK_USART2|RCC_PERIPHCLK_I2C1
                              |RCC_PERIPHCLK_ADC;
  PeriphClkInit.Usart2ClockSelection = RCC_USART2CLKSOURCE_PCLK1;
  PeriphClkInit.I2c1ClockSelection = RCC_I2C1CLKSOURCE_PCLK1;
  PeriphClkInit.AdcClockSelection = RCC_ADCCLKSOURCE_PLLSAI1;
  PeriphClkInit.PLLSAI1.PLLSAI1Source = RCC_PLLSOURCE_MSI;
  PeriphClkInit.PLLSAI1.PLLSAI1M = 1;
  PeriphClkInit.PLLSAI1.PLLSAI1N = 16;
  PeriphClkInit.PLLSAI1.PLLSAI1P = RCC_PLLP_DIV7;
  PeriphClkInit.PLLSAI1.PLLSAI1Q = RCC_PLLQ_DIV2;
  PeriphClkInit.PLLSAI1.PLLSAI1R = RCC_PLLR_DIV2;
  PeriphClkInit.PLLSAI1.PLLSAI1ClockOut = RCC_PLLSAI1_ADC1CLK;
  if (HAL_RCCEx_PeriphCLKConfig(&PeriphClkInit) != HAL_OK)
  {
    Error_Handler();
  }
  /** Configure the main internal regulator output voltage
  */
  if (HAL_PWREx_ControlVoltageScaling(PWR_REGULATOR_VOLTAGE_SCALE1) != HAL_OK)
  {
    Error_Handler();
  }
  /** Enable MSI Auto calibration
  */
  HAL_RCCEx_EnableMSIPLLMode();
}
 
/* USER CODE BEGIN 4 */
 
/* USER CODE END 4 */
 
/**
  * @brief  This function is executed in case of error occurrence.
  * @retval None
  */
void Error_Handler(void)
{
  /* USER CODE BEGIN Error_Handler_Debug */
  /* User can add his own implementation to report the HAL error return state */
  __disable_irq();
  while (1)
  {
  }
  /* USER CODE END Error_Handler_Debug */
}
 
#ifdef  USE_FULL_ASSERT
/**
  * @brief  Reports the name of the source file and the source line number
  *         where the assert_param error has occurred.
  * @param  file: pointer to the source file name
  * @param  line: assert_param error line source number
  * @retval None
  */
void assert_failed(uint8_t *file, uint32_t line)
{
  /* USER CODE BEGIN 6 */
  /* User can add his own implementation to report the file name and line number,
     ex: printf("Wrong parameters value: file %s on line %d\r\n", file, line) */
  /* USER CODE END 6 */
}
#endif /* USE_FULL_ASSERT */
 
/************************ (C) COPYRIGHT STMicroelectronics *****END OF FILE****/