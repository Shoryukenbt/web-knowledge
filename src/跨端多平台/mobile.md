## 移动端适配

1. 像素

    设备像素：指设备的物理像素, 取决于硬件

    逻辑像素(设备独立像素)：操作系统定义到像素单位，为了保证在各种不分辨率的设备上显示基本一致，统一程序逻辑

    css像素：css中定义的px值

    ***缩放比例(scale)*** = css像素边长 / 设备独立像素边长

    ***设备像素比(devicePixelRatio)*** = (相同长度下)设备像素数量 / css像素数量

2. 视口

    ***layout viewport(布局视口)***：网页布局的区域

    ***visual viewport(视图视口)***：显示在屏幕上的网页区域，可能只是布局视口的一部分，需要通过滚动查看完整布局视口内容

    ***ideal viewport(理想视口)***：宽度等于设备宽度，针对该视口的设计不用手动缩放也不需要出现横向滚动条

    >移动设备默认的视口为`layout viewport`, 所以我们需要使用meta标签转换为`ideal viewport`, 如：`<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0">`

3. 方案选择
    
    3.1. 使用css的媒体查询`@media`。 页面所有元素都需要根据设备重新定义，工作量大

    3.2. 使用rem单位。 接祖@media为不同设备定义不同的根元素font-size，所有需要使用rem的地方都需要计算具体的值

    3.3. flexible 使用动态js设置根字体

        3.3.1. 使用rem模拟vw特性
        3.3.2. 设置viewport的width的scale适配高倍屏。设置width=device-width, 而等比设置scale, 保证1css像素=1物理像素， `scale=1/dpr`

        > 存在如下问题：不在兼容@media，因为@media查询的是物理像素, 和flexible意境相悖
    
4. viewport方案

    使用vw作为单位，从根本上解决了不同尺寸屏幕的适配
    
    4.1. 设置meta标签，使viewport宽度等于设备宽度, 不允许用户缩放

    4.2. 将像素单位转换成vw (postcss-px-to-viewport)
    > postcss-px-to-viewport不能转换行内样式

5. Retina屏
    
    根据dpr设置不同分辨率的图片

6. 1px问题  
    对于Retina屏，1px线会显得比较粗，因为1px css像素并不等于1px 物理像素
    
    可以使用如下方法解决:

    > a. transform: scale(0.5)

    > b. transform: scale(0.5) + :before/:after 推荐

    > c. box-shadow

    > d. viewport缩放, border-image, background-image, liner-gradient, svg

6. vw兼容
    

