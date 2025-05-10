# 绪论 
- 可简化和加速使用 C++ 开发的以网络为中心的可移植应用程序   
- 与 C++ 标准库完美集成，并填补了其留下的许多功能空白    
- 由 4 个核心库组成：Foundation、XML、Util、Net    
- 两个附加库：NetSSL（为 Net 库提供 SSL 支持）和 Data（用于访问 SQL 数据库）
- 在使用 C++ 高级特性和代码可维护性之间找到良好平衡 
![image](https://github.com/user-attachments/assets/a9c2a3e5-525b-44ee-9746-ccd841f3c06d)

# Foundation库
- 核心
- 包含底层平台抽象层，以及常用的实用程序类和函数
- 包含固定大小整数的类型、用于在字节顺序之间转换整数的函数、Poco::Any 类（基于 boost::Any）、用于错误处理和调试的实用程序（包括各种异常类和对断言的支持）
- 大量的内存管理类
- 字符串处理类
- 支持格式化和解析数字
- 提供了处理各种日期和时间的类
- 为了访问文件系统，POCO 有 Poco::File、Poco::Path、 Poco::DirectoryIterator 类
- 通知记录进程中部分状态、结果, 有Poco::NotificationCenter、Poco::NotificationQueue、events
- 数据流类（ Poco::BinaryReader 和 Poco::BinaryWriter 增强，用于将二进制数据写入流，自动且透明地处理字节顺序问题。）
- POCO 提供了一个强大且可扩展的日志框架，支持过滤、路由到不同的渠道以及日志消息的格式化
- 运行时加载（和卸载）共享库，POCO 有一个低级Poco::SharedLibrary 类
- Poco::ClassLoader 类模板和支持框架，允许在运行时动态加载和卸载 C++ 类
- 包含不同级别的多线程抽象。活跃对象是指拥有在自己的线程中执行方法的对象，这使得异步成员函数调用成为可能。

# XML库
- 可扩展标记语言（英语：Extensible Markup Language，简称：XML）是一种标记语言和用于存储、传输和重构松散数据的文件格式，它定义了一系列编码文档的规则以使其在人类可读的同时机器可读
- 提供对读取、处理和写入 XML 的支持

# Util库
- 包含用于创建命令行和服务器应用程序的框架
- 包括对处理命令行参数（验证、绑定到配置属性等）和管理配置信息的支持
- 支持不同的配置文件格式——Windows 风格的 INI 文件、Java 风格的属性文件、XML 文件和 Windows 注册表
- 对于服务器应用程序，该框架为 Windows 服务和 Unix 守护进程提供透明支持；每个服务器应用程序都可以注册并作为 Windows 服务运行，无需额外代码

# Net库
- Net 库使得编写基于网络的应用程序变得容易；无论应用程序只需要通过普通的 TCP 套接字发送数据，还是需要一个功能齐全的内置 HTTP 服务器，都会在 Net 库中找到一些有用的东西
- 在最低级别，Net 库包含套接字类，支持 TCP 流和服务器套接字、UDP 套接字、多播套接字、ICMP 和原始套接字
- 如果应用程序需要安全套接字，可以使用 NetSSL 库中的安全套接字（使用 OpenSSL 实现）

# 使用POCO库实现简单的HTTP服务器（例子）
- 服务器返回显示当前日期和时间的 HTML 文档
- 该应用程序框架用于构建可以作为 Windows 服务或 Unix 守护进程运行的服务器应用程序
- 定义了一个 TimeRequestHandler 类，它通过返回包含当前日期和时间的 HTML 文档来处理传入的请求；对于每个传入请求，都会使用日志框架记录一条消息
- 需要一个对应的工厂类 TimeRequestHandlerFactory；这个工厂类的实例会被传递给 HTTP 服务器对象
- HTTPTimeServer应用程序类通过重写Poco::Util::ServerApplication的defineOptions()成员函数来定义命令行参数帮助
- 在启动 HTTP 服务器之前，它还会读取默认应用程序配置文件（在 initialize() 中）并在 main() 中获取一些配置属性的值
```
#include "Poco/Net/HTTPServer.h"
#include "Poco/Net/HTTPRequestHandler.h"
#include "Poco/Net/HTTPRequestHandlerFactory.h"
#include "Poco/Net/HTTPServerParams.h"
#include "Poco/Net/HTTPServerRequest.h"
#include "Poco/Net/HTTPServerResponse.h"
#include "Poco/Net/HTTPServerParams.h"
#include "Poco/Net/ServerSocket.h"
#include "Poco/Timestamp.h"
#include "Poco/DateTimeFormatter.h"
#include "Poco/DateTimeFormat.h"
#include "Poco/Exception.h"
#include "Poco/ThreadPool.h"
#include "Poco/Util/ServerApplication.h"
#include "Poco/Util/Option.h"
#include "Poco/Util/OptionSet.h"
#include "Poco/Util/HelpFormatter.h"
#include <iostream>

using Poco::Net::ServerSocket;
using Poco::Net::HTTPRequestHandler;
using Poco::Net::HTTPRequestHandlerFactory;
using Poco::Net::HTTPServer;
using Poco::Net::HTTPServerRequest;
using Poco::Net::HTTPServerResponse;
using Poco::Net::HTTPServerParams;
using Poco::Timestamp;
using Poco::DateTimeFormatter;
using Poco::DateTimeFormat;
using Poco::ThreadPool;
using Poco::Util::ServerApplication;
using Poco::Util::Application;
using Poco::Util::Option;
using Poco::Util::OptionSet;
using Poco::Util::OptionCallback;
using Poco::Util::HelpFormatter;

class TimeRequestHandler: public HTTPRequestHandler
{
public:
    TimeRequestHandler(const std::string& format): _format(format)
    {
    }

    void handleRequest(HTTPServerRequest& request, HTTPServerResponse& response)
    {
        Application& app = Application::instance();
        app.logger().information("Request from %s",
            request.clientAddress().toString());

        Timestamp now;
        std::string dt(DateTimeFormatter::format(now, _format));

        response.setChunkedTransferEncoding(true);
        response.setContentType("text/html");

        std::ostream& ostr = response.send();
        ostr << "<html><head><title>HTTPTimeServer powered by "
                "POCO C++ Libraries</title>";
        ostr << "<meta http-equiv=\"refresh\" content=\"1\"></head>";
        ostr << "<body><p style=\"text-align: center; "
                "font-size: 48px;\">";
        ostr << dt;
        ostr << "</p></body></html>";
    }

private:
    std::string _format;
};

class TimeRequestHandlerFactory: public HTTPRequestHandlerFactory
{
public:
    TimeRequestHandlerFactory(const std::string& format):
        _format(format)
    {
    }

    HTTPRequestHandler* createRequestHandler(const HTTPServerRequest& request)
    {
        if (request.getURI() == "/")
            return new TimeRequestHandler(_format);
        else
            return 0;
    }

private:
    std::string _format;
};

class HTTPTimeServer: public Poco::Util::ServerApplication
{
protected:
    void initialize(Application& self)
    {
        loadConfiguration();
        ServerApplication::initialize(self);
    }

    void defineOptions(OptionSet& options)
    {
        ServerApplication::defineOptions(options);

        options.addOption(
        Option("help", "h", "display argument help information")
            .required(false)
            .repeatable(false)
            .callback(OptionCallback<HTTPTimeServer>(
                this, &HTTPTimeServer::handleHelp)));
    }

    void handleHelp(const std::string& name, const std::string& value)
    {
        HelpFormatter helpFormatter(options());
        helpFormatter.setCommand(commandName());
        helpFormatter.setUsage("OPTIONS");
        helpFormatter.setHeader(
            "A web server that serves the current date and time.");
        helpFormatter.format(std::cout);
        stopOptionsProcessing();
        _helpRequested = true;
    }

    int main(const std::vector<std::string>& args)
    {
        if (!_helpRequested)
        {
            unsigned short port = static_cast<unsigned short>(
                config().getInt("HTTPTimeServer.port", 9980));
            std::string format(config().getString(
                "HTTPTimeServer.format",
                DateTimeFormat::SORTABLE_FORMAT));

            ServerSocket svs(port);
            HTTPServer srv(
                new TimeRequestHandlerFactory(format),
                svs, new HTTPServerParams);
            srv.start();
            waitForTerminationRequest();
            srv.stop();
        }
        return Application::EXIT_OK;
    }

private:
    bool _helpRequested = false;
};

int main(int argc, char** argv)
{
    HTTPTimeServer app;
    return app.run(argc, argv);
}
```



