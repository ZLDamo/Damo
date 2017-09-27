---
layout: post
title:  "蓝牙小工具"
date:   2017-09-26 10:35:21 +0800
categories: jekyll update
---

先说一下我的目录结构(swift4.0)
##  1.目录结构



| 文件 | 内容 |
|---------|:--------------:|
| Const  | 定义ServiceUUID 和CharacteristicUUID  |      
|ZLBleDocument|使用方法 |
| ZLBluetoothManagerDelegate|自定义代理 |
|ZLBluetoothManager| 核心文件,扫描,连接|
| ZLBluetoothConfig| 搜索产品配置文件|
| ZLPeripheral| 广播数据转化|
| ZLBluetoothMessage| 发送的蓝牙数据|
| ZLBluetoothCallBackMessage| 接受的蓝牙数据解析|
|ZLString + Extension|16进制String和Data的转化|
| ZLData + Extension |分类为取指定范围的数据 |



下面直接代码走起

## 2. Const.Swift

这个文件定义的是一些SeriveUUID和CharacteristicUUID,根据情况自己定义
```
let CUSTOM_SERVICE_UUID     = CBUUID.init(string: "0000FF00-0000-1234-8000-00805F9B34FB")
let WRITE_CHARACTERISTIC_UUID       = CBUUID.init(string: "0000FF01-0000-1234-8000-00805F9B34FB")
let NOTIFY_CHARACTERISTIC_UUID      = CBUUID.init(string: "0000FF02-0000-1234-8000-00805F9B34FB")

//解释一下这个是做OTA的一个专用通道
let CUSTOM_BOOT_LOADER_SERVICE_UUID = CBUUID.init(string: "00060000-F8CE-11E4-ABF4-1234A5D5C51B")
let BOOT_LOADER_CHARACTERISTIC_UUID = CBUUID.init(string: "00060001-F8CE-11E4-ABF4-1234A5D5C51B")
```

## 3. ZLBleDocument.Swift(使用方法)

```
1.初始化
//设置扫描的产品码和代理,读写和升级的characteristic已经设置默认值,也可在参数中设置
manager = ZLBluetoothManager(config: ZLBluetoothConfig(scanProductCode: "7010"))
manager?.delegate = self

2.扫描
manager?.scan(progresshandle: { [weak self] (per) in
guard let _ = per?.peripheral else {
print("设备为空")
return
}
//这里可以直接将per(ZLPeripheral)中的数据拿出来展示UI
self?.connectPer(per: per!.peripheral!)

}) {[weak self] (perArr, error) in
//扫描结束回调
可能结果 :
nil,centralStateError //蓝牙状态错误
nil,busy              //正在扫描
perArr,timeout        //扫描的指定设备列表,超时
perArr,stop           //连接成功,结束扫描,回调结果

}

3.连接
manager?.connect(peripheral: per, handle: {[weak self] (per,progress, error) in
连接可能结果:
nil,.noConnect,.busy              // 连接超时,没有搜索到产品
peripheral,.connect,nil           // 连接上设备
peripheral,.discoverServices,nil  // 发现服务
peripheral,.discoverCharacteristics,nil   //发现读或者写特征
peripheral,.disconnect,nil        // 断开连接(复杂逻辑可在后面代理设置)
})

```

## 4. ZLBluetoothConfig.Swift

```
class ZLBluetoothConfig {

var scanProductCode: String?  //需要扫描产品名
var writeCharacteristic: CBUUID
var notifyCharacteristic: CBUUID
var bootCharacteristic: CBUUID

init(scanProductCode: String,
writeCharacteristic: CBUUID = WRITE_CHARACTERISTIC_UUID,
notifyCharacteristic: CBUUID = NOTIFY_CHARACTERISTIC_UUID,
bootCharacteristic: CBUUID = BOOT_LOADER_CHARACTERISTIC_UUID) {
self.scanProductCode = scanProductCode
self.writeCharacteristic = writeCharacteristic
self.notifyCharacteristic = notifyCharacteristic
self.bootCharacteristic = bootCharacteristic
}
}
```

