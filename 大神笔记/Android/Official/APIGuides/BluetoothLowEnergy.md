[toc]

## Bluetooth Low Energy

Android 4.3 (API Level 18) introduces built-in platform support for Bluetooth Low Energy *in the central role*，并提供API，用于发现设备、查询设备、读写特征（characteristics）。Bluetooth Low Energy (BLE)的功耗比经典蓝牙显著降低。This allows Android apps to communicate with BLE devices that have low power requirements, such as proximity sensors, heart rate monitors, fitness devices, and so on.

### 关键术语和概念

- **Generic Attribute Profile (GATT)**：GATT profile是一个通用的规范（general specification），通过BLE链路（link）收发短数据（称为"attributes"）。目前所有的的Low Energy应用profiles都基于GATT。The Bluetooth SIG defines many [profiles](https://www.bluetooth.org/en-us/specification/adopted-specifications) for Low Energy devices. A profile is a specification for how a device works in a particular application. 设备可以实现超过一个profile。例如，一个设备可以包含一个心率监控器和一个battery level detector。
- **Attribute Protocol (ATT)**：GATT基于Attribute Protocol (ATT)。This is also referred to as GATT/ATT. ATT被优化运行于BLE设备。此协议尽量用最少的字节。每个attribute由一个UUID唯一的识别（128位字符串）。The attributes transported by ATT are formatted as *characteristics* and *services*.
- **Characteristic**：一个characteristic包含一个值，和0到n个descriptors描述这个值。一个characteristic可以看做一个类型，或一个类。
- **Descriptor**：Descriptors are defined attributes that describe a characteristic value. For example, a descriptor might specify a human-readable description, an acceptable range for a characteristic's value, or a unit of measure that is specific to a characteristic's value.
- **Service**：一个服务是特征的集合。For example, you could have a service called "Heart Rate Monitor" that includes characteristics such as "heart rate measurement." You can find a list of existing GATT-based profiles and services on [bluetooth.org](https://www.bluetooth.org/en-us/specification/adopted-specifications).

#### 角色与职责

Here are the roles and responsibilities that apply when an Android device interacts with a BLE device:

- 中央 vs 外围（peripheral）。This applies to the BLE connection itself. 处于中心角色的设备扫描、搜索advertisement，外围设备发送advertisement。
- GATT服务器 vs GATT客户端。This determines how two devices talk to each other once they've established the connection.

To understand the distinction, imagine that you have an Android phone and an activity tracker that is a BLE device. 手机支持中心角色，活动追踪器支持外围角色。要建立连接，两步必须是两个不同角色。相同角色两个设备不能互联。

一旦手机和活动追踪器建立了连接，他们开始传送GATT元数据。根据传输的数据类型，其中一个作为服务器。例如，如果活动追踪器想向手机报告传感器数据，由活动追踪器做服务器是合理的。如果活动追踪器想从手机接收数据，则手机做服务器是合理的。

在本章的例子中，Android App是GATT客户端。App从GATT服务器接收数据，服务器是一个BLE heart rate monitor that supports the [Heart Rate Profile](http://developer.bluetooth.org/TechnologyOverview/Pages/HRP.aspx)。当然也可以设计App做GATT服务器。See [BluetoothGattServer](http://developer.android.com/reference/android/bluetooth/BluetoothGattServer.html) for more information.

### BLE权限

In order to use Bluetooth features in your application, you must declare the Bluetooth permission `BLUETOOTH`. You need this permission to perform any Bluetooth communication, such as requesting a connection, accepting a connection, and transferring data.

If you want your app to initiate device discovery or manipulate Bluetooth settings, you must also declare the `BLUETOOTH_ADMIN` permission. Note: If you use the `BLUETOOTH_ADMIN` permission, then you must also have the `BLUETOOTH` permission.

Declare the Bluetooth permission(s) in your application manifest file. For example:

    <uses-permission android:name="android.permission.BLUETOOTH"/>
    <uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>

如果想限定你的App只对有BLE功能的设备可用，include the following in your app's manifest:

	<uses-feature android:name="android.hardware.bluetooth_le" android:required="true"/>

However, if you want to make your app available to devices that don't support BLE, you should still include this element in your app's manifest, but set `required="false"`. Then at run-time you can determine BLE availability by using `PackageManager.hasSystemFeature()`:

    // Use this check to determine whether BLE is supported on the device. Then
    // you can selectively disable BLE-related features.
    if (!getPackageManager().hasSystemFeature(PackageManager.FEATURE_BLUETOOTH_LE)) {
        Toast.makeText(this, R.string.ble_not_supported, Toast.LENGTH_SHORT).show();
        finish();
    }

### Setting Up BLE

Before your application can communicate over BLE, you need to verify that BLE is supported on the device, and if so, ensure that it is enabled. Note that this check is only necessary if `<uses-feature.../>` is set to false.

If BLE is not supported, then you should gracefully disable any BLE features. If BLE is supported, but disabled, then you can request that the user enable Bluetooth without leaving your application. This setup is accomplished in two steps, using the `BluetoothAdapter`.

1、Get the `BluetoothAdapter`

The `BluetoothAdapter` is required for any and all Bluetooth activity. `BluetoothAdapter`表示蓝牙适配器（无线电）。整个系统只有一个适配器，利用此对象与之交互。`BluetoothManager`是Android 4.3 (API Level 18)才引入的。

    // Initializes Bluetooth adapter.
    final BluetoothManager bluetoothManager =
            (BluetoothManager) getSystemService(Context.BLUETOOTH_SERVICE);
    mBluetoothAdapter = bluetoothManager.getAdapter();

2、Enable Bluetooth

Next, you need to ensure that Bluetooth is enabled. Call `isEnabled()` to check whether Bluetooth is currently enabled. 下面的代码检查Bluetooth是否已启用。若未，让用户进入设置界面启用蓝牙：

    private BluetoothAdapter mBluetoothAdapter;
    ...
    // Ensures Bluetooth is available on the device and it is enabled. If not,
    // displays a dialog requesting user permission to enable Bluetooth.
    if (mBluetoothAdapter == null || !mBluetoothAdapter.isEnabled()) {
        Intent enableBtIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
        startActivityForResult(enableBtIntent, REQUEST_ENABLE_BT);
    }

### 寻找 BLE 设备

使用`startLeScan()`寻找BLE设备。该方法需要一个`BluetoothAdapter.LeScanCallback`做参数，扫描结果返回给这个回调。由于搜索很耗电，因此应遵循：

- 找到目标设备后，停止扫描
- 不要在循环内扫描，给扫描设置一个时间限制。之前能访问的设备可能已经移走，持续扫描可能耗光电量。

下面展示如何启动和停止扫描：

    public class DeviceScanActivity extends ListActivity {

        private BluetoothAdapter mBluetoothAdapter;
        private boolean mScanning;
        private Handler mHandler;

        // Stops scanning after 10 seconds.
        private static final long SCAN_PERIOD = 10000;
        ...
        private void scanLeDevice(final boolean enable) {
            if (enable) {
                // Stops scanning after a pre-defined scan period.
                mHandler.postDelayed(new Runnable() {
                    @Override
                    public void run() {
                        mScanning = false;
                        mBluetoothAdapter.stopLeScan(mLeScanCallback);
                    }
                }, SCAN_PERIOD);

                mScanning = true;
                mBluetoothAdapter.startLeScan(mLeScanCallback);
            } else {
                mScanning = false;
                mBluetoothAdapter.stopLeScan(mLeScanCallback);
            }
            ...
        }
    ...
    }

如果只想扫描特定类型的外围设备，可以调用`startLeScan(UUID[], BluetoothAdapter.LeScanCallback)`，providing an array of UUID objects that specify the GATT services your app supports.

Here is an implementation of the `BluetoothAdapter.LeScanCallback`, which is the interface used to deliver BLE scan results:

    private LeDeviceListAdapter mLeDeviceListAdapter;
    ...
    // Device scan callback.
    private BluetoothAdapter.LeScanCallback mLeScanCallback =
            new BluetoothAdapter.LeScanCallback() {
        @Override
        public void onLeScan(final BluetoothDevice device, int rssi, byte[] scanRecord) {
            runOnUiThread(new Runnable() {
               @Override
               public void run() {
                   mLeDeviceListAdapter.addDevice(device);
                   mLeDeviceListAdapter.notifyDataSetChanged();
               }
           });
       }
    };

注意：不能同时扫描Bluetooth LE和经典蓝牙设备。一次只能扫描一种。

## 链接到GATT服务器

与BLE设备交互的第一步是连接，更准确的说，连接到设备的GATT服务器。使用`connectGatt()`连接到BLE设备上的GATT服务器。该方法有三个参数：一个Context对象，autoConnect（布尔值，indicating whether to automatically connect to the BLE device as soon as it becomes available），and a reference to a `BluetoothGattCallback`:

	mBluetoothGatt = device.connectGatt(this, false, mGattCallback);

返回一个`BluetoothGatt`实例，which you can then use to conduct GATT client operations. Android App是GATT客户端。The `BluetoothGattCallback` is used to deliver results to the client, such as connection status, as well as any further GATT client operations.

In this example, the BLE app provides an activity (DeviceControlActivity) to connect, display data, and display GATT services and characteristics supported by the device. 根据用户输入，活动与一个Android服务（BluetoothLeService)交互，服务负责通过Android BLE API与BLE设备通讯。

    // A service that interacts with the BLE device via the Android BLE API.
    public class BluetoothLeService extends Service {
        private final static String TAG = BluetoothLeService.class.getSimpleName();

        private BluetoothManager mBluetoothManager;
        private BluetoothAdapter mBluetoothAdapter;
        private String mBluetoothDeviceAddress;
        private BluetoothGatt mBluetoothGatt;
        private int mConnectionState = STATE_DISCONNECTED;

        private static final int STATE_DISCONNECTED = 0;
        private static final int STATE_CONNECTING = 1;
        private static final int STATE_CONNECTED = 2;

        public final static String ACTION_GATT_CONNECTED =
                "com.example.bluetooth.le.ACTION_GATT_CONNECTED";
        public final static String ACTION_GATT_DISCONNECTED =
                "com.example.bluetooth.le.ACTION_GATT_DISCONNECTED";
        public final static String ACTION_GATT_SERVICES_DISCOVERED =
                "com.example.bluetooth.le.ACTION_GATT_SERVICES_DISCOVERED";
        public final static String ACTION_DATA_AVAILABLE =
                "com.example.bluetooth.le.ACTION_DATA_AVAILABLE";
        public final static String EXTRA_DATA =
                "com.example.bluetooth.le.EXTRA_DATA";

        public final static UUID UUID_HEART_RATE_MEASUREMENT =
                UUID.fromString(SampleGattAttributes.HEART_RATE_MEASUREMENT);

        // Various callback methods defined by the BLE API.
        private final BluetoothGattCallback mGattCallback =
                new BluetoothGattCallback() {
            @Override
            public void onConnectionStateChange(BluetoothGatt gatt, int status,
                    int newState) {
                String intentAction;
                if (newState == BluetoothProfile.STATE_CONNECTED) {
                    intentAction = ACTION_GATT_CONNECTED;
                    mConnectionState = STATE_CONNECTED;
                    broadcastUpdate(intentAction);
                    Log.i(TAG, "Connected to GATT server.");
                    Log.i(TAG, "Attempting to start service discovery:" +
                            mBluetoothGatt.discoverServices());

                } else if (newState == BluetoothProfile.STATE_DISCONNECTED) {
                    intentAction = ACTION_GATT_DISCONNECTED;
                    mConnectionState = STATE_DISCONNECTED;
                    Log.i(TAG, "Disconnected from GATT server.");
                    broadcastUpdate(intentAction);
                }
            }

            @Override
            // New services discovered
            public void onServicesDiscovered(BluetoothGatt gatt, int status) {
                if (status == BluetoothGatt.GATT_SUCCESS) {
                    broadcastUpdate(ACTION_GATT_SERVICES_DISCOVERED);
                } else {
                    Log.w(TAG, "onServicesDiscovered received: " + status);
                }
            }

            @Override
            // Result of a characteristic read operation
            public void onCharacteristicRead(BluetoothGatt gatt,
                    BluetoothGattCharacteristic characteristic,
                    int status) {
                if (status == BluetoothGatt.GATT_SUCCESS) {
                    broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);
                }
            }
         ...
        };
    ...
    }

When a particular callback is triggered, it calls the appropriate broadcastUpdate() helper method and passes it an action. Note that the data parsing in this section is performed in accordance with the Bluetooth Heart Rate Measurement [profile specifications](http://developer.bluetooth.org/gatt/characteristics/Pages/CharacteristicViewer.aspx?u=org.bluetooth.characteristic.heart_rate_measurement.xml):

    private void broadcastUpdate(final String action) {
        final Intent intent = new Intent(action);
        sendBroadcast(intent);
    }

    private void broadcastUpdate(final String action,
                                 final BluetoothGattCharacteristic characteristic) {
        final Intent intent = new Intent(action);

        // This is special handling for the Heart Rate Measurement profile. Data
        // parsing is carried out as per profile specifications.
        if (UUID_HEART_RATE_MEASUREMENT.equals(characteristic.getUuid())) {
            int flag = characteristic.getProperties();
            int format = -1;
            if ((flag & 0x01) != 0) {
                format = BluetoothGattCharacteristic.FORMAT_UINT16;
                Log.d(TAG, "Heart rate format UINT16.");
            } else {
                format = BluetoothGattCharacteristic.FORMAT_UINT8;
                Log.d(TAG, "Heart rate format UINT8.");
            }
            final int heartRate = characteristic.getIntValue(format, 1);
            Log.d(TAG, String.format("Received heart rate: %d", heartRate));
            intent.putExtra(EXTRA_DATA, String.valueOf(heartRate));
        } else {
            // For all other profiles, writes the data formatted in HEX.
            final byte[] data = characteristic.getValue();
            if (data != null && data.length > 0) {
                final StringBuilder stringBuilder = new StringBuilder(data.length);
                for(byte byteChar : data)
                    stringBuilder.append(String.format("%02X ", byteChar));
                intent.putExtra(EXTRA_DATA, new String(data) + "\n" +
                        stringBuilder.toString());
            }
        }
        sendBroadcast(intent);
    }

Back in `DeviceControlActivity`, these events are handled by a `BroadcastReceiver`:

    // Handles various events fired by the Service.
    // ACTION_GATT_CONNECTED: connected to a GATT server.
    // ACTION_GATT_DISCONNECTED: disconnected from a GATT server.
    // ACTION_GATT_SERVICES_DISCOVERED: discovered GATT services.
    // ACTION_DATA_AVAILABLE: received data from the device. This can be a
    // result of read or notification operations.
    private final BroadcastReceiver mGattUpdateReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            final String action = intent.getAction();
            if (BluetoothLeService.ACTION_GATT_CONNECTED.equals(action)) {
                mConnected = true;
                updateConnectionState(R.string.connected);
                invalidateOptionsMenu();
            } else if (BluetoothLeService.ACTION_GATT_DISCONNECTED.equals(action)) {
                mConnected = false;
                updateConnectionState(R.string.disconnected);
                invalidateOptionsMenu();
                clearUI();
            } else if (BluetoothLeService.
                    ACTION_GATT_SERVICES_DISCOVERED.equals(action)) {
                // Show all the supported services and characteristics on the
                // user interface.
                displayGattServices(mBluetoothLeService.getSupportedGattServices());
            } else if (BluetoothLeService.ACTION_DATA_AVAILABLE.equals(action)) {
                displayData(intent.getStringExtra(BluetoothLeService.EXTRA_DATA));
            }
        }
    };

