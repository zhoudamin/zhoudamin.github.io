---
title: Java 运动模糊
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2018-04-23 20:41:58
categories: 图像
tags: Java
---

想用Java 写个运动模糊的效果，无奈本人水平有限，国内也没找到资源，于是Google到了一个文档，特地分享出来！

<!--more-->

本代码源自   http://www.jhlabs.com/ip/blurring.html

Java运动模糊算法：
```
import java.awt.*;
import java.awt.geom.*;
import java.awt.image.*;

public class MotionBlurOp extends AbstractBufferedImageOp {
    private float centreX = 0.5f, centreY = 0.5f;
    private float distance=20.0f;                //这里设置运动距离
    private float angle;
    private float rotation;
    private float zoom;

    public MotionBlurOp() {
    }

    public MotionBlurOp( float distance, float angle, float rotation, float zoom ) {
        this.distance = distance;
        this.angle = angle;
        this.rotation = rotation;
        this.zoom = zoom;
    }

    public void setAngle( float angle ) {
        this.angle = angle;
    }

    public float getAngle() {
        return angle;
    }

    public void setDistance( float distance ) {
        this.distance = distance;
    }

    public float getDistance() {
        return distance;
    }

    public void setRotation( float rotation ) {
        this.rotation = rotation;
    }

    public float getRotation() {
        return rotation;
    }

    public void setZoom( float zoom ) {
        this.zoom = zoom;
    }

    public float getZoom() {
        return zoom;
    }

    public void setCentreX( float centreX ) {
        this.centreX = centreX;
    }

    public float getCentreX() {
        return centreX;
    }

    public void setCentreY( float centreY ) {
        this.centreY = centreY;
    }

    public float getCentreY() {
        return centreY;
    }

    public void setCentre( Point2D centre ) {
        this.centreX = (float)centre.getX();
        this.centreY = (float)centre.getY();
    }

    public Point2D getCentre() {
        return new Point2D.Float( centreX, centreY );
    }

    private int log2( int n ) {
        int m = 1;
        int log2n = 0;

        while (m < n) {
            m *= 2;
            log2n++;
        }
        return log2n;
    }

    public BufferedImage filter( BufferedImage src, BufferedImage dst ) {
        if ( dst == null )
            dst = createCompatibleDestImage( src, null );
        BufferedImage tsrc = src;
        float cx = (float)src.getWidth() * centreX;
        float cy = (float)src.getHeight() * centreY;
        float imageRadius = (float)Math.sqrt( cx*cx + cy*cy );
        float translateX = (float)(distance * Math.cos( angle ));
        float translateY = (float)(distance * -Math.sin( angle ));
        float scale = zoom;
        float rotate = rotation;
        float maxDistance = distance + Math.abs(rotation*imageRadius) + zoom*imageRadius;
        int steps = log2((int)maxDistance);

        translateX /= maxDistance;
        translateY /= maxDistance;
        scale /= maxDistance;
        rotate /= maxDistance;

        if ( steps == 0 ) {
            Graphics2D g = dst.createGraphics();
            g.drawRenderedImage( src, null );
            g.dispose();
            return dst;
        }

        BufferedImage tmp = createCompatibleDestImage( src, null );
        for ( int i = 0; i < steps; i++ ) {
            Graphics2D g = tmp.createGraphics();
            g.drawImage( tsrc, null, null );
            g.setRenderingHint( RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON );
            g.setRenderingHint( RenderingHints.KEY_INTERPOLATION, RenderingHints.VALUE_INTERPOLATION_BILINEAR );
            g.setComposite( AlphaComposite.getInstance( AlphaComposite.SRC_OVER, 0.5f ) );

            g.translate( cx+translateX, cy+translateY );
            g.scale( 1.0001+scale, 1.0001+scale );  // The .0001 works round a bug on Windows where drawImage throws an ArrayIndexOutofBoundException
            if ( rotation != 0 )
                g.rotate( rotate );
            g.translate( -cx, -cy );

            g.drawImage( dst, null, null );
            g.dispose();
            BufferedImage ti = dst;
            dst = tmp;
            tmp = ti;
            tsrc = dst;

            translateX *= 2;
            translateY *= 2;
            scale *= 2;
            rotate *= 2;
        }
        return dst;
    }

    public String toString() {
        return "Blur/Motion Blur...";
    }
}
```

测试代码：
```
import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.File;
import java.io.IOException;

/**
 * Created by zdmein on 2018/1/10.
 */
public class MotionBlurOpTest {
    public static void main(String [] args) throws IOException {
        BufferedImage sourceImage = ImageIO.read(new File("flower.jpg"));
        MotionBlurOp filter=new MotionBlurOp();
        BufferedImage destImage=filter.filter(sourceImage,null);
        ImageIO.write(destImage, "PNG", new File("MotionBlurOpflower.jpg"));

    }
}

```
![原图](http://images2017.cnblogs.com/blog/1300781/201801/1300781-20180110111018113-450639113.jpg)
![Motion](http://images2017.cnblogs.com/blog/1300781/201801/1300781-20180110111047176-161279166.jpg)

![](http://pic.sc.chinaz.com/files/pic/pic9/201508/apic14250.jpg)




![](http://img.hb.aicdn.com/402338e39ee91c9f2f55116942593e37d009fd15418553-ulMGQY_fw658)

