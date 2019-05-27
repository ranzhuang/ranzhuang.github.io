---
title: iOS简单实用的手写签名实现
date: 2018-11-26
tags: Object-C
---

在iOS中实现手写签名主要运用到的知识点：UIBezierPath、手势、图片的截取、图片的压缩

### 签名封装

新建"HJSignatureView"继承自UIView，在分装类的.h文件中提供"清除签名"与"保存签名"方法，.m文件中实现签名的具体封装

.h文件

    /**
     清除签名
     */
    - (void)clear;



    /**
     保存签名

     @return 保存在本地的图片路径
     */
    - (NSString *)saveTheSignatureWithError:(void(^)(NSString *errorMsg))errorBlock;

.m文件所提供的属性与懒加载，导入头文件Photos、UIImage图片压缩的分类头文件

    #import "HJSignatureView.h"
    #import <Photos/Photos.h>
    #import "UIImage+HJImage.h"

    @interface HJSignatureView()
    @property (nonatomic, strong) UIBezierPath *signaturePath;
    @property (nonatomic, assign) CGPoint oldPoint;

    /**
     是否有签名
     */
    @property (nonatomic, assign) BOOL isHaveSignature;
    //绘画区域
    @property (nonatomic, assign) CGFloat minX;
    @property (nonatomic, assign) CGFloat minY;
    @property (nonatomic, assign) CGFloat maxX;
    @property (nonatomic, assign) CGFloat maxY;
    @end    
    @implementation HJSignatureView

    //......其他代码

    #pragma mark - set & get
    - (UIBezierPath *)signaturePath {
        if (!_signaturePath) {
            _signaturePath = [UIBezierPath bezierPath];
        }
        return _signaturePath;
    }

    @end

重写初始化方法与drawRect方法，实现视图基本的配置功能

    - (instancetype)initWithFrame:(CGRect)frame
    {
        self = [super initWithFrame:frame];
        if (self) {
            [self config];
        }
        return self;
    }

    - (instancetype)initWithCoder:(NSCoder *)aDecoder {
        self = [super initWithCoder:aDecoder];
        if (self) {
           [self config];
        }
        return self;
    }

    - (void)drawRect:(CGRect)rect {
        self.signaturePath.lineWidth = 2;
        [[UIColor blackColor] setStroke];
        [self.signaturePath stroke];
    }

    #pragma mark - 基础配置
    - (void)config {
        self.backgroundColor = [UIColor whiteColor];
        self.oldPoint = CGPointZero;
        self.isHaveSignature = NO;
        self.minX = 0;
        self.maxX = 0;
        self.minY = 0;
        self.maxY = 0;
    }

重写 touchesBegan 与 touchesMoved方法，在手势中设置贝塞尔曲线路径，设置最大最小值、签名是否存在、点等属性值，最后记住调用setNeedsDisplay方法

    - (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
        UITouch *touch = [touches anyObject];
        CGPoint currentPoint = [touch locationInView:self];
        [self.signaturePath moveToPoint:currentPoint];
        self.oldPoint = currentPoint;
    }

    - (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
        UITouch *touch = [touches anyObject];
        CGPoint currentPoint = [touch locationInView:self];
        [self.signaturePath addQuadCurveToPoint:currentPoint controlPoint:self.oldPoint];
        self.oldPoint = currentPoint;
        //设置剪切图片的区域
        [self getImageRect:currentPoint];
        //设置签名存在
        if (!self.isHaveSignature) {
            self.isHaveSignature = YES;
        }
        [self setNeedsDisplay];
    }

    - (void)getImageRect:(CGPoint)currentPoint {
        if (self.maxX == 0 && self.minX == 0) {
            self.maxX = currentPoint.x;
            self.minX = currentPoint.x;
        } else {
            if (self.maxX <= currentPoint.x) {
                self.maxX = currentPoint.x;
            }
            if (self.minX >= currentPoint.x) {
                self.minX = currentPoint.x;
            }
        }
        if (self.maxY == 0 && self.minY == 0) {
            self.maxY = self.minY = currentPoint.y;
        } else {
            if (self.maxY <= currentPoint.y) {
                self.maxY = currentPoint.y;
            }
            if (self.minY >= currentPoint.y) {
                self.minY = currentPoint.y;
            }
        }

    }

