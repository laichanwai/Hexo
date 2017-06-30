---
title: NSLog 源码阅读笔记
date: 2017-03-06 13:46:09
tags: 
  - Blog
  - iOS
permalink: nslog-reading-notes
---

> 你曾经有过这样的疑问吗，为什么 NSLog 的速度这么慢？

*声明：代码来自 [GNUstep](https://zh.wikipedia.org/wiki/GNUstep) 。

NSLog() 是个 C 函数，看看 NSLog 调用三部曲：

```C
NSLog(NSString *format, ...) 
    -> NSLogv(NSString *format, va_list args) 
        -> _NSLog_standard_printf_handler(NSString *message)
```

代码中使用了几个宏：`_WIN32`、`HAVE_SYSLOG`、`HAVE_SLOGF`

其中 `HAVE_SYSLOG` 是引入 `#include <syslog.h>`，`HAVE_SLOGF` 是引入 `# include <sys/slog.h>`

### 第一步：NSLog(NSString *, ...)

看源码我们就能发现，其实这个方法只是为了得到 va_list，然后调用 NSLogv()

```
void
NSLog(NSString* format, ...)
{
    va_list ap;
    
    va_start(ap, format);
    NSLogv(format, ap);
    va_end(ap);
}
```

### 第二步：NSLogv(NSString *, va_list)

这个方法是用来封装打印信息的，控制台看到的 时间，进程，线程等等信息都是在这个方法里操作的。

#### 1. PID（进程）

```objc
if (pid == 0)
{
#if defined(_WIN32)
   pid = (int)GetCurrentProcessId();
#else
   pid = (int)getpid();
#endif
}
```
    
#### 2.  thread（线程）

```
if (GSPrivateDefaultsFlag(GSLogThread) == YES)
{
   /* If no name has been set for the current thread,
    * we log the address of the NSThread object instead.
    */
   t = GSCurrentThread();
   threadName = [t name];
}
```
    
有一点要提一下，`NSUserDefaults` 初始化的时候会读取 `NSPrecessInfo` 的信息，`GSPrivateDefaultsFlag(GSUserDefaultFlagType)` 这个函数是用来判断初始化的时候有没有读取到该信息。
    
```objc
typedef enum {
 GSMacOSXCompatible,			// General behavior flag.
 GSOldStyleGeometry,			// Control geometry string output.
 GSLogSyslog,				// Force logging to go to syslog.
 GSLogThread,				// Include thread name in log message.
 GSLogOffset,			        // Include time zone offset in message.
 NSWriteOldStylePropertyLists,		// Control PList output.
 GSUserDefaultMaxFlag			// End marker.
} GSUserDefaultFlagType;
```
    
#### 3. Message（打印信息）

这一步就是拼接字符串了，它会判断之前获取的线程信息。
    
##### 首先是获取拼接当前时间：
    
```objc
if (GSPrivateDefaultsFlag(GSLogOffset) == YES)
{
   fmt = @"%Y-%m-%d %H:%M:%S.%F %z";
}
else
{
   fmt = @"%Y-%m-%d %H:%M:%S.%F";
}
cal = [[NSCalendarDate calendarDate] descriptionWithCalendarFormat: fmt];
    
[prefix appendString: cal];
[prefix appendString: @" "];
```
    
##### 进程、线程信息
    
```objc
#ifdef	HAVE_SYSLOG
if (GSPrivateDefaultsFlag(GSLogSyslog) == YES)
{
   if (nil == t)
   {
       [prefix appendFormat: @"[thread:%"PRIuPTR"] ",
     GSPrivateThreadID()];
   }
   else if (nil == threadName)
   {
       [prefix appendFormat: @"[thread:%"PRIuPTR",%p] ",
     GSPrivateThreadID(), t];
   }
   else
   {
       [prefix appendFormat: @"[thread:%"PRIuPTR",%@] ",
     GSPrivateThreadID(), threadName];
   }
}
else {
   [prefix appendString: [[NSProcessInfo processInfo] processName]];
   if (nil == t)
   {
       [prefix appendFormat: @"[%d:%"PRIuPTR"] ",
        pid, GSPrivateThreadID()];
   }
   else if (nil == threadName)
   {
       [prefix appendFormat: @"[%d:%"PRIuPTR",%p] ",
        pid, GSPrivateThreadID(), t];
   }
   else
   {
       [prefix appendFormat: @"[%d:%"PRIuPTR",%@] ",
        pid, GSPrivateThreadID(), threadName];
   }
}
#endif
```

##### 拼接打印信息
    
```objc
message = [[NSString alloc] initWithFormat: format arguments: args];
[prefix appendString: message];
[message release];
if ([prefix hasSuffix: @"\n"] ==  NO)
{
 [prefix appendString: @"\n"];
}
```

#### 4. 加锁，调用 `_NSLog_printf_handler`

调用 `_NSLog_printf_handler` 就是调用 `_NSLog_standard_printf_handler`：
    
```objc
NSLog_printf_handler *_NSLog_printf_handler = _NSLog_standard_printf_handler;
    
// ...
    
if (_NSLog_printf_handler == NULL)
{
   _NSLog_printf_handler = *_NSLog_standard_printf_handler;
}
```

这里加锁用的是 `NSRecursiveLock`，有兴趣的可以去看看。

```c
if (nil == myLock)
{
   GSLogLock();
}
    
(*lockImp)(myLock, @selector(lock));
   
_NSLog_printf_handler(prefix);
    
(*unlockImp)(myLock, @selector(unlock));
```

### 第三步：_NSLog_standard_printf_handler(NSString *)

这个函数中定义了几个变量

```objc
    NSData	*d;
    const char	*buf;
    unsigned	len;
#if	defined(_WIN32)
    LPCWSTR	null_terminated_buf;
#else
#if	defined(HAVE_SYSLOG) || defined(HAVE_SLOGF)
    char	*null_terminated_buf = NULL;
#endif
#endif
    static NSStringEncoding enc = 0;
```

#### Buffer

不管在什么平台，不管使用的是什么输出，数据都是以流的形式输出，所以第一步就是将 `message` 转成 `buffer`。

```
if (enc == 0)
{
    enc = [NSString defaultCStringEncoding];
}
d = [message dataUsingEncoding: enc allowLossyConversion: NO];
if (d == nil)
{
    d = [message dataUsingEncoding: NSUTF8StringEncoding
    allowLossyConversion: NO];
}

if (d == nil)		// Should never happen.
{
    buf = [message lossyCString];
    len = strlen(buf);
}
else
{
    buf = (const char*)[d bytes];
    len = [d length];
}
```

#### _WIN32

先看看在 WIN32 下的操作：

```C
#ifdefined(_WIN32)
null_terminated_buf = UNISTR(message);

OutputDebugStringW(null_terminated_buf);

if ((GSPrivateDefaultsFlag(GSLogSyslog) == YES
|| write(_NSLogDescriptor, buf, len) != (int)len) && !IsDebuggerPresent())
{
    static HANDLE eventloghandle = 0;
    
    if (!eventloghandle)
    {
        eventloghandle = RegisterEventSourceW(NULL,
        UNISTR([[NSProcessInfo processInfo] processName]));
    }
    if (eventloghandle)
    {
        ReportEventW(eventloghandle, // event log handle
        EVENTLOG_WARNING_TYPE, // event type
        0,				// category zero
        0,				// event identifier
        NULL,			// no user security identifier
        1,				// one substitution string
        0,				// no data
        &null_terminated_buf, // pointer to string array
        NULL);			// pointer to data
    }
}
#else
```

#### HAVE_SYSLOG

使用 `syslog.h`

```c
if (GSPrivateDefaultsFlag(GSLogSyslog) == YES
    || write(_NSLogDescriptor, buf, len) != (int)len)
{
    null_terminated_buf = malloc(sizeof (char) * (len + 1));
    strncpy (null_terminated_buf, buf, len);
    null_terminated_buf[len] = '\0';
    
    syslog(SYSLOGMASK, "%s",  null_terminated_buf);
    
    free(null_terminated_buf);
}
```

#### HAVE_SLOGF

使用 `slog.h`

```c
if (GSPrivateDefaultsFlag(GSLogSyslog) == YES
    || write(_NSLogDescriptor, buf, len) != (int)len)
{
   /* QNX's slog has a size limit per entry. We might need to iterate over
    * _SLOG_MAXSIZEd chunks of the buffer
    */
   const char *newBuf = buf;
   unsigned newLen = len;
      
   // Allocate at most _SLOG_MAXSIZE bytes
   null_terminated_buf = malloc(sizeof(char) * MIN(newLen, _SLOG_MAXSIZE));
   // If it's shorter than that, we never even enter the loop
   while (newLen >= _SLOG_MAXSIZE)
   {
       strncpy(null_terminated_buf, newBuf, (_SLOG_MAXSIZE - 1));
       null_terminated_buf[_SLOG_MAXSIZE] = '\0';
       slogf(_SLOG_SETCODE(_SLOG_SYSLOG, 0), _SLOG_ERROR, "%s",
         null_terminated_buf);
       newBuf += (_SLOG_MAXSIZE - 1);
       newLen -= (_SLOG_MAXSIZE - 1);
   }
   /* Write out the rest (which will be at most (_SLOG_MAXSIZE - 1) chars,
   * so the terminator still fits.
   */
   if (0 != newLen)
   {
       strncpy(null_terminated_buf, newBuf, newLen);
       null_terminated_buf[newLen] = '\0';
       slogf(_SLOG_SETCODE(_SLOG_SYSLOG, 0), _SLOG_ERROR, "%s",
       null_terminated_buf);
   }
       free(null_terminated_buf);
}
```

#### 默认

```c
write(_NSLogDescriptor, buf, len);
```

### 最后

> 现在你知道为什么 NSLog 这么慢了吗？