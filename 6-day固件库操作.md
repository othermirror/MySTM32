# STM32应用的三种框架

### 1、裸奔

- 指没有操作系统或者没有其他中间层的代码
- **特点：**只有两层，应用直接操作硬件寄存器，简单
- **缺点：**必须懂得硬件，同时还有看得懂简单的原理图，只能单任务进行

### 2、带库的框架（固件库）

- **特点：**做APP的可以无需掌握硬件的实现的细节，可以将全部精力放在应用的逻辑开发
- **缺点：**单任务进行

### 3、带OS

- **特点：**在包含中间层库的基础上，提供了OS，无需关心硬件的具体实现
- OS可以提供并发



***

# STM32固件库操作GPIO

> APP -> 固件库API -> Register

### 1、使能GPIO分组的时钟（GPIO都属于AHB1总线）

- **函数名：**RCC_XXXPeriphClockCmd，xxx代表的是对应的总线名，如下例子的总线名位AHB1
- **函数原型：**void ==RCC_AHB1PeriphClockCmd==(uint32_t RCC_AHB1Periph, FunctionalState NewState);
- ***@RCC_AHB1Periph:***  指定总线上需要使能的外设名 
  - 一些可用的宏定义
  - RCC_AHB1Periph_GPIOA
  - RCC_AHB1Periph_GPIOB
  - ……
  - RCC_AHB1Periph_GPIOI
  - ……
- ***@NewState:*** 指定该外设的时钟状态
  - 一些可用的宏定义
  - ENABLE ：使能
  - DISABLE：禁止
- 具体的使用的例子（使能GPIOF组的时钟）
  - RCC_AHB1PeriphClockCmd(RCC_AHB1Periph_GPIOF,ENABLE);

### 2、初始化配置GPIO引脚的功能模式（MODER）

- **函数原型：**void ==GPIO_Init==(GPIO_TypeDef* GPIOx, GPIO_InitTypeDef* GPIO_InitStruct);

- ***@GPIOx：***指定要初始化的GPIO分组 

  - 一些可用的宏定义
  - GPIOA
  - GPIOB
  - ……
  - GPIOI

- ***@GPIO_InitStruct：***结构体指针，指向保存GPIO初始化信息的结构体

  - ~~~C
    typedef struct
    			{
    			  uint32_t GPIO_Pin; 
    				//指定要配置的GPIO引脚(可以位或多个引脚，表示配置同一组
    					的多个引脚为相同功能模式)。
    					GPIO_Pin_0
    					GPIO_Pin_1
    					.....
    					GPIO_Pin_15
    		
    			  GPIOMode_TypeDef GPIO_Mode;
    				//指定要配置的GPIO引脚的功能模式(GPIOx_MODER)
    					  GPIO_Mode_IN   /*!< GPIO Input Mode */
    					  GPIO_Mode_OUT  /*!< GPIO Output Mode */
    					  GPIO_Mode_AF   /*!< GPIO Alternate function Mode */
    					  GPIO_Mode_AN   /*!< GPIO Analog Mode */
    					  
    			  GPIOSpeed_TypeDef GPIO_Speed;  
    				//指定引脚的输出速率(GPIOx_OSEEPDR) 
    					GPIO_Speed_2MHz    低速模式
    					GPIO_Speed_25MHz   中速模式
    					GPIO_Speed_50MHz   快速模式
    					GPIO_Speed_100MHz  高速模式 
    			  
    			  GPIOOType_TypeDef GPIO_OType; 
    				//指定要配置的引脚的输出类型(GPIOx_OTYPER)
    					GPIO_OType_PP 推挽输出 
    					GPIO_OType_OD 开漏输出 
    						
    			  GPIOPuPd_TypeDef GPIO_PuPd;   
    				//指定要配置引脚的上下拉电阻的情况 
    					  GPIO_PuPd_NOPULL 输入悬空
    					  GPIO_PuPd_UP     带上拉电阻 
    					  GPIO_PuPd_DOWN   带下拉电阻 
    			}GPIO_InitTypeDef;
    ~~~

  - 例子：配置PE2、PE3、PE输入模式并带上拉电阻

    - ~~~C
      GPIO_InitTypeDef GPIO_InitStruct; // 定义结构体
      // 设置哪些引脚				
      GPIO_InitStruct.GPIO_Pin = GPIO_Pin_2|GPIO_Pin_3|GPIO_Pin_4; 
      // 设置的模式（输出、输入、复用等）
      GPIO_InitStruct.GPIO_Mode = GPIO_Mode_IN;
      // 带上拉电阻或者下拉电阻或者悬空
      GPIO_InitStruct.GPIO_PuPd = GPIO_PuPd_UP;
      // 初始化函数
      GPIO_Init(GPIOE, &GPIO_InitStruct);
      ~~~