## 5. ZLBluetoothManagerDelegate.Swift

```
@objc protocol ZLBluetoothManagerDelegate: NSObjectProtocol  {

@objc func peripheral(_ peripheral: CBPeripheral, didUpdateValueForNotifyCharacteristic: Data?)

@objc optional
func centralManager(_ central: CBCentralManager, didDisconnectPeripheral peripheral: CBPeripheral, error: Error?)

@objc optional
func peripheral(_ peripheral: CBPeripheral, didUpdateValueForBootCharacteristic: Data?)
}
```

自定义的代理方法,其中`peripheral(_ peripheral: didUpdateValueForNotifyCharacteristic:)` 和 ` peripheral(_ peripheral, didUpdateValueForBootCharacteristic)`是用于接受普通和OTA的蓝牙消息回调

`centralManager(_ central: didDisconnectPeripheral peripheral: error:)`用于断开连接的处理,还可以通过连接进度来处理(见ZLBluetoothManager)

## 6.定义扫描和连接的状态

```
enum FKBleScanError: Error {
case timeout    //超时
case busy       //正在扫描
case stop       //主动停止扫描
case centralStateError    //蓝牙状态错误
}

enum FKBleConnectError: Error {
case timeout
case busy
}

enum FkBleConnectProgress {
case noConnect                   //超时,还未连接
case connect                     //已连接
case discoverServices            //已发现服务
case discoverCharacteristics     //一发现普通通道特征
case disconnect
}
```

## 7. ZLPeripheral.Swift
对广播的数据进行处理,数据为可选类型取到的广播数据内容为"optional<xxxx>",计算字符串的时候注意要去除optional和<>

```
class ZLPeripheral: NSObject {
var peripheral: CBPeripheral?
var advertisementData: [String : Any]?
var RSSI: NSNumber?
var productName: String?
var dataStr: String?

class func zl_Peripheral(peripheral: CBPeripheral?,advertisementData: [String : Any]?,RSSI: NSNumber?) -> ZLPeripheral {
let zl_peripheral = ZLPeripheral()
zl_peripheral.peripheral = peripheral
zl_peripheral.advertisementData = advertisementData
zl_peripheral.RSSI = RSSI
zl_peripheral.productName = peripheral?.name
zl_peripheral.dataStr = String.init(format: "%@", advertisementData?["kCBAdvDataManufacturerData"] as? CVarArg ?? "")
.replacingOccurrences(of: " ", with: "")
.replacingOccurrences(of: "<", with: "")
.replacingOccurrences(of: ">", with: "")

return zl_peripheral
}
}
```
## 8.ZLBluetoothManager.Swift

