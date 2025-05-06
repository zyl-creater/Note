开机动画跑起来除了需要自身进程的启动外，还肯定以来显示系统的相关进程，即一定需要SurfaceFlinger的进程的合成和送显，所以这里需要启动SurfaceFlinger服务和bootanim服务，两者是在init.rc中启动。
## init服务声明

BootAnimation 的源码路径为: `frameworks/base/cmds/bootanimation`。
```shell
bootanimation/
├── Android.bp
├── audioplay.cpp
├── audioplay.h
├── BootAnimation.cpp
├── BootAnimation.h
├── bootanimation_main.cpp
├── BootAnimationUtil.cpp
├── BootAnimationUtil.h
├── bootanim.rc
└── FORMAT.md
```

我们看到有一个 bootanim.rc 文件，内容如下：
```rc
service bootanim /system/bin/bootanimation
    class core animation
    user graphics
    group graphics audio
    disabled
    oneshot
    ioprio rt 0
    task_profiles MaxPerformance
```
其中定义了一个 bootanim 服务，但这个服务是被 disable 的状态，也就是 init 解析 init.rc 的时候不会自动启动这个服务。

再看 Android.bp :
```bp
// bootanimation executable
// =========================================================

cc_binary {
    name: "bootanimation",
    defaults: ["bootanimation_defaults"],

    header_libs: ["jni_headers"],

    shared_libs: [
        "libOpenSLES",
        "libbootanimation",
    ],

    srcs: [
        "BootAnimationUtil.cpp",

        "bootanimation_main.cpp",
        "audioplay.cpp",
    ],

    init_rc: ["bootanim.rc"],

    cflags: [
        "-Wno-deprecated-declarations",
    ],
}
```
有个 `init_rc: ["bootanim.rc"]` 的配置。加了这个配置之后，在编译系统的时候会把 `bootanim.rc` 文件安装到 `/system/etc/init/` 目录。  
[[init进程]]在启动的时候会加载这个目录下的所有 rc 文件。

至此，我们已经定义好了 bootanim 服务，但是并没有自动启动。

**我们先想一下为什么 bootanim 服务要定义成 disable 的？为什么不直接在 init 阶段启动？**  
因为动画是需要显示屏幕上的。如果图像显示服务那个时候还没有初始化完，那 BootAnimation 就没法播放动画了。所以 bootanim 服务必须要等图形显示相关的服务初始化完了才启动。

在 Android 系统中，图像合成显示的服务是 [[SurfaceFlinger]]，源码路径为：
frameworks/native/services/surfaceflinger

我们来看一下surfaceflinger.rc
```rc
service surfaceflinger /system/bin/surfaceflinger
    class core animation
    user system
    group graphics drmrpc readproc
    capabilities SYS_NICE
    onrestart restart zygote
    task_profiles HighPerformance
    socket pdx/system/vr/display/client     stream 0666 system graphics u:object_r:pdx_display_client_endpoint_socket:s0
    socket pdx/system/vr/display/manager    stream 0666 system graphics u:object_r:pdx_display_manager_endpoint_socket:s0
    socket pdx/system/vr/display/vsync      stream 0666 system graphics u:object_r:pdx_display_vsync_endpoint_socket:s0
```
可以看到 surfaceflinger 没有被 disable，那就是系统启动的时候，由 init 进程自动启动的了。
## surfaceflinger服务的启动
动画播放依赖于 surfaceflinger 进程，所以它希望先启动，SF进程启动后会加载它的main方法
frameworks/native/services/surfaceflinger/main_surfaceflinger.cpp
```cpp
int main(int, char**) {
	...
    // instantiate surfaceflinger
    sp<SurfaceFlinger> flinger = surfaceflinger::createSurfaceFlinger();
	...
    // initialize before clients can connect
    flinger->init(); // 这里init会去设置开机动画相关的属性

    // publish surface flinger
    sp<IServiceManager> sm(defaultServiceManager());
    sm->addService(String16(SurfaceFlinger::getServiceName()), flinger, false,
                   IServiceManager::DUMP_FLAG_PRIORITY_CRITICAL | IServiceManager::DUMP_FLAG_PROTO);

    // publish gui::ISurfaceComposer, the new AIDL interface
    sp<SurfaceComposerAIDL> composerAIDL = new SurfaceComposerAIDL(flinger);
    sm->addService(String16("SurfaceFlingerAIDL"), composerAIDL, false,
                   IServiceManager::DUMP_FLAG_PRIORITY_CRITICAL | IServiceManager::DUMP_FLAG_PROTO);

    startDisplayService(); // dependency on SF getting registered above
	...
    // run surface flinger in this thread
    flinger->run();  //SurfaceFinger启动，并运行在主线程，通过消息机制循环等待任务

    return 0;
}
```