### 3、操作GPIO对应的引脚

- 可读（GPIOx_IDR、GPIOx_ODR）
- 可写（GPIOx_ BSRR、GPIOx_ODR）

##### 1.有关输入的函数（读）

- ==以下两个函数是获取GPIOx_IDR寄存器中的数据==

- **函数原型：**uint8_t ==GPIO_ReadInputDataBit==(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);
- **作用：**获取指定的GPIO分组的某个引脚的电平状态（一个引脚）
- **返回值：**为读取到的电平状态
- ***@GPIOx：***需要读取的GPIO组，如GPIOA、GPIOB……
- ***GPIO_Pin：***需要读取的具体的引脚，如GPIO_Pin_0、GPIO_Pin_1……
- 例子：获取PER3引脚的电平状态值
  - uint8_t ret = GPIO_ReadInputDataBit(GPIOE,GPIO_Pin_3);
- **函数原型：**uint16_t ==GPIO_ReadInputData==(GPIO_TypeDef* GPIOx);
- **作用：**获取指定的GPIO分组的整组电平状态（16个引脚）
- **返回值：**为读取到的电平状态
- ***@GPIOx：***需要读取的GPIO组，如GPIOA、GPIOB……
- 例子： 获取PE3引脚的电平状态值 
  - uint16_t ret = GPIO_ReadInputData(GPIOE);
- ==以下的两个函数是读取GPIOx_ODR寄存器中的数据==
- uint8_t ==GPIO_ReadOutputDataBit==(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);
- uint16_t ==GPIO_ReadOutputData==(GPIO_TypeDef* GPIOx);
- **用法和组以上面的两个读取GPIOx_IDR的函数一样**

##### 2.有关输出的函数（写/设置）

> 专门设置高低电平的函数

- **将特定的分组的特定的引脚设置为高电平**
- **函数原型：**void ==GPIO_SetBits==(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);

- **将特定的分组的特定的引脚设置为低电平**
- **函数原型：**void ==GPIO_ResetBits==(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);

**例子：**将PF9设置为高电平，将PF10设置为低电平

- GPIO_SetBits(GPIOF,  GPIO_Pin_9);
- GPIO_ResetBits(GPIOF,GPIO_Pin_10);  

> 也提供了设置高低电平都由一个函数来决定的版本

- void ==GPIO_WriteBit==(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, BitAction BitVal);

- void ==GPIO_Write==(GPIO_TypeDef* GPIOx, uint16_t PortVal); 

- 应用的例子：将PF9设置为高电平，将PF10设置为低电平

  - GPIO_WriteBit(GPIOF,  GPIO_Pin_9,  Bit_SET);
  - GPIO_WriteBit(GPIOF,  GPIO_Pin_10,  Bit_RESET);

- 应用的例子：将PF9设置为高电平

  - ~~~C
    uint16_t ret = GPIO_ReadOutputData(GPIOF);
    ret |= (1 << 9);
    GPIO_Write(GPIOF, ret); 
    ~~~

> 将引脚的电平反转 0->1 1->0

- void ==GPIO_ToggleBits==(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);

### 4、功能复用选择函数

> 只有GPIO设置为复用功能模式下才需要调用，其余模式不需要调用

- **函数原型：**void ==GPIO_PinAFConfig==(GPIO_TypeDef* GPIOx, uint16_t GPIO_PinSource, uint8_t GPIO_AF);
- **作用：**当GPIO被设置为复用模式时，设置需要复用的模式
- ***@GPIOx ：***指定GPIO引脚的分组 
  - GPIOA
  - ……
  - GPIOI 
- ***@GPIO_PinSource:*** 指定作为复用功能的引脚的编号 
  - GPIO_PinSource0 
  - GPIO_PinSource1
  - ……
  - GPIO_PinSource15
- ***@GPIO_AF：***指定引脚的复用功能 
  - GPIO_AF_TIM1
  - GPIO_AF_TIM2 
  - GPIO_AF_USART1 
  - GPIO_AF_USART2 
  - ……



# TIPS

以上只是例句了一小部分的函数的用法，其余的函数用法可以查阅资料或者翻阅头文件（有相应的能力的话）

