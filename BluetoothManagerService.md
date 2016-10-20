#BluetoothManagerService

##启动
 在SystemServer中启动BluetoothService
```
            if (isEmulator) {
                Slog.i(TAG, "No Bluetooh Service (emulator)");
            } else if (mFactoryTestMode == FactoryTest.FACTORY_TEST_LOW_LEVEL) {
                Slog.i(TAG, "No Bluetooth Service (factory test)");
            } else if (!context.getPackageManager().hasSystemFeature
                       (PackageManager.FEATURE_BLUETOOTH)) {
                Slog.i(TAG, "No Bluetooth Service (Bluetooth Hardware Not Present)");
            } else if (disableBluetooth) {
                Slog.i(TAG, "Bluetooth Service disabled by config");
            } else {
                Slog.i(TAG, "Bluetooth Service");
                mSystemServiceManager.startService(BluetoothService.class);
            }

```
而BluetoothService.java的构造函数中有会new BluetoothManagerService(),在该service起来的时候即onStart()中会调用publishBinderService将该service添加到serviceManager中，这样其他应用程序和服务就都能访问该service了。
```java
publishBinderService(BluetoothAdapter.BLUETOOTH_MANAGER_SERVICE, mBluetoothManagerService);
```

当phase 是SystemService.PHASE_SYSTEM_SERVICES_READY的时候，那么变回调用BluetoothManagerService.handleOnBootPhase()，在这个过程中会去初始化bt设备，enable bt device等。代码如下:
```java
    /**
     * Send enable message and set adapter name and address. Called when the boot phase becomes
     * PHASE_SYSTEM_SERVICES_READY.
     */
    public void handleOnBootPhase() {
        if (DBG) Log.d(TAG, "Bluetooth boot completed");
        // MStar Android Patch Begin
        // detect bt dongle thread
        new Thread(new Runnable() {
            public void run() {
                mBluetoothMonitor.initDetectBluetoothDevice();
                if (!("".equals(SystemProperties.get("mstar.bt.driver")))) {
                    mDongleState = true;
                    enablingBluetooth();
                }
                for (;;) {
                    int res = mBluetoothMonitor.detectBluetoothDevice();
                    if (res == 1) {
                        Log.d(TAG, "bt device added!");
                        mDongleState = true;
                        sendPlugInMsg();
                    } else if (res == 0) {
                        Log.d(TAG, "bt device Removed!");
                        mDongleState = false;
                        sendPlugOutMsg();
                        if (BluetoothAdapter.getDefaultAdapter().getBtSuspend()) {
                            Log.d(TAG, "BT usb disconnect caused by suspending");
                        } else if (mHardwarReset) {
                            Log.d(TAG, "BT usb disconnect caused by recovery reset");
                            mHardwarReset = false;
                        } else {
                            Log.e(TAG, "BT usb disconnect caused by unknown reason, reset it");
                            hardwareResetBluetooth();
                        }
                    } else if (res == -2) {
                        Log.e(TAG,"No BT usb device within 10s, reset it!");
                        hardwareResetBluetooth();
                    }
                }
            }
        }).start();
        // MStar Android Patch End
    }

```
这里会创建一个线程不停地去监测 bt 设备的状态，可以实时的得到当前bt device added 还是bt device removed, 如果检测到当前device状态不对，甚至可以硬件复位Bluetooth。
如果检测到bt added的话那么就会给Handle发送MSG_ENABlE消息，主要来看下handleEnable()函数:
```java
 private void handleEnable(boolean quietMode) {
        mQuietEnable = quietMode;

        synchronized(mConnection) {
            if ((mBluetooth == null) && (!mBinding)) {
                //Start bind timeout and bind
                Message timeoutMsg=mHandler.obtainMessage(MESSAGE_TIMEOUT_BIND);
                mHandler.sendMessageDelayed(timeoutMsg,TIMEOUT_BIND_MS);
                mConnection.setGetNameAddressOnly(false);
                // MStar Android Patch Begin
                Intent i;
                if (isBcmChip()) {
                    i = new Intent("android.bluetooth.bcm.IBluetooth");
                }else {
                    i = new Intent(IBluetooth.class.getName());
                }
                // MStar Android Patch End
                if (!doBind(i, mConnection,Context.BIND_AUTO_CREATE | Context.BIND_IMPORTANT,
                        UserHandle.CURRENT)) {
                    mHandler.removeMessages(MESSAGE_TIMEOUT_BIND);
                } else {
                    mBinding = true;
                }
            } else if (mBluetooth != null) {
                if (mConnection.isGetNameAddressOnly()) {
                    // if GetNameAddressOnly is set, we can clear this flag,
                    // so the service won't be unbind
                    // after name and address are saved
                    mConnection.setGetNameAddressOnly(false);
                    //Register callback object
                    try {
                        mBluetooth.registerCallback(mBluetoothCallback);
                    } catch (RemoteException re) {
                        Log.e(TAG, "Unable to register BluetoothCallback",re);
                    }
                    //Inform BluetoothAdapter instances that service is up
                    sendBluetoothServiceUpCallback();
                }

                //Enable bluetooth
                try {
                    if (!mQuietEnable) {
                        if(!mBluetooth.enable()) {
                            Log.e(TAG,"IBluetooth.enable() returned false");
                        }
                    }
                    else {
                        if(!mBluetooth.enableNoAutoConnect()) {
                            Log.e(TAG,"IBluetooth.enableNoAutoConnect() returned false");
                        }
                    }
                } catch (RemoteException e) {
                    Log.e(TAG,"Unable to call enable()",e);
                }
            }
        }
    }
```
可以看到这边调用了mBluetooth.enable() 即调用AdapterService.enable() 而BluetoothManagerService和AdapterService之间通过Binder进行进程间通讯，
而之后又调用mAdapterStateMachine.sendMessage(obtainMessage(BLE_TURN_ON)),这里就会涉及到状态机的概念，因为当前处于OffState，我们来看
```java
 case BLE_TURN_ON:
                   notifyAdapterStateChange(BluetoothAdapter.STATE_BLE_TURNING_ON);
                   mPendingCommandState.setBleTurningOn(true);
                   transitionTo(mPendingCommandState);
                   sendMessageDelayed(BLE_START_TIMEOUT, BLE_START_TIMEOUT_DELAY);
                   adapterService.BleOnProcessStart();
                   break;

```
通过notifyAdapterStateChange()之后调用到AdapterService.updateAdapterState(),这里会对mCallback里register的callback挨个通知状态的变化，可以看到mCallback是IBluetoothCallback类型的，而只有BluetoothManagerService是register到这里的 因此即调用BluetoothManagerService的onBluetoothStateChange()
```java
void updateAdapterState(int prevState, int newState){
        if (mCallbacks !=null) {
            int n=mCallbacks.beginBroadcast();
            debugLog("updateAdapterState() - Broadcasting state to " + n + " receivers.");
            for (int i=0; i <n;i++) {
                try {
                    mCallbacks.getBroadcastItem(i).onBluetoothStateChange(prevState,newState);
                }  catch (RemoteException e) {
                    debugLog("updateAdapterState() - Callback #" + i + " failed ("  + e + ")");
                }
            }
            mCallbacks.finishBroadcast();
        }
    }

```
而在BluetoothManagerService的onBluetoothStateChange()只是给自己的Handler发一个message.