```
final class ZLBluetoothManager: NSObject {
typealias ScanProgressHandle = (ZLPeripheral?) -> Void
typealias ScanCompleteHandle = ([ZLPeripheral]?, FKBleScanError?) -> Void
typealias ConnectHandle = (CBPeripheral?,FkBleConnectProgress,FKBleConnectError?) -> Void

weak var delegate:ZLBluetoothManagerDelegate?

var writeCodeCharacteristic: CBCharacteristic?
var notifyCharacteristic: CBCharacteristic?
var bootLoaderCharacteristic: CBCharacteristic?
//当前连接的设备
var peripheral: CBPeripheral?
//断开连接的设备
var oldPeripheral: CBPeripheral?
var centralMgr:CBCentralManager!

//扫描指定产品的设备(有重复)
fileprivate var scanProgressHandle: ScanProgressHandle?
//扫描结束回调扫描到的指定产品列表和错误
fileprivate var scanCompleteHandle: ScanCompleteHandle?
//是否正在扫描
fileprivate var scanIsBusy: Bool = false
fileprivate var scanTime: Timer?
//扫描到的产品集合
var deviceArr = [ZLPeripheral]()

//包含设备蓝牙,进度,错误
fileprivate var connectHandle: ConnectHandle?
fileprivate var connectIsBusy: Bool = false
fileprivate var connectTime: Timer?
//配置文件(需要搜索的产品)
fileprivate var config: ZLBluetoothConfig?

init(config: ZLBluetoothConfig) {
super.init()
self.centralMgr = CBCentralManager.init(delegate: self, queue: nil)
self.config = config
}

//默认扫描30.0s,超时停止扫描,Error = timeout
internal func scan(timeout: TimeInterval = 30.0,progresshandle: ScanProgressHandle?,comleteHandle: ScanCompleteHandle?) {

guard  scanIsBusy == false  else {
comleteHandle?(nil,.busy)
return
}

guard self.centralMgr.state == .poweredOn else {
comleteHandle?(nil,.centralStateError)
return
}

scanProgressHandle = nil
scanCompleteHandle = nil

if progresshandle != nil {
scanProgressHandle = progresshandle
}

if comleteHandle != nil {
scanCompleteHandle = comleteHandle
}

if (self.centralMgr.state == .poweredOn) {
self.centralMgr.scanForPeripherals(withServices: nil,options:[CBCentralManagerScanOptionAllowDuplicatesKey: false])

scanIsBusy = true
scanTime = Timer.scheduledTimer(timeInterval: timeout, target: self, selector: #selector(scanTimeout), userInfo: nil, repeats: false)
return
}

print("打开蓝牙失败:%d",self.centralMgr.state.rawValue)
}

//连接时重置属性
fileprivate func restoreState() {
self.peripheral = nil
self.notifyCharacteristic = nil
self.writeCodeCharacteristic = nil
self.bootLoaderCharacteristic = nil
}

//扫描超时
@objc fileprivate func scanTimeout() {
if scanTime != nil {
scanCompleteHandle?(deviceArr,.timeout)
}
invalidateScanState()
}

//主动停止扫描(连接成功后自动停止)
func stopScan() {
scanCompleteHandle?(deviceArr,.stop)
invalidateScanState()
}

fileprivate func invalidateScanState() {
scanTime?.invalidate()
scanTime = nil
scanIsBusy = false
deviceArr = [ZLPeripheral]()
self.centralMgr.stopScan()
}


//连接设备
internal func connect(peripheral:CBPeripheral?,timeout: TimeInterval = 5.0,handle: ConnectHandle?) {
guard connectIsBusy == false else {
connectHandle?(nil,.noConnect,.busy)
return
}

connectHandle = nil
if let per = peripheral {
connectTime = Timer.scheduledTimer(timeInterval: timeout, target: self, selector: #selector(disConnect), userInfo: nil, repeats: false)

connectHandle = handle
self.centralMgr.connect(per, options: nil)
connectIsBusy = true
}
}

@objc internal func disConnect() {

if connectTime != nil {
connectHandle?(nil,.noConnect,.timeout)
}

if let per = self.peripheral {
self.centralMgr.cancelPeripheralConnection(per)
self.peripheral = nil
}

connectTime?.invalidate()
connectTime = nil
connectIsBusy = false
}


//重新连接,如果没有特殊逻辑,重连的处理同连接
internal func reconnect(handle: ConnectHandle? = nil) {
if #available(iOS 9.0, *) {
if oldPeripheral != nil &&
(oldPeripheral!.state == .disconnected || oldPeripheral!.state == .disconnecting) {
self.connect(peripheral: oldPeripheral, handle: handle != nil ? handle : connectHandle)

}
} else {
if oldPeripheral != nil &&
oldPeripheral!.state == .disconnected {
self.connect(peripheral: oldPeripheral, handle: handle != nil ? handle : connectHandle)
}
}
}

internal func removeData() {
deviceArr.removeAll()
}

//向设备写入数据
internal func write(data: Data) {
guard let _ = peripheral else {
print("未找到设备")
return
}

guard peripheral?.state == .connected && writeCodeCharacteristic != nil else {
print("设备蓝牙未连接")
return
}

peripheral?.writeValue(data, for:writeCodeCharacteristic! , type: .withResponse)
}
}

extension ZLBluetoothManager:CBCentralManagerDelegate,CBPeripheralDelegate {

func centralManagerDidUpdateState(_ central: CBCentralManager) {
print("检查蓝牙状态")
switch central.state {
case .poweredOn:
print("蓝牙打开成功")
default:
print("蓝牙打开失败")
scanCompleteHandle?(nil,.centralStateError)
if (self.peripheral != nil) {
self.centralMgr .cancelPeripheralConnection(self.peripheral!)
}
}
}

func centralManager(_ central: CBCentralManager, didDiscover peripheral: CBPeripheral, advertisementData: [String : Any], rssi RSSI: NSNumber) {
//        print("name = \(peripheral.name ?? ""),identify: \(peripheral.identifier),RSSI = \(RSSI)")

guard config?.scanProductCode != nil else {
print("\nError: 产品编号为空\n")
return
}

if (peripheral.name?.contains((config?.scanProductCode)!)) == true {
print("\n\n\n---------------\n\(String(describing: config?.scanProductCode)) = \(advertisementData)\n-------------\n\n\n")

let zl_peripheral = ZLPeripheral.zl_Peripheral(peripheral: peripheral, advertisementData: advertisementData, RSSI: RSSI);

//返回扫描到的指定产品
scanProgressHandle?(zl_peripheral)

var container = false

for z_perpheral in deviceArr {
if (z_perpheral.peripheral?.isEqual(peripheral))! {
container = true
}
}

if !container {
deviceArr.append(zl_peripheral)
}

let str = advertisementData["kCBAdvDataManufacturerData"].debugDescription
print(str.replacingOccurrences(of: " ", with: ""))
}

}

func centralManager(_ central: CBCentralManager, didConnect peripheral: CBPeripheral) {
print("已连接%@",peripheral)
self.restoreState()

self.peripheral = peripheral
self.peripheral?.delegate = self
self.peripheral?.discoverServices(nil)

stopScan()

oldPeripheral = nil;
connectIsBusy = false
connectTime?.invalidate()
connectTime = nil
connectHandle?(peripheral,.connect,nil)
}

func peripheral(_ peripheral: CBPeripheral, didDiscoverServices error: Error?) {
print("发现服务")

if error != nil {
print(error ?? "didDiscoverServicesError")
return
}

if let services = peripheral.services {
for service in services {
self.peripheral?.discoverCharacteristics(nil, for: service)
}

connectHandle?(peripheral,.discoverServices,nil)
}
}

func peripheral(_ peripheral: CBPeripheral, didDiscoverCharacteristicsFor service: CBService, error: Error?) {
print("发现特征")

if (error != nil) {
print(error ?? "discoverError")
return
}

guard let _ = service.characteristics else {
print("没有发现特征")
return
}

for characteristic in service.characteristics! {
if characteristic.uuid == config?.writeCharacteristic {
writeCodeCharacteristic = characteristic
}

if characteristic.uuid == config?.notifyCharacteristic {
notifyCharacteristic = characteristic
self.peripheral?.readValue(for: characteristic)
self.peripheral?.setNotifyValue(true, for: characteristic)
print("监听:%@",characteristic)
}

if characteristic.uuid == config?.writeCharacteristic ||
characteristic.uuid == config?.notifyCharacteristic {
connectHandle?(peripheral,.discoverCharacteristics,nil)
}

if characteristic.uuid == config?.bootCharacteristic {
bootLoaderCharacteristic = characteristic

self.peripheral?.readValue(for: characteristic)
self.peripheral?.setNotifyValue(true, for: characteristic)
print("监听:%@",characteristic)
}
}
}

func peripheral(_ peripheral: CBPeripheral, didUpdateValueFor characteristic: CBCharacteristic, error: Error?) {
guard  error == nil else {
print("didUPdateValueError = \(error!)")
return
}

if characteristic.uuid == config?.notifyCharacteristic {
print("收到监听的设备值改变发送的通知\(String(describing: characteristic.value))")

delegate?.peripheral(peripheral, didUpdateValueForNotifyCharacteristic: characteristic.value)
return
}

if characteristic.uuid == config?.bootCharacteristic {
print("设备升级:\(String(describing: characteristic.value?.description))")

delegate?.peripheral?(peripheral, didUpdateValueForBootCharacteristic: characteristic.value)
}
}

func peripheral(_ peripheral: CBPeripheral, didWriteValueFor characteristic: CBCharacteristic, error: Error?) {

if error == nil {
print("写入成功")
} else {
print("写入失败")
}

}

func centralManager(_ central: CBCentralManager, didDisconnectPeripheral peripheral: CBPeripheral, error: Error?) {
print("设备断开连接")

oldPeripheral = peripheral;
self.restoreState()
self.peripheral = nil


connectHandle?(peripheral,.disconnect,nil)

delegate?.centralManager?(central, didDisconnectPeripheral: peripheral, error: error)
}
}
```

