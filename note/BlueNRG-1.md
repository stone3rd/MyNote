**服务注册**  
0x02,0x36,0x6e,0x80, 0xcf,0x3a, 0x11,0xe1, 0x9a,0xb4, 0x00,0x02,0xa5,0xd5,0xc5,0x1b



	tBleStatus aci_gatt_add_char(uint16_t Service_Handle,
                             uint8_t Char_UUID_Type,
                             Char_UUID_t *Char_UUID,
                             uint16_t Char_Value_Length,
                             uint8_t Char_Properties,
                             uint8_t Security_Permissions,
                             uint8_t GATT_Evt_Mask,
                             uint8_t Enc_Key_Size,
                             uint8_t Is_Variable,
                             uint16_t *Char_Handle);
**数据的收发:**  
APP->BLE：

	//接收APP发送的数据
	void aci_gatt_attribute_modified_event(uint16_t Connection_Handle,
                                       uint16_t Attr_Handle,
                                       uint16_t Offset,
                                       uint8_t Attr_Data_Length,
                                       uint8_t Attr_Data[])
	{
		Acc_Update(Attr_Data_Length,Attr_Data);	
	}

BLE->APP  
	
	//BLE发送数据
	tBleStatus aci_gatt_update_char_value(uint16_t Service_Handle,
                                      uint16_t Char_Handle,
                                      uint8_t Val_Offset,
                                      uint8_t Char_Value_Length,
                                      uint8_t Char_Value[]);

---
`aci_gap_terminate_gap_proc()` 终止连接,暂时测试还没有成功。`hci_reset();`断开蓝牙连接  





**sleepmode测试**  

技术人员回复的邮件内容：
  
	请贵公司工程师确认下，只要你设置了蓝牙可连接的情况下，蓝牙必须要周期性向外广播自己的信息直至被被连接上，但被连接上后也需要周期性和手机透过无线沟通，因此此时不可能进入NOTIMER的状态。  

	所以若要进入NOTIMER的状态后，只能透过外部GPIO中断唤醒它，谢谢！  

调用`aci_gap_set_non_discoverable()`函数设置BlueNRG为不可发现状态，但是第一次调用上面的函数时