frameworks/native/services/surfaceflinger/SurfaceFlinger.cpp
```cpp
// Do not call property_set on main thread which will be blocked by init
// Use StartPropertySetThread instead.
void SurfaceFlinger::init() {
    ALOGI(  "SurfaceFlinger's main thread ready to run. "
            "Initializing graphics H/W...");
	...
    mStartPropertySetThread = getFactory().createStartPropertySetThread(presentFenceReliable);
	// 开启一个子现场去设置开机动画相关的属性（主线程设置属性会造成block）
    if (mStartPropertySetThread->Start() != NO_ERROR) {
        ALOGE("Run StartPropertySetThread failed!");
    }

    ALOGV("Done initializing");
}
```

frameworks/native/services/surfaceflinger/StartPropertySetThread.cpp
```cpp
bool StartPropertySetThread::threadLoop() {
    // Set property service.sf.present_timestamp, consumer need check its readiness
    property_set(kTimestampProperty, mTimestampPropertyValue ? "1" : "0");
    // Clear BootAnimation exit flag
    property_set("service.bootanim.exit", "0"); //重置开机动画退出属性
    property_set("service.bootanim.progress", "0"); // 开机动画进度
    // Start BootAnimation if not started
    property_set("ctl.start", "bootanim"); //启动开机动画的属性，这里保险起见还是会去启动bootanim服务
    // Exit immediately
    return false;
}
```
这里主要是做了两个事：
1. 重置开机动画的退出属性，开机动画进程会循环check这个属性，如果为1就结束播放并退出，这里先初始化为0
2. 设置开机动画启动属性，如果开机动画进程没有被前面的init进程拉起来，那么这里还会再次主动让属性服务拉起开机动画进程。
3. 设置`ctl.start=bootanim` 属性，即让 init进程启动 bootanim 服务。

