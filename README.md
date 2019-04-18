# һ. usb����
    stm32��usbҲ�������˾��ʹ�õĵĽӿڣ�usbȫ�ٿ��Դﵽ12M/s, ��Ϊ���⴮�ڽӿڣ����ǲ���ġ�

# ��. cubemx����usb���⴮�ڵ�demo����

## 2.1 ���裺
cuebe���ɴ��ں�heapҪ���ó�0x600��������windows���ָ�̾�����⡣
ͼƬ�뿴: ./piture/1-9.png
![avatar](../picture/1.png)
![avatar](../picture/2.png)
![avatar](../picture/3.png)
![avatar](../picture/4.png)
![avatar](../picture/5.png)
![avatar](../picture/6.png)
![avatar](../picture/7.png)
![avatar](../picture/8.png)
![avatar](../picture/9.png)

## 2.2 ���Թ����� 
PC �� WIN10 
���������֮�۴�������
Keil: MDK5
���壺stm32f4 discory

## 2.3 ���Թ��̣�
1. KEIL��Ҫ����������ɺ��Զ���λ
2. �ڵ����ϲ鿴COM�����ĸ�
3. ������ÿ�뷢��1���ַ�'a', 
4. ��������ÿ�뷢��1���ַ�'a'

## �������̱����Ѿ��ύ����github��
��ַ:

# ��. ��δ����ͺͽ���

���պͷ��͵Ľӿڶ���usbd_cdc_if.c��

## 3.1 ���ݻ����С����������
```
#define APP_RX_DATA_SIZE  4
#define APP_TX_DATA_SIZE  4
```

## 3.2 ��������

�����������������api�ӿ�����
uint8_t CDC_Transmit_FS(uint8_t* Buf, uint16_t Len)
������1s���ڷ���"a"����
```
HAL_Delay(1000);
char *string = "a";
CDC_Transmit_FS((uint8_t*)string, 1);
```

## 3.3 ��������

���ջ��������õ� uint8_t UserRxBufferFS[APP_RX_DATA_SIZE];
�������ݵĳ���   int recv_len = 0;
�������ݵĽӿ��� static int8_t CDC_Receive_FS (uint8_t* Buf, uint32_t *Len), ��������
```
static int8_t CDC_Receive_FS (uint8_t* Buf, uint32_t *Len)
{ 
  recv_len = *Len;
  USBD_CDC_SetRxBuffer(&hUsbDeviceFS, &Buf[0]);
  USBD_CDC_ReceivePacket(&hUsbDeviceFS);
  return (USBD_OK);
}
```

# ��. ���ͺͽ���ԭ��

| ��� | ���� |  ��Ҫ�ļ�|
| :------| ------: | :------: |
| APP |  Ӧ�ò�| usbd_cdc_if.c |
| calss | ��ͬ���豸��ͬ������ | usb_cdc.c |
| core | USB���ݻ���Ĵ��� | usb_core.c |
| HAL  | Ӳ���жϴ��� | stm32f4_ll_usb.c |

## 4.1 ��������

USB_EPStartXfer: ����һ���˵㴫��
USBx_INEP: ʹ��EP
USB_WritePacket:����д��fifo
```
CDC_Transmit_FS((uint8_t*)string, 15); 
    -> USBD_CDC_TransmitPacket(&hUsbDeviceFS);
        ->   USBD_LL_Transmit(pdev,  CDC_IN_EP, hcdc->TxBuffer,hcdc->TxLength);
            ->   hal_status = HAL_PCD_EP_Transmit(pdev->pData, ep_addr, pbuf, size);
                -> HAL_StatusTypeDef USB_EPStartXfer(USB_OTG_GlobalTypeDef *USBx , USB_OTG_EPTypeDef *ep, uint8_t dma)
                    ->USBx_INEP(ep->num)->DIEPCTL |= (USB_OTG_DIEPCTL_CNAK | USB_OTG_DIEPCTL_EPENA);
                    ->USB_WritePacket(USBx, ep->xfer_buff, ep->num, ep->xfer_len, dma);
```            

## 4.2 ��������
```
HAL_PCD_IRQHandler(&hpcd_USB_OTG_FS);
    -> HAL_PCD_DataOutStageCallback(hpcd, epnum);
        -> USBD_LL_DataOutStage((USBD_HandleTypeDef*)hpcd->pData, epnum, hpcd->OUT_ep[epnum].xfer_buff);
            -> pdev->pClass->DataOut(pdev, epnum); 
                ->  ((USBD_CDC_ItfTypeDef *)pdev->pUserData)->Receive(hcdc->RxBuffer, &hcdc->RxLength);
``` 
### 4.2.1 �ӿ� DataOut
�ӿڽṹ��
``` 
  typedef struct _Dev   ice_cb{
  uint8_t  (*DataOut)(struct _USBD_HandleTypeDef *pdev , uint8_t epnum); 
``` 
ע��DataOut����ӿ�
``` 
USBD_ClassTypeDef  USBD_CDC = {
  USBD_CDC_DataOut,
``` 
### 4.2.2 �ӿ� Receive

�ӿڽṹ��
``` 
typedef struct _USBD_CDC_Itf
{
  int8_t (* Init)          (void);
  int8_t (* DeInit)        (void);
  int8_t (* Control)       (uint8_t, uint8_t * , uint16_t);   
  int8_t (* Receive)       (uint8_t *, uint32_t *);  
}USBD_CDC_ItfTypeDef;
``` 

ע��(* Receive)����ӿں���
``` 
USBD_CDC_ItfTypeDef USBD_Interface_fops_FS = 
{
  CDC_Init_FS,
  CDC_DeInit_FS,
  CDC_Control_FS,  
  CDC_Receive_FS
};
``` 