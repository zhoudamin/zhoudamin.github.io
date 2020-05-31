---
title: Java Draw
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-07-23 20:27:16
categories: Java
tags: 绘图
---
Java绘图基础！
![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1501417779&di=1b7bb238ccf0eada0f82bcaad7a98ca2&imgtype=jpg&er=1&src=http%3A%2F%2Ftupian.enterdesk.com%2Fuploadfile%2F2014%2F0809%2F20140809103712945.jpg.200.150.jpg)
<!--more-->
# 简单绘画
直线
矩形
圆
根据矩阵画图

```
package com.zhoudm;
import java.awt.*;
import javax.swing.*;

public class Draw extends JFrame
{
    MyPanel mp = null ;

    public static void main(String[] args)
    {
        // TODO Auto-generated method stub
        Draw qwe = new Draw();

    }

    public Draw()
    {
        mp = new MyPanel();

        this.add(mp);
        this.setSize(400,300);
        this.setVisible(true);
        this.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    }
}

class MyPanel extends JPanel    //我自己的面板，用于绘图和实现绘图区域
{
    //覆盖JPanel的paint方法
    //Graphics是绘图的重要类，可以理解成一支画笔
    public void paint(Graphics g)
    {
        //1.调用父类函数完成初始化
        super.paint(g);     //这句话不能少
        //System.out.println("被调用");        //证明别调用

        //画圆
        int [][] drawnum={{1,0,1,1},
                          {0,1,0,1},
                          {1,0,1,1},
                          {1,1,0,1}};
        for(int i=0;i<drawnum.length;i++){
            for(int j=0;j<drawnum[0].length;j++){
                if(drawnum[i][j]==1){
                    g.drawOval(30*i+50,30*j+50,25,25);
                    g.setColor(Color.BLUE);
                }
            }
        }
      //  g.drawOval(10, 10, 30, 30);
        //画直线
      //  g.drawLine(20, 30, 20, 80);
        //画出矩形边框
    //    g.drawRect(50, 50, 100, 50);
        //画填充矩形
     //   g.setColor(Color.BLUE);     //设置颜色
      //  g.fillRect(80,60,40,60);

    }
}
```
<br/>
# Java嵌入图片
```
class MyPanel extends JPanel    //我自己的面板，用于绘图和实现绘图区域
{
    //覆盖JPanel的paint方法
    //Graphics是绘图的重要类，可以理解成一支画笔
    public void paint(Graphics g)
    {
        //放置图片
        Image im = Toolkit.getDefaultToolkit().getImage
                (Panel.class.getResource("/sysu.jpg"));
        g.drawImage(im, 50, 50, 70, 70, this);      //this代指JPanel本身，意思是把图片放这上面
	}
}
```
# 将矩形图片切成圆形
周边透明！
```
import java.awt.*;
import java.awt.geom.Ellipse2D;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

import javax.imageio.ImageIO;

public class Main {
    public static void main(String[] args) throws IOException {
        BufferedImage bi1 = ImageIO.read(new File("G:/code/Java/leetcode/src/mm.jpg"));


        // 根据需要是否使用 BufferedImage.TYPE_INT_ARGB
        BufferedImage image = new BufferedImage(bi1.getWidth(), bi1.getHeight(),
                BufferedImage.TYPE_INT_ARGB);

        Ellipse2D.Double shape = new Ellipse2D.Double(0, 0, bi1.getWidth(), bi1
                .getHeight());

        Graphics2D g2 = image.createGraphics();
        image = g2.getDeviceConfiguration().createCompatibleImage(bi1.getWidth(), bi1.getHeight(), Transparency.TRANSLUCENT);
        g2 = image.createGraphics();
        g2.setComposite(AlphaComposite.Clear);
        g2.fill(new Rectangle(image.getWidth(), image.getHeight()));
        g2.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC, 1.0f));
        g2.setClip(shape);
        // 使用 setRenderingHint 设置抗锯齿
        g2.drawImage(bi1, 0, 0, null);
        g2.dispose();

        try {
            ImageIO.write(image, "PNG", new File("G:/code/Java/leetcode/src/mm2.jpg"));
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
}
```

# 保存图片
但是保存不了组建图片
```
Dimension imageSize = qwe.getSize();
        BufferedImage image = new BufferedImage(imageSize.width,
                imageSize.height, BufferedImage.TYPE_INT_ARGB);
        Graphics2D g = image.createGraphics();
        qwe.paint(g);
        g.dispose();
        try {
            ImageIO.write(image, "png", new File("G:/code/Java/leetcode/src/sysu2.jpg"));
        } catch (IOException e) {
            e.printStackTrace();
        }
```




![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1501417829&di=92f1cfa22efa1d88ac0fea28f541f177&imgtype=jpg&er=1&src=http%3A%2F%2Fp2.qhimg.com%2Ft01734e32fa1e77bbc0.jpg)