## 9  ZLBluetoothMessage和 ZLBluetoothCallBackMessage
分别为发送的蓝牙数据和蓝牙数据解析,为了减轻Controller的工作

Example:

发送数据:

```
//回复稳定数据
public func z_steadyBodyDataMessage() ->Data {
let start  = "5ad2"
let retain = "00000000000000000000000000000000"

let str = start + retain
var data = Data()
//+~ 为空返回自身,反之拼接
data +~ str.z_hexStringToData()
addCheckSum(data: &data)
data +~ "aa".z_hexStringToData()

print("steadyDataMessage:\(data.description)")
return data
}
```

校验位计算规则:从第1位开始到校验位前一个数据结束,做^运算:

```
//MARK:校验位
public func addCheckSum(data: inout Data) {
var out = UInt8.init(0x00)

let array = data.withUnsafeBytes {
[UInt8](UnsafeBufferPointer(start: $0, count: data.count))
}

print(array.count)

for i in 1..<array.count {
out ^= array[i]
}

data.append(out)
}

```


其中 +~为自定义运算符
```
precedencegroup dataAssignedPredence {
associativity:  right
higherThan: DefaultPrecedence
}

infix operator +~ : dataAssignedPredence

public func +~(x:inout Data,y: Data?) {

guard  let _ = y else {
return
}
x =  x + y!
}

```

