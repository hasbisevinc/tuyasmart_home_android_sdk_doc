# Device Control

## **Acquisition of Device Information**

**[Description]**

Tuya Smart provides a lot of interfaces for developers to realize the acquisition of device information and management capabilities (removal, and so on). Receivers shall be informed of the device-related return data by means of asynchronous messages.

**[Notes]**

- For the device control, the data must be initialized first; namely, TuyaHomeSdk.newHomeInstance(homeId).getHomeDetail(ITuyaHomeResultCallback callback) must be invoked first.
- Introduction to schema dp data [see related concepts of function points for details] [3]

![pastedGraphic.png](images/pastedGraphic-1064908.png)

**Device Operation Control**

The ITuyaDevice class provides the device status notification capability. By registering callback functions, developers can easily obtain notifications on device data reception, device removal, online/offline status of device, and mobile network changes. In addition, it also provides an interface for controlling command issuing and device firmware upgrade.

```java
// Initialize the device control class based on the device id.
ITuyaDevice mDevice = TuyaHomeSdk.newDeviceInstance(deviceBean.getDevId());
```
## **Function Points of Device**

The dps attribute of the DeviceBean class defines the state of the device, and is called the data point (DP) or the function point. Each key in the dps dictionary refers to a dpId of a function point, and the dpValue is the value of the function point. Refer to the functions of product on the [Tuya developer platform ](https://developer.tuya.com/)for definition of function points of products. For details of function points, please refer to the [QuickStart-Related Concepts of Function Points](https://docs.tuya.com/cn/creatproduct/#_7)

**[Command Format]**

The control commands shall be sent in the format given below. {"(dpId)":"(dpValue)"}

**[Example of Function Point]**

A product interface on the development platform is as follows:![pastedGraphic_1.png](../../../../../../Library/Application Support/typora-user-images/B8B015F7-C08C-4E83-8DCE-A2FE553C169B/pastedGraphic_1.png)

According to the definition of function points of the product in the back end, the example codes is as follows.
```java
// Example for boolean type function point with dpId set to 1. Function: turn on switch. 

dps = {"1": true};



// Example of string type function point with dpId set to 4. Function: set RGB to ff5500.

dps = {"4": "ff5500"};



// Example of enumeration type function point with dpId set to 5. Function: set gear to position 2.

dps = {"5": "2"};



// Example of value type function point with dpId set to 6. Function: set temperature to 20 centigrade degree.

dps = {"6": 20};



// Example of transparent (byte array) function point with dpId set to 15. Function: transparently transfer infrared data (i.e., 1122).

dps = {"15": "1122"};

// Send multiple functions in one time.

dps = {"1": true, "4": "ff5500"};

mDevice.publishDps(dps, new IControlCallback() {

@Override

public void onError(String code, String error) {}

@Override

public void onSuccess() {

}

});
```
**[Notes]**

- Special attention shall be paid to the type of data in sending the control commands. 
	 For example, the data type of function points shall be value, and {"2": 25} shall be sent as the control command, instead of {"2": "25"}.
- For the transparent transmission, the byte array shall be the string format, and the string must have even bits. 
	 The correct format shall be: {"1": "0110"}, instead of {"1": "110"}.

### **Initializing Data Listener**

**[Description]**

The TuyaHomeDevice provides listeners for device-related information (dp data, device name, online status of device, and device removal), and the information will be synchronized here in real time.

**[Callback]**
```java
mDevice.registerDevListener(new IDevListener() {
    @Override
    public void onDpUpdate(String devId, String dpStr) {
    // dp data update: devId and corresponding dp data.
    }
    @Override
    public void onRemoved(String devId) {

    // The device is removed.
    }
    @Override
    public void onStatusChanged(String devId, boolean online) {
    // Status of device: online.
    }

    @Override
    public void onNetworkStatusChanged(String devId, boolean status) {
    // Network status listener
    }
    @Override
    public void onDevInfoUpdate(String devId) {
    // Device information change (currently, only the device name change requires the interface invocation)
    }

});
```
## Data Issuing

**[Description]**

Send control commands to the device through the LAN or the cloud.

**[Method Invocation]**
```java
// Send control commands to the hardware.

mDevice.send(String command,IControlCallback callback);
```
**[Example Codes]**

Take the light product as an example.

1. Defining the dp point of light switch
```java
public static final String STHEME_LAMP_DPID_1 = "1"; //light switch 
```
2. Data structure of light switch
```java
public class LampBean {
	private boolean open;
	public boolean isOpen() {
		return open;
	}
	public void setOpen(boolean open) {
		this.open = open;
	}
}
```
3. Device initialization
```java
/**

 \* Device object. All dp changes for this device will be returned via callback.

 *

 \* Before the device initialization, please make sure that the connection server has been initialized; otherwise, the information returned by the server cannnot be obtained.

 */

mDevice = new TuyaHomeSdk.newDeviceInstance(mDevId);



mDevice.registerDevListener(new IDevListener() {
    @Override
    public void onDpUpdate(String devId, String dpStr) {
        // dp data update: devId and corresponding dp data.
    }
    @Override
    public void onRemoved(String devId) {
        // The device is removed.
    }
    @Override
    public void onStatusChanged(String devId, boolean online) {
        // Status of device: online.
        // The statusChange refers to whether the communication between the hardware device and the cloud is normal.
    }
    @Override
    public void onNetworkStatusChanged(String devId, boolean status) {
        // Network status listener
        // The onNetworkStatusChanged refers to whether the communication between the mobile phone and the cloud is normal.
    }
    @Override
    public void onDevInfoUpdate(String devId) {
        // Device information change (currently, only the device name change requires the interface invocation)
    }
});
```
\4. Code segment for switching on the light
```java
 public void openLamp() {
    LampBean bean = new LampBean();
    bean.setOpen(true);
    HashMap<String, Object> hashMap = new HashMap<>();
    hashMap.put(STHEME_LAMP_DPID_1, bean.isOpen());
    mDevice.publishDps(JSONObject.toJSONString(hashMap), new IControlCallback() {
        @Override
        public void onError(String code, String error) {
           Toast.makeText(mContext, "turning on the light failed", Toast.LENGTH_SHORT).show();
        }
        @Override
        public void onSuccess() {
            Toast.makeText(mContext, "turning on the light failed", Toast.LENGTH_SHORT).show();
        }
    });
}
```
5. Cancel device listener event
```java
mDevice.unRegisterDevListener();
```
\6. Device resource destruction
```java
mDevice.onDestroy();
```
**[Notes]**

- The successful issuing of a command does not mean that the device is successfully operated, but only means that the command has been successfully sent. If the operation succeeds, the dp data information will be reported, and returned through the IDevListener onDpUpdate interface.
- The command string is converted to JsonString in the format of Map<String dpId,Object dpValue>.
- The command can send multiple dp data at a time.



## Querying Device Information

**[Description]**

Query single dp data; query the latest data of the dp from the device; those data will be called back via the IDevicePanelCallback onDpUpdate interface.

**[Method Invocation]**

mDevice.getDp(String dpId, IResultCallback callback);

**[Example Codes]**

```java
1. It is achieved by invoking the mTuyaDevice.getDp method.
2. The data will be reported via the dp data upgrade listener.

IDevListener.onDpUpdate(String devId,String dpStr)
```
**[Notes]**

- This interface is mainly for the dp points where the data will not be reported automatically. The dp data values for regular query can be obtained through getDps() in DeviceBean.

## Device Renaming

**[Description]**

Device renaming can be conducted in multiple devices synchronously.

**[Method Invocation]**
```java
// Rename
mDevice.renameDevice(String name,IResultCallback callback);
```
**[Example Codes]**
```java
mDevice.renameDevice("device name", new IResultCallback() {
    @Override
    public void onError(String code, String error) {
        // Renaming failed
    }
    @Override
    public void onSuccess() {
        // Renaming succeeded
    }
});
```
After renaming succeeded, IDevListener.onDevInfoUpdate() will be notified.

Invoke the following method to get the latest data, and then refresh the device information.
```java
TuyaHomeDataManager.getInstance().getDeviceBean(String devId);
```
## **Obtaining Historical Data for Data Points**

**[Description]**

Obtain historical status data of dp points, such as information about power.

**[Method Invocation]**

```java
* @param type           obtaining type values of historical data (hour, day and month)
* @param number         obtaining the maximum number of historical data points (value (1-50))
* @param dpId           obtaining dp point values of historical data
* @param startTime      obtaining the coordinate and date of the historical data point

getDataPointStat(DataPointTypeEnum type, long startTime, int number, String dpId, final IGetDataPointStatCallback callback)

long startTime = System.currentTimeMillis(); // startTime

int number = 12;// to obtain the number of resulting values of historical data (maximum: 50)

String dpId = "1";

mDevice.getDataPointStat(DataPointTypeEnum.DAY, startTime, number, dpId, new IGetDataPointStatCallback() {
    @Override
    public void onError(String errorCode, String errorMsg) {
       Toast.makeText(mContext, "obtaining historical data failed" + errorMsg, Toast.LENGTH_SHORT).show();
    }


    @Override
    public void onSuccess(DataPointStatBean bean) {
        Toast.makeText(mContext, "obtaining historical data succeeded:", Toast.LENGTH_SHORT).show();
    }
});
```
## Remove Device

**[Description]**

It is used to remove a device from the list of user devices.

**[Method Invocation]**
```java
/**
* Remove device
*
* @param callback
*/

void removeDevice(IResultCallback callback);
```
**[Example Codes]**
```java
mDevice.removeDevice(new IResultCallback() {
    @Override
    public void onError(String errorCode, String errorMsg) {
    }
    @Override
    public void onSuccess() {
    }

});
```