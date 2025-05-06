在[[BootAnimation启动流程]]中提到过，设置 ctl.start 属性值为 bootanim，怎么就能启动 bootanim 呢？

在init进程启动的第二阶段过程，会开启属性服务

//system\core\init\init.cpp
int SecondStageMain(int argc, char** argv) {
    //......
    StartPropertyService(&property_fd);
    //......
1
2
3
4
5
使用epoll来监听是否有事件可供读入。对于epoll的操作，可以查看system/core/init/epoll.cpp文件

//system\core\init\property_service.cpp
static void PropertyServiceThread() {
    Epoll epoll;
    if (auto result = epoll.Open(); !result.ok()) {//创建监听的fd，
        LOG(FATAL) << result.error();
    }

    if (auto result = epoll.RegisterHandler(property_set_fd, handle_property_set_fd);//调用 epoll_ctl，将fd加到检测列表，并且将property_set_fd和handle_property_set_fd存入链表,后面可以通过property_set_fd找到handle_property_set_fd函数
        !result.ok()) {
        LOG(FATAL) << result.error();
    }
    
	//......

    while (true) {
        auto pending_functions = epoll.Wait(std::nullopt);//等待事件，没有事件休眠，有事件时，根据fd找到对应的函数并返回
        if (!pending_functions.ok()) {
            LOG(ERROR) << pending_functions.error();
        } else {
            for (const auto& function : *pending_functions) {
                (*function)();//执行函数
            }
        }
    }
}

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
调用property_set(“ctl.start”, “bootanim”); 会导致epoll返回，调用handle_property_set_fd函数

//system\core\init\property_service.cpp
static void handle_property_set_fd() {
    //......

    switch (cmd) {
    case PROP_MSG_SETPROP: {
        char prop_name[PROP_NAME_MAX];
        char prop_value[PROP_VALUE_MAX];
		//接收数据
        //......
        uint32_t result =
                HandlePropertySet(prop_name, prop_value, source_context, cr, nullptr, &error);
        //......
      }
1
2
3
4
5
6
7
8
9
10
11
12
13
14
对于设置属性，继续调用HandlePropertySet函数

//system\core\init\property_service.cpp
uint32_t HandlePropertySet(const std::string& name, const std::string& value,
                           const std::string& source_context, const ucred& cr,
                           SocketConnection* socket, std::string* error) {
   //......

    if (StartsWith(name, "ctl.")) {
        return SendControlMessage(name.c_str() + 4, value, cr.pid, socket, error);
    }
    
    //......
1
2
3
4
5
6
7
8
9
10
11
对于ctl. 开头的属性名，调用SendControlMessage，注意，这里传入的socket为null

//system\core\init\property_service.cpp
static uint32_t SendControlMessage(const std::string& msg, const std::string& name, pid_t pid,
                                   SocketConnection* socket, std::string* error) {
    //......

    bool queue_success = QueueControlMessage(msg, name, pid, fd);
	
    //......
}
1
2
3
4
5
6
7
8
9
在QueueControlMessage 函数中，将消息放入队列，并唤醒init进程

//system\core\init\init.cpp
bool QueueControlMessage(const std::string& message, const std::string& name, pid_t pid, int fd) {
	//......
    pending_control_messages.push({message, name, pid, fd});
    WakeMainInitThread();
    return true;
}

//写fd
static void WakeMainInitThread() {
    uint64_t counter = 1;
    TEMP_FAILURE_RETRY(write(wake_main_thread_fd, &counter, sizeof(counter)));
}
1
2
3
4
5
6
7
8
9
10
11
12
13
init进程在启动的第二阶段，无事件时休眠，被唤醒后继续往下执行

//system\core\init\init.cpp
int SecondStageMain(int argc, char** argv) {
    while (true) {
        //......
        
        auto pending_functions = epoll.Wait(epoll_timeout);
        //......
        if (!IsShuttingDown()) {
            HandleControlMessages();
            SetUsbController();
        }
    
 //......   
}
1
2
3
4
5
6
7
8
9
10
11
12
13
14
在HandleControlMessages中，查找对应的函数，并执行

//system\core\init\init.cpp
static bool HandleControlMessage(std::string_view message, const std::string& name,
                                 pid_t from_pid) {
  	//......

    const auto& map = GetControlMessageMap();
    const auto it = map.find(action);//根据action，查找对应的函数
  
    const auto& function = it->second;

    if (auto result = function(service); !result.ok()) {//执行函数
        
        return false;
    }

   	//......
}


//附:
static const std::map<std::string, ControlMessageFunction, std::less<>>& GetControlMessageMap() {
    // clang-format off
    static const std::map<std::string, ControlMessageFunction, std::less<>> control_message_functions = {
        {"sigstop_on",        [](auto* service) { service->set_sigstop(true); return Result<void>{}; }},
        {"sigstop_off",       [](auto* service) { service->set_sigstop(false); return Result<void>{}; }},
        {"oneshot_on",        [](auto* service) { service->set_oneshot(true); return Result<void>{}; }},
        {"oneshot_off",       [](auto* service) { service->set_oneshot(false); return Result<void>{}; }},
        {"start",             DoControlStart},
        {"stop",              DoControlStop},
        {"restart",           DoControlRestart},
    };
    // clang-format on

    return control_message_functions;
}


对于start，调用 DoControlStart函数

//system\core\init\init.cpp
static Result<void> DoControlStart(Service* service) {
    return service->Start();
}
1
2
3
4
调用Start 最终fork一个子进程，并调用ExpandArgsAndExecv函数，执行/system/bin/bootanimation，开机动画就启动起来了

static bool ExpandArgsAndExecv(const std::vector<std::string>& args, bool sigstop) {
    std::vector<std::string> expanded_args;
    std::vector<char*> c_strings;

    expanded_args.resize(args.size());
    c_strings.push_back(const_cast<char*>(args[0].data()));
    for (std::size_t i = 1; i < args.size(); ++i) {
        auto expanded_arg = ExpandProps(args[i]);
        if (!expanded_arg.ok()) {
            LOG(FATAL) << args[0] << ": cannot expand arguments': " << expanded_arg.error();
        }
        expanded_args[i] = *expanded_arg;
        c_strings.push_back(expanded_args[i].data());
    }
    c_strings.push_back(nullptr);

    if (sigstop) {
        kill(getpid(), SIGSTOP);
    }

    return execv(c_strings[0], c_strings.data()) == 0;//执行
}