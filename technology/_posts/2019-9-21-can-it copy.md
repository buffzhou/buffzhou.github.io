---
layout: post
title: CAN的中断配置和过滤器配置
description: >
  CAN的中断接受和过滤器配置，码云地址
author: author1
---
# CAN的中断配置和过滤器配置 （更加贴和实际使用）

-----------
## 快速目录
{:.no_toc}
1. this unordered seed list will be replaced by toc as unordered list
{:toc}

-----------
## CAN

**API使用：**
HAL_CAN_Start(CAN_HandleTypeDef *hcan);
	
**接收过程：**
1. CAN_Filter_Mask_Conf(CAN_HandleTypeDef _hcan,...)
2. 记得配置can的NVIC
3. 配置中断服务函数内的等待读取回调函数
	void HAL_CAN_RxFifo0MsgPendingCallback(CAN_HandleTypeDef *hcan);
	void HAL_CAN_RxFifo1MsgPendingCallback(CAN_HandleTypeDef *hcan);	
	在回调函数当中写 
	HAL_CAN_GetRxMessage(...)



~~~c


MSG_R temp_RxMes;

void (*can1_recv_callback_reg) (MSG_R* rece_msg); 
void (*can2_recv_callback_reg) (MSG_R* rece_msg); 

void CAN_Filter_Mask_Conf(CAN_HandleTypeDef *_hcan, uint8_t object_para,uint32_t Id,uint32_t MaskId)
{
  CAN_FilterTypeDef  CAN_FilterInitStructure;
  if( (object_par a&0x02))//判断是否为数据帧
  {
    CAN_FilterInitStructure.FilterIdHigh         = Id<<3<<16;             							/* 掩码后ID的高16bit */
    CAN_FilterInitStructure.FilterIdLow          = Id<<3| ((object_para&0x03)<<1);    				/* 掩码后ID的低16bit */
    CAN_FilterInitStructure.FilterMaskIdHigh     = MaskId<<3<<16;             						/* ID掩码值高16bit */
    CAN_FilterInitStructure.FilterMaskIdLow      = MaskId<<3| ((object_para&0x03)<<1);	 			/* ID掩码值低16bit */
  }
  else
  {
    CAN_FilterInitStructure.FilterIdHigh         = Id<<5;             								/* 掩码后ID的高16bit */
    CAN_FilterInitStructure.FilterIdLow          = ((object_para&0x03)<<1);   						/* 掩码后ID的低16bit */
    CAN_FilterInitStructure.FilterMaskIdHigh     = MaskId<<5;             							/* ID掩码值高16bit */
    CAN_FilterInitStructure.FilterMaskIdLow      = ((object_para&0x03)<<1);; 					  	/* ID掩码值低16bit */
  }
	CAN_FilterInitStructure.FilterBank     = object_para>>3;              							/* 滤波器序号，0-13，共14个滤波器 */
	CAN_FilterInitStructure.FilterFIFOAssignment = (object_para>>2)&0x01;       					/* 滤波器绑定FIFO 0 */
	CAN_FilterInitStructure.FilterActivation     = ENABLE;                								/* 使能滤波器 */
	CAN_FilterInitStructure.FilterMode       = CAN_FILTERMODE_IDMASK;       							/* 滤波器模式，设置ID掩码模式 */
	CAN_FilterInitStructure.FilterScale      = CAN_FILTERSCALE_32BIT;      							  /* 32位滤波 */
	
	HAL_CAN_ConfigFilter(_hcan, &CAN_FilterInitStructure);
}

uint8_t CANx_SendNormalData(CAN_HandleTypeDef* hcan,uint16_t ID,uint8_t *pData,uint16_t Len)
{
    CAN_TxHeaderTypeDef     TxMeg;
    HAL_StatusTypeDef   HAL_RetVal;
    TxMeg.StdId=ID;
    TxMeg.ExtId=0;
    TxMeg.IDE=0;
    TxMeg.RTR=0;		  // 消息类型为数据帧，一帧8位
    TxMeg.DLC=Len;
	
    if(HAL_CAN_AddTxMessage(hcan,&TxMeg,pData,(uint32_t*)CAN_TX_MAILBOX0) != HAL_OK)
		{
			if(HAL_CAN_AddTxMessage(hcan,&TxMeg,pData,(uint32_t*)CAN_TX_MAILBOX1) != HAL_OK)
			{
				if(HAL_CAN_AddTxMessage(hcan,&TxMeg,pData,(uint32_t*)CAN_TX_MAILBOX2) != HAL_OK)
				{
					return 1;
				}
			}
		}
    return 0;
}

/**
* @brief  CAN数据包邮箱接收
  * @param  ctrl  1：CAN1  2：CAN2
  * @param  _fifo 0：fifo0 1：fifo1
  * @retval 0x01:未接收到CAN数据
  */
void CANx_Recv_Callback_Register(uint8_t ctrl,void (*fun) (MSG_R* rece_msg))
{
	if(ctrl == 1)
	{
		can1_recv_callback_reg = fun;
	}
	else
	{
		can2_recv_callback_reg = fun;
	}
}

void HAL_CAN_RxFifo0MsgPendingCallback(CAN_HandleTypeDef *hcan)
{
    MSG_R*	RxMes;
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;

    /* 初始化结构体指针 */
    RxMes = &temp_RxMes;

    /* 从CAN接口接收数据 */
    HAL_CAN_GetRxMessage(hcan,CAN_FILTER_FIFO0,&RxMes->RxMeg,RxMes->Data);

//		Motor_Pack_Handle(RxMes);

    /* 向消息队列发数据 */
    if(hcan == &hcan1)
        (*can1_recv_callback_reg)(RxMes);
    else if(hcan == &hcan2)
        (*can2_recv_callback_reg)(RxMes);

    /* 如果 xHigherPriorityTaskWoken = pdTRUE，那么退出中断后切到当前最高优先级任务执行 */
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
}

void HAL_CAN_RxFifo1MsgPendingCallback(CAN_HandleTypeDef *hcan)
{
    MSG_R* RxMes;
    BaseType_t xHigherPriorityTaskWoken = pdFALSE;

    /* 初始化结构体指针 */
    RxMes = &temp_RxMes;

    /* 从CAN接口接收数据 */
    HAL_CAN_GetRxMessage(hcan,CAN_FILTER_FIFO1,&RxMes->RxMeg,RxMes->Data);

//		Motor_Pack_Handle(RxMes);

    /* 向消息队列发数据 */
    if(hcan == &hcan1)
        (*can1_recv_callback_reg)(RxMes);
    else if(hcan == &hcan2)
        (*can2_recv_callback_reg)(RxMes);

    /* 如果 xHigherPriorityTaskWoken = pdTRUE，那么退出中断后切到当前最高优先级任务执行 */
    portYIELD_FROM_ISR(xHigherPriorityTaskWoken);
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

