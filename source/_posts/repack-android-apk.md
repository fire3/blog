title: "repack android apk"
date: 2015-03-24 22:33:33
tags: [android]
---

最近经常`hack`一些android的应用，常常需要修改smali后重新打包，特地编写了一个简单的打包脚本：

<!-- more -->

``` bash

#!/bin/bash -x

if [ $# -ne 1 ]
then
echo "pack your.apk"
exit 1
fi

apk=$1

apkdir=$(basename $apk .apk)

package_grep=$(grep -o -e package=\".*\" $apkdir/AndroidManifest.xml)
package=$(echo $package_grep | sed "s/package=//g" | sed "s/\"//g")
echo $package

rm -rf build
mkdir build

adb shell pm uninstall $package
apktool build $apkdir -o build/$apk

rm -rf signed
mkdir signed

d2j-apk-sign.sh -f -o  signed/$apk build/$apk 

cd signed
adb install $apk
cd -
```
