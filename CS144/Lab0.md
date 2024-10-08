# Networking warmup

## 1 Set up GNU/Linux on your computer
根据各自系统选择下载镜像。注意，你可能不知道用户和密码。
```
The VM will boot to a textual interface. Your username is cs144 , and the default password is also cs144 .
```
![alt text](image.png)

## 2 Networking by hand
### 2.1 Fetch a web page
![alt text](image-1.png)

#### 3 Assigment 
最紧要就是快，不然就是408timeout。<br> 
Connection: close 是默认设置 可以不传入headers.<br>

![alt text](image-2.png)

### 2.2 Send yourself an email
需要学校内部账号或者赞助，工作人员。我不符合条件，选择了解过程。

### 2.3 Listening and connecting
切换tty以达到不同terminal的通信
![alt text](image-3.png)

## 3 Writing a network program using an OS stream socket

介绍了TCP以及展望后续Lab内容
在实际网络中，互联网并不提供可靠的字节流。<br>
他们往往“尽力而为”，也被叫做互联网数据报。
* headers (some metadata): source, destination address
* payload data

数据报可能出现的问题
* lost
* delivered out of order
* delivered with the contents altered
* duplicated and delivered more than once

### 3.1 Let’s get started—fetching and building the starter code
下载安装好对应资源
![alt text](image-4.png)

### 3.2 Modern C++: mostly safe but still fast and low-level
一些规范
*  Use the language documentation at https://en.cppreference.com as a resource. (We’d
recommend you avoid cplusplus.com which is more likely to be out-of-date.)
*  Never use malloc() or free().
*  Never use new or delete.
*  Essentially never use raw pointers (*), and use “smart” pointers (unique ptr or
shared ptr) only when necessary. (You will not need to use these in CS144.)
*  Avoid templates, threads, locks, and virtual functions. (You will not need to use these
in CS144.)
*  Avoid C-style strings (char *str) or string functions (strlen(), strcpy()). These
are pretty error-prone. Use a std::string instead.
*  Never use C-style casts (e.g., (FILE *)x). Use a C++ static cast if you have to (you
generally will not need this in CS144).
*  Prefer passing function arguments by const reference (e.g.: const Address & address).
*  Make every variable const unless it needs to be mutated.
*  Make every method const unless it needs to mutate the object.
*  Avoid global variables, and give every variable the smallest scope possible.
*  Before handing in an assignment, run cmake --build build --target tidy for
suggestions on how to improve the code related to C++ programming practices, and
cmake --build build --target format to format the code consistently.

### 3.3 Reading the Minnow support code
一些关于Minnow须知
Please read over the public interfaces (the part that comes after “public:” in the files
util/socket.hh and util/file descriptor.hh. (Please note that a Socket is a type of
FileDescriptor, and a TCPSocket is a type of Socket.)

## Writing webget
主要是读老师给的文件，注意format代码。
```c++
void get_URL( const string& host, const string& path )
{
  Address addr = Address( host, "http" );
  TCPSocket tcp;
  string request = "GET " + path + " HTTP/1.1\r\n";
  request += "Host: " + host + "\r\n";
  request += "Connection: close \r\n";
  request += "\r\n";

  tcp.connect( addr );
  tcp.write( request );
  string response;
  while ( !tcp.eof() ) {
    tcp.read( response );
    cout << response;
  }
  tcp.close();
}
```

## An in-memory reliable byte stream

```c++
#include "byte_stream.hh"

using namespace std;

ByteStream::ByteStream( uint64_t capacity ) : capacity_( capacity )
{
  buffers_.reserve( capacity );
}

bool Writer::is_closed() const
{
  return is_closed_;
}

void Writer::push( string data )
{
  if ( is_closed() ) {
    return;
  }
  uint64_t dataSize = data.size();
  if ( dataSize > available_capacity() ) {
    dataSize = available_capacity();
  }
  buffers_ += data.substr( 0, dataSize );
  bytes_pushed_ += dataSize;
}

void Writer::close()
{
  is_closed_ = true;
}

uint64_t Writer::available_capacity() const
{
  // Your code here.
  return capacity_ - buffers_.size();
}

uint64_t Writer::bytes_pushed() const
{
  // Your code here.
  return bytes_pushed_;
}

bool Reader::is_finished() const
{
  // Your code here.
  return is_closed_ && ( bytes_buffered() == 0 );
}

uint64_t Reader::bytes_popped() const
{
  // Your code here.
  return bytes_popped_;
}

string_view Reader::peek() const
{
  uint64_t len = bytes_buffered() > 1024 ? 1024 : bytes_buffered();
  string_view sv( buffers_.data(), len );
  return sv;
}

void Reader::pop( uint64_t len )
{
  if ( is_finished() ) {
    return;
  }
  if ( len > bytes_buffered() ) {
    len = bytes_buffered();
  }
  buffers_.erase( 0, len );
  bytes_popped_ += len;
}

uint64_t Reader::bytes_buffered() const
{
  // Your code here.
  return buffers_.size();
}

```

没有什么难点，前面一直出现超时是因为环境问题。
拉镜像又大又慢，毫无必要。
强烈建议用docker自己构造一个配置环境使用。

```dockerfile
FROM ubuntu:23.10

RUN apt update && apt install -y git cmake gdb build-essential clang \
    clang-tidy clang-format gcc-doc pkg-config glibc-doc tcpdump tshark
```

注意docker国内镜像已经关闭，自己找新的镜像或者代理。