---
title: iOS直播技术分享-视频编码（三）
date: 2016-07-11 14:06:46
categories: 音视频
tags: [音视频,iOS]
---
x264是一种免费的、具有更优秀算法的符合H.264/MPEG-4 AVC视频压缩编码标准格式的编码库。它同xvid一样都是开源项目，但x264是采用H.264标准的，而xvid是采用MPEG-4早期标准的。由于H.264是2003年正式发布的最新的视频编码标准，因此，在通常情况下，x264压缩出的视频文件在相同质量下要比xvid压缩出的文件要小，或者也可以说，在相同体积下比xvid压缩出的文件质量要好。它符合GPL许可证。 

iOS视频编码分为硬编码和软编码：硬编码就是利用手机专用的硬件进行编码，软编码是用CPU进行编码。由于苹果在iOS8开放的硬编码的API，故现在大多数的直播应用都是采用的硬编码。
## iOS硬编码

从iOS8开始，苹果开放了硬解码和硬编码API，框架为 VideoToolbox.framework， 此框架需要在iOS8及以上的系统上才能使用。  

此框架中的硬解码API是几个纯C函数，在任何OC或者 C++代码里都可以使用。使用的时候，首先，要把VideoToolbox.framework 添加到工程里，并且在要使用该API的文件中包含头文件#include <VideoToolbox/VideoToolbox.h>，然后，就可以畅快的高效的对视频流进行硬编码了。

直接上代码来说明，首先是定义了编码所需的变量
```
@interface CLHardwareVideoEncoder (){
    VTCompressionSessionRef compressionSession;
    NSInteger frameCount;
    NSData *sps;
    NSData *pps;
    FILE *fp;
    BOOL enabledWriteVideoFile;
}

@property (nonatomic, strong) CLLiveVideoConfiguration *configuration;
@property (nonatomic,weak) id<CLVideoEncodingDelegate> h264Delegate;
@property (nonatomic) BOOL isBackGround;
@property (nonatomic) NSInteger currentVideoBitRate;

@end
```

初始化编码session
```
- (void)initCompressionSession{
    if(compressionSession){
        VTCompressionSessionCompleteFrames(compressionSession, kCMTimeInvalid);
        
        VTCompressionSessionInvalidate(compressionSession);
        CFRelease(compressionSession);
        compressionSession = NULL;
    }
    
    OSStatus status = VTCompressionSessionCreate(NULL, _configuration.videoSize.width, _configuration.videoSize.height, kCMVideoCodecType_H264, NULL, NULL, NULL, VideoCompressonOutputCallback, (__bridge void *)self, &compressionSession);
    if(status != noErr){
        return;
    }
    
    _currentVideoBitRate = _configuration.videoBitRate;
    VTSessionSetProperty(compressionSession, kVTCompressionPropertyKey_MaxKeyFrameInterval,(__bridge CFTypeRef)@(_configuration.videoMaxKeyframeInterval));
    VTSessionSetProperty(compressionSession, kVTCompressionPropertyKey_MaxKeyFrameIntervalDuration,(__bridge CFTypeRef)@(_configuration.videoMaxKeyframeInterval));
    VTSessionSetProperty(compressionSession, kVTCompressionPropertyKey_ExpectedFrameRate, (__bridge CFTypeRef)@(_configuration.videoFrameRate));
    VTSessionSetProperty(compressionSession, kVTCompressionPropertyKey_AverageBitRate, (__bridge CFTypeRef)@(_configuration.videoBitRate));
    NSArray *limit = @[@(_configuration.videoBitRate * 1.5/8),@(1)];
    VTSessionSetProperty(compressionSession, kVTCompressionPropertyKey_DataRateLimits, (__bridge CFArrayRef)limit);
    VTSessionSetProperty(compressionSession, kVTCompressionPropertyKey_RealTime, kCFBooleanFalse);
    VTSessionSetProperty(compressionSession, kVTCompressionPropertyKey_ProfileLevel, kVTProfileLevel_H264_Main_AutoLevel);
    VTSessionSetProperty(compressionSession, kVTCompressionPropertyKey_AllowFrameReordering, kCFBooleanFalse);
    VTSessionSetProperty(compressionSession, kVTCompressionPropertyKey_H264EntropyMode, kVTH264EntropyMode_CABAC);
    VTCompressionSessionPrepareToEncodeFrames(compressionSession);
}
```

