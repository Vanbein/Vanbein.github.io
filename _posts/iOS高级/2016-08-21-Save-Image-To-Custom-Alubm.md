---
layout: post
title: 保存图片到应用自定义的相册
category: iOS高级
tags: 相册
image: /images/head-800x400/-58.png
description: 在很多应用中，浏览图片时，可以长按图片选择保存到手机相册中的自定义名称相册中，iOS8以后，我们可以使用 Photos 框架非常轻松的实现这个功能。
homepage: false
toc: true
---

在很多应用中，浏览图片时，可以长按图片选择保存到手机相册中的自定义名称相册中，比如微博，它会在系统相册中自动创建一个名为“微博”的相册，并添加要保存的图片到其中。在iOS8以后，我们可以使用 Photos 框架非常轻松的实现这个功能。

## 一、Photos 框架简单介绍

### 1、重要的类
该框架有几个非常重要的类：PHAsset、PHAssetCollection 和 PHLibrary。

PHAsset 表示一个图片或者视频文件
PHAssetCollection 表示图片集合或者视频集合，其实就是指相册（包括了系统相册和自定义相册）
PHLibrary 表示整个相册库，包括整个相册和图片等

### 2、查询操作
查询操作，直接使用 PHAsset 和 PHAssetCollection 类本身的方法

//1 获取相册中的图片--传入相册图片的 ID---返回一组图片

{% highlight objc  %}
[PHAsset fetchAssetsWithLocalIdentifiers:@[ID] options:nil];
{% endhighlight %}

//2 查询手机中所有的相册列表(分为系统相册和自定义相册，通过控制传入的参数来确定)---返回相册组--类似数组-forin遍历即可

{% highlight objc  %}
[PHAssetCollection fetchAssetCollectionsWithType:PHAssetCollectionTypeAlbum subtype:PHAssetCollectionSubtypeAlbumRegular options:nil];
{% endhighlight %}

### 3、增删改操作

1. 如果做查询之外的操作，比如说保存图片、创建自定义相册、向自定义相册中添加图片等，都需要使用另外两个类：`PHAssetChangeRequest` 和 `PHAssetCollectionChangeRequest`

2. 这些操作必须在 `[[PHPhotoLibrary sharedPhotoLibrary]performChange...]` 的 block 中间调用

//1 保存图片

{% highlight objc  %}
[PHAssetChangeRequest creationRequestForAssetFromImage:image];
{% endhighlight %}

//2 创建相册

{% highlight objc  %}
[PHAssetCollectionChangeRequest creationRequestForAssetCollectionWithTitle:@"自定义相册"];
{% endhighlight %}


## 二、保存图片到自定义相册

### 1、判断相册访问权限
保存图片时，首先我们要判断应用是否有对相册的访问权限，这个可以通过下面的代码进行简单的判断：

{% highlight objc  %}
- (BOOL)achiveAuthorizationStatus{
    
    // 1、判断相册访问权限
    /*
     * PHAuthorizationStatusNotDetermined = 0, 用户未对这个应用程序的权限做出选择
     * PHAuthorizationStatusRestricted, 此应用程序没有被授权访问的照片数据。可能是家长控制权限。
     * PHAuthorizationStatusDenied, 用户已经明确拒绝了此应用程序访问照片数据.
     * PHAuthorizationStatusAuthorized, 用户已授权此应用访问照片数据.
     */
    PHAuthorizationStatus status = [PHPhotoLibrary authorizationStatus];
    if (status == PHAuthorizationStatusDenied || status == PHAuthorizationStatusRestricted) {
        // 没权限
        UIAlertController *authorizationAlert = [UIAlertController alertControllerWithTitle:@"提示" message:@"没有照片的访问权限，请在设置中开启" preferredStyle:UIAlertControllerStyleAlert];
        UIAlertAction *cancel = [UIAlertAction actionWithTitle:@"确定" style:UIAlertActionStyleCancel handler:NULL];
        [authorizationAlert addAction:cancel];
        [self presentViewController:authorizationAlert animated:YES completion:nil];
        return NO;
    } else {
        return YES;
    }
}
{% endhighlight %}

### 2、将图片保存到系统相册

关于图片或视频保存到自定义相册，其实是先保存到系统相册，再将该图片引用到我们创建的自定义相册，保存图片或视频一共有三种方式，代码如下：

