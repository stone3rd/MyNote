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
