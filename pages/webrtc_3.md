互动直播 500ms  WebRtc（UDP/RTP）
娱乐直播 3s     rtmp(TCP,不支持浏览器) http-flv(TCP/http) hls(TCP/Http，苹果, 10s）DASH(TCP/Http, 10s)

HLS 是基于HTTP的，它首先对媒体流（文件）进行切片，然后通过http传输，接收端则需要将接收到的切片进行缓冲，之后才能将媒体流平稳地播放出来。

基于上述机制，HTS在实时性方面比RTMP差很多
但它的好处也是显而易见的（苹果产品原生支持）

但Adobe不再维护RTMP时，有人提出了http-flv方案，即传输的内容仍然使用RTMP格式，但底层传输协议换成http，
这种方案即可以保证其实时性比HLS好，又可以节约升级成本

不过http-flv的扩展性比较差，厂商不得不多写多套代码，费时费力，
于是，ffmpeg推出了dash方案，该方案与hls类似，也是以切片的方式传输数据，
最终该方案成为国际标准，从而使直播厂商只要写一套代码就可以实现切片传输了


有了AI、深度学习技术，我们可以利用他们对音视频数据（非结构化数据）做二次处理，转化为结构化的数据，之后再用大数据技术对他们进行分析，生成各种报表

如果将AI和大数据分析速度提升到实时处理的级别，实时改变服务内容

每一路音频或每一路视频为一条轨
轨：取两条轨永远不相交的意思，也就是说，音频数据与视频数据是永远不会交叉存放到一起的。

pcm->opus/aac->pcm
yuv->h264/vp8->yuv/rgb

G.711/G.722主要用于电话系统，音视频直播客户端要想与电话系统对接，就要支持这种编解码格式
Opus主要用于实时通话
AAC主要用于音乐类的应用，如钢琴教学等

音视频不同步
回音，噪声，声音过小，统称为3A问题
音视频实时性，网络拥塞，丢包，延时，抖动
混音

3A是指：Acoustic Echo Cancelling（AEC），即回音消除；
       Automatic Gain Control（AGC），即自动增益；
       Active Noise Control（ANC，也称为Noise Cancellation、NoiseSuppression），即降噪

目前开源项目中，只要WebRtc和Speex有开源的回音消除算法


webrtc接入层
      web api 或 c++ api
 webrtc会话层
      Session

webrtc核心引擎层
      音频引擎层 voice engine
        netEQ
        qpus/iLBC codec
        3A
      网络传输层 transport
        SRTP
        网络IO多路复用 multiplexing
        P2P
      视频引擎层 video engine
        JitterBuffer
        VP8/VP9 codec
        图像增强
        
webrtc设备层
        Capture/Render
        Network
        Capture
        
        
Session层的主要作用是：控制业务逻辑，如媒体协商、收集Candiate等，这些操作都是在Session层处理的

两个指标：
一是实时通信中的延迟指标；
  200ms内，面对面效果
  300ms内，难感觉延时
  400ms内，有迟滞现象
  >500ms，明显迟滞现象，影响互动
二是音视频服务质量指标

分辨率
  图像默认分辨率 640x480或640x360，如果低于该值，则图像中包含的信息太少
帧率
  视频每秒播放帧的数量
  一般动画片、电影的帧率在24帧以上
  高清视频在60以上
  对于实时通信的视频视频来说，15帧是一个分水岭
  当小于15时，大部分人会觉得视频质量不佳，卡顿严重
  
码率
  指视频压缩后，每秒数据流的大小
  原则上，分辨率越大，码率也越大
  如果出现分辨率大，而码率小的情况，说明在视频编码时丢弃了大量的图像信息，这将导致解码时无法将图像完整复原，从而造成失真。
  因此我们可以得到结论：相同分辨率的情况下，码率越大还原度越好，图像越清晰。
  码率大小是有限制的，超过一定阈值(MOS=5)后，再大的码率也没有意义了。
  
MOS值 Mean Option Score 平均意见值 
  是用来评估业务服务质量好坏的
  MOS值越高，业务质量越好
  5 优秀
  4 较好
  3 还可以
  2 差
  1 很坏
  
  MOS=4时
    640x480 1900kbps
    1920x1080 7Mbps
  MOS=3时
    640x480 500kbps
    1920x1080 2.5Mbps

实时通信的主要矛盾

解决方法
  一 增加带宽
  二 减少数据量
  三 适当增加时延
  四 提高网络质量
  五 快速准确评估带宽
  
  
一 增加宽带
  在众多的解决方案中，增加宽带的方案无疑是解决音视频实时通信服务质量的根本
  除了提升网络能力这种被动的方法外，还有一些变相增加宽带的方案，
  客户端
    最经典的就是WebRTC支持的选路方案
    它可以按优先级选择最优质的网络连接线路
  服务端
    有三种
    提供更优质的接入服务
    保证云端网络的带宽和质量
    更合理的路由调度策略

  【用户终端】------最后一公里-----> 【接入服务器】-----服务器内部网络--->【服务器核心节点】-----网络调度-----> 【其他节点】
  
  提供更优质的接入服务，指定是最后一公里问题
    如果可以提高用户终端接入的网络质量，就相当于提高了用户的网络带宽
  
    目前国内存在多家网络运营商，如联通、电信、移动、长城宽带、铁通等，因此国内的网络十分复杂。
    一般情况下，同类型运营商的用户相互通信时，都不会遇到什么问题，
    但跨运营商的用户进行通信时，网络质量就很难得到有效保障。
    解决这一问题的一般办法是，让用户连接同一地区、同一运营商的接入服务器，这样就可以有效保障用户与服务器之间的连接通道。
    
  保证云端网络的宽度和质量，值得是数据进入云端后，云内部的网络质量一定要好
    因为云内部的宽带大小和质量是可以控制的，所以提升这部分的网络能力相对简单一些。
    最简单的办法是，可以购买优质的BGP网络作为云内部使用。
    但优质的BGP的费用也是比较高的
  
更合理的路由调度策略方
    选路的基本原则是距离最近、网络质量最好、服务器负载最小的路线是最优质的线路
    
    
二 减少数据量
  要知道，减少音视频数据量一定是以牺牲音视频服务质量为代价的，但这就是一种平衡
  5种方法：
  采用更好的压缩算法
  SVC技术
  Simulcast技术
  动态码率
  甩帧或减少业务
  
  采用更好的压缩算法
    H265、AVI是最近几年才推出的编解码器，它的压缩率要比现在流行的H264高得多
    在一些测试中，H265比H264提高了25%的压缩率，而AVI在veryslow模式下压缩率比H264提高了40%左右，这是相当可观的
    但从实时性方面看，目前无论是H265还是AVI，其编码速度都与H264有不小的差距，要达到商用级别还需要一段时间。
    
  SVC技术
    是减少传输数据量非常好的一种方法。
    其基本原理是将视频按时间、空间及质量分成多层编码，然后将他们装在一路流中发给服务器
    服务端收到后，再根据每个用户的宽带情况选择不同的层下发。
    其好处是，可以让不同网络状况的用户都得到较好的服务质量。
    但它也有缺点：
    一是上行麻溜不但没有减少反而增加了，所以需要上行用户配置很好的带宽
    二是由于SVC实现复杂，又没有硬件支持，所以终端解码时对CPU消耗很大
    
  Simulcast技术
    Simulcast与SVC技术类似，不过它的实现要比SVC简单得多
    其基本原理是，将视频编码出多种不同分辨率的多路码流，然后上传给服务端
    服务端收到码流后，根据每个用户不同的宽带情况，选择其中一路最合适的码流发给用户
    
    它与SVC技术相比有以下几点不同：
    一是Simulcast上传的每一路流可以单独解码，而SVC做不到
    二是由于Simulcast的每一路都可以单独解码，所以它的解码复杂度与普通解码的是一样的
    三是由于Simulcast上传的是多路单独的流，所以上传码率要比SVC多很多
    
    SVC pc ≡＞ SVR ≡> pc
                   -> phone
    
                 |->
    Simulcast pc |-> SVR -> pc
                 |->     -> phone
 
  动态码率
    也是一种减少数据的方法
    当网络带宽评估出用户宽带不够时，会通过编译器让其减少输出码率
    当评估出带宽增大时，又会增加输出码率。
    这就是动态码率
    如果你发现再网络抖动比较大时，某个音视频产品的图像一会清晰，一会模糊，那多半是因为其采用了动态码率的策略
    
  甩帧或减少业务
    除了上面介绍的那些方法外，还有一种不太友好的方法，就是甩帧或关闭某些不重要的业务来减少数据量。
    当然，这种方法是在用户宽带严重不足的情况下才使用的，只有到了万不得已的时候才会使用这种策略。
    
  Simulcast和动态码率，用得最多
  
  三 适当增加时延
  
    将数据传输时出现的时快时慢现象成为网络抖动。
    如果不对网络抖动加以处理的话，它会对音视频服务质量造成严重影响：
      对于视频来说，网络抖动会造成频繁卡顿和快播现象
      对于音频而言，则会出现断音、吞音等问题
    解决的方法：增加时延，即先将数据放到队列中缓冲一下，然后再从队列中获取数据进行处理，这样数据就变得平滑了
    
    不过对于实时音视频直播而言，必须把时延控制在一定范围之内
    只要让单向延迟小于500ms，大部分人都是可以接受的
    由于音视频的采集、编解码、渲染等时间是固定的，所以只要将网络时延计算出来，就可以确定缓冲区的时延了
    
    虽然实时通信对延迟有着极严格的要求，但通过增加适当的、小幅度的延迟是可以提升音视频质量且不影响实时通信效果的
    
  四 提高网络质量
    提高网络质量是有默认前提条件的，即网络没有发生拥塞时，才能提高网络质量，否则提高网络质量无从谈起
    
    影响网络质量的问题：丢包、延迟、抖动
    
    丢包
      是网络传输过程中网络质量好坏的最重要标志，对网络的影响是最大的
      优质的网络丢包率不超过2%
      对于WebRTC而言，大于2%且小于10%的丢包率是正常的网络
      
    延迟
      也是网络质量的重要指标，但与丢包相比，其对网络的影响要少一些
      如果在两端之间数据传输的延迟持续增大，说明网络线路很可能发生了拥塞
      
    抖动
      对网络质量的影响是最小的
      一般情况下，网络都会发生一些抖动，
      如果抖动很小，可以通过循环队列将其消除
      如果抖动过大，则将乱序包当作丢包处理
      在WebRTC中，抖动时长不能超过10ms，也就是说，如果有包乱序了，最多等待该乱序包10ms，超过10ms就认为该包丢了（即使在第11ms时，乱序的包来了，也仍然认为它丢失了）
      
    解决丢包、延迟和抖动的问题，5种方法：NACK/RTX、FEC前向纠错、JitterBufer防抖动、NetEQ、拥塞控制
    
    NACK/RTX
      NACK是RTCP中的一种消息类型，由接收端向发送端报告一段时间内有哪些包丢失了
      RTX是指发送端重传丢失包，并使用新的SSRC（将传输的音视频包与重传包进行区分）
      
    FEC前向纠错
      使用异或操作传输数据，以便在丢包时可以通过这种机制恢复丢失的包。FEC特别适合随机少量丢包的场景
      
    JitterBufer
      用于防抖动，可以将抖动较小的乱序包恢复成有序包
      
    NetEQ
      专用于音频控制，里面包括了JitterBufer
      除此之外，它还可以利用音频的变速不变调机制将积攒的音频数据快速播放或将不足的音频拉长播放，以实现音频的防抖动
      
    拥塞控制
      这部分内容很丰富，略
      
      
五 快速准确评估宽带
  网络质量提升的前提是网络没有发生拥塞，而为了防止发生网络拥塞，直播客户端就要有快速、准确的评估宽带的方法
  
  在实时通信领域，有四种常见的宽带评估方法，分别是：Goog-REMB、Goog-TCC、NADA、SCReAM
  他们对网络宽带的评估各有优劣
  但整体上来看Google最新的宽带评估算法Goog-TCC是最优的


