我们接着回到BLE_TURN_ON的地方分析，之后把stateMachine设置未PendingCommandState，然后delay 发送一个BLE_START_TIMEOUT消息，然后调用AdapterService.BleOnProcessStart(),我们看下这个函数的代码：
```java
   void BleOnProcessStart() {
        debugLog("BleOnProcessStart()");
        Class[] supportedProfileServices = ProfileConfig.getSupportedProfiles();
        for (int i=0; i < supportedProfileServices.length;i++) {
            String profileName = supportedProfileServices[i].getName();
            if (ProfileConfig.isProfileConfiguredEnabled(profileName)){
                if (isProfileStarted(profileName) == false) {
                    mProfileServicesState.put(profileName, BluetoothAdapter.STATE_OFF);
                }
            } else {
                Log.w(TAG,"processStart(): profile not enabled: "  + profileName);
            }
        }

        mRemoteDevices = new RemoteDevices(this);
        mAdapterProperties.init(mRemoteDevices);

        debugLog("BleOnProcessStart() - Make Bond State Machine");
        mBondStateMachine = BondStateMachine.make(this, mAdapterProperties, mRemoteDevices);

        mJniCallbacks.init(mBondStateMachine,mRemoteDevices);

        //FIXME: Set static instance here???
        setAdapterService(this);

        //Start Gatt service
        setGattProfileServiceState(supportedProfileServices,BluetoothAdapter.STATE_ON);
    }
```
上面的代码首先先查询当前系统支持的所有的Bluetooth Profile, 因为所有在AdapterApp的onCreate回去调用ProfileConfig.init(),而在init里面会调用checkAndAdjustDeviceModeConfiguration(),这个函数会根据当前系统处于何种模式而enable相应的Profile，那么在上面的代码中就可以知道每一种Profile当前是否start，如果没有启动需要更新mProfileServicesState里的状态。
之后会新建RemoteDevices和AdapterProperties对象，并且创建BondStateMachine状态机，之后start Gatt ProfileService，我们看下该代码：(关于Bluetooth 各个Profile会在其他文档里介绍。)
```java
    @SuppressWarnings("rawtypes")
    private void setGattProfileServiceState(Class[] services, int state) {
        if (state != BluetoothAdapter.STATE_ON && state != BluetoothAdapter.STATE_OFF) {
            Log.w(TAG,"setGattProfileServiceState(): invalid state...Leaving...");
            return;
        }

        int expectedCurrentState= BluetoothAdapter.STATE_OFF;
        int pendingState = BluetoothAdapter.STATE_TURNING_ON;

        if (state == BluetoothAdapter.STATE_OFF) {
            expectedCurrentState= BluetoothAdapter.STATE_ON;
            pendingState = BluetoothAdapter.STATE_TURNING_OFF;
        }

        for (int i=0; i <services.length;i++) {
            String serviceName = services[i].getName();
            String simpleName = services[i].getSimpleName();

            if (simpleName.equals("GattService")) {
                Integer serviceState = mProfileServicesState.get(serviceName);

                if(serviceState != null && serviceState != expectedCurrentState) {
                    debugLog("setProfileServiceState() - Unable to "
                        + (state == BluetoothAdapter.STATE_OFF ? "start" : "stop" )
                        + " service " + serviceName
                        + ". Invalid state: " + serviceState);
                        continue;
                }
                debugLog("setProfileServiceState() - "
                    + (state == BluetoothAdapter.STATE_OFF ? "Stopping" : "Starting")
                    + " service " + serviceName);

                mProfileServicesState.put(serviceName,pendingState);
                Intent intent = new Intent(this,services[i]);
                intent.putExtra(EXTRA_ACTION,ACTION_SERVICE_STATE_CHANGED);
                intent.putExtra(BluetoothAdapter.EXTRA_STATE,state);
                startService(intent);
                return;
            }
        }
    }


```
上面的代码主要就是去start GattService。然后当GattService起来之后会去通知AdapterService调用onProfileServiceStateChanged(),之后会给Handler发送MESSAGE_PROFILE_SERVICE_STATE_CHANGED消息，处理消息的时候就会给AdapterStateMachine发送BLE_STARTED消息，那么当AdapterStateMachine收到该消息之后进行如下处理：
```java
         case BLE_STARTED:
                      //Remove start timeout
                      removeMessages(BLE_START_TIMEOUT);

                      //Enable
                      if (!adapterService.enableNative()) {
                          errorLog("Error while turning Bluetooth on");
                          notifyAdapterStateChange(BluetoothAdapter.STATE_OFF);
                          transitionTo(mOffState);
                      } else {
                          sendMessageDelayed(ENABLE_TIMEOUT, ENABLE_TIMEOUT_DELAY);
                      }
                      break;

```
首先会将之前设置的BLE_START_TIMEOUT移除，并且调用JNI曾的enableNative()去使能蓝牙模块，并且设置ENABLE_TIMEOUT，这个和之前BLE_START_TIMEOUT一样。当stack将bluetooth成功enable起来变为STATE_ON的话就会网上回调stateChangeCallback(),这里判断状态是否是STATE_ON的，如果是就会发送ENABLED_READY消息，那么AdapterStateMachine此时处于PendingCommandState，处理该消息如下：
```java
                case ENABLED_READY:
                      removeMessages(ENABLE_TIMEOUT);
                      mPendingCommandState.setBleTurningOn(false);
                      transitionTo(mBleOnState);
                      notifyAdapterStateChange(BluetoothAdapter.STATE_BLE_ON);
                      break;

```
上面的代码同样会先移除ENABLE_TIMEOUT，然后将状态机转移到BleOnState