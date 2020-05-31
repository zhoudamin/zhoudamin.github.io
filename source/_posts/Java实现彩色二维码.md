---
title: Java实现彩色二维码
categories:
  - 编程语言
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2018-05-02 19:30:05
tags: Java
  	  二维码
---
使用Zxing库实现彩色二维码的生成，代码如下：
<!--more-->
```java
package com;

import com.google.zxing.BarcodeFormat;
import com.google.zxing.EncodeHintType;
import com.google.zxing.MultiFormatWriter;
import com.google.zxing.client.j2se.MatrixToImageWriter;
import com.google.zxing.common.BitMatrix;
import com.google.zxing.qrcode.QRCodeWriter;
import com.google.zxing.qrcode.decoder.ErrorCorrectionLevel;
import com.google.zxing.qrcode.encoder.Encoder;

import javax.imageio.ImageIO;
import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;
import java.util.Hashtable;
import java.util.Map;
import java.util.Random;

public class TexturedEncoderHandler {
    /**
     * 编码
     * @param contents
     * @param width
     * @param height
     * @param imgPath
     */
    public void encode(String contents, int width, int height, ErrorCorrectionLevel level, String imgPath) {
        Hashtable<EncodeHintType, Object> hints = new Hashtable<EncodeHintType, Object>();
        // 指定纠错等级
        hints.put(EncodeHintType.ERROR_CORRECTION,  level);
        // 指定编码格式
        hints.put(EncodeHintType.CHARACTER_SET, "GBK");
        try {
            int [][] bitMatrix = new QRCodeWriter().encode(contents, BarcodeFormat.QR_CODE, width, height, hints);
            int len=bitMatrix.length;
            width=len*12;
            boolean [][]matrixBoo=new boolean[width][width];
            MyPic(bitMatrix,matrixBoo,imgPath);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    
    public  void MyPic(int [][] bitmatrix,boolean [][] matrixboo,String pathname) throws IOException {
        BufferedImage matrix = new BufferedImage(matrixboo.length , matrixboo[0].length, BufferedImage.TYPE_INT_ARGB);
        Random rand = new Random();
        for(int i=0;i<bitmatrix.length;i++){
            for(int j=0;j<bitmatrix[0].length;j++){
                if(bitmatrix[i][j]==1){
                    int rr= rand.nextInt(255);
                    int gg= rand.nextInt(255);
                    int bb= rand.nextInt(255);
                    Color color = new Color(rr, gg, bb);
                    int colorInt = color.getRGB();
                    for(int n=i*12;n<(i+1)*12;n++){
                        for(int m=j*12;m<(j+1)*12;m++) {
                            matrixboo[n][m] = true;
                            matrix.setRGB(n, m, colorInt );
                        }
                    }
                }
            }
        }
        ImageIO.write(matrix, "PNG", new File(pathname));
    }

    /**
     * @param args
     */
    public static void main(String[] args) {
        long startTime=System.currentTimeMillis();
        String imgPath = "guangzhou.jpg";
        String contents = "guangzhou";
        int width = 300, height = 300;
        ErrorCorrectionLevel level=ErrorCorrectionLevel.L;
        TexturedEncoderHandler handler = new TexturedEncoderHandler();
        handler.encode(contents, width, height,  level, imgPath);
        long endTime=System.currentTimeMillis(); //获取结束时间
        System.out.println("程序运行时间： "+(endTime-startTime)+"ms");
    }
}
```
