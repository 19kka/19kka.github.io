---
layout:     post
title:      iOS Bulid Tips
date:       2016-05-15 14:03:18
summary:    NULL
categories: iOS,Xcode,Bulid
---

#iOS Bulid Tips
--
汇总了一些编译碰到的问题


###Framework Error
--

``` cmake
CodeSign /Users/xx/Library/Developer/Xcode/DerivedData//xx//xx/xx/x/x/x/x/x/xxx/x/xxxx.app/Frameworks/xxxx.framework
    cd //xx//xx/xx/x/x/x/x/x/xxx/x/x
    export CODESIGN_ALLOCATE=/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/codesign_allocate
    export PATH="/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin:/Applications/Xcode.app/Contents/Developer/usr/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
    
Signing Identity:     "iPhone Developer: xxxxxxxxx"

    /usr/bin/codesign --force --sign 0B9F657BF598KJDSHFJF*^&FDSKFD229CB0 --preserve-metadata=identifier,entitlements --timestamp=none /Users/xxxxx/Library/Developer/Xcode/DerivedData/xxxxxemo-ffgsdfgsdgejrsdfgvi/Build/Products/Debug-iphoneos/rightAVDemo.app/Frameworks/xxxxxx.framework

/Users/xxx/Library/Developer/Xcode/DerivedData/xxxx-xxxxxdfssdfsdf/Build/Products/Debug-iphoneos/xxxxxx.app/Frameworks/xxxxxxx.framework: bundle format unrecognized, invalid, or unsuitable
Command /usr/bin/codesign failed with exit code 1

```

#####Try to Remove:  
**General -> Embedded Binaries : xxx.Framework**

--
###Pods Error
```
diff: /../Podfile.lock: No such file or directory 
diff: /Manifest.lock: No such file or directory error: 
The sandbox is not in sync with the Podfile.lock. Run 'pod install' or update your CocoaPods installation.
```

#####Add User-Defined Setting

Key:PODS_ROOT
Value:${SRCROOT}/Pods

--
###Pods Bulid Error
####xxxxx.h header not found

1
>  
-Use the `$(inherited)` flag, or   
-Remove the build settings from the target.

2.
>
Pods/Target Support Files/Pods/Pods.debug.xcconfig Set to None