实现.h提供的清除手势与确认手势方法

    #pragma mark - 清理
    - (void)clear {
        self.signaturePath = [UIBezierPath bezierPath];
        self.isHaveSignature = NO;
        self.oldPoint = CGPointZero;
        self.minX = 0;
        self.maxX = 0;
        self.minY = 0;
        self.maxY = 0;
        [self setNeedsDisplay];
    }

    #pragma mark - 确认
    - (NSString *)saveTheSignatureWithError:(void(^)(NSString *errorMsg))errorBlock {
        if (self.isHaveSignature) {
            //开启上下文
            UIGraphicsBeginImageContextWithOptions(self.bounds.size,NO, [UIScreen mainScreen].scale);
            //获取上下文
            CGContextRef ref = UIGraphicsGetCurrentContext();
            //截图
            [self.layer drawInContext:ref];
            //获取整个视图图片
            UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
            //关闭上下文
            UIGraphicsEndImageContext();
            //获取绘画区域图片
            //需要获取图片真正的像素，求出比例
            NSInteger scale = CGImageGetHeight(image.CGImage) / image.size.height;
            //区域
            CGFloat space = 4;
            CGFloat x = self.minX * scale - space;
            CGFloat y = self.minY * scale - space;
            CGFloat width = (self.maxX - self.minX + space) * scale;
            CGFloat height = (self.maxY - self.minY + space) * scale;
            UIImage *drawImage = [UIImage imageWithCGImage:CGImageCreateWithImageInRect(image.CGImage, CGRectMake(x, y, width, height))];
            //压缩图片 将图片压缩到100KB以内
            NSData *imageData = [drawImage compressImageToSize:100 * 1024];
            //保存到本地
            NSString *path = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES).firstObject;
            NSString *fileName = [NSUUID UUID].UUIDString;
            NSString *filePath = [path stringByAppendingPathComponent:[NSString stringWithFormat:@"%@.png",fileName]];
            [imageData writeToFile:filePath atomically:YES];
            //清除签名
            [self clear];
            return filePath;
        } else {
            if (errorBlock) {
                errorBlock(@"签名不存在");
            }
            return nil;
        }

    }

### 图片压缩

创建名为"UIImage+HJImage"的UIImage分类，提供压缩图片到指定大小的方法"compressImageToSize"。先对图片采用二分法进行优化循环的质量压缩，可以保证图片的清晰度；如果未达到所需要的大小，再对图片进行大小的压缩，图片的大小压缩可能会降低清晰度，但会确保能够得到想要的大小。这种压缩方式也是图片压缩较优的一种选择

.h方法

    /**
     压缩图片到指定大小

     @param maxSize 例如100 * 1024  100kb以内
     @return data
     */
    - (NSData *)compressImageToSize:(NSInteger)maxSize;

.m方法实现

    #pragma mark - 压缩图片到指定大小
    - (NSData *)compressImageToSize:(NSInteger)maxSize {
        CGFloat compress = 1;
        NSData *data = UIImageJPEGRepresentation(self, compress);
        //如果图片本身大小小于maxSize就直接返回
        NSLog(@"------------%.1lu KB", data.length / 1024);
        if (data.length < maxSize) return data;
        //压缩图片的质量优点在于可以更好的保留图片的清晰度，缺点在于可能我们compress继续
        //减小，data也不会再减小，不能保证压缩后小于指定的大小
        //为了快速压缩我们使用二分法来进行优化循环
        CGFloat max = 1;
        CGFloat min = 0;
        for (int i = 0; i < 6; i++) {
            compress = (max + min) / 2;
            data = UIImageJPEGRepresentation(self, compress);
            //如果data.length小于maxSize * 0.9 代表data.length过小那么就让最小数变大
            //此处为何是0.9，因为我们需要返回data的大小在出入指定大小maxSize在90% ~ 100%
            if (data.length < maxSize * 0.9) {
                min = compress;
            } else if (data.length > maxSize) {
                //如果data.length大于maxSize 代表data.length过大那么就让最大数变小
                max = compress;
            } else {
                break;
            }
        }

        //当对质量进行压缩并没达到我们需要的大小时我们可以对图片大小进行压缩
        //压缩图片尺寸会达到我们理想的大小，但会让图片变得相对模糊
        //判断质量压缩后的大小是否小于maxSize
        NSLog(@"------------%.1lu KB", data.length / 1024);
        if (data.length < maxSize) return data;
        UIImage *image = [UIImage imageWithData:data];
        NSInteger lastLength = 0;
        while (data.length > maxSize && data.length != lastLength) {
            lastLength = data.length;
            //计算比例
            CGFloat ratio = (CGFloat)maxSize / data.length;
            //计算在比例下的size，注意要将宽高转变成整数，否则可能会出现图片百边的情况
            CGSize size = CGSizeMake((NSInteger)(image.size.width * sqrtf(ratio)), (NSInteger)(image.size.height * sqrtf(ratio)));
            UIGraphicsBeginImageContext(size);
            [image drawInRect:CGRectMake(0, 0, size.width, size.height)];
            image = UIGraphicsGetImageFromCurrentImageContext();
            UIGraphicsEndImageContext();
            data = UIImageJPEGRepresentation(image, 1);
            NSLog(@"------------%.1lu KB", data.length / 1024);
        }
        NSLog(@"------------%.1lu KB", data.length / 1024);
        return data;
    }


> 示例项目
[手写签名Demo](https://coding.net/u/Ranzhuang/p/HJSignatureDemo/git)
