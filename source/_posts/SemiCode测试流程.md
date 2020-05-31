---
title: SemiCode测试流程
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-07-19 14:50:28
categories: 编程语言
tags: Java
---
第一次写脚本测试！
![](http://www.easyicon.net/api/resizeApi.php?id=1178455&size=64)
<!--more-->
# 测试目的
随机生成大量SemiCode，从译码检测算法完备性！

# 测试结果
1、第一次测试10万张码有5%的无法译出，单独调试发现是寻像算法寻像出错
2、为了补充寻像算法，写了个图片旋转的算法
3、第二次测试重新生成10万张码，全部译出
4、第三次测试10万张，全部译出
5、结论：代码的健壮性非常好！

# 测试流程
## 生成
1、用随机字符串生成内容，用随机数字生成输入内容大小
2、用时间戳给每张二维码命名
3、将生成的二维码的名字（时间戳）保存到encode.txt文件
4、将生成的二维码图片保存到encodePic文件夹
5、for循环生成10万张SemiCode二维码

## 解码
1、从encode.txt文本中分别读取每个图片的名字
2、根据图片name从encodePic中调取图片
3、for循环译码
4、将无法译出的图片旋转测试（补充寻像算法）
5、将译码结果保存到decode.txt
6、从decode.txt中查找译码信息为null的二维码单独调试


<br/>
# 关键代码
## 生成随机字符串以及随机汉字 
```
public static String RandomString(int length) {
        String str = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789:/?@#$&*()+";
        Random random = new Random();
        StringBuffer buf = new StringBuffer();
        for (int i = 0; i < length-1; i++) {
            int num = random.nextInt(73);
            if(num==72){
                length-=1;
            }
            buf.append( num!=72?  str.charAt(num):(char) (0x4e00 + (int) (Math.random() * (0x9fa5 - 0x4e00 + 1))));
        }
        return buf.toString();
    }
```
## 生成随机数字
```
  public static int suijinum(int max) {
          int min=2;
          Random random = new Random();
          int s = random.nextInt(max)%(max-min+1) + min;
          return s;
      }
```
## 时间戳生成随机图片name
```
String pname = "a";
Random rand = new Random();
SimpleDateFormat df = new SimpleDateFormat("yyyyMMddHHmmss");//设置日期格式
String time = df.format(new Date());
int randnum1 = rand.nextInt(900)+100;
time=time.concat(String.valueOf(randnum1));
int randnum2 = rand.nextInt(90)+10;
time=time.concat(String.valueOf(randnum2));
pname=time;
```
## 读写txt文件
```
import java.io.File;  
import java.io.InputStreamReader;  
import java.io.BufferedReader;  
import java.io.BufferedWriter;  
import java.io.FileInputStream;  
import java.io.FileWriter;  
  
public class cin_txt {  
    static void main(String args[]) {  
        try { // 防止文件建立或读取失败，用catch捕捉错误并打印，也可以throw  
  
            /* 读入TXT文件 */  
            String pathname = "D:\\twitter\\13_9_6\\dataset\\en\\input.txt"; // 绝对路径或相对路径都可以，这里是绝对路径，写入文件时演示相对路径  
            File filename = new File(pathname); // 要读取以上路径的input。txt文件  
            InputStreamReader reader = new InputStreamReader(  
                    new FileInputStream(filename)); // 建立一个输入流对象reader  
            BufferedReader br = new BufferedReader(reader); // 建立一个对象，它把文件内容转成计算机能读懂的语言  
            String line = "";  
            line = br.readLine();  
            while (line != null) {  
                line = br.readLine(); // 一次读入一行数据  
            }  
  
            /* 写入Txt文件 */  
            File writename = new File(".\\result\\en\\output.txt"); // 相对路径，如果没有则要建立一个新的output。txt文件  
            writename.createNewFile(); // 创建新文件  
            BufferedWriter out = new BufferedWriter(new FileWriter(writename));  
            out.write("我会写入文件啦\r\n"); // \r\n即为换行  
            out.flush(); // 把缓存区内容压入文件  
            out.close(); // 最后记得关闭文件  
  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
}
```
## 计算代码时间
```
long startTime=System.currentTimeMillis();   //获取开始时间  
doSomeThing();  //测试的代码段  
long endTime=System.currentTimeMillis(); //获取结束时间  
System.out.println("程序运行时间： "+(endTime-startTime)+"ms");
```
## 旋转图片测试,返回图片name
```
 /**
     * 旋转
     *
     * @param degree
     *            旋转角度
     * @throws Exception
     */
    public static String spin(int degree ,String imgPath,String line) throws Exception {
        int swidth = 0; // 旋转后的宽度
        int sheight = 0; // 旋转后的高度
        int x; // 原点横坐标
        int y; // 原点纵坐标

        File file = new File(imgPath);
        if (!file.isFile()) {
            throw new Exception("ImageDeal>>>" + file + " 不是一个图片文件!");
        }
        BufferedImage bi = ImageIO.read(file); // 读取该图片
        // 处理角度--确定旋转弧度
        degree = degree % 360;
        if (degree < 0)
            degree = 360 + degree;// 将角度转换到0-360度之间
        double theta = Math.toRadians(degree);// 将角度转为弧度

        // 确定旋转后的宽和高
        if (degree == 180 || degree == 0 || degree == 360) {
            swidth = bi.getWidth();
            sheight = bi.getHeight();
        } else if (degree == 90 || degree == 270) {
            sheight = bi.getWidth();
            swidth = bi.getHeight();
        } else {
            swidth = (int) (Math.sqrt(bi.getWidth() * bi.getWidth()
                    + bi.getHeight() * bi.getHeight()));
            sheight = (int) (Math.sqrt(bi.getWidth() * bi.getWidth()
                    + bi.getHeight() * bi.getHeight()));
        }

        x = (swidth / 2) - (bi.getWidth() / 2);// 确定原点坐标
        y = (sheight / 2) - (bi.getHeight() / 2);

        BufferedImage spinImage = new BufferedImage(swidth, sheight,
                bi.getType());
        // 设置图片背景颜色
        Graphics2D gs = (Graphics2D) spinImage.getGraphics();
        gs.setColor(Color.white);
        gs.fillRect(0, 0, swidth, sheight);// 以给定颜色绘制旋转后图片的背景

        AffineTransform at = new AffineTransform();
        at.rotate(theta, swidth / 2, sheight / 2);// 旋转图象
        at.translate(x, y);
        AffineTransformOp op = new AffineTransformOp(at,
                AffineTransformOp.TYPE_BICUBIC);
        spinImage = op.filter(bi, spinImage);
        String linesp=line+degree;
        File sf = new File("G:/ProjectQRcode/SpringPro/Test/spanpic", linesp + "." +"png");
        ImageIO.write(spinImage, "png", sf); // 保存图片
        return linesp;
    }
```



![](http://image54.360doc.com/DownloadImg/2012/08/2512/26415405_1.gif)