[[init进程#^d0f6f4|设置 ctl.start 属性值为 bootanim，怎么就能启动 bootanim 呢？]]
详细分析请参考 [[init进程]] ^a92e6e

## BootAnimation 播放流程

BootAnimation 的执行入口为 bootanimation_main.cpp 的 main 函数 ^e2af50
```cpp
int main()
{
    setpriority(PRIO_PROCESS, 0, ANDROID_PRIORITY_DISPLAY);
	//是否支持开机动画，　由 ro.boot.quiescent 这个属性决定。
    bool noBootAnimation = bootAnimationDisabled();
    ALOGI_IF(noBootAnimation,  "boot animation disabled");
    if (!noBootAnimation) {

        sp<ProcessState> proc(ProcessState::self());
        ProcessState::self()->startThreadPool();

        // create the boot animation object (may take up to 200ms for 2MB zip)
        sp<BootAnimation> boot = new BootAnimation(audioplay::createAnimationCallbacks());//创建动画对象，这个过程会去解析的动画文件，文件越大越耗时
		//等待SurfaceFlinger初始化完成并加入到 ServiceManager。  
        //因为 bootanim 服务是在 SurfaceFlinger.init() 里面启动的，但在 SurfaceFlinger 服务是在执行完 SurfaceFlinger.init() 之后才加入到 ServiceManager 的，所以这里需要等一下。
        waitForSurfaceFlinger();//向servicemanager检查SF进程是否正常启动了, 死循环等待SF注册成功
		//运行动画线程，开始播放动画。
        boot->run("BootAnimation", PRIORITY_DISPLAY);
        ALOGV("Boot animation set up. Joining pool.");
        IPCThreadState::self()->joinThreadPool();
    }
    return 0;
}
```
主要作用是：
1. 检查是否禁用了开机动画，如果禁用了进程直接结束，不播放Android动画（也许厂商自己用其他方式实现）
2. 创建BootAnimation对象，加载并解析动画文件，加载时间根据文件大小有关（图片大小，数量）
3. 循环等待SurfaceFlinger服务的启动（没有SF也播放不了动画）
4. 启动动画线程(BootAnimation继承于Thread类，它本身就是一个thread)，开始播放动画

### 检查是否禁用了开机动画
BootAnimationUtil.cpp
```cpp
bool bootAnimationDisabled() {
    char value[PROPERTY_VALUE_MAX];
    property_get("debug.sf.nobootanimation", value, "0");
    if (atoi(value) > 0) {
        return true;
    }

    property_get("ro.boot.quiescent", value, "0");
    if (atoi(value) > 0) {
        // Only show the bootanimation for quiescent boots if this system property is set to enabled
        if (!property_get_bool("ro.bootanim.quiescent.enabled", false)) {
            return true;
        }
    }

    return false;
}
```
主要受三个属性控制，三个属性优先级有前后。

### 创建BootAnimation对象，预加载动画文件
BootAnimation.cpp
```cpp
BootAnimation::BootAnimation(sp<Callbacks> callbacks)
        : Thread(false), mLooper(new Looper(false)), mClockEnabled(true), mTimeIsAccurate(false),
        mTimeFormat12Hour(false), mTimeCheckThread(nullptr), mCallbacks(callbacks) {
    mSession = new SurfaceComposerClient();

    std::string powerCtl = android::base::GetProperty("sys.powerctl", "");
    if (powerCtl.empty()) {
        mShuttingDown = false;
    } else {
        mShuttingDown = true;
    }
    ALOGD("%sAnimationStartTiming start time: %" PRId64 "ms", mShuttingDown ? "Shutdown" : "Boot",
            elapsedRealtime());
}
```
上面的构造方法中仅仅中只是获取了SF的session代理对象, 真正的加载逻辑在这个对象的第一次引用回调方法中（在前面的sp指针实例化时被回调）。

BootAnimation 继承了 Thread, Thread 又继承了 RefBase , RefBase 有一个函数 onFirstRef()，这个函数在对象第一次被引用的时候会调用，这是 RefBase 引用计数的一个机制。 BootAnimation 类重写了这个函数。因此在执行 `sp<BootAnimation> boot = new BootAnimation(audioplay::createAnimationCallbacks());` 这行的时候， BootAnimation.onFirstRef() 会被自动调用。
```cpp
void BootAnimation::onFirstRef() {
    status_t err = mSession->linkToComposerDeath(this);
    SLOGE_IF(err, "linkToComposerDeath failed (%s) ", strerror(-err));
    if (err == NO_ERROR) {
        // Load the animation content -- this can be slow (eg 200ms)
        // called before waitForSurfaceFlinger() in main() to avoid wait
        ALOGD("%sAnimationPreloadTiming start time: %" PRId64 "ms",
                mShuttingDown ? "Shutdown" : "Boot", elapsedRealtime());
        preloadAnimation();
        ALOGD("%sAnimationPreloadStopTiming start time: %" PRId64 "ms",
                mShuttingDown ? "Shutdown" : "Boot", elapsedRealtime());
    }
}
```
加载的逻辑主要是preloadAnimation()，这个过程主要是解析bootanimation.zip文件
```cpp
bool BootAnimation::preloadAnimation() {
    findBootAnimationFile();// 检索系统的几个预设定路径下是否存在bootanimation.zip文件
    if (!mZipFileName.isEmpty()) {
        mAnimation = loadAnimation(mZipFileName);// 加载动画文件
        return (mAnimation != nullptr);
    }

    return false;
}
```
#### findBootAnimationFile方法
是去检索系统的几个预设定路径下是否存在bootanimation.zip文件, 如果存在赋值给mZipFileName, 几个预设定路径是：（寻找是根据系统是否加密、是否是深色主题进行选择。）
```cpp
static const char OEM_BOOTANIMATION_FILE[] = "/oem/media/bootanimation.zip";
static const char PRODUCT_BOOTANIMATION_DARK_FILE[] = "/product/media/bootanimation-dark.zip";
static const char PRODUCT_BOOTANIMATION_FILE[] = "/product/media/bootanimation.zip";
static const char SYSTEM_BOOTANIMATION_FILE[] = "/system/media/bootanimation.zip";
static const char APEX_BOOTANIMATION_FILE[] = "/apex/com.android.bootanimation/etc/bootanimation.zip";
static const char PRODUCT_ENCRYPTED_BOOTANIMATION_FILE[] = "/product/media/bootanimation-encrypted.zip";
static const char SYSTEM_ENCRYPTED_BOOTANIMATION_FILE[] = "/system/media/bootanimation-encrypted.zip";

void BootAnimation::findBootAnimationFile() {
    // If the device has encryption turned on or is in process
    // of being encrypted we show the encrypted boot animation.
    char decrypt[PROPERTY_VALUE_MAX];
    property_get("vold.decrypt", decrypt, "");

    bool encryptedAnimation = atoi(decrypt) != 0 ||
        !strcmp("trigger_restart_min_framework", decrypt);

    if (!mShuttingDown && encryptedAnimation) {
        static const std::vector<std::string> encryptedBootFiles = {
            PRODUCT_ENCRYPTED_BOOTANIMATION_FILE, SYSTEM_ENCRYPTED_BOOTANIMATION_FILE,
        };
        if (findBootAnimationFileInternal(encryptedBootFiles)) {
            return;
        }
    }

    const bool playDarkAnim = android::base::GetIntProperty("ro.boot.theme", 0) == 1;
    static const std::vector<std::string> bootFiles = {
        APEX_BOOTANIMATION_FILE, playDarkAnim ? PRODUCT_BOOTANIMATION_DARK_FILE : PRODUCT_BOOTANIMATION_FILE,
        OEM_BOOTANIMATION_FILE, SYSTEM_BOOTANIMATION_FILE
    };
    static const std::vector<std::string> shutdownFiles = {
        PRODUCT_SHUTDOWNANIMATION_FILE, OEM_SHUTDOWNANIMATION_FILE, SYSTEM_SHUTDOWNANIMATION_FILE, ""
    };
    static const std::vector<std::string> userspaceRebootFiles = {
        PRODUCT_USERSPACE_REBOOT_ANIMATION_FILE, OEM_USERSPACE_REBOOT_ANIMATION_FILE,
        SYSTEM_USERSPACE_REBOOT_ANIMATION_FILE,
    };

    if (android::base::GetBoolProperty("sys.init.userspace_reboot.in_progress", false)) {
        findBootAnimationFileInternal(userspaceRebootFiles);
    } else if (mShuttingDown) {
        findBootAnimationFileInternal(shutdownFiles);
    } else {
        findBootAnimationFileInternal(bootFiles);
    }
}
```
#### loadAnimation方法
是去解析`bootanimation.zip`文件
```cpp
BootAnimation::Animation* BootAnimation::loadAnimation(const String8& fn) {
    if (mLoadedFiles.indexOf(fn) >= 0) {
        SLOGE("File \"%s\" is already loaded. Cyclic ref is not allowed",
            fn.string());
        return nullptr;
    }
    ZipFileRO *zip = ZipFileRO::open(fn);//打开zip文件
    if (zip == nullptr) {
        SLOGE("Failed to open animation zip \"%s\": %s",
            fn.string(), strerror(errno));
        return nullptr;
    }

    Animation *animation =  new Animation;
    animation->fileName = fn;
    animation->zip = zip;
    animation->clockFont.map = nullptr;
    mLoadedFiles.add(animation->fileName);//整个zip也用一个Animation来描述

    parseAnimationDesc(*animation);//解析‘desc.txt’文件，根据这个文件创建每个part的Animation对象
    if (!preloadZip(*animation)) {//加载每一个part对应的图片，并填充到animation.part.frames
        releaseAnimation(animation);
        return nullptr;
    }

    mLoadedFiles.remove(fn);
    return animation;
}
```
解析`desc.txt`文件
```cpp
bool BootAnimation::parseAnimationDesc(Animation& animation)  {
	...
    // Parse the description file
    for (;;) {
        const char* endl = strstr(s, "\n");
        if (endl == nullptr) break;
        String8 line(s, endl - s);
        const char* l = line.string();
        int fps = 0;
        int width = 0;
        int height = 0;
        int count = 0;
        int pause = 0;
        int progress = 0;
        int framesToFadeCount = 0;
        int colorTransitionStart = 0;
        int colorTransitionEnd = 0;
        char path[ANIM_ENTRY_NAME_MAX];
        char color[7] = "000000"; // default to black if unspecified
        char clockPos1[TEXT_POS_LEN_MAX + 1] = "";
        char clockPos2[TEXT_POS_LEN_MAX + 1] = "";
        char dynamicColoringPartNameBuffer[ANIM_ENTRY_NAME_MAX];
        char pathType;
        // start colors default to black if unspecified
        char start_color_0[7] = "000000";
        char start_color_1[7] = "000000";
        char start_color_2[7] = "000000";
        char start_color_3[7] = "000000";

        int nextReadPos;

        int topLineNumbers = sscanf(l, "%d %d %d %d", &width, &height, &fps, &progress);
        if (topLineNumbers == 3 || topLineNumbers == 4) { //解析第一行，获取width/height/fps参数
            // SLOGD("> w=%d, h=%d, fps=%d, progress=%d", width, height, fps, progress);
            animation.width = width;
            animation.height = height;
            animation.fps = fps;
            if (topLineNumbers == 4) {  //如果配置了progress，一般不会配置
              animation.progressEnabled = (progress != 0);
            } else {
              animation.progressEnabled = false;
            }
        } else if (sscanf(l, "dynamic_colors %" STRTO(ANIM_PATH_MAX) "s #%6s #%6s #%6s #%6s %d %d",
            dynamicColoringPartNameBuffer,加载每一个part对应的图片，并填充到animation.part.frames
            start_color_0, start_color_1, start_color_2, start_color_3,
            &colorTransitionStart, &colorTransitionEnd)) {  //动态颜色，我们一般不配置，所以不会走这里
            animation.dynamicColoringEnabled = true;
            parseColor(start_color_0, animation.startColors[0]);
            parseColor(start_color_1, animation.startColors[1]);
            parseColor(start_color_2, animation.startColors[2]);
            parseColor(start_color_3, animation.startColors[3]);
            animation.colorTransitionStart = colorTransitionStart;
            animation.colorTransitionEnd = colorTransitionEnd;
            dynamicColoringPartName = std::string(dynamicColoringPartNameBuffer);
        } else if (sscanf(l, "%c %d %d %" STRTO(ANIM_PATH_MAX) "s%n",
                          &pathType, &count, &pause, path, &nextReadPos) >= 4) {  //解析第二行开始的part部分
            if (pathType == 'f') {
                sscanf(l + nextReadPos, " %d #%6s %16s %16s", &framesToFadeCount, color, clockPos1,
                       clockPos2);
            } else {
                sscanf(l + nextReadPos, " #%6s %16s %16s", color, clockPos1, clockPos2);
            }
            // SLOGD("> type=%c, count=%d, pause=%d, path=%s, framesToFadeCount=%d, color=%s, "
            //       "clockPos1=%s, clockPos2=%s",
            //       pathType, count, pause, path, framesToFadeCount, color, clockPos1, clockPos2);
            Animation::Part part; 
            if (path == dynamicColoringPartName) {
                // Part is specified to use dynamic coloring.
                part.useDynamicColoring = true;
                part.postDynamicColoring = false;
                postDynamicColoring = true;
            } else {   //我们一般只配置pathType、count、pause，其他的都是空
                // Part does not use dynamic coloring.
                part.useDynamicColoring = false;
                part.postDynamicColoring =  postDynamicColoring;
            }
            part.playUntilComplete = pathType == 'c';
            part.framesToFadeCount = framesToFadeCount;
            part.count = count;
            part.pause = pause;加载
            part.path = path;
            part.audioData = nullptr;  //新版本还加入了开机动画每个part可以添加对应的音频文件，但是一般不配置，且不是在这里加载
            part.animation = nullptr;  //这里每个part下面一般不会再嵌套动画文件了，所以它已经是节点了
            if (!parseColor(color, part.backgroundColor)) {  //color和backgroundcolor没有特殊配置使用默认的black
                SLOGE("> invalid color '#%s'", color);
                part.backgroundColor[0] = 0.0f;
                part.backgroundColor[1] = 0.0f;
                part.backgroundColor[2] = 0.0f;
            }
            parsePosition(clockPos1, clockPos2, &part.clockPosX, &part.clockPosY);
            animation.parts.add(part);  //将part加入animation对象中，这样就能拿到他的fps/type/count/pause, 但是frames数据还没有填充
        }
        else if (strcmp(l, "$SYSTEM") == 0) {  //一种特殊情况，不会走这里
            // SLOGD("> SYSTEM");
            Animation::Part part;
            part.playUntilComplete = false;
            part.framesToFadeCount = 0;
            part.count = 1;
            part.pause = 0;
            part.audioData = nullptr;
            part.animation = loadAnimation(String8(SYSTEM_BOOTANIMATION_FILE));
            if (part.animation != nullptr)
                animation.parts.add(part);
        }
        s = ++endl;
    }

    return true;
}
```
解析完`desc.txt`，已经将配置文件中每个part对应到每个实际的part文件夹，但是真正的图片，还没有去加载，接下来`preloadZip`加载每一个part对应的图片，并填充到`animation.part.frames`
```cpp
bool BootAnimation::preloadZip(Animation& animation) {
    // read all the data structures
    const size_t pcount = animation.parts.size();
    void *cookie = nullptr;
    ZipFileRO* zip = animation.zip;
    if (!zip->startIteration(&cookie)) {
        return false;
    }

    ZipEntryRO entry;
    char name[ANIM_ENTRY_NAME_MAX];
    while ((entry = zip->nextEntry(cookie)) != nullptr) {  //遍历整个zip下的文件树，一般我们都不会嵌套多层，叶子节点就是每个part文件夹下的文件
        const int foundEntryName = zip->getEntryFileName(entry, name, ANIM_ENTRY_NAME_MAX);
        if (foundEntryName > ANIM_ENTRY_NAME_MAX || foundEntryName == -1) {
            SLOGE("Error fetching entry file name");
            continue;
        }

        const String8 entryName(name);
        const String8 path(entryName.getPathDir());
        const String8 leaf(entryName.getPathLeaf());
        if (leaf.size() > 0) {
			...
            for (size_t j = 0; j < pcount; j++) {  //遍历每个part
                if (path == animation.parts[j].path) {
                    uint16_t method;
                    // supports only stored png files
                    if (zip->getEntryInfo(entry, &method, nullptr, nullptr, nullptr, nullptr, nullptr)) {
                        if (method == ZipFileRO::kCompressStored) {  //从直接可以知道bootanimation.zip必须使用存储方式打包，不能压缩，否则将不能播放
                            FileMap* map = zip->createEntryFileMap(entry);
                            if (map) {
                                Animation::Part& part(animation.parts.editItemAt(j));
                                if (leaf == "audio.wav") { // 如果是音频文件
                                    // a part may have at most one audio file
                                    part.audioData = (uint8_t *)map->getDataPtr();
                                    part.audioLength = map->getDataLength();
                                } else if (leaf == "trim.txt") {  //一般不会使用trim
                                    part.trimData.setTo((char const*)map->getDataPtr(),
                                                        map->getDataLength());
                                } else {
                                    Animation::Frame frame;
                                    frame.name = leaf;
                                    frame.map = map;
                                    frame.trimWidth = animation.width;
                                    frame.trimHeight = animation.height;
                                    frame.trimX = 0;
                                    frame.trimY = 0;
                                    part.frames.add(frame); // 将part下面的图片添加到part的frames
                                }
                            }
                        } else {
                            SLOGE("bootanimation.zip is compressed; must be only stored");
                        }
                    }
                }
            }
        }
    }
	...
    zip->endIteration(cookie);

    return true;
}
```
到这里动画文件的预加载就完成了，这时候需要check SF是否已经正常启动。
#### 等待SF服务的启动
frameworks/base/cmds/bootanimation/BootAnimationUtil.cpp
```cpp
void waitForSurfaceFlinger() {
    // TODO: replace this with better waiting logic in future, b/35253872
    int64_t waitStartTime = elapsedRealtime();
    sp<IServiceManager> sm = defaultServiceManager();
    const String16 name("SurfaceFlinger");
    const int SERVICE_WAIT_SLEEP_MS = 100;
    const int LOG_PER_RETRIES = 10;
    int retry = 0;
    while (sm->checkService(name) == nullptr) {
        retry++;
        if ((retry % LOG_PER_RETRIES) == 0) {
            ALOGW("Waiting for SurfaceFlinger, waited for %" PRId64 " ms",
                  elapsedRealtime() - waitStartTime);
        }
        usleep(SERVICE_WAIT_SLEEP_MS * 1000);
    };
    int64_t totalWaited = elapsedRealtime() - waitStartTime;
    if (totalWaited > SERVICE_WAIT_SLEEP_MS) {
        ALOGI("Waiting for SurfaceFlinger took %" PRId64 " ms", totalWaited);
    }
}
```
向ServiceMananger询问SF是否注册，如果没注册死循环，每秒问一次，启动了就开始播放动画

### 启动播放线程，播放动画

前面的 [[#^e2af50|main]] 方法的`boot->run("BootAnimation", PRIORITY_DISPLAY);`
会先走BootAnimation线程的`readyToRun`, 然后才执行真正的线程循环体`threadLoop`，先看下`readyToRun`


```cpp
status_t BootAnimation::readyToRun() {
	mAssets.addDefaultAssets();
	
    mDisplayToken = SurfaceComposerClient::getInternalDisplayToken();  //获取DisplayToken， 用于获取Display(屏幕)的参数
    if (mDisplayToken == nullptr)
        return NAME_NOT_FOUND;

    DisplayMode displayMode;
    const status_t error =
            SurfaceComposerClient::getActiveDisplayMode(mDisplayToken, &displayMode);  //获取DisplayMode，用于获取分辨率等信息
	if (error != NO_ERROR)
        return error;

    mMaxWidth = android::base::GetIntProperty("ro.surface_flinger.max_graphics_width", 0);
    mMaxHeight = android::base::GetIntProperty("ro.surface_flinger.max_graphics_height", 0);
    ui::Size resolution = displayMode.resolution;
    resolution = limitSurfaceSize(resolution.width, resolution.height);
    // create the native surface
    sp<SurfaceControl> control = session()->createSurface(String8("BootAnimation"),
            resolution.getWidth(), resolution.getHeight(), PIXEL_FORMAT_RGB_565);  //创建Surface并返回一个surfacecontrol对象，动画需要他通过opengl绘制到surface上

    SurfaceComposerClient::Transaction t;  //创建与SF通信的事务

	// this guest property specifies multi-display IDs to show the boot animation
    // multiple ids can be set with comma (,) as separator, for example:
    // setprop persist.boot.animation.displays 19260422155234049,19261083906282754
    Vector<PhysicalDisplayId> physicalDisplayIds;
    char displayValue[PROPERTY_VALUE_MAX] = "";
    property_get(DISPLAYS_PROP_NAME, displayValue, "");
    bool isValid = displayValue[0] != '\0';
    if (isValid) {
        char *p = displayValue;
        while (*p) {
            if (!isdigit(*p) && *p != ',') {
                isValid = false;
                break;
            }
            p ++;
        }
        if (!isValid)
            SLOGE("Invalid syntax for the value of system prop: %s", DISPLAYS_PROP_NAME);
    }
    if (isValid) {
        std::istringstream stream(displayValue);
        for (PhysicalDisplayId id; stream >> id.value; ) {
            physicalDisplayIds.add(id);
            if (stream.peek() == ',')
                stream.ignore();
        }

        // In the case of multi-display, boot animation shows on the specified displays
        // in addition to the primary display
        auto ids = SurfaceComposerClient::getPhysicalDisplayIds();
        constexpr uint32_t LAYER_STACK = 0;
        for (auto id : physicalDisplayIds) {
            if (std::find(ids.begin(), ids.end(), id) != ids.end()) {
                sp<IBinder> token = SurfaceComposerClient::getPhysicalDisplayToken(id);
                if (token != nullptr)
                    t.setDisplayLayerStack(token, LAYER_STACK);
            }
        }
        t.setLayerStack(control, LAYER_STACK);
    }

    t.setLayer(control, 0x40000000) //将Layer和SurfaceControl绑定
        .apply(); //提交事务

    sp<Surface> s = control->getSurface(); //获取一个surface

	//初始化opengl and egl
    // initialize opengl and egl
    EGLDisplay display = eglGetDisplay(EGL_DEFAULT_DISPLAY);
    eglInitialize(display, nullptr, nullptr);
    EGLConfig config = getEglConfig(display);
    EGLSurface surface = eglCreateWindowSurface(display, config, s.get(), nullptr);
    // Initialize egl context with client version number 2.0.
    EGLint contextAttributes[] = {EGL_CONTEXT_CLIENT_VERSION, 2, EGL_NONE};
    EGLContext context = eglCreateContext(display, config, nullptr, contextAttributes);
    EGLint w, h;
    eglQuerySurface(display, surface, EGL_WIDTH, &w);
    eglQuerySurface(display, surface, EGL_HEIGHT, &h);

    if (eglMakeCurrent(display, surface, surface, context) == EGL_FALSE)
        return NO_INIT;

    mDisplay = display;
    mContext = context;
    mSurface = surface;
    mInitWidth = mWidth = w;
    mInitHeight = mHeight = h;
    mFlingerSurfaceControl = control;
    mFlingerSurface = s;
    mTargetInset = -1;

    projectSceneToWindow(); // 裁剪窗口、转化坐标

    // Register a display event receiver
    mDisplayEventReceiver = std::make_unique<DisplayEventReceiver>();
    status_t status = mDisplayEventReceiver->initCheck();
    SLOGE_IF(status != NO_ERROR, "Initialization of DisplayEventReceiver failed with status: %d",
            status);
    mLooper->addFd(mDisplayEventReceiver->getFd(), 0, Looper::EVENT_INPUT,
            new DisplayEventCallback(this), nullptr);

    /*resync display mode after bind event handle, for displaymode may changed*/
    const status_t mode_error = SurfaceComposerClient::getActiveDisplayMode(
        mDisplayToken, &displayMode);
    if (mode_error != NO_ERROR) {
        SLOGE("Can't get active display mode for RESIZE");
    } else {
        resizeSurface(displayMode.resolution.getWidth(),
            displayMode.resolution.getHeight());
    }

    return NO_ERROR;
}
```

![[Pasted image 20240902155820.png]]