# nRF52832-BLE-UART

## 一、功能概述
	本 BLE_UART 模块(以下简称为 BLE_UART )主要功能如下：
* BLE 和串口数据转发。
	* 模块将从串口接收到的数据通过 BLE 转发出去；
	* 模块将 BLE 接收到的数据通过串口转发出去。
* 串口 AT 指令配置模块功能。

## 二、BLE通信
	BLE_UART 支持 BLE 4.0/4.1/4.2 通信。
	Android 4.3及以后 和 iPhone 4s + iOS 7.0及以后 的智能手机才支持 BLE 4.0，才可以与 BLE_UART 进行通信。  

### 2.1 广播Advertising
	广播数据包包括 Advertising Data 和 Scan Response Data。

#### 2.1.1 Advertising Data

![adv](https://github.com/langxing08/nRF52832-BLE-UART/blob/master/picture/1-ADV_IND.png)

	BLE_UART 中 Advertising Data 的 PDU 格式为 ADV_IND。<br>flag 值为 0x06，表示只支持 BLE，不支持 BR/EDR，通用发现模式。<br>Device Name 为设备名称。

#### 2.1.2 Scan Response Data

![scan](https://github.com/langxing08/nRF52832-BLE-UART/blob/master/picture/2-SCAN_RESP.png)

	Scan Response Data 中包括服务 UUID，为 128 位自定义 UUID。

### 2.2 服务Service

![profile](https://github.com/langxing08/nRF52832-BLE-UART/blob/master/picture/3-Profile.png)

	BLE_UART 的 profile 中包含 4 个首要服务(Primary Service)，包括Generic Access Service、Generic Attribute Service、Nordic UART Service 和 Device Information Service。
	
#### 2.2.1 Generic Access Service
	该服务中包括Device Name、Appearance、Peripheral Preferred Connection Parameters和Central Address Resolution 4个特征值。
	服务的UUID和各个特征值的权限及UUID详见下图。

![access](https://github.com/langxing08/nRF52832-BLE-UART/blob/master/picture/4-Generic%20Access%20Service.png)
	
	Device Name 特征值为设备名称。
	Appearance 特征值为外观特性，本模块没有设置该值，默认值为0。
	Peripheral Preferred Connection Parameters 特征值为周边设备推荐的连接参数，本模块默认值为连接间隔10ms - 30ms，从机延迟0，监督超时72*10=720ms。该连接参数也是iOS默认的连接参数。
	Central Address Resolution 特征值为是否支持中央设备私有设备地址解析，BLE_UART 默认值为支持。

#### 2.2.2 Generic Attribute Service

![attribute](https://github.com/langxing08/nRF52832-BLE-UART/blob/master/picture/5-Generic%20Attribute%20Service.png)

	该服务为空。

#### 2.2.3 Nordic UART Service
	该服务不是SIG定义的标准服务，是用户自定义服务。服务UUID和各特征值的权限及UUID见下图。该服务中包含RX和TX 2个特征值。

![uart](https://github.com/langxing08/nRF52832-BLE-UART/blob/master/picture/6-Nordic%20Uart%20Service.png)
	
* RX特征值用于将中央设备通过BLE write到本模块的数据通过串口转发出去。BLE每包write最多支持 ATT_MTU(默认值为23，可通过ATT_MTU_UPDATED事件协商更改) - 3 个字节，如果中央设备需要发送的数据超过20字节，需要中央设备分包后再write，否则会导致write失败，甚至可能导致模块重启。
* TX特征值用于将模块串口接收到的数据通过BLE以notify方式转发出去，每包最多 ATT_MTU - 3 个字节，如果串口接收到的数据超过 ATT_MTU - 3 个字节，则模块自动按照 ATT_MTU - 3 个字节分包发送。另外，如果串口接收到的数据为AT指令，则模块会进行相应的操作，并返回执行结果。注意：必须使能Notification才能将数据从本模块notify给中央设备。

## 三、串口通信
	模块上电默认串口配置：115200bps，8位数据位，1位停止位，无校验位，无流控。
	串口每帧最多接收256字节，超过ATT_MTU(默认值为23，可通过ATT_MTU_UPDATED事件协商更改)-3个字节自动分包，支持不定帧长度，支持任意字符结尾。接收完一个字符后3ms不再接收到新的字符，则认为本帧结束。

## 四、串口AT指令
	BLE_UART 支持的 AT 指令如下表所示。

|命令|说明|
|---|---|
|AT|测试串口通信
|AT+RESET|复位模块
|AT+MAC|查询模块MAC地址
|AT+VER|查询模块版本信息
|AT+STATUS|查询模块BLE连接状态
|AT+DISCON|断开BLE连接
|AT+BAUD|配置串口波特率
|AT+TXPW|配置射频发射功率
|AT+ADV|配置广播间隔
|AT+CON|配置连接参数

	所有AT指令(包括命令和响应)必须"AT"作为开头，以回车新行(<CR><LF>)结尾，为描述方便，文档中<CR><LF>被有意忽略了。


### 4.1 AT 测试串口通信

|查询命令|响应|命令参数说明|响应参数说明|
|---|---|---|---|
|AT?|AT:\<status>|无|\<status> 执行结果：<br>OK - 串口通信正常

### 4.2 AT+RESET 复位模块

|设置命令|响应|命令参数说明|响应参数说明|
|---|---|---|---|
|AT+RESET|AT+RESET:\<status>|无|\<status> 执行结果：<br>OK - 复位模块成功

### 4.3 AT+MAC 查询模块MAC地址

|查询命令|响应|命令参数说明|响应参数说明|
|---|---|---|---|
|AT+MAC?|AT+MAC:\<MAC address>|无|\<MAC address> 模块MAC地址(16进制字符串，12 Bytes)
	
### 4.4 AT+VER 查询模块版本信息

|查询命令|响应|命令参数说明|响应参数说明|
|---|---|---|---|
|AT+VER?|AT+VER:\<HW version>,\<FW version>,\<SW version>|无|\<HW version> 硬件版本<br>\<FW version> 协议栈固件版本<br>\<SW version> 应用软件版本

### 4.5 AT+STATUS 查询模块BLE连接状态

|查询命令|响应|命令参数说明|响应参数说明|
|---|---|---|---|
|AT+STATUS?|AT+STATUS:\<status>|无|\<status> BLE连接状态：<br>1 - 已连接，<br>0 - 未连接或断开连接

### 4.6 AT+DISCON 断开BLE连接

|设置命令|响应|命令参数说明|响应参数说明|
|---|---|---|---|
|AT+DISCON|AT+DISCON:\<status>|无|\<status> 断开BLE连接执行结果：<br>OK - 执行成功，<br>FAIL - 执行失败，<br>ERP - 参数错误

### 4.7 AT+BAUD 配置串口波特率

|设置命令|响应|命令参数说明|响应参数说明|
|---|---|---|---|
|AT+BAUD=\<level>|AT+DISCON:\<status>|\<level> 波特率：<br>0 - 4800bps，<br>1 - 9600bps，<br>2 - 19200bps，<br>3 - 38400bps，<br>4 - 57600bps，<br>5 - 115200bps，<br>6 - 230400bps，<br>7 - 460800bps，<br>8 - 921600bps<br>|\<status> 设置串口波特率执行结果：<br>OK - 执行成功，<br>FAIL - 执行失败，<br>ERP - 参数错误

### 4.8 AT+TXPW 配置射频发射功率TX Power

|设置命令|响应|命令参数说明|响应参数说明|
|---|---|---|---|
|AT+TXPW=\<level>|AT+TXPW:\<status>|\<level> 发射功率：<br>0 - -40dBm，<br>1 - -20dBm，<br>2 - -16dBm，<br>3 - -12dBm，<br>4 - -8dBm，<br>5 - -4dBm，<br>6 - 0dBm，<br>7 - +3dBm，<br>8 - +4dBm<br>|\<status> 设置发射功率执行结果：<br>OK - 执行成功，<br>FAIL - 执行失败，<br>ERP - 参数错误

### 4.9 AT+ADV 配置广播间隔

|设置命令|响应|命令参数说明|响应参数说明|
|---|---|---|---|
|AT+ADV=\<adv_interval>|AT+ADV:\<status>|\<adv_interval> 广播间隔：单位0.625ms，<br>最小值0x0020，最大值0x4000<br>|\<status> 设置广播间隔执行结果：<br>OK - 执行成功，<br>FAIL - 执行失败，<br>ERP - 参数错误

### 4.10 AT+CON 配置连接参数

|设置命令|响应|命令参数说明|响应参数说明|
|---|---|---|---|
|AT+CON=\<min_connect_interval>,<br>\<max_connect_interval>,\<slave_latency>,<br>\<conn_sup_timeout>|AT+CON:\<status>|\<min_connect_interval> 最小连接间隔：单位1.25ms，6 ≤ min_connect_interval ≤ max_connect_interval<br>\<max_connect_interval> 最大连接间隔：单位1.25ms，min_connect_interval ≤ max_connect_interval ≤ 0x0C80<br>\<slave_latency> 从机延迟：最大值499<br>\<conn_sup_timeout>监督超时：单位10ms，1 ≤ conn_sup_timeout ≤ 3200<br>注意：(1 + slave_latency) * (max_connect_interval) < conn_sup_timeout|\<status> 设置连接参数执行结果：<br>OK - 执行成功，<br>FAIL - 执行失败，<br>ERP - 参数错误

	