### Reading BLE Attributes

Once your Android app has connected to a GATT server and discovered services, it can read and write attributes, where supported. For example, this snippet iterates through the server's services and characteristics and displays them in the UI:

    public class DeviceControlActivity extends Activity {
        ...
        // Demonstrates how to iterate through the supported GATT
        // Services/Characteristics.
        // In this sample, we populate the data structure that is bound to the
        // ExpandableListView on the UI.
        private void displayGattServices(List<BluetoothGattService> gattServices) {
            if (gattServices == null) return;
            String uuid = null;
            String unknownServiceString = getResources().
                    getString(R.string.unknown_service);
            String unknownCharaString = getResources().
                    getString(R.string.unknown_characteristic);
            ArrayList<HashMap<String, String>> gattServiceData =
                    new ArrayList<HashMap<String, String>>();
            ArrayList<ArrayList<HashMap<String, String>>> gattCharacteristicData
                    = new ArrayList<ArrayList<HashMap<String, String>>>();
            mGattCharacteristics =
                    new ArrayList<ArrayList<BluetoothGattCharacteristic>>();

            // Loops through available GATT Services.
            for (BluetoothGattService gattService : gattServices) {
                HashMap<String, String> currentServiceData =
                        new HashMap<String, String>();
                uuid = gattService.getUuid().toString();
                currentServiceData.put(
                        LIST_NAME, SampleGattAttributes.
                                lookup(uuid, unknownServiceString));
                currentServiceData.put(LIST_UUID, uuid);
                gattServiceData.add(currentServiceData);

                ArrayList<HashMap<String, String>> gattCharacteristicGroupData =
                        new ArrayList<HashMap<String, String>>();
                List<BluetoothGattCharacteristic> gattCharacteristics =
                        gattService.getCharacteristics();
                ArrayList<BluetoothGattCharacteristic> charas =
                        new ArrayList<BluetoothGattCharacteristic>();
               // Loops through available Characteristics.
                for (BluetoothGattCharacteristic gattCharacteristic :
                        gattCharacteristics) {
                    charas.add(gattCharacteristic);
                    HashMap<String, String> currentCharaData =
                            new HashMap<String, String>();
                    uuid = gattCharacteristic.getUuid().toString();
                    currentCharaData.put(
                            LIST_NAME, SampleGattAttributes.lookup(uuid,
                                    unknownCharaString));
                    currentCharaData.put(LIST_UUID, uuid);
                    gattCharacteristicGroupData.add(currentCharaData);
                }
                mGattCharacteristics.add(charas);
                gattCharacteristicData.add(gattCharacteristicGroupData);
             }
        ...
        }
    ...
    }