编码输入
```
- (void)encodeVideoData:(CVImageBufferRef)pixelBuffer timeStamp:(uint64_t)timeStamp{
    if(_isBackGround) return;
    
    frameCount ++;
    CMTime presentationTimeStamp = CMTimeMake(frameCount, (int32_t)_configuration.videoFrameRate);
    VTEncodeInfoFlags flags;
    CMTime duration = CMTimeMake(1, (int32_t)_configuration.videoFrameRate);
    
    NSDictionary *properties = nil;
    if(frameCount % (int32_t)_configuration.videoMaxKeyframeInterval == 0){
        properties = @{(__bridge NSString *)kVTEncodeFrameOptionKey_ForceKeyFrame: @YES};
    }
    NSNumber *timeNumber = @(timeStamp);
    
    VTCompressionSessionEncodeFrame(compressionSession, pixelBuffer, presentationTimeStamp, duration, (__bridge CFDictionaryRef)properties, (__bridge_retained void *)timeNumber, &flags);
}
```

回调
```
static void VideoCompressonOutputCallback(void *VTref, void *VTFrameRef, OSStatus status, VTEncodeInfoFlags infoFlags, CMSampleBufferRef sampleBuffer)
{
    if(!sampleBuffer) return;
    CFArrayRef array = CMSampleBufferGetSampleAttachmentsArray(sampleBuffer, true);
    if(!array) return;
    CFDictionaryRef dic = (CFDictionaryRef)CFArrayGetValueAtIndex(array, 0);
    if(!dic) return;
    
    BOOL keyframe = !CFDictionaryContainsKey(dic, kCMSampleAttachmentKey_NotSync);
    uint64_t timeStamp = [((__bridge_transfer NSNumber*)VTFrameRef) longLongValue];
    
    CLHardwareVideoEncoder *videoEncoder = (__bridge CLHardwareVideoEncoder *)VTref;
    if(status != noErr){
        return;
    }
    
    if (keyframe && !videoEncoder->sps)
    {
        CMFormatDescriptionRef format = CMSampleBufferGetFormatDescription(sampleBuffer);
        
        size_t sparameterSetSize, sparameterSetCount;
        const uint8_t *sparameterSet;
        OSStatus statusCode = CMVideoFormatDescriptionGetH264ParameterSetAtIndex(format, 0, &sparameterSet, &sparameterSetSize, &sparameterSetCount, 0 );
        if (statusCode == noErr)
        {
            size_t pparameterSetSize, pparameterSetCount;
            const uint8_t *pparameterSet;
            OSStatus statusCode = CMVideoFormatDescriptionGetH264ParameterSetAtIndex(format, 1, &pparameterSet, &pparameterSetSize, &pparameterSetCount, 0 );
            if (statusCode == noErr)
            {
                videoEncoder->sps = [NSData dataWithBytes:sparameterSet length:sparameterSetSize];
                videoEncoder->pps = [NSData dataWithBytes:pparameterSet length:pparameterSetSize];
                
                if(videoEncoder->enabledWriteVideoFile){
                    NSMutableData *data = [[NSMutableData alloc] init];
                    uint8_t header[] = {0x00,0x00,0x00,0x01};
                    [data appendBytes:header length:4];
                    [data appendData:videoEncoder->sps];
                    [data appendBytes:header length:4];
                    [data appendData:videoEncoder->pps];
                    fwrite(data.bytes, 1,data.length,videoEncoder->fp);
                }
            }
        }
    }
    
    
    CMBlockBufferRef dataBuffer = CMSampleBufferGetDataBuffer(sampleBuffer);
    size_t length, totalLength;
    char *dataPointer;
    OSStatus statusCodeRet = CMBlockBufferGetDataPointer(dataBuffer, 0, &length, &totalLength, &dataPointer);
    if (statusCodeRet == noErr) {
        size_t bufferOffset = 0;
        static const int AVCCHeaderLength = 4;
        while (bufferOffset < totalLength - AVCCHeaderLength) {
            // Read the NAL unit length
            uint32_t NALUnitLength = 0;
            memcpy(&NALUnitLength, dataPointer + bufferOffset, AVCCHeaderLength);
            
            NALUnitLength = CFSwapInt32BigToHost(NALUnitLength);

            CLVideoFrame *videoFrame = [CLVideoFrame new];
            videoFrame.timestamp = timeStamp;
            videoFrame.data = [[NSData alloc] initWithBytes:(dataPointer + bufferOffset + AVCCHeaderLength) length:NALUnitLength];
            videoFrame.isKeyFrame = keyframe;
            videoFrame.sps = videoEncoder->sps;
            videoFrame.pps = videoEncoder->pps;
            
            if(videoEncoder.h264Delegate && [videoEncoder.h264Delegate respondsToSelector:@selector(videoEncoder:videoFrame:)]){
                [videoEncoder.h264Delegate videoEncoder:videoEncoder videoFrame:videoFrame];
            }
            
            if(videoEncoder->enabledWriteVideoFile){
                NSMutableData *data = [[NSMutableData alloc] init];
                if(keyframe){
                    uint8_t header[] = {0x00,0x00,0x00,0x01};
                    [data appendBytes:header length:4];
                }else{
                    uint8_t header[] = {0x00,0x00,0x01};
                    [data appendBytes:header length:3];
                }
                [data appendData:videoFrame.data];
                fwrite(data.bytes, 1,data.length,videoEncoder->fp);
            }
            bufferOffset += AVCCHeaderLength + NALUnitLength;
        }
    }
}
```