## 10.  ZLString + Extension.Swift
- 十六进制Str转十进制Str
- 十六进制Str转Data

```
extension String {
public func  z_hexStringToString() -> String {

var str = self.replacingOccurrences(of: " ", with: "")

if str.characters.count % 2 != 0 {
print("解析数据为单数")
}

str = str.uppercased()
var sum = 0
for i in str.utf8 {
sum = sum * 16

if i >= 48 && i <= 57 {
sum += Int(i) - 48
}

if i >= 65 {
sum += Int(i) - 55
}
}

return String(sum)
}

public func z_hexStringToData() ->Data?  {
var value = self
var data:Data = Data()

if value.characters.count % 2 != 0 {
print("Error: 解析数据为单数")
return nil
}

for i in 0..<value.characters.count - 1 where i % 2 == 0 {
//
let str = (self as NSString).substring(with: NSRange(location: i, length: 2)).z_hexStringToString()
let temp = UInt8.init(str)
data.append(temp!)
}
return data
}

}

```

## 11.ZLData + EXtension.Swift

- 取指定字节的字符串
- 取指定范围的字符串

```
extension Data {
public func z_strWithIndex(_ index: Int) ->String? {
guard index < self.count else {
return nil
}

let array = self.withUnsafeBytes {
[UInt8](UnsafeBufferPointer(start: $0, count: self.count))
}

guard  array.count > 0 else {
return nil
}

let result = String(format: "%02x", array[index])
return result
}

public func z_strWithRange(location: Int,length: Int) -> String? {

var str: String = ""
for i in location..<location+length {
let subStr = self.z_strWithIndex(i)

guard  let _ = subStr else {
return nil
}

str = str + subStr!
}

return str
}
}

```