### Receiving GATT Notifications

It's common for BLE apps to ask to be notified when a particular characteristic changes on the device. This snippet shows how to set a notification for a characteristic, using the `setCharacteristicNotification()` method:

    private BluetoothGatt mBluetoothGatt;
    BluetoothGattCharacteristic characteristic;
    boolean enabled;
    ...
    mBluetoothGatt.setCharacteristicNotification(characteristic, enabled);
    ...
    BluetoothGattDescriptor descriptor = characteristic.getDescriptor(
            UUID.fromString(SampleGattAttributes.CLIENT_CHARACTERISTIC_CONFIG));
    descriptor.setValue(BluetoothGattDescriptor.ENABLE_NOTIFICATION_VALUE);
    mBluetoothGatt.writeDescriptor(descriptor);
    Once notifications are enabled for a characteristic, an onCharacteristicChanged() callback is triggered if the characteristic changes on the remote device:

    @Override
    // Characteristic notification
    public void onCharacteristicChanged(BluetoothGatt gatt,
            BluetoothGattCharacteristic characteristic) {
        broadcastUpdate(ACTION_DATA_AVAILABLE, characteristic);
    }

### 关闭客户端App

Once your app has finished using a BLE device, it should call close() so the system can release resources appropriately:

    public void close() {
        if (mBluetoothGatt == null) {
            return;
        }
        mBluetoothGatt.close();
        mBluetoothGatt = null;
    }