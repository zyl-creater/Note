```cpp
// \system\core\init\init.cpp main()
/* 
* 1.C++中主函数有两个参数，第一个参数argc表示参数个数，第二个参数是参数列表，也就是具体 的参数 
* 2.init的main函数有两个其它入口，一是参数中有ueventd，进入ueventd_main,二是参数中 有watchdogd，进入watchdogd_main 
*/ 
int main(int argc, char** argv) {
    /* 
    * 1.strcmp是String的一个函数，比较字符串，相等返回0 
    * 2.C++中0也可以表示false 
    * 3.basename是C库中的一个函数，得到特定的路径中的最后一个'/'后面的内容， 
    * 比如/sdcard/miui_recovery/backup，得到的结果是backup 
    */
    if (!strcmp(basename(argv[0]), "ueventd")) {
        //当argv[0]的内容为ueventd 时，strcmp的值为0,！strcmp为1 
		//1表示true，也就执行ueventd_main,ueventd主要是负责设备节点的创建、权限设定等一系列工作
        return ueventd_main(argc, argv);
    }

    if (!strcmp(basename(argv[0]), "watchdogd")) {
        //watchdogd俗称看门狗，用于 系统出问题时重启系统
        return watchdogd_main(argc, argv);
    }

    if (REBOOT_BOOTLOADER_ON_PANIC) {
        //初始化重启系统的处理信号，内部通过sigaction 注册信号，当监听到该信号时重启系统
        install_reboot_signal_handlers();
    }

    add_environment("PATH", _PATH_DEFPATH);

    // 查看是否有环境变量INIT_SECOND_STAGE
    bool is_first_stage = (getenv("INIT_SECOND_STAGE") == nullptr);

    // 1.init的main方法会执行两次，由is_first_stage控制,first_stage就是第一阶段要做的事
    if (is_first_stage) {
        boot_clock::time_point start_time = boot_clock::now();

        // Clear the umask.
        //清空文件权限
        umask(0);

        // Get the basic filesystem setup we need put together in the initramdisk
        // on / and then we'll let the rc file figure out the rest.
        //mount是用来挂载文件系统的，mount属于Linux系统调用
        mount("tmpfs", "/dev", "tmpfs", MS_NOSUID, "mode=0755");
        mkdir("/dev/pts", 0755);;//创建目录，第一个参数是目录路径，第二个是读写权限
        mkdir("/dev/socket", 0755);
        mount("devpts", "/dev/pts", "devpts", 0, NULL);
        #define MAKE_STR(x) __STRING(x)
        mount("proc", "/proc", "proc", 0, "hidepid=2,gid=" MAKE_STR(AID_READPROC));
        // Don't expose the raw commandline to unprivileged processes.
        chmod("/proc/cmdline", 0440);//用于修改文件/目录的读写权限
        gid_t groups[] = { AID_READPROC };
        // 用来将list数组中所标明的组加入到目前进程的组设置中
        setgroups(arraysize(groups), groups);
        mount("sysfs", "/sys", "sysfs", 0, NULL);
        mount("selinuxfs", "/sys/fs/selinux", "selinuxfs", 0, NULL);
        //mknod用于创建Linux中的设备文件
        mknod("/dev/kmsg", S_IFCHR | 0600, makedev(1, 11));
        mknod("/dev/random", S_IFCHR | 0666, makedev(1, 8));
        mknod("/dev/urandom", S_IFCHR | 0666, makedev(1, 9));

        // Now that tmpfs is mounted on /dev and we have /dev/kmsg, we can actually
        // talk to the outside world...
        //将标准输入输出重定向到"/sys/fs/selinux/null"
        InitKernelLogging(argv);

        LOG(INFO) << "init first stage started!";

        if (!DoFirstStageMount()) {
            LOG(ERROR) << "Failed to mount required partitions early ...";
            panic();
        }

        //Avb即Android Verfied boot,功能包括Secure Boot, verfying boot 和dm-verity, 
		//原理都是对二进制文件进行签名，在系统启动时进行认证，确保系统运行的是合法的二进制镜像文件。 
		//其中认证的范围涵盖：bootloader，boot.img，system.img
        // 通过调用FsManagerAvbHandle::Open()->FsManagerAvbOps::AvbSlotVerify->avb_slot_verify最终去验证每个分区。
        // 当验证成功之后，会将版本信息写到环境变量 INIT_AVB_VERSION 中
        SetInitAvbVersionInRecovery();

        // Set up SELinux, loading the SELinux policy.
        //加载SELinux policy，也就是安全策略，
        selinux_initialize(true);

        // We're in the kernel domain, so re-exec init to transition to the init domain now
        // that the SELinux policy has been loaded.
        // 我们执行第一遍init的main方法是在kernel domain，所以要重新执行init文件，切换到加载了selinux策略的init domain，
        if (restorecon("/init") == -1) {
            PLOG(ERROR) << "restorecon failed";
            security_failure();
        }

        // 设置进入第二阶段标志位，进入第二阶段
        setenv("INIT_SECOND_STAGE", "true", 1);

        static constexpr uint32_t kNanosecondsPerMillisecond = 1e6;
        uint64_t start_ms = start_time.time_since_epoch().count() / kNanosecondsPerMillisecond;
        setenv("INIT_STARTED_AT", StringPrintf("%" PRIu64, start_ms).c_str(), 1);

        char* path = argv[0];
        char* args[] = { path, nullptr };
        // 重新执行init,由于标志位置为了，因此再次执行init会进入阶段。
        execv(path, args);

        // execv() only returns if an error happened, in which case we
        // panic and never fall through this conditional.
        PLOG(ERROR) << "execv(\"" << path << "\") failed";
        security_failure();
    }

    // At this point we're in the second stage of init.
    InitKernelLogging(argv);
    LOG(INFO) << "init second stage started!";

    // Set up a session keyring that all processes will have access to. It
    // will hold things like FBE encryption keys. No process should override
    // its session keyring.
    keyctl(KEYCTL_GET_KEYRING_ID, KEY_SPEC_SESSION_KEYRING, 1);

    // Indicate that booting is in progress to background fw loaders, etc.
    close(open("/dev/.booting", O_WRONLY | O_CREAT | O_CLOEXEC, 0000));

    //初始化属性系统，并从指定文件读取属性
    // bionic/libc/bionic/system_properties.c
    // __system_property_area_init->map_prop_area_rw->打开/dev/__properties__文件->并且映射128kb空间大小内存来存属性键值对。
    property_init();

    // If arguments are passed both on the command line and in DT,
    // properties set in DT always have priority over the command-line ones.
    //接下来的一系列操作都是从各个文件读取一些属性，然后通过property_set设置系统属性
    // 1.这句英文的大概意思是，如果参数同时从命令行和DT传过来，DT的优先级总是大于命令行的。
    // 2.DT即device-tree，中文意思是设备树，这里面记录自己的硬件配置和系统运行参数，参考http://www.wowotech.net/linux_kenrel/why-dt.html
    process_kernel_dt();//处理DT属性
    process_kernel_cmdline();//处理命令行属性

    // Propagate the kernel variables to internal variables
    // used by init as well as the current required properties.
    export_kernel_boot_props();//处理其他的一些属性

    // Make the time that init started available for bootstat to log.
    property_set("ro.boottime.init", getenv("INIT_STARTED_AT"));
    property_set("ro.boottime.init.selinux", getenv("INIT_SELINUX_TOOK"));

    // Set libavb version for Framework-only OTA match in Treble build.
    const char* avb_version = getenv("INIT_AVB_VERSION");
    // 将avb版本设置到系统属性中
    if (avb_version) property_set("ro.boot.avb_version", avb_version);

    // Clean up our environment.
    // 清除环境变量
    unsetenv("INIT_SECOND_STAGE");
    unsetenv("INIT_STARTED_AT");
    unsetenv("INIT_SELINUX_TOOK");
    unsetenv("INIT_AVB_VERSION");

    // Now set up SELinux for second stage.
    selinux_initialize(false);
    selinux_restore_context();

    //创建epoll实例，并返回epoll的文件描述符
    epoll_fd = epoll_create1(EPOLL_CLOEXEC);
    if (epoll_fd == -1) {
        PLOG(ERROR) << "epoll_create1 failed";
        exit(1);
    }

    // 主要是创建handler处理子进程终止信号，创建一个匿名 socket并注册到epoll进行监听
    signal_handler_init();

    // 从文件中加载一些属性，读取usb配置
    // 涉及到的文件，/system/etc/prop.default，/odm/default.prop，/vendor/default.prop
    property_load_boot_defaults();
    export_oem_lock_status();// 设置ro.boot.flash.locked 属性
    start_property_service();//开启一个socket监听系统属性的设置
    set_usb_controller();//设置sys.usb.controller 属性

    //init.rc文件中方法映射,例如“class_start”-> "do_class_start"
    const BuiltinFunctionMap function_map;
    Action::set_function_map(&function_map);//将function_map存放到Action中作 为成员属性

    // 使用不同的parse解析init.rc文件中的不同字段，并且将service保存在servicelist中
    Parser& parser = Parser::GetInstance();
    parser.AddSectionParser("service",std::make_unique<ServiceParser>());
    parser.AddSectionParser("on", std::make_unique<ActionParser>());
    parser.AddSectionParser("import", std::make_unique<ImportParser>());
    std::string bootscript = GetProperty("ro.boot.init_rc", "");
    if (bootscript.empty()) {
        // 以下目录的所有rc文件默认都会被解析
        parser.ParseConfig("/init.rc");
        parser.set_is_system_etc_init_loaded(
                parser.ParseConfig("/system/etc/init"));
        parser.set_is_vendor_etc_init_loaded(
                parser.ParseConfig("/vendor/etc/init"));
        parser.set_is_odm_etc_init_loaded(parser.ParseConfig("/odm/etc/init"));
    } else {
        parser.ParseConfig(bootscript);
        parser.set_is_system_etc_init_loaded(true);
        parser.set_is_vendor_etc_init_loaded(true);
        parser.set_is_odm_etc_init_loaded(true);
    }

    // Turning this on and letting the INFO logging be discarded adds 0.2s to
    // Nexus 9 boot time, so it's disabled by default.
    if (false) parser.DumpState();//打印一些当前Parser的信息，默认是不执行的

    ActionManager& am = ActionManager::GetInstance();

    // QueueEventTrigger用于触发Action,这里触发early-init事件
    am.QueueEventTrigger("early-init");

    // Queue an action that waits for coldboot done so we know ueventd has set up all of /dev...
    //QueueBuiltinAction用于添加Action，第一个参数是 Action要执行的Command,第二个是Trigger
    am.QueueBuiltinAction(wait_for_coldboot_done_action, "wait_for_coldboot_done");
    // ... so that we can start queuing up actions that require stuff from /dev.
    am.QueueBuiltinAction(mix_hwrng_into_linux_rng_action, "mix_hwrng_into_linux_rng");
    am.QueueBuiltinAction(set_mmap_rnd_bits_action, "set_mmap_rnd_bits");
    am.QueueBuiltinAction(set_kptr_restrict_action, "set_kptr_restrict");
    am.QueueBuiltinAction(keychord_init_action, "keychord_init");
    am.QueueBuiltinAction(console_init_action, "console_init");

    // Trigger all the boot actions to get us started.
    am.QueueEventTrigger("init");

    // Repeat mix_hwrng_into_linux_rng in case /dev/hw_random or /dev/random
    // wasn't ready immediately after wait_for_coldboot_done
    am.QueueBuiltinAction(mix_hwrng_into_linux_rng_action, "mix_hwrng_into_linux_rng");

    // Don't mount filesystems or start core system services in charger mode.
    std::string bootmode = GetProperty("ro.bootmode", "");
    if (bootmode == "charger") {
        am.QueueEventTrigger("charger");
    } else {
        am.QueueEventTrigger("late-init");
    }

    // Run all property triggers based on current state of the properties.
    am.QueueBuiltinAction(queue_property_triggers_action, "queue_property_triggers");

    while (true) {
        // By default, sleep until something happens.
        int epoll_timeout_ms = -1; //epoll超时时间，相当于阻塞时间

        if (!(waiting_for_prop || ServiceManager::GetInstance().IsWaitingForExec())) {
            am.ExecuteOneCommand();//执行一个command
        }
        if (!(waiting_for_prop || ServiceManager::GetInstance().IsWaitingForExec())) {
            restart_processes();

            // If there's a process that needs restarting, wake up in time for that.
            if (process_needs_restart_at != 0) {
                epoll_timeout_ms = (process_needs_restart_at - time(nullptr)) * 1000;
                if (epoll_timeout_ms < 0) epoll_timeout_ms = 0;
            }

            // If there's more work to do, wake up again immediately.
            //当还有命令要执行时，将epoll_timeout_ms设置为0
            if (am.HasMoreCommands()) epoll_timeout_ms = 0;
        }

        epoll_event ev;
        /* 
        * 1.epoll_wait与epoll_create1、epoll_ctl是一起使用的 
        * 2.epoll_create1用于创建epoll的文件描述符，epoll_ctl、epoll_wait都把它创建的fd作为第一个参数传入 
        * 3.epoll_ctl用于操作epoll，EPOLL_CTL_ADD：注册新的fd到epfd中， 
        EPOLL_CTL_MOD：修改已经注册的fd的监听事件，EPOLL_CTL_DEL：从epfd中删除一个fd； 
        * 4.epoll_wait用于等待事件的产生，epoll_ctl调用EPOLL_CTL_ADD时会传入需要监听什么类型的事件， 
        *比如EPOLLIN表示监听fd可读，当该fd有可读的数据时，调用epoll_wait经过epoll_timeout_ms时间就会把该事件的信息返回给&ev 
        */
        int nr = TEMP_FAILURE_RETRY(epoll_wait(epoll_fd, &ev, 1, epoll_timeout_ms));
        if (nr == -1) {
            PLOG(ERROR) << "epoll_wait failed";
        } else if (nr == 1) {
            //当有event返回时，取出 ev.data.ptr（之前epoll_ctl注册时的回调函数），直接执行 
            //在signal_handler_init和start_property_service有注册两个fd的监听，一个用于监听SIGCHLD(子进程结束信号)，一个用于监听属性设置
            ((void (*)()) ev.data.ptr)();
        }
    }

    return 0;
}
```