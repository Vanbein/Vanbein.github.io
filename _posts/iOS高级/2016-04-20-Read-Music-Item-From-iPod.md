---
layout: post
title: iOS 获取 iPod 音乐库歌曲文件信息并拷贝文件到个人的 App 中
category: iOS高级
tags: iPod
image: http://o7rxin1of.qnssl.com/images/head-800x400/-57.png
description: 最近有需要从 iOS 系统的音乐库里面拷贝歌曲到 APP 的 Documents 目录下进行共享使用，于是总结一下使用的方法。其中使用了 MPMediaQuery 来获得音乐文件的信息，比如歌手、专辑名、专辑封面、时长、 ASSetURL 地址等，然后通过 ASSetURL 地址使用 AVAssetExportSession 导出其数据。
homepage: false
toc: true
---

最近有需要从 iOS 系统的音乐库里面拷贝歌曲到 APP 的 Documents 目录下进行共享使用，于是总结一下使用的方法。其中使用了 MPMediaQuery 来获得音乐文件的信息，比如歌手、专辑名、专辑封面、时长、 ASSetURL 地址等，然后通过 ASSetURL 地址使用 AVAssetExportSession 导出其数据。

### 一. 访问 MediaLibrary

[Apple官方文档](https://developer.apple.com/library/ios/documentation/Audio/Conceptual/iPodLibraryAccess_Guide/AboutiPodLibraryAccess/AboutiPodLibraryAccess.html#//apple_ref/doc/uid/TP40008765-CH103-SW9)指出，访问iPod Library的方法有两种，分别是 **MediaPicker** 和 **MediaQuery**。

#### MediaPicker

MediaPicker是一个高度封装的iPod Library访问方式，通过使用 `MPMediaPickerController`类来访问iPod Library。这是一个UI控件，用户可以根据需要选择其中的音乐。这个类使用时非常方便，只需要生成一个 `MPMediaPickerController` 的实例，设置一下属性和 delegate 后 present 出来，接下来只要等待回调即可，在回调时需要手动 dismiss picker。

{% highlight objc  %}
- (void)convertMediaPickerController{

    //MPMediaPicker
    MPMediaPickerController *picker = [[MPMediaPickerController alloc] initWithMediaTypes:MPMediaTypeAnyAudio];
    picker.prompt = @"请选择需要播放的歌曲";
    picker.showsCloudItems = NO;
    picker.allowsPickingMultipleItems = YES;
    picker.delegate = self;
    [self presentViewController:picker animated:YES completion:nil];
}

#pragma mark - MPMediaPicker Controller Delegate

- (void)mediaPickerDidCancel:(MPMediaPickerController *)mediaPicker
{
    [mediaPicker dismissViewControllerAnimated:YES completion:nil];
}

- (void)mediaPicker:(MPMediaPickerController *)mediaPicker didPickMediaItems:(MPMediaItemCollection *)mediaItemCollection
{
    [mediaPicker dismissViewControllerAnimated:YES completion:nil];
    //do something
}
{% endhighlight %}

得到的效果图如下：

![MPMediaPickerController](http://o7rxin1of.qnssl.com/images/2016/04/MPMediaPickerController.png)

通过 MediaPicker 最终可以得到 MPMediaItemCollection，其中存放着所有在Picker中选中的歌曲，每一个歌曲使用一个 `MPMediaItem` 对象表示。对于MediaPicker的使用也可以参考[Apple官方文档](https://developer.apple.com/library/ios/documentation/Audio/Conceptual/iPodLibraryAccess_Guide/UsingtheMediaItemPicker/UsingtheMediaItemPicker.html#//apple_ref/doc/uid/TP40008765-CH104-SW1)。





#### MediaQuery

如果觉得 MeidaPicker 的功能或者 UI 不能满足你的要求那么可以使用 **MediaQuery**。MediaQuery 可以直接访问 iPod Library 的数据库，并根据需求获取数据。详情可以参见[Apple官方文档](https://developer.apple.com/library/ios/documentation/Audio/Conceptual/iPodLibraryAccess_Guide/UsingTheiPodLibrary/UsingTheiPodLibrary.html#//apple_ref/doc/uid/TP40008765-CH101-SW1)

MediaQuery 功能十分强大，它可以根据一个或多个条件查询满足需要的 MediaItem。

你可以使用 MPMediaQuery 的类方法来生成一些已经预置了条件的 Query

也可以自己生成 `MPMediaPredicate` 设置条件，并把它加到 Query 中，最后通过 items 和 collections 访问查询到的结果，例如：

{% highlight objc  %}
MPMediaPropertyPredicate *artistNamePredicate =
[MPMediaPropertyPredicate predicateWithValue:@"Happy the Clown"
                                 forProperty:MPMediaItemPropertyArtist
                              comparisonType:MPMediaPredicateComparisonEqualTo];

MPMediaQuery *quert = [[MPMediaQuery alloc] init];
[quert addFilterPredicate: artistNamePredicate];
quert.groupingType = MPMediaGroupingArtist;

NSArray *itemsFromArtistQuery = [quert items];
NSArray *collectionsFromArtistQuery = [quert collections];
{% endhighlight %}

这一过程用官方给图片表示为：

[mediaQuery](https://developer.apple.com/library/ios/documentation/Audio/Conceptual/iPodLibraryAccess_Guide/Art/mediaQuery.jpg)

于是可以得到从 iPod 读取音乐文件信息的方法为：

{% highlight objc  %}
@implementation VNBMusicManager

/**
 *  读取音乐文件
 *
 *  @param completion 完成后返回一个包含所有音乐信息的 array，
 */
- (void)readMusicItemsFromeAlbumCompletion:(void (^)(NSMutableArray *musicArray, NSError *error))completion{
    __block NSMutableArray *musicItemsArray = [NSMutableArray array];
    __block NSError *err = nil;
    
    MPMediaPropertyPredicate *predicate = [MPMediaPropertyPredicate predicateWithValue:@(MPMediaTypeAnyAudio) forProperty:MPMediaItemPropertyMediaType];
    MPMediaQuery *mediaQuery = [[MPMediaQuery alloc] initWithFilterPredicates:nil];
    [mediaQuery addFilterPredicate:predicate];

    NSArray *items = [mediaQuery items];
    
    for (int i = 0; i < [items count]; i++) {
        
        // 保存所有音乐信息
        VNBMusicItem *musicItem = [[VNBMusicItem alloc] initWithMPMediaItem:items[i]];

        // 不显示Apple Music里面下载的音乐
        /**
         *  由于iPhone 自带的音乐软件Music的推出.从iPod取出来的音乐MPMediaItemPropertyAssetURL属性可能为空.
         *  这是因为iPhone自带软件Music对音乐版权的保护,对于所有进行过 DRM Protection(数字版权加密保护)的音乐
         *  都不能被第三方APP获取并播放.即使这些音乐已经下载到本地.但是还是可以播放本地未进行过数字版权加密的音乐.
         *  也就是自己手动导入的音乐.
         */
        if (musicItem.assetUrl) {
            [musicItemsArray addObject:musicItem];
        }
    }
    
    if (completion) {
        completion(musicItemsArray, err);
    }
}
@end

@implementation VNBMusicItem

- (id)initWithMPMediaItem:(MPMediaItem *)item{
    self = [super init];
    if (self) {
        
        self.audioItem = item;

        //URL 地址
        self.assetUrl = [item valueForProperty:MPMediaItemPropertyAssetURL];
        //歌手
        self.artistName = [item valueForProperty:MPMediaItemPropertyArtist];
        //歌曲名
        self.trackName = [item valueForProperty:MPMediaItemPropertyTitle];

        //文件名
        NSString *fileExt = @".mp3";
        self.fileName = [NSString stringWithFormat:@"%@%@", self.trackName,fileExt];
        
        //专辑名
        self.albumName = [item valueForProperty: MPMediaItemPropertyAlbumTitle];

        //专辑缩略图
        MPMediaItemArtwork *artwork = [item valueForProperty:MPMediaItemPropertyArtwork];
        if (artwork!=nil) {
            self.thumbnail = [artwork imageWithSize:artwork.bounds.size];
        }
        //播放时长
        self.playBackDuration = [self getDurationOfAudioWithItem:item];
        //时长
        self.IntDuration = [self getIntDurationOfAudioWithItem:item];
        
    }
    return self;
}
@end
{% endhighlight %}


#### MediaItem

通过 **MediaPicker** 和 **MediaQuery** 最终都会得到 `MPMediaItem`，这个item中包含了许多信息。这些信息都可以通过 `MPMediaEntity` 的方法访问，其中参数非常多，具体可以参照`MPMediaItem.h`。


### 导出数据

前面代码使用 MPMediaItem 的 MPMediaItemPropertyAssetURL 属性可以得到一个表示当前 MediaItem 的NSURL，有了这个NSURL 我们便可以使用 AVFoundation 中的类进行播放，也可以使用 AVFoundation 中两个有趣的类：AVAssetReader和AVAssetExportSession，它们可以把 iPod Library 中的指定歌曲以指定的音频格式导出到内存中或者硬盘中，这个指定的格式包括 PCM。这是一个非常炫酷的特性，有了 PCM 数据后我们就可以实现很多很多功能了。

这部分如果要展开的话还会有相当多的内容，国外的先辈们早在2010年就已经发掘了这两个类的用法，详细参见[这里](http://www.subfurther.com/blog/2010/07/19/from-iphone-media-library-to-pcm-samples-in-dozens-of-confounding-potentially-lossy-steps/)和[这里](http://www.subfurther.com/blog/2010/12/13/from-ipod-library-to-pcm-samples-in-far-fewer-steps-than-were-previously-necessary/)。这两篇讲的比较详细并且附有Sample（其中还涉及了一些Extended Audio File Services的内容），如果里面Sample无法下载可以从点击[MediaLibraryExportThrowaway1.zip](http://msching.github.io/files/MediaLibraryExportThrowaway1.zip) 和 [VTM_AViPodReader.zip](http://msching.github.io/files/VTM_AViPodReader.zip) 下载。

#### 示例代码

下面的代码就是利用了 AVAssetExportSession 将 iPod 库中的歌曲数据导出为 m4a 文件：

{% highlight objc  %}

#define Cache_Directory [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) objectAtIndex:0] stringByAppendingPathComponent:@"Web"]

@interface VNBMusicManager ()

@property (nonatomic, strong) AVAssetExportSession* exportSessionMusic;

@end

@implementation VNBMusicManager

/**
 *  准备共享音频：先把音频文件从 ipod 库中拷贝出来，再上传给电视播放
 *
 *  @param item       需要共享的音乐对象信息
 *  @param error      错误信息
 *  @param completion 完成后开始共享
 *
 *  @return 是否读取完成
 */
- (BOOL)prepareToShareAlbumItem:(VNBMusicItem *)item error:(NSError **)error completion:(void (^)(NSError *err))completion{
    
    __block NSError *err = nil;
    
    NSFileManager *fm = [NSFileManager defaultManager];
    //若文件夹不存在，创建文件夹
    if (![fm fileExistsAtPath:Cache_Directory]) {
        [fm createDirectoryAtPath:Cache_Directory withIntermediateDirectories:YES attributes:nil error:error];
        if (*error) {
            NSLog(@"创建文件夹失败：%@", *error);
            return NO;
        }
    }
    
    NSString *localPath = [Cache_Directory stringByAppendingPathComponent:item.fileName];

    [self clearCacheWithOutMusicItem:item];
    
    //音乐 m4a
    NSString *outPathUrl = [[Cache_Directory stringByAppendingPathComponent:item.trackName] stringByAppendingPathExtension:@"m4a"];
    
    AVURLAsset *urlAsset = [AVURLAsset assetWithURL:item.assetUrl];
    
    self.exportSessionMusic = [AVAssetExportSession exportSessionWithAsset:urlAsset presetName:AVAssetExportPresetAppleM4A];
    self.exportSessionMusic.outputFileType = AVFileTypeAppleM4A;
    self.exportSessionMusic.shouldOptimizeForNetworkUse = YES;
    self.exportSessionMusic.outputURL = [NSURL fileURLWithPath:outPathUrl];
    
    [self.exportSessionMusic exportAsynchronouslyWithCompletionHandler:^{
        
        switch (self.exportSessionMusic.status) {
            case AVAssetExportSessionStatusUnknown:{//0
                NSLog(@"exportSession.status AVAssetExportSessionStatusUnknown");
                break;
            }
            case AVAssetExportSessionStatusWaiting:{//1
                NSLog(@"exportSession.status AVAssetExportSessionStatusWaiting");
                break;
            }
            case AVAssetExportSessionStatusExporting:{//2
                NSLog(@"exportSession.status AVAssetExportSessionStatusExporting");
                break;
            }
            case AVAssetExportSessionStatusCompleted:{//3
                NSLog(@"exportSession.status AVAssetExportSessionStatusCompleted");
                
                //do something
                NSError *error;
                [fm moveItemAtPath:outPathUrl toPath:localPath error:&error];
                //只能导出为 m4a 文件，导出后再更名为 MP3
                //outPathUrl: ~/Documents/Web/musicName.m4a
                //localPath: ~/Documents/Web/musicName.mp3

                dispatch_sync(dispatch_get_main_queue(), ^{
                    if (completion) {
                        completion(err);
                    }
                });
                
                break;
            }
            case AVAssetExportSessionStatusFailed:{//4
                
                NSLog(@"exportSession.status2 AVAssetExportSessionStatusFailed");
                if (completion) {
                    completion(self.exportSessionMusic.error);
                }
                NSLog(@"error:%@",self.exportSessionMusic.error);
                break;
            }
            case AVAssetExportSessionStatusCancelled:{//5
                NSLog(@"exportSession.status2 AVAssetExportSessionStatusCancelled");
                break;
            }
            default:
                break;
        }
    }];
    
    return YES;
}

/**
 *  清除缓存
 *
 *  @param item 准备上传的音乐对象
 */
-(void)clearCacheWithOutMusicItem:(VNBMusicItem *)item{
    NSFileManager *fm = [NSFileManager defaultManager];
    NSError *errors = nil;
    NSArray *fileList = [[NSArray alloc] init];
    fileList = [fm contentsOfDirectoryAtPath:Cache_Directory error:&errors];
    for (NSString *object in fileList) {
        NSString * filePath=[Cache_Directory stringByAppendingPathComponent:object];
        BOOL Ret = [fm fileExistsAtPath:filePath];
        if (Ret) {
            NSError *err;
            [fm removeItemAtPath:filePath error:&err];
        }
    }
}
@end
{% endhighlight %}

* 参考文章
    
    * [iOS音频播放 (七)：播放iPod Library中的歌曲](http://www.360doc.com/content/15/0202/16/21064743_445737510.shtml#)

    * [如何从iPhone/iPod的音乐库中拷贝音乐到自己的App里](http://swordinhand.iteye.com/blog/1942043)
    
    
> 附 demo 的代码地址[ReadMusicItemFromiPod](https://github.com/Vanbein/ReadMusicItemFromiPod)