{% highlight objc  %}
- (void)saveImageFromImage:(UIImage *)image{
    _saveType = 0;
    _targetImage = image;
    [self saveIntoAlbum];
}

- (void)saveImageFromFileURL:(NSURL *)url{
    
    _saveType = 1;
    _targetImageURL = url;
    [self saveIntoAlbum];
}

- (void)saveVideoFromFileURL:(NSURL *)url{
    _saveType = 2;
    _targetVideoURL = url;
    [self saveIntoAlbum];
}

- (void)saveIntoAlbum{
    
    // 1、判断相册访问权限
    [PHPhotoLibrary requestAuthorization:^(PHAuthorizationStatus status) {
        if (status != PHAuthorizationStatusAuthorized) {
            // 没权限
            return ;
        }
        
        NSError *error = nil;
        // 2、有权限，保存相片到相机胶卷
        __block PHObjectPlaceholder *targetObject = nil;
        __weak typeof (self) weakself = self;

        switch (_saveType) {
            case 0:{
                [[PHPhotoLibrary sharedPhotoLibrary] performChangesAndWait:^{
                    targetObject = [PHAssetCreationRequest creationRequestForAssetFromImage:_targetImage].placeholderForCreatedAsset;
                } error:&error];
                break;
            }
            case 1:{
                
                [[PHPhotoLibrary sharedPhotoLibrary] performChangesAndWait:^{
                    targetObject = [PHAssetCreationRequest creationRequestForAssetFromImageAtFileURL:_targetImageURL].placeholderForCreatedAsset;
                } error:&error];
                break;
            }
            case 2:{
                [[PHPhotoLibrary sharedPhotoLibrary] performChangesAndWait:^{
                    targetObject = [PHAssetCreationRequest creationRequestForAssetFromVideoAtFileURL:_targetVideoURL].placeholderForCreatedAsset;
                } error:&error];
                break;
            }
            default:
                break;
        }
        
        if (error) {
            NSLog(@"保存失败：%@", error);
            return;
        }
        // TODO: 将保存到系统的图片或视频引用保存到自定义相册
    }];
}
{% endhighlight %}

### 3、创建/获取自定义相册

想要将图片保存到自定义的相册，在进行保存操作之前，需要获取到我们创建的相册对象，如果没有获取到，则创建，代码如下：


{% highlight objc  %}
// 获取应用名作为自定义相册名
#define albumName [NSBundle mainBundle].infoDictionary[(NSString *)kCFBundleNameKey]
- (PHAssetCollection *)createdCollection {
    
    // 获得所有的自定义相册
    PHFetchResult<PHAssetCollection *> *collections = [PHAssetCollection fetchAssetCollectionsWithType:PHAssetCollectionTypeAlbum subtype:PHAssetCollectionSubtypeAlbumRegular options:nil];
    for (PHAssetCollection *collection in collections) {
        if ([collection.localizedTitle isEqualToString:albumName]) {
            return collection;
        }
    }
    
    // 代码执行到这里，说明还没有自定义相册
    __block NSString *createdCollectionId = nil;
    
    // 创建一个新的相册
    [[PHPhotoLibrary sharedPhotoLibrary] performChangesAndWait:^{
        createdCollectionId = [PHAssetCollectionChangeRequest creationRequestForAssetCollectionWithTitle:albumName].placeholderForCreatedAssetCollection.localIdentifier;
    } error:nil];
    
    if (createdCollectionId == nil) return nil;
    
    // 创建完毕后再取出相册
    return [PHAssetCollection fetchAssetCollectionsWithLocalIdentifiers:@[createdCollectionId] options:nil].firstObject;
}
{% endhighlight %}

### 4、保存到自定义相册

如上所述，在以上步骤都完成后，则可以通过下面的代码保存到自定义相册了：

{% highlight objc  %}
- (void)saveImageToCustomAlbum{
 
    // 3、拿到自定义的相册对象
    PHAssetCollection *collection = [self createdCollection];
    if (collection == nil) return;
    
    // 4、保存
    [[PHPhotoLibrary sharedPhotoLibrary] performChangesAndWait:^{
        [[PHAssetCollectionChangeRequest changeRequestForAssetCollection:collection] insertAssets:@[targetObject] atIndexes:[NSIndexSet indexSetWithIndex:0]];
    } error:&error];
    
    if (error) {
        NSLog(@"保存失败：%@", error);
    } else {
        NSLog(@"保存成功");
    }
}
{% endhighlight %}

