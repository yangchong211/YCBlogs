## 动机

最近因为想要英语学习，特下载了「扇贝阅读」App,保证自己抽空能够提升一下自己的英语水平。这个App有一个功能，就是打卡功能，每天成功阅读完两篇英语短文，就能完成每日打卡，并领取一些奖励。

问题就出现在这里，因为这个App的设定是，如果天天都坚持打卡，那么你就能持续的获得奖励，这些奖励可用来兑换付费的英语书。为了保证能够最大化每日奖励，我就必须坚持阅读打卡，平时这个设定没啥问题，但是有时候（就是前两天的五一放假），我可能一天都没有时间阅读，但是我又不想错过每日的奖励，该怎么办呢？

有朋友说了，你直接跳过阅读，点击完成阅读，然后打卡可以吗，这里就要说要扇贝阅读APP的一个奇葩的设定了，那就是：

> 直接快速滑到底部，点击完成阅读，APP会检测到你这个异常的操作，然后提示你「请认真阅读」。

就是这样：

![error1.gif](https://upload-images.jianshu.io/upload_images/7293029-d086361b9dba9d79.gif?imageMogr2/auto-orient/strip)

不止是扇贝阅读这个App,市面上其他一些主流英语阅读App都有这种类似的设定，我想策划是想告诉我们：

> **既然想要学习，为何不好好认真下来学习呢？不要骗自己。**

好好，你说的我都懂，可是我真的抽不出时间认真阅读2篇文章，又不想断了自己的连续签到奖励怎么办？

> 有朋友说了，既然他让我们多花点时间阅读，我们就搁在那里不动，过两分钟再点击完成阅读按钮呗。

很遗憾，我进行了尝试，然并卵，APP依旧冷漠无情，让我「再认真阅读一遍」。

经过尝试，我发现，APP的处理逻辑大概是这样，**确保用户每一行文字都会被展示一段时间（就像我们正常阅读的效果一样），当所有段落都经过一段时间被「阅读」后，才能正常「完成阅读」**。

> 有朋友又说了，那我们就模拟阅读，慢慢往下划，让文章被「慢速」、「均匀」地拉到底部，怎么样？

这就是笔者五一期间「作弊」式打卡的方式，事实证明完全可行，但是这种方式的弊端也很明显，一篇1.2百词汇的文章，也许1分钟就能拉到底，要是4.5百词汇的文章，就得花数分钟来模拟「阅读」操作。

我不禁深深被APP这种奇葩的设定感动到了，这种完全是「防君子不防小人」的设定，究竟有什么意义？并且，随着第二天，第三天这样的操作过来，我不禁无语了，这种毫无技术含量的作弊手段，可以称得上既无聊又繁琐，让我感觉自己是被APP强行交了一波智商税。

## 那就写个工具吧

基于上述事实，我决定写个工具，尽量代替双手解决目前的窘境。

我选择了Groovy作为开发语言，写了一个脚本，模拟用户操作，缓慢阅读文章，并自动点击「完成阅读」按钮。

先来看看脚本运行的效果，完全由ADB命令控制：

![因为gif图的原因，看起来很快，实际上ADB是控制屏幕缓慢地匀速下拉](https://upload-images.jianshu.io/upload_images/7293029-bd8b1858bc4ad70e.gif?imageMogr2/auto-orient/strip)

看一下命令行的输出：

![开始阅读](https://upload-images.jianshu.io/upload_images/7293029-062cb376628ca514.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![结束阅读](https://upload-images.jianshu.io/upload_images/7293029-59825faa68c1afc0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


我的基本思路是这样：

* 1、通过adb命令模拟用户向下翻页的操作；
* 2、每次模拟完翻页操作后，将当前屏幕截图保存；
* 3、然后将上次翻页完成后的截图和本次截图进行图像识别分析，得到2张屏幕截图的相似度；
* 4、当2张屏幕截图的相似度匹配不高时，视为两张图片不同，即应该继续向下翻页，并重复1~3的行为；
* 5、当2张屏幕截图的相似度匹配很高时，视为该两次操作达到了文章最底部（无法继续下翻，所以截图基本一样），点击「完成阅读」按钮，并清除截图缓存文件夹，结束本次脚本任务。

## 脚本代码

前期是补充一些基本的属性和配置：

```groovy 
final int actionInterval = 250        //两次下翻操作的时间间隔，单位毫秒
final float threshold = 0.95          //图片分析相似度的阈值，当相似度大于阈值时，视为图片相同

println "已选中的Android设备："

println "————————————————————————————————————"

println "adb devices".execute().text

println "————————————————————————————————————"

println "开始执行自动阅读"

//因为系统原因，很多情况下该命令实际的效果为对界面元素的长按，因此抛弃该命令
//println 'input swipe 540 1300 540 500 100 '

boolean clearScreenShotCacheWhenFinishTask = true       //可选项，当脚本执行结束时，是否自动清除截图缓存
def ending = false              //是否已结束
def duration = 0                //本次操作已执行时间
String rootPath = System.getProperty("user.dir") + "/screenshots/"
String lastScreenShot = null
String newScreenShot = null
```

本来想用 adb shell input swipe 命令模拟滑动操作，但是发现某些系统的设备不支持该命令，实现效果会变成「长按界面上某个元素」，而非我们想要的「滑动界面」操作，无奈，使用adb shell input keyevent 20命令代替。

我们提供了几个方法方便调用：

```groovy
/**
 * 为当前的屏幕截图，并保存在默认路径
 */
def task_screenShot(String rootPath) {

    def millis = currentTimeMillis()

    def screenShotPath = rootPath + millis  //要截图的路径

    println "screenShotPath = $screenShotPath"

    println "adb shell screencap -p /sdcard/${millis}".execute().text
    println "adb pull /sdcard/${millis} $screenShotPath".execute().text

    return screenShotPath
}

/**
 * 为当前app执行向下翻页操作
 */
def task_downPage(Integer interval = 500) {
    Thread.sleep(interval)
    println "adb shell input keyevent 20".execute().text
}

/**
 * 通过比较获取图片的相似度
 */
def task_compareSimilar(String pic1, String pic2) {
    def print1 = new FingerPrint(ImageIO.read(new File(pic1)))
    def print2 = new FingerPrint(ImageIO.read(new File(pic2)))

    return print1.compare(print2)
}

/**
 * 结束阅读，自动点击屏幕下方按钮「完成阅读」或者「读后感」
 */
def task_finishReading() {
    println "——————————————————————————————————————————————"
    println "执行结束阅读操作..."
    println "adb shell input tap 540 1730".execute().text         //模拟点击按钮完成阅读，这里以1920*1080的屏幕分辨率为准
    println "执行结束阅读操作完毕."
    println "——————————————————————————————————————————————"
}

/**
 * 清除文件目录下截图文件
 */
def task_clearDir(boolean clear = true, String rootPath = System.getProperty("user.dir") + "/screenshots/") {
    if (clear) {
        println '清除图片文件夹中...'
        new File(rootPath).deleteDir()
        println '清除完毕'
    } else {
        println '本次任务不清除screenshots文件夹下缓存图片文件,若要修改该配置,请将脚本文件中clearScreenShotCacheWhenFinishTask设置为true'
    }
}
```

有几点补充的：

* **截图和保存截图功能我们依靠adb的命令实现。**

adb shell screencap -p /sdcard/${millis} 是截图保存到手机；

adb pull /sdcard/${millis} $screenShotPath是将截图保存到自己的PC项目的指定目录下。

* **结束阅读，自动点击屏幕下方按钮「完成阅读」**

这个功能也不难，关键是获取该按钮的位置，通过Android设备自带的开发者选项，轻松获取到按钮的位置。

![打开指针位置选项](https://upload-images.jianshu.io/upload_images/7293029-8ef3613ee31d6559.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![手指放在按钮中间，上方显示坐标点](https://upload-images.jianshu.io/upload_images/7293029-54323b19725326f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

因为我的MI6分辨率是1920*1080，只需要确认Y值即可，约为1730左右，X轴自然是1080/2=540，因此模拟点击按钮的adb命令为：

> adb shell input tap 540 1730

时间原因，没有做不同分辨率下不同机型的适配，而是写死了自己的机型1920*1080，以后有机会再补充其他主流的分辨率吧。

## 均值哈希实现图像内容相似度比较

脚本代码中，「图像内容相似度比较」的算法是很重要的一部分，对此我参考了@10km前辈的文章：[java:均值哈希实现图像内容相似度比较](https://blog.csdn.net/10km/article/details/70949272)，并将代码基本原封不动放入了项目中：

```groovy
class FingerPrint {

    /**
     * 图像指纹的尺寸,将图像resize到指定的尺寸，来计算哈希数组
     */
    def static HASH_SIZE = 16

    /**
     * 保存图像指纹的二值化矩阵
     */
    private final byte[] binaryzationMatrix

    FingerPrint(byte[] hashValue) {
        if (hashValue.length != HASH_SIZE * HASH_SIZE)
            throw new IllegalArgumentException(String.format("length of hashValue must be %d", HASH_SIZE * HASH_SIZE))
        this.binaryzationMatrix = hashValue
    }

    FingerPrint(String hashValue) {
        this(toBytes(hashValue))
    }

    FingerPrint(BufferedImage src) {
        this(hashValue(src))
    }

    private static byte[] hashValue(BufferedImage src) {
        BufferedImage hashImage = resize(src, HASH_SIZE, HASH_SIZE)
        byte[] matrixGray = (byte[]) toGray(hashImage).getData().getDataElements(0, 0, HASH_SIZE, HASH_SIZE, null)
        return binaryzation(matrixGray)
    }
    /**
     * 从压缩格式指纹创建{@link FingerPrint}对象
     * @param compactValue
     * @return
     */
    static FingerPrint createFromCompact(byte[] compactValue) {
        return new FingerPrint(uncompact(compactValue))
    }

    static boolean validHashValue(byte[] hashValue) {
        if (hashValue.length != HASH_SIZE)
            return false
        for (byte b : hashValue) {
            if (0 != b && 1 != b) return false
        }
        return true
    }

    static boolean validHashValue(String hashValue) {
        if (hashValue.length() != HASH_SIZE)
            return false
        for (int i = 0; i < hashValue.length(); ++i) {
            if ('0' != hashValue.charAt(i) && '1' != hashValue.charAt(i)) return false
        }
        return true
    }

    byte[] compact() {
        return compact(binaryzationMatrix)
    }

    /**
     * 指纹数据按位压缩
     * @param hashValue
     * @return
     */
    static byte[] compact(byte[] hashValue) {
        byte[] result = new byte[(hashValue.length + 7) >> 3]
        byte b = 0
        for (int i = 0; i < hashValue.length; ++i) {
            if (0 == (i & 7)) {
                b = 0
            }
            if (1 == hashValue[i]) {
                b |= 1 << (i & 7)
            } else if (hashValue[i] != 0)
                throw new IllegalArgumentException("invalid hashValue,every element must be 0 or 1")
            if (7 == (i & 7) || i == hashValue.length - 1) {
                result[i >> 3] = b
            }
        }
        return result
    }

    /**
     * 压缩格式的指纹解压缩
     * @param compactValue
     * @return
     */
    private static byte[] uncompact(byte[] compactValue) {
        byte[] result = new byte[compactValue.length << 3]
        for (int i = 0; i < result.length; ++i) {
            if ((compactValue[i >> 3] & (1 << (i & 7))) == 0)
                result[i] = 0
            else
                result[i] = 1
        }
        return result
    }
    /**
     * 字符串类型的指纹数据转为字节数组
     * @param hashValue
     * @return
     */
    private static byte[] toBytes(String hashValue) {
        hashValue = hashValue.replaceAll("\\s", "")
        byte[] result = new byte[hashValue.length()]
        for (int i = 0; i < result.length; ++i) {
            char c = hashValue.charAt(i)
            if ('0' == c)
                result[i] = 0
            else if ('1' == c)
                result[i] = 1
            else
                throw new IllegalArgumentException("invalid hashValue String")
        }
        return result
    }
    /**
     * 缩放图像到指定尺寸
     * @param src
     * @param width
     * @param height
     * @return
     */
    private static BufferedImage resize(Image src, int width, int height) {
        BufferedImage result = new BufferedImage(width, height,
                BufferedImage.TYPE_3BYTE_BGR)
        Graphics g = result.getGraphics()
        try {
            g.drawImage(src.getScaledInstance(width, height, Image.SCALE_SMOOTH), 0, 0, null)
        } finally {
            g.dispose()
        }
        return result
    }
    /**
     * 计算均值
     * @param src
     * @return
     */
    private static int mean(byte[] src) {
        long sum = 0
        // 将数组元素转为无符号整数
        for (byte b : src) sum += (long) b & 0xff
        return (int) (Math.round((float) sum / src.length))
    }
    /**
     * 二值化处理
     * @param src
     * @return
     */
    private static byte[] binaryzation(byte[] src) {
        byte[] dst = src.clone()
        int mean = mean(src)
        for (int i = 0; i < dst.length; ++i) {
            // 将数组元素转为无符号整数再比较
            dst[i] = (byte) (((int) dst[i] & 0xff) >= mean ? 1 : 0)
        }
        return dst

    }
    /**
     * 转灰度图像
     * @param src
     * @return
     */
    private static BufferedImage toGray(BufferedImage src) {
        if (src.getType() == BufferedImage.TYPE_BYTE_GRAY) {
            return src
        } else {
            // 图像转灰
            BufferedImage grayImage = new BufferedImage(src.getWidth(), src.getHeight(),
                    BufferedImage.TYPE_BYTE_GRAY)
            new ColorConvertOp(ColorSpace.getInstance(ColorSpace.CS_GRAY), null).filter(src, grayImage)
            return grayImage
        }
    }

    @Override
    String toString() {
        return toString(true)
    }
    /**
     * @param multiLine 是否分行
     * @return
     */
    String toString(boolean multiLine) {
        StringBuffer buffer = new StringBuffer()
        int count = 0
        for (byte b : this.binaryzationMatrix) {
            buffer.append(0 == b ? '0' : '1')
            if (multiLine && ++count % HASH_SIZE == 0)
                buffer.append('\n')
        }
        return buffer.toString()
    }

    @Override
    boolean equals(Object obj) {
        if (obj instanceof FingerPrint) {
            return Arrays.equals(this.binaryzationMatrix, ((FingerPrint) obj).binaryzationMatrix)
        } else
            return super.equals(obj)
    }

    /**
     * 与指定的压缩格式指纹比较相似度
     * @param compactValue
     * @return
     * @see #compare(FingerPrint)
     */
    float compareCompact(byte[] compactValue) {
        return compare(createFromCompact(compactValue))
    }
    /**
     * @param hashValue
     * @return
     * @see #compare(FingerPrint)
     */
    float compare(String hashValue) {
        return compare(new FingerPrint(hashValue))
    }
    /**
     * 与指定的指纹比较相似度
     * @param hashValue
     * @return
     * @see #compare(FingerPrint)
     */
    float compare(byte[] hashValue) {
        return compare(new FingerPrint(hashValue))
    }
    /**
     * 与指定图像比较相似度
     * @param image2
     * @return
     * @see #compare(FingerPrint)
     */
    float compare(BufferedImage image2) {
        return compare(new FingerPrint(image2))
    }

    /**
     * 比较指纹相似度
     * @param src
     * @return
     * @see #compare(byte [ ], byte [ ])
     */
    float compare(FingerPrint src) {
        if (src.binaryzationMatrix.length != this.binaryzationMatrix.length)
            throw new IllegalArgumentException("length of hashValue is mismatch")
        return compare(binaryzationMatrix, src.binaryzationMatrix)
    }
    /**
     * 判断两个数组相似度，数组长度必须一致否则抛出异常
     * @param f1
     * @param f2
     * @return 返回相似度 ( 0.0 ~ 1.0 )
     */
    static float compare(byte[] f1, byte[] f2) {
        if (f1.length != f2.length)
            throw new IllegalArgumentException("mismatch FingerPrint length")
        int sameCount = 0
        for (int i = 0; i < f1.length; ++i) {
            if (f1[i] == f2[i]) ++sameCount
        }
        return (float) sameCount / f1.length
    }

    static float compareCompact(byte[] f1, byte[] f2) {
        return compare(uncompact(f1), uncompact(f2))
    }

    static float compare(BufferedImage image1, BufferedImage image2) {
        return new FingerPrint(image1).compare(new FingerPrint(image2))
    }
}
```

## 小结

写到这里，这篇文章基本就结束了，我把自己的代码也托管到了我的github上。

[ShanbayAutoReader:扇贝英语阅读app，首页短文自动阅读脚本](https://github.com/qingmei2/ShanbayAutoReader)

其实这个脚本意义不是很大，写这个东西的动机也很简单：

1、不想自己被APP套路
2、巩固一下自己的groovy知识体系

