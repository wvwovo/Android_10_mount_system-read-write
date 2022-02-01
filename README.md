# Android_10_mount_system(read/write)
# 只在ONLY A动态分区的手机做过测试,像AB分区，VAB分区的不知道是什么情况
- 解锁安卓10的system分区并自动扩大分区(可以扩大多少取决于整个super分区还剩多少空间，有点super最大大小是8GB，有的是4.5GB，也许还有别的大小)
- 可以指定选择扩大某个分区，要刷入模块两次，第一次生成基本的文件，然后去找'/data/local/super2rw/setting.txt'进行修改 
- 默认可能长这个样子
```
把1改成0为不扩大分区,1为扩大分区。每个分区能扩大多少，取决于剩余空间/需要扩区的数量
product 1
system 1
vendor 1
```
把1改成0就是这个分区不扩区，全1就是平均扩大所有分区，全0就是只解锁分区(因为resize2fs，所以分区大小还是有轻微变化，暂时不动)

- 一个super分区可以包含 system ，product ，vendor ，odm , system_ext (odm不一定在super里面)，每个手机的情况可能都有所不同 https://source.android.google.cn/devices/tech/ota/dynamic_partitions/implement?hl=zh-cn
- 把文件用压缩文件打包成zip即可使用，内部文件目录必须是
```
- bin
- META-INF
- .gitignore
- .gitattributes
- LICENSE
- README.md
```
- 刷一遍扩大，两遍会缩小，三遍会扩大，四遍会缩小。。。
- 仅对因分区filesystem带有shared_blocks属性的锁分区有效
- 最好先用twrp备份super分区以便未来不时之需
```
- https://forum.xda-developers.com/t/script-android-10-universal-mount-system-r-w-read-write.4247311/
- To check whether or not your device has the notorious shared_blocks read-only feature you can run these commands in Android:
下面是查询是否带shared_blocks的代码,结果保存在内部存储里的my_tune2fs_output.txt文件里
- Bash:
adb shell
su
touch /sdcard/my_tune2fs_output.txt
for i in `seq 0 5`; do tune2fs -l /dev/block/dm-$i>>/sdcard/my_tune2fs_output.txt; done
```
## 参考&使用
- https://gitee.com/sharpeter/DNA 拆twrp，找到\ramdisk\tools\makerw.zip
- https://androidfilehost.com/?fid=7161016148664805646 twrp-3.5.2_10-3-lime-brigudav , 红米 note9 4g 的twrp(该twrp支持安卓10的miui12和安卓11的miui12.5)
- https://github.com/YAWAsau/backup_script 一些可执行文件和代码
- https://forum.xda-developers.com/t/script-android-10-universal-mount-system-r-w-read-write.4247311/ systemrw_1.32_flashable.zip 一些可执行文件和代码

## 其它
- 1
  有些手机的分区格式是erofs，天然就是只读属性
  https://zhuanlan.zhihu.com/p/365461315
- 2
 修改y=0.9的数值大小可以解决super.mod.bin太大刷不进super分区的问题  
```
//META-INF\com\google\android\update-binary
//343行
temp4=`awk -v x=$temp3 -v y=0.9 'BEGIN{printf("%.0f",x*y)}'`
```
- 3
 红米9a 联发科 32位系统 安卓10和11也可以用，但要直接在twrp修改system分区还需要这个https://github.com/wvwovo/redmi9a_android10_twrp_mount_system_rw
