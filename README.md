# pyGregorian2LunarCalendar
前言：
由于三体运动（主要地球、太阳、月球）无法准确预测，目前二十四节气依然还是靠天文台观测，Yovey使用传说中[Y*D+C]-L方法实际有很多天数不准，def getSolarTerms(_date)12个if嵌套判断让代码变得十分冗余，由简书网友“大咖_247c”首先发现计算不准问题……
【方案过程】
![image.png](https://upload-images.jianshu.io/upload_images/2369108-a121d5e561adc30b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

1、在校验过数据与公式差异后，第一方案是采用15%黄经夹角计算，计算后差异依然存在，地球椭圆形轨道一年内公转速度差异超过7%；
![开普勒.gif](https://upload-images.jianshu.io/upload_images/2369108-671532f9183aad1d.gif?imageMogr2/auto-orient/strip)

2、在发现椭圆轨道问题后，尝试使用平均值、开普勒第二定律计算行星轨道，计算值已经十分贴近了，但依然和香港天文台数据有差异，恍然大悟，地球、月亮、太阳三个天体呈现无法完美预测的三体运动，甚至还包括木星引力影响（比如传说中九星连珠，虽然引力抵消微弱），而且太阳本身也在运动，地球和太阳的质量也不是永恒不变的，这就导致地球的恒星年相对稳定，但回归年浮动振荡；
![image.png](https://upload-images.jianshu.io/upload_images/2369108-2cdf50bbe6b62989.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3、当试过所有技术物理原理还无法调节误差后，最终使用了Chen Jian的核心设计理念，添加了十六进制加存二十四节气。（还是要靠香港天文台）
coding=UTF-8
1901~2100年农历数据表
Author: cuba3
base code by Yovey , https://www.jianshu.com/p/8dc0d7ba2c2a
powered by Late Lee, http://www.latelee.org/python/python-yangli-to-nongli.html#comment-78
other author:Chen Jian, http://www.cnblogs.com/chjbbs/p/5704326.html
数据来源: http://data.weather.gov.hk/gts/time/conversion1_text_c.htm

[【GITHUB/cuba3】:pyGregorian2LunarCalendar项目地址>>https://github.com/cuba3/pyGregorian2LunarCalendar](https://github.com/cuba3/pyGregorian2LunarCalendar)

跟进Chen Jian的设计思路，增加了一层向量压缩。
因为24节气每个月新历月固定有两个，所以list保持顺序，月份就不用存了，一定是1、1、2、2、3、3……
只记录日期的话，日期数据过大，所以对200年内4800个数据进行分组比对，求出最小公约数，得出最小公约年向量[4, 19, 3, 18, 4, 19, 4, 19, 4, 20, 4, 20, 6, 22, 6, 22, 6, 22, 7, 22, 6, 21, 6, 21]，将爬取数据减去这个最小公约向量，就得到了一个元素最大值不大于3的矩阵。
所有数字不大于3，两个二进制就可满足存储一个日期，一个十六进制就能存储一个月，利用Python3 位移算法 << 2 将原本庞大的txt文本压缩成长度200的12位16进制list。
```
# 1901-2100年二十节气最小公约数序列 向量压缩法
encryptionVectorList=[4, 19, 3, 18, 4, 19, 4, 19, 4, 20, 4, 20, 6, 22, 6, 22, 6, 22, 7, 22, 6, 21, 6, 21]

# 1901-2100年二十节气数据 每个元素的存储格式如下：
# 1-24
# 节气所在天（减去节气最小公约数）
# 1901-2100年香港天文台公布二十四节气按年存储16进制，1个16进制为4个2进制
solarTermsData=[
    0x6aaaa6aa9a5a, 0xaaaaaabaaa6a, 0xaaabbabbafaa, 0x5aa665a65aab, 0x6aaaa6aa9a5a, # 1901 ~ 1905
    0xaaaaaaaaaa6a, 0xaaabbabbafaa, 0x5aa665a65aab, 0x6aaaa6aa9a5a, 0xaaaaaaaaaa6a,
    0xaaabbabbafaa, 0x5aa665a65aab, 0x6aaaa6aa9a56, 0xaaaaaaaa9a5a, 0xaaabaabaaeaa,
    0x569665a65aaa, 0x5aa6a6a69a56, 0x6aaaaaaa9a5a, 0xaaabaabaaeaa, 0x569665a65aaa,
    0x5aa6a6a65a56, 0x6aaaaaaa9a5a, 0xaaabaabaaa6a, 0x569665a65aaa, 0x5aa6a6a65a56,
    0x6aaaa6aa9a5a, 0xaaaaaabaaa6a, 0x555665665aaa, 0x5aa665a65a56, 0x6aaaa6aa9a5a,
    0xaaaaaabaaa6a, 0x555665665aaa, 0x5aa665a65a56, 0x6aaaa6aa9a5a, 0xaaaaaaaaaa6a,
    0x555665665aaa, 0x5aa665a65a56, 0x6aaaa6aa9a5a, 0xaaaaaaaaaa6a, 0x555665665aaa,
    0x5aa665a65a56, 0x6aaaa6aa9a5a, 0xaaaaaaaaaa6a, 0x555665655aaa, 0x569665a65a56,
    0x6aa6a6aa9a56, 0xaaaaaaaa9a5a, 0x5556556559aa, 0x569665a65a55, 0x6aa6a6a65a56,
    0xaaaaaaaa9a5a, 0x5556556559aa, 0x569665a65a55, 0x5aa6a6a65a56, 0x6aaaa6aa9a5a,
    0x5556556555aa, 0x569665a65a55, 0x5aa665a65a56, 0x6aaaa6aa9a5a, 0x55555565556a,
    0x555665665a55, 0x5aa665a65a56, 0x6aaaa6aa9a5a, 0x55555565556a, 0x555665665a55,
    0x5aa665a65a56, 0x6aaaa6aa9a5a, 0x55555555556a, 0x555665665a55, 0x5aa665a65a56,
    0x6aaaa6aa9a5a, 0x55555555556a, 0x555665655a55, 0x5aa665a65a56, 0x6aa6a6aa9a5a,
    0x55555555456a, 0x555655655a55, 0x5a9665a65a56, 0x6aa6a6a69a5a, 0x55555555456a,
    0x555655655a55, 0x569665a65a56, 0x6aa6a6a65a56, 0x55555155455a, 0x555655655955,
    0x569665a65a55, 0x5aa6a5a65a56, 0x15555155455a, 0x555555655555, 0x569665665a55,
    0x5aa665a65a56, 0x15555155455a, 0x555555655515, 0x555665665a55, 0x5aa665a65a56,
    0x15555155455a, 0x555555555515, 0x555665665a55, 0x5aa665a65a56, 0x15555155455a,
    0x555555555515, 0x555665665a55, 0x5aa665a65a56, 0x15555155455a, 0x555555555515,
    0x555655655a55, 0x5aa665a65a56, 0x15515155455a, 0x555555554515, 0x555655655a55,
    0x5a9665a65a56, 0x15515151455a, 0x555551554515, 0x555655655a55, 0x569665a65a56,
    0x155151510556, 0x555551554505, 0x555655655955, 0x569665665a55, 0x155110510556,
    0x155551554505, 0x555555655555, 0x569665665a55, 0x55110510556, 0x155551554505,
    0x555555555515, 0x555665665a55, 0x55110510556, 0x155551554505, 0x555555555515,
    0x555665665a55, 0x55110510556, 0x155551554505, 0x555555555515, 0x555655655a55,
    0x55110510556, 0x155551554505, 0x555555555515, 0x555655655a55, 0x55110510556,
    0x155151514505, 0x555555554515, 0x555655655a55, 0x54110510556, 0x155151510505,
    0x555551554515, 0x555655655a55, 0x14110110556, 0x155110510501, 0x555551554505,
    0x555555655555, 0x14110110555, 0x155110510501, 0x555551554505, 0x555555555555,
    0x14110110555, 0x55110510501, 0x155551554505, 0x555555555555, 0x110110555,
    0x55110510501, 0x155551554505, 0x555555555515, 0x110110555, 0x55110510501,
    0x155551554505, 0x555555555515, 0x100100555, 0x55110510501, 0x155151514505,
    0x555555555515, 0x100100555, 0x54110510501, 0x155151514505, 0x555551554515,
    0x100100555, 0x54110510501, 0x155150510505, 0x555551554515, 0x100100555,
    0x14110110501, 0x155110510505, 0x555551554505, 0x100055, 0x14110110500,
    0x155110510501, 0x555551554505, 0x55, 0x14110110500, 0x55110510501,
    0x155551554505, 0x55, 0x110110500, 0x55110510501, 0x155551554505,
    0x15, 0x100110500, 0x55110510501, 0x155551554505,0x555555555515]
```
4、根据《四库全书 - 协纪辨方书》添加了部分的吉神凶煞、八字，修正了部分八字月柱算法，关于值神方面的算法没有采用农历月算，而是根据农历八字月柱为依据，不同算法之间有细微区别。主要参考了老黄历、聚宝堂日历等，待持续优化

铺注条例：
凡铺注《万年历》、《通书》，先依用事次第察其所宜忌之日，于某日下注宜某事，某日下注忌某事，次按宜忌，较量其凶吉之轻重，以定去取。
凡宜宣政事，布政事之日，只注宜宣政事。
凡宜营建宫室、修宫室之日，只注宜营建宫室。

凡吉足胜凶，从宜不从忌者，如遇德犹忌之事，则仍注忌。

凡吉凶相抵，不注宜亦不注忌者，如遇德犹忌之事，则仍注忌。

凡德合、赦愿、月恩、四相、时德等日，不注忌进人口、安床、经络、酝酿、开市、立券、交易、纳财、开仓库、出货财。如遇德犹忌，及从忌不从宜之日，则仍注忌。
凡天狗寅日忌祭祀，不注宜求福、祈嗣。

凡卯日忌穿井，不注宜开渠。壬日忌开渠，不注宜穿井。
凡巳日忌出行，不注宜出师、遣使。
凡酉日忌宴会，亦不注宜庆赐、赏贺。
凡丁日忌剃头，亦不注宜整容。

凡吉凶相抵，不注忌祈福，亦不注忌求嗣。

凡忌诏命公卿、招贤，不注宜施恩、封拜、举正直、袭爵受封。
凡忌施恩、封拜、举正直、袭爵受封，亦不注宜诏命公卿、招贤。
凡宜宣政事之日遇往亡则改宣为布。
凡月厌忌行幸、上官，不注宜颁诏、施恩封拜、诏命公卿、招贤、举正直。遇宜宣政事之日，则改宣为布。

凡吉凶相抵，不注忌结婚姻，亦不注忌冠带、纳采问名、嫁娶、进人口，如遇德犹忌之日则仍注忌。
凡吉凶相抵，不注忌嫁娶，亦不注忌冠带、结婚姻、纳采问名、进人口、搬移、安床，如遇德犹忌之日，则仍注忌。遇不将而不注忌嫁娶者，亦仍注忌。遇亥日、厌对、八专、四忌、四穷而仍注忌嫁娶者，只注所忌之事，其不忌者仍不注忌。
凡吉凶相抵，不注忌搬移，亦不注忌安床。不注忌安床，亦不注忌搬移。如遇德犹忌之日，则仍注忌。
凡吉凶相抵，不注忌解除，亦不注忌整容、剃头、整手足甲。如遇德犹忌之日，则仍注忌。
凡吉凶相抵，不注忌修造动土、竖柱上梁，亦不注忌修宫室、缮城郭、筑提防、修仓库、鼓铸、苫盖、修置产室、开渠穿井、安碓硙、补垣塞穴、修饰垣墙、平治道涂、破屋坏垣。如遇德犹忌之日，则仍注忌。
凡吉凶相抵，不注忌开市，亦不注忌立券、交易、纳财。不注忌纳财，亦不注忌开市、立券、交易。不注忌立券、交易，亦不注忌开市、纳财。
凡吉凶相抵，不注忌开市、立券、交易，亦不注忌开仓库、出货财。如遇专忌之日，则仍注忌。
凡吉凶相抵，不注忌牧养，亦不注忌纳畜。不注忌纳畜，亦不注忌牧养。
凡吉凶相抵，有宜安葬不注忌启攒，有宜启攒不注忌安葬。

凡土府、土符、地囊，只注忌补垣，亦不注宜塞穴。
凡开日，不注宜破土、安葬、启攒，亦不注忌。遇忌则注。
凡四忌、四穷只忌安葬。如遇鸣吠、鸣吠对亦不注宜破土、启攒。
凡天吏、大时不以死败论者，遇四废、岁薄、逐阵仍以死败论。
凡岁薄、逐阵日所宜事，照月厌所忌删去，所忌依然依从本日。

凡二月甲戌、四月丙申、六月甲午、七月戊申、八月庚辰、九月辛卯、十月甲子、十二月甲子、德合与赦（天赦）、愿（天愿）所会之辰，诸事不忌。

5、星次
星纪、玄枵、娵訾、降娄、大梁、实沈、鹑首、鹑火、鹑尾、寿星、大火、析木