## iOS软编码
软编码主要是利用CPU进行编码的过程, 具体的编码通常会用FFmpeg+x264，需要自己先编译FFmpeg(iOS)和X264。

- 将编译好的文件夹拖入到工程中
- 添加依赖库:
  libiconv.dylib/libz.dylib/libbz2.dylib/CoreMedia.framework/AVFoundation.framework
- FFmpeg编码两个重要的类
- AVFormat
  保存的是解码后和原始的音视频信息
- AVPacket
  解码完成的数据及附加信息（解码时间戳、显示时间戳、时长等）
  
```
/*
 *  设置X264
 */
- (int)setX264ResourceWithVideoWidth:(int)width height:(int)height bitrate:(int)bitrate
{
    // 1.默认从第0帧开始(记录当前的帧数)
    framecnt = 0;

    // 2.记录传入的宽度&高度
    encoder_h264_frame_width = width;
    encoder_h264_frame_height = height;

    // 3.注册FFmpeg所有编解码器(无论编码还是解码都需要该步骤)
    av_register_all();

    // 4.初始化AVFormatContext: 用作之后写入视频帧并编码成 h264，贯穿整个工程当中(释放资源时需要销毁)
    pFormatCtx = avformat_alloc_context();

    // 5.设置输出文件的路径
    fmt = av_guess_format(NULL, out_file, NULL);
    pFormatCtx->oformat = fmt;

    // 6.打开文件的缓冲区输入输出，flags 标识为  AVIO_FLAG_READ_WRITE ，可读写
    if (avio_open(&pFormatCtx->pb, out_file, AVIO_FLAG_READ_WRITE) < 0){
        printf("Failed to open output file! \n");
        return -1;
    }

    // 7.创建新的输出流, 用于写入文件
    video_st = avformat_new_stream(pFormatCtx, 0);

    // 8.设置 20 帧每秒 ，也就是 fps 为 20
    video_st->time_base.num = 1;
    video_st->time_base.den = 25;

    if (video_st==NULL){
        return -1;
    }

    // 9.pCodecCtx 用户存储编码所需的参数格式等等
    // 9.1.从媒体流中获取到编码结构体，他们是一一对应的关系，一个 AVStream 对应一个  AVCodecContext
    pCodecCtx = video_st->codec;

    // 9.2.设置编码器的编码格式(是一个id)，每一个编码器都对应着自己的 id，例如 h264 的编码 id 就是 AV_CODEC_ID_H264
    pCodecCtx->codec_id = fmt->video_codec;

    // 9.3.设置编码类型为 视频编码
    pCodecCtx->codec_type = AVMEDIA_TYPE_VIDEO;

    // 9.4.设置像素格式为 yuv 格式
    pCodecCtx->pix_fmt = PIX_FMT_YUV420P;

    // 9.5.设置视频的宽高
    pCodecCtx->width = encoder_h264_frame_width;
    pCodecCtx->height = encoder_h264_frame_height;

    // 9.6.设置帧率
    pCodecCtx->time_base.num = 1;
    pCodecCtx->time_base.den = 15;

    // 9.7.设置码率（比特率）
    pCodecCtx->bit_rate = bitrate;

    // 9.8.视频质量度量标准(常见qmin=10, qmax=51)
    pCodecCtx->qmin = 10;
    pCodecCtx->qmax = 51;

    // 9.9.设置图像组层的大小(GOP-->两个I帧之间的间隔)
    pCodecCtx->gop_size = 250;

    // 9.10.设置 B 帧最大的数量，B帧为视频图片空间的前后预测帧， B 帧相对于 I、P 帧来说，压缩率比较大，也就是说相同码率的情况下，
    // 越多 B 帧的视频，越清晰，现在很多打视频网站的高清视频，就是采用多编码 B 帧去提高清晰度，
    // 但同时对于编解码的复杂度比较高，比较消耗性能与时间
    pCodecCtx->max_b_frames = 5;

    // 10.可选设置
    AVDictionary *param = 0;
    // H.264
    if(pCodecCtx->codec_id == AV_CODEC_ID_H264) {
        // 通过--preset的参数调节编码速度和质量的平衡。
        av_dict_set(&param, "preset", "slow", 0);

        // 通过--tune的参数值指定片子的类型，是和视觉优化的参数，或有特别的情况。
        // zerolatency: 零延迟，用在需要非常低的延迟的情况下，比如视频直播的编码
        av_dict_set(&param, "tune", "zerolatency", 0);
    }

    // 11.输出打印信息，内部是通过printf函数输出（不需要输出可以注释掉该局）
    av_dump_format(pFormatCtx, 0, out_file, 1);

    // 12.通过 codec_id 找到对应的编码器
    pCodec = avcodec_find_encoder(pCodecCtx->codec_id);
    if (!pCodec) {
        printf("Can not find encoder! \n");
        return -1;
    }

    // 13.打开编码器，并设置参数 param
    if (avcodec_open2(pCodecCtx, pCodec,&param) < 0) {
        printf("Failed to open encoder! \n");
        return -1;
    }

    // 13.初始化原始数据对象: AVFrame
    pFrame = av_frame_alloc();

    // 14.通过像素格式(这里为 YUV)获取图片的真实大小，例如将 480 * 720 转换成 int 类型
    avpicture_fill((AVPicture *)pFrame, picture_buf, pCodecCtx->pix_fmt, pCodecCtx->width, pCodecCtx->height);

    // 15.h264 封装格式的文件头部，基本上每种编码都有着自己的格式的头部，想看具体实现的同学可以看看 h264 的具体实现
    avformat_write_header(pFormatCtx, NULL);

    // 16.创建编码后的数据 AVPacket 结构体来存储 AVFrame 编码后生成的数据
    av_new_packet(&pkt, picture_size);

    // 17.设置 yuv 数据中 y 图的宽高
    y_size = pCodecCtx->width * pCodecCtx->height;

    return 0;
}
```
- 编码每一帧数据
```
/*
 * 将CMSampleBufferRef格式的数据编码成h264并写入文件
 *
 */
- (void)encoderToH264:(CMSampleBufferRef)sampleBuffer
{
    // 1.通过CMSampleBufferRef对象获取CVPixelBufferRef对象
    CVPixelBufferRef imageBuffer = CMSampleBufferGetImageBuffer(sampleBuffer);

    // 2.锁定imageBuffer内存地址开始进行编码
    if (CVPixelBufferLockBaseAddress(imageBuffer, 0) == kCVReturnSuccess) {
        // 3.从CVPixelBufferRef读取YUV的值
        // NV12和NV21属于YUV格式，是一种two-plane模式，即Y和UV分为两个Plane，但是UV（CbCr）为交错存储，而不是分为三个plane
        // 3.1.获取Y分量的地址
        UInt8 *bufferPtr = (UInt8 *)CVPixelBufferGetBaseAddressOfPlane(imageBuffer,0);
        // 3.2.获取UV分量的地址
        UInt8 *bufferPtr1 = (UInt8 *)CVPixelBufferGetBaseAddressOfPlane(imageBuffer,1);

        // 3.3.根据像素获取图片的真实宽度&高度
        size_t width = CVPixelBufferGetWidth(imageBuffer);
        size_t height = CVPixelBufferGetHeight(imageBuffer);
        // 获取Y分量长度
        size_t bytesrow0 = CVPixelBufferGetBytesPerRowOfPlane(imageBuffer,0);
        size_t bytesrow1  = CVPixelBufferGetBytesPerRowOfPlane(imageBuffer,1);
        UInt8 *yuv420_data = (UInt8 *)malloc(width * height *3/2);

        /* convert NV12 data to YUV420*/
        // 3.4.将NV12数据转成YUV420数据
        UInt8 *pY = bufferPtr ;
        UInt8 *pUV = bufferPtr1;
        UInt8 *pU = yuv420_data + width*height;
        UInt8 *pV = pU + width*height/4;
        for(int i =0;i<height;i++)
        {
            memcpy(yuv420_data+i*width,pY+i*bytesrow0,width);
        }
        for(int j = 0;j<height/2;j++)
        {
            for(int i =0;i<width/2;i++)
            {
                *(pU++) = pUV[i<<1];
                *(pV++) = pUV[(i<<1) + 1];
            }
            pUV+=bytesrow1;
        }

        // 3.5.分别读取YUV的数据
        picture_buf = yuv420_data;
        pFrame->data[0] = picture_buf;              // Y
        pFrame->data[1] = picture_buf+ y_size;      // U
        pFrame->data[2] = picture_buf+ y_size*5/4;  // V

        // 4.设置当前帧
        pFrame->pts = framecnt;
        int got_picture = 0;

        // 4.设置宽度高度以及YUV各式
        pFrame->width = encoder_h264_frame_width;
        pFrame->height = encoder_h264_frame_height;
        pFrame->format = PIX_FMT_YUV420P;

        // 5.对编码前的原始数据(AVFormat)利用编码器进行编码，将 pFrame 编码后的数据传入pkt 中
        int ret = avcodec_encode_video2(pCodecCtx, &pkt, pFrame, &got_picture);
        if(ret < 0) {
            printf("Failed to encode! \n");

        }

        // 6.编码成功后写入 AVPacket 到 输入输出数据操作着 pFormatCtx 中，当然，记得释放内存
        if (got_picture==1) {
            framecnt++;
            pkt.stream_index = video_st->index;
            ret = av_write_frame(pFormatCtx, &pkt);
            av_free_packet(&pkt);
        }

        // 7.释放yuv数据
        free(yuv420_data);
    }

    CVPixelBufferUnlockBaseAddress(imageBuffer, 0);
}
```
- 释放资源
```
/*
 * 释放资源
 */
- (void)freeX264Resource
{
    // 1.释放AVFormatContext
    int ret = flush_encoder(pFormatCtx,0);
    if (ret < 0) {
        printf("Flushing encoder failed\n");
    }

    // 2.将还未输出的AVPacket输出出来
    av_write_trailer(pFormatCtx);

    // 3.关闭资源
    if (video_st){
        avcodec_close(video_st->codec);
        av_free(pFrame);
    }
    avio_close(pFormatCtx->pb);
    avformat_free_context(pFormatCtx);
}

int flush_encoder(AVFormatContext *fmt_ctx,unsigned int stream_index)
{
    int ret;
    int got_frame;
    AVPacket enc_pkt;
    if (!(fmt_ctx->streams[stream_index]->codec->codec->capabilities &
          CODEC_CAP_DELAY))
        return 0;

    while (1) {
        enc_pkt.data = NULL;
        enc_pkt.size = 0;
        av_init_packet(&enc_pkt);
        ret = avcodec_encode_video2 (fmt_ctx->streams[stream_index]->codec, &enc_pkt,
                                     NULL, &got_frame);
        av_frame_free(NULL);
        if (ret < 0)
            break;
        if (!got_frame){
            ret=0;
            break;
        }
        ret = av_write_frame(fmt_ctx, &enc_pkt);
        if (ret < 0)
            break;
    }
    return ret;
}
```
