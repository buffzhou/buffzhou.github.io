---
layout: post
title: PID调控单C610电调电机（CAN和PID实操，还有用STM32IDE的好处）
description: >
  简单的用CAN的收发电机，用STM32IDE调参运用其中的曲线和改参功能，码云地址https://gitee.com/buffzhou/PID_motor
author: author2
---
# PID调控单C610电调电机（CAN和PID实操，STM32IDE的图像和现场改参）

-----------
## 快速目录
{:.no_toc}
1. this unordered seed list will be replaced by toc as unordered list
{:toc}

-----------
## PID部分

~~~c

extern short _speed;                                    //全局变量便于监控速度
short roll = 0;                                         //全局变量便于监控差值
static float p = 1.2,i = 0.03;                          //全局可以暂停改P，I，D值
static float d = 0.03,error = 0,error_i = 0,error_d = 0;//静态可以保存上一次差值

short PID_MOTOR(short target)
{
	short speed;
	uint8_t id;
	MOTOR_GET_SPEED(&speed, &id);                        //获取电机速度和ID

	roll = error;                                        //为了实时监控速度
	error_d = (target - speed) - error;                  //算出差值微分
	error = target - speed;                              //算出差值比例
	error_i += error;                                    //算出差值积分
	MOTOR_SPEED(error * p + error_i * i + error_d * d);  //PID公式
	return speed;                                        //返回当前速度
}

~~~
   
---

## CAN收发部分

**发送部分**

~~~c

void CAN_Transmit(uint8_t data[])
{
	uint32_t pTxMailbox;                                                    //获取邮箱编号
	CAN_TxHeaderTypeDef pHeader;                                            //定义配置结构体，具体可以点进去或者参考HAL手册
	pHeader.StdId = 0x200;                                                  //报文ID，相当于收信人姓名
	pHeader.RTR = CAN_RTR_DATA;                                             //报文类型，这里是传DATA
	pHeader.IDE = CAN_ID_STD;                                               //报文ID是否加长，这里不加长
	pHeader.DLC = 8;                                                        //报文长度，8位
	if(HAL_CAN_AddTxMessage(&hcan1, &pHeader, data, &pTxMailbox) != HAL_OK) //新HAL库收发，DATA与其他部分分开（2019.9）
		Error_Handler();                                                    //发送失败进入Error_Handler()在main里面（偏下）
	return ;
}


~~~

**接受部分**

* 过滤器配置（这里全部接受）

~~~c

void CAN_FilterInit()
{
	  CAN_FilterTypeDef can_filter_st;						//定义CAN报文句柄
	  can_filter_st.FilterActivation = ENABLE;				//启用过滤器
	  can_filter_st.FilterMode = CAN_FILTERMODE_IDMASK;		//启用过滤器掩码形式
	  can_filter_st.FilterScale = CAN_FILTERSCALE_32BIT;	//指定过滤器范围


	  can_filter_st.FilterIdHigh = 0x0000;					//高位全部0
	  can_filter_st.FilterIdLow = 0x0000;					//低位全部0
	  can_filter_st.FilterMaskIdHigh = 0x0000;				//掩码高位都不检测
	  can_filter_st.FilterMaskIdLow = 0x0000;				//掩码低位都不检测


	  can_filter_st.FilterBank = 0;							//过滤器编码，单个CAN 0~13
	  can_filter_st.FilterFIFOAssignment = CAN_RX_FIFO0;	//被配置的邮箱
	  HAL_CAN_ConfigFilter(&hcan1, &can_filter_st);
	return ;
}

~~~

* 接受函数

~~~c

void CAN_Receive(uint8_t data[],uint8_t *id)                          //这里有个小技巧，data传数组要加[],预定义也要，c不能用&
{
	CAN_RxHeaderTypeDef RxHeader;                                     //接受句柄结构体
	HAL_CAN_GetRxMessage(&hcan1, CAN_RX_FIFO0, &RxHeader, data);      //CAN邮箱为0，这里其实邮箱会满以后可能要加中断
	*id = RxHeader.StdId;                                             //获得ID
	return ;
}

~~~

---

