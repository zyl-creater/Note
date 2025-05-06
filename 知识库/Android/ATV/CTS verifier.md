cts verifier测试方法更新，以后测试时安装apk的时候请执行一下6条命令，不要只是adb install：  
1、adb shell settings put global hidden_api_policy 1  
2、adb install -r -g CtsVerifier-14_r4.apk  
3、adb shell appops set com.android.cts.verifier android:read_device_identifiers allow  
4、adb shell appops set com.android.cts.verifier MANAGE_EXTERNAL_STORAGE 0  
5、adb shell am compat enable ALLOW_TEST_API_ACCESS com.android.cts.verifier  
6、adb shell appops set com.android.cts.verifier TURN_SCREEN_ON 0  
其中：  
1）android 10 是1-3条  
2）android 11 是1-4条  
3）android 12 是1-5条  
4）android 14 是1-6条