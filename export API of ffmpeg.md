## 添加私有的编解码协议方法

在MSVC环境下编译ffmpeg，目的是在dll中获取到 ffurl_register_protocol 函数。    
   目前采取了2个方案，都可以达到这个目的 ：    
### 方案1 ： 修改代码 
在 libavformat/allformat.h ， av_register_all() 后面添加 av_ffurl_register_protocol(URLProtocol *protocol, int size)声明在 libavformat/allformats.c , av_register_all() 后面添加实现        
   
      int av_ffurl_register_protocol(URLProtocol *protocol, int size)   
      {   
         return ffurl_register_protocol( protocol, size );    
      }
    
  msvc环境下编译ffmpeg代码，获取到的avformat.dll中有函数API   av_ffurl_register_protocol()可以调用

### 方案2 ：  修改Makefile, 添加url.h  

   文件路径 /libavformat/Makefile  ,     
   ```  
   HEADERS = avformat.h avio.h url.h version.h    
   OBJS = allformats.o         \     
   cutils.o             \   
   
  ```
  修改文件     /libavformat/libavformat.v 文件    
  ```
    LIBAVFORMAT_$MAJOR {   
        global: av*;   
                #FIXME those are for ffserver   
                ffurl_register_protocol ;   
                ff_inet_aton;   
                ff_socket_nonblock;   
                ff_rtsp_parse_line;   
                ff_rtp_get_local_rtp_port;   
                ff_rtp_get_local_rtcp_port;   
                ffio_open_dyn_packet_buf;   
                ffio_set_buf_size;   
                ffurl_close;   
                ffurl_open;   
                ffurl_read_complete;   
                ffurl_seek;   
                ffurl_size;   
                ffurl_write;   
                #those are deprecated, remove on next bump   
                url_feof;   
        local: *;   
};   
```

重新进行编译，avformat.dll 中获取到函数 ffurl_register_protocol ()

## 对比结论
目前来看，方案二更优
