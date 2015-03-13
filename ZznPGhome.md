# 我的地盘我做主 #

# Introduction #

游戏进展.


# Details #

Add your content here.  Format your content with:
  * 为游戏功能做准备，实现在屏幕下方显示两个按钮，当按其中一个按钮的时候，在屏幕中间显示按钮上的文字
  * 这个功能在以后可能用的上
  * 代码下载地址 http://androidteam.googlecode.com/files/test.rar
# SurfaceView类 #
  * urfaceView在游戏开发中有着举足轻重的地位，它对于画面的控制有着更大的自由度（不像View要用handler来更新，关于View的），但这方面的参考资料并不是太多，能找到的例子都有点喧宾夺主的感觉，不能把使用的流程清晰展示出来，下面是个简单的示例，力求把流程清楚展示，其他的可简则简。


```
package com.ray.test;   
/*  
 * SurfaceView的示例程序  
 * 演示其流程  
 */  
import android.app.Activity;   
import android.content.Context;   
import android.graphics.Canvas;   
import android.graphics.Color;   
import android.graphics.Paint;   
import android.graphics.RectF;   
import android.os.Bundle;   
import android.view.SurfaceHolder;   
import android.view.SurfaceView;   
  
public class Test extends Activity {   
    public void onCreate(Bundle savedInstanceState) {   
        super.onCreate(savedInstanceState);   
        setContentView(new MyView(this));   
    }   
       
    //内部类   
    class MyView extends SurfaceView implements SurfaceHolder.Callback{   
  
        SurfaceHolder holder;   
        public MyView(Context context) {   
            super(context);   
            holder = this.getHolder();//获取holder   
            holder.addCallback(this);   
            //setFocusable(true);   
               
        }   
  
        @Override  
        public void surfaceChanged(SurfaceHolder holder, int format, int width,   
                int height) {   
               
        }   
  
        @Override  
        public void surfaceCreated(SurfaceHolder holder) {   
            new Thread(new MyThread()).start();   
        }   
  
        @Override  
        public void surfaceDestroyed(SurfaceHolder holder) {   
               
        }   
           
        //内部类的内部类   
        class MyThread implements Runnable{   
  
            @Override  
            public void run() {   
                Canvas canvas = holder.lockCanvas(null);//获取画布   
                Paint mPaint = new Paint();   
                mPaint.setColor(Color.BLUE);   
                   
                canvas.drawRect(new RectF(40,60,80,80), mPaint);   
                holder.unlockCanvasAndPost(canvas);//解锁画布，提交画好的图像   
                   
            }   
               
        }   
           
    }   
} 
```