---
title: Core Image人脸检测
date: 2016-05-17 20:04:34
categories: iOS
tags: [iOS]
---
此次iOS5的一个新特性就是提供了人脸检测的API，这也是被媒体关注的一个功能，基本上，我看到的报道都是说iOS5提供了人脸识别的功能，然后又是设想要通过人脸来实现解锁屏幕等等，如何如何的。一开始，我也以为iOS5确实提供了这样的功能，这意味着可能不用opencv等静态库来实现了，免去了一旦OS版本升级要重新编译静态库的麻烦。研究了几天，发现并不是这么回事。  
首先，此次iOS5提供的人脸检测API是Core Image这个Framework中，Core Image实际上在Mac OS下早就有了，只不过这次才引入到iOS5中。但是，apple还是比较谨慎的，并没有引入Core Image所有的功能到iOS下，只是提供了少数的API而已，而且可以看出基本都是对系统性能要求不怎么高的功能，可能apple对手机上做图像处理的性能问题还是有些顾虑的。例如，Core Image下图形处理的上百个filter，到了iOS下就只剩下20来个了，关键的轮廓提取，高斯模糊等filter都没有提供，很多复杂的处理都无法直接用Core Image来实现。但我相信随着iOS设备硬件性能的不断提上，开放所有的功能应该是迟早的事。  
其次，谈谈人脸识别。严格的说，此次提供的API并不能叫人脸识别（face recognition），只能叫人脸检测（face detection），这二者在图像处理领域是有很大差别的，作用和实现难度都是天差地别。简单的说，人脸检测就是检测出图像中是否包含人脸，此次apple提供的API就是这个功能，可以给你指出图像中每一个人脸的位置，还有人脸中眼睛、嘴巴的位置。而人脸识别则是更加高级的技术，可以告诉你几张照片中的人是不是同一人。是不是觉得在哪见过，没错，就是iPhoto里提供的功能。对apple来说，这样的技术也应该不是什么难事，关键是什么时候可以开放给开发者的问题了。  
最后，当然要谈谈怎么用了。具体的使用，其实很简单，看看后面的代码你就明白了，如果要实现一些高级的应用，目前来说可能还得结合opencv或image-processing等开源的处理库来用。但如果只是简单的人脸检测，Core Image的效率就我个人感觉来说，还是很不错的，不比opencv的差。

人脸检测代码：

```
-(void)DetectFace {
    UIImage* image = [UIImage imageNamed:@"face.png"];
    UIImageView *testImage = [[UIImageView alloc] initWithImage: image];
    [testImage setTransform:CGAffineTransformMakeScale(1, -1)];
    [[[UIApplication sharedApplication] delegate].window setTransform:CGAffineTransformMakeScale(1, -1)];
    [testImage setFrame:CGRectMake(0, 0, testImage.image.size.width, testImage.image.size.height)];
    [self.view addSubview:testImage];
    
    CIImage* ciimage = [CIImage imageWithCGImage:image.CGImage];
    NSDictionary* opts = [NSDictionary dictionaryWithObject:CIDetectorAccuracyHigh
                                                     forKey:CIDetectorAccuracy];
    CIDetector* detector = [CIDetector detectorOfType:CIDetectorTypeFace
                                              context:nil options:opts];
    NSArray* features = [detector featuresInImage:ciimage];
    
    for (CIFaceFeature *faceFeature in features) {
        CGFloat faceWidth = faceFeature.bounds.size.width;
        
        // create a UIView using the bounds of the face
        UIView* faceView = [[UIView alloc] initWithFrame:faceFeature.bounds];
        
        // add a border around the newly created UIView
        faceView.layer.borderWidth = 1;
        faceView.layer.borderColor = [[UIColor redColor] CGColor];
        
        [self.view addSubview:faceView];
        
        if(faceFeature.hasLeftEyePosition) {
            // create a UIView with a size based on the width of the face
            UIView* leftEyeView = [[UIView alloc] initWithFrame:CGRectMake(faceFeature.leftEyePosition.x-faceWidth*0.15, faceFeature.leftEyePosition.y-faceWidth*0.15, faceWidth*0.3, faceWidth*0.3)];
            
            // change the background color of the eye view
            [leftEyeView setBackgroundColor:[[UIColor blueColor] colorWithAlphaComponent:0.3]];
            
            // set the position of the leftEyeView based on the face
            [leftEyeView setCenter:faceFeature.leftEyePosition];
            
            // round the corners
            leftEyeView.layer.cornerRadius = faceWidth*0.15;
            
            // add the view to the window
            [self.view  addSubview:leftEyeView];
            
        }
        
        if(faceFeature.hasRightEyePosition) {
            // create a UIView with a size based on the width of the face
            UIView* leftEye = [[UIView alloc] initWithFrame:CGRectMake(faceFeature.rightEyePosition.x-faceWidth*0.15, faceFeature.rightEyePosition.y-faceWidth*0.15, faceWidth*0.3, faceWidth*0.3)];
            
            // change the background color of the eye view
            [leftEye setBackgroundColor:[[UIColor blueColor] colorWithAlphaComponent:0.3]];
            
            // set the position of the rightEyeView based on the face
            [leftEye setCenter:faceFeature.rightEyePosition];
            
            // round the corners
            leftEye.layer.cornerRadius = faceWidth*0.15;
            
            // add the new view to the window
            [self.view  addSubview:leftEye];
        }
        
        if(faceFeature.hasMouthPosition) {
            // create a UIView with a size based on the width of the face
            UIView* mouth = [[UIView alloc] initWithFrame:CGRectMake(faceFeature.mouthPosition.x-faceWidth*0.2, faceFeature.mouthPosition.y-faceWidth*0.2, faceWidth*0.4, faceWidth*0.4)];
            
            // change the background color for the mouth to green
            [mouth setBackgroundColor:[[UIColor greenColor] colorWithAlphaComponent:0.3]];
            
            // set the position of the mouthView based on the face
            [mouth setCenter:faceFeature.mouthPosition];
            
            // round the corners
            mouth.layer.cornerRadius = faceWidth*0.2;
            
            // add the new view to the window
            [self.view  addSubview:mouth];
        }       
    }
}
```
