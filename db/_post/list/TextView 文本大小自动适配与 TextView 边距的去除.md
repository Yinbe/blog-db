> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7106714269730914312)

本文已参与「新人创作礼」活动，一起开启掘金创作之路。

TextView 文本大小自动适配与 TextView 边距的去除
-----------------------------------------------

标题太难取了，其实本文主要就是讲如何控制文本大小，让其自动适配宽度，其次我们还需要精准控制 Text 的高度和宽度间距等属性。

一般我们的布局都是分 match parent 和 wrap content 而他们的自动方式又有所不同。下面看看都有哪些方式来实现！

#### 一、Autosizing 的方式 (固定宽度)

官方推出的 TextView 的 Autosizing 方式，在宽度固定的情况下，可以设置最大文本 Size 和最小文本 Size 和每次缩放粒度，非常方便的就能实现该功能。

```
 <TextView
        android:layout_width="340dp"
        android:layout_height="50dp"
        android:background="@drawable/shape_bg_008577"
        android:gravity="center_vertical"
        android:maxLines="1"
        android:text="这是标题，该标题的名字比较长，产品要求不换行全部显示出来"
        android:textSize="18sp"
        android:autoSizeTextType="uniform"
        android:autoSizeMaxTextSize="18sp"
        android:autoSizeMinTextSize="10sp"
        android:autoSizeStepGranularity="1sp"/>


```

1. autoSizeTextType：设置 TextView 是否支持自动改变文本大小，none 表示不支持，uniform 表示支持。
2. autoSizeMinTextSize：最小文字大小，例如设置为 10sp，表示文字最多只能缩小到 10sp。
3. autoSizeMaxTextSize：最大文字大小，例如设置为 18sp，表示文字最多只能放大到 18sp。
4. autoSizeStepGranularity：缩放粒度，即每次文字大小变化的数值，例如设置为 1sp，表示每次缩小或放大的值为 1sp。

效果：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e2d64944a41499bb3dc6688e800179b~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

如果在 Java 代码中使用，我们也可以这么用

```
TextView tvText = findViewById(R.id.tv_text);
TextViewCompat.setAutoSizeTextTypeWithDefaults(tvText,TextViewCompat.AUTO_SIZE_TEXT_TYPE_UNIFORM);
TextViewCompat.setAutoSizeTextTypeUniformWithConfiguration(tvText,10,18,1, TypedValue.COMPLEX_UNIT_SP);


```

#### 二、自定义 View 的方式（固定宽度）

github 上有很多这种的 TextView 自定义，[类似这样](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Figlaweb%2FAutoSizeTextView%2Fblob%2Fmaster%2Flibrary%2Fsrc%2Fmain%2Fjava%2Fru%2Figla%2Fwidget%2FAutoSizeTextView.java "https://github.com/iglaweb/AutoSizeTextView/blob/master/library/src/main/java/ru/igla/widget/AutoSizeTextView.java")的。

其核心思想和上面的 Autosizing 的方式类似，一般是测量 TextView 字体所占的宽度与 TextView 控件的宽度对比，动态改变 TextView 的字体大小。

它们的类似用法如下：

```
    <ru.igla.widget.AutoSizeTextView
        android:id="@+id/tvFullscreen"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:text="Long ancestry"
        android:textColor="@android:color/black"
        android:background="@android:color/white"
        android:textSize="500sp"
        android:maxLines="500"
        android:gravity="center"
        android:ellipsize="@null"
        android:autoText="false"
        android:autoLink="none"
        android:linksClickable="false"
        android:singleLine="false"
        android:padding="0px"
        android:includeFontPadding="false"
        android:textAlignment="center"
        android:typeface="normal"
        android:layout_gravity="center"
        android:textStyle="normal"
        app:minTxtSize="8sp"
        />


```

效果和方案一类似

#### 三、使用工具类自行计算（非控件固定宽度）

把第二步中自定义 View 计算宽度的方法抽取出来，我们可以可以得到一个工具类如下：

```
private void adjustTvTextSize(TextView tv, int maxWidth, String text) {
        int avaiWidth = maxWidth - tv.getPaddingLeft() - tv.getPaddingRight();


        if (avaiWidth <= 0) {
            return;
        }

        TextPaint textPaintClone = new TextPaint(tv.getPaint());
   
        float trySize = textPaintClone.getTextSize();


        while (textPaintClone.measureText(text) > avaiWidth) {
            trySize--;
            textPaintClone.setTextSize(trySize);
        }


        tv.setTextSize(TypedValue.COMPLEX_UNIT_PX, trySize);
    }


```

Demo 如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f197c81e8cda4d3bacfa725f2c7e1442~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

右侧的 LinearLayout 中需要包含 2 个文本 一个 14sp 一个是 30sp，同时居中但是要金额的文本自动适配大小。

```
            <LinearLayout      
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="@dimen/d_15dp"
                android:layout_marginRight="@dimen/d_15dp"
                android:gravity="center"
                android:orientation="horizontal">

                <TextView
                    android:id="@+id/tv_job_detail_dollar"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="$"
                    android:textColor="@color/black"
                    android:textSize="@dimen/job_detail_message_size"/>

                <TextView
                    android:id="@+id/text_view_hourly_rate"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginLeft="@dimen/d_2dp"
                    android:singleLine="true"
                    android:text="-"
                    android:textColor="@color/job_detail_black"
                    android:textSize="30sp" />

            </LinearLayout>


```

可以看到 2 个都是 wrap content，那么如何实现这种适应宽度 + 多布局的变长宽度效果呢。其实就是需要我们调用方法手动的计算金额 TextView 的宽度

```
    int mFullNameTVMaxWidth = CommUtils.dip2px(60);

    //    mTextViewHourlyRate.setText(totalMoney);
    //     while (true) {
    //         float measureTextWidth = mTextViewHourlyRate.getPaint().measureText(totalMoney);

    //         if (measureTextWidth > mFullNameTVMaxWidth) {
    //             int textSize = (int) mTextViewHourlyRate.getTextSize();
    //             textSize = textSize - 2;
    //             mTextViewHourlyRate.setTextSize(TypedValue.COMPLEX_UNIT_PX, textSize);
    //         } else {
    //             break;
    //         }
    //     }

    adjustTvTextSize(mTextViewHourlyRate,mFullNameTVMaxWidth,totalMoney)


```

效果如下：（该效果是去除边距之后的对齐效果）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/968195e8976848098853090d64338152~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4f33598f77de4b299742473b4933ba47~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 四、去除 TextView 的边距

我们都知道 TextView 绘制的时候并非是我们平常自定义 View 那种 drawText，而是分为几块区域，基于基线绘制文本，并加入了上下左右的间距。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c1d5078c77344049f0722a8a9e5c162~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

而不同的 TextSize 它的间距还不同，比如上文中我们一个很小的 TextView 和一个很大的 TextView 在一起排列的时候，特别是大的 TextView 还是 AutoSize 的情况下，实现一些对齐效果就很难实现，并且本来宽度就极其有限了，再加上这么多无用的间距，搞得我们很难实现效果嘛！那我们就需要考虑到去除间距，只保留上图灰色的矩形框来绘制文本。

代码如下：

```
public class NoPaddingTextView extends AppCompatTextView {
    private Paint mPaint = getPaint();
    private Rect mBounds = new Rect();

    private Boolean mRemoveFontPadding = false;//是否去除字体内边距，true：去除 false：不去除

    public NoPaddingTextView(Context context) {
        super(context);
    }

    public NoPaddingTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initAttributes(context, attrs);
    }

    public NoPaddingTextView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initAttributes(context, attrs);
    }

    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        if (mRemoveFontPadding) {
            calculateTextParams();
            setMeasuredDimension(mBounds.right - mBounds.left, -mBounds.top + mBounds.bottom);
        }
    }

    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
    }

    protected void onDraw(Canvas canvas) {
        drawText(canvas);
    }

    /**
     * 初始化属性
     */
    private void initAttributes(Context context, AttributeSet attrs) {
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.NoPaddingTextView);
        mRemoveFontPadding = typedArray.getBoolean(R.styleable.NoPaddingTextView_removeDefaultPadding, false);
        typedArray.recycle();
    }

    /**
     * 计算文本参数
     */
    private String calculateTextParams() {
        String text = getText().toString();
        int textLength = text.length();
        mPaint.getTextBounds(text, 0, textLength, mBounds);
        if (textLength == 0) {
            mBounds.right = mBounds.left;
        }
        return text;
    }

    /**
     * 绘制文本
     */
    private void drawText(Canvas canvas) {
        String text = calculateTextParams();
        int left = mBounds.left;
        int bottom = mBounds.bottom;
        mBounds.offset(-mBounds.left, -mBounds.top);
        mPaint.setAntiAlias(true);
        mPaint.setColor(getCurrentTextColor());
        canvas.drawText(text, (float) (-left), (float) (mBounds.bottom - bottom), mPaint);
    }
}


```

使用：

```
            <LinearLayout         
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginLeft="@dimen/d_15dp"
                android:layout_marginRight="@dimen/d_15dp"
                android:gravity="center"
                android:orientation="horizontal">

                <com.guadou.componentservice.widget.view.NoPaddingTextView
                    android:id="@+id/tv_job_detail_dollar"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="$"
                    android:textColor="@color/black"
                    android:background="@color/yellow"
                    android:textSize="@dimen/job_detail_message_size"
                    app:removeDefaultPadding="true" />

                <com.guadou.componentservice.widget.view.NoPaddingTextView
                    android:id="@+id/text_view_hourly_rate"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:layout_marginLeft="@dimen/d_2dp"
                    android:singleLine="true"
                    android:text="-"
                    android:background="@color/red"
                    android:textColor="@color/job_detail_black"
                    android:textSize="30sp"
                    app:removeDefaultPadding="true" />

            </LinearLayout>


```

效果如下：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7d7745b87ae4310a80144a3e75f0ecf~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

到此我们就能随心的控制 TextView 的大小和间距，想让文本多大就多大，想在哪展示就在哪展示，很方便的实现对齐和绝大部分文本的展示效果了。

#### 补充

有同学讲到 `includeFontPadding` 属性去除边距，无需自定义 View 的方式。那么我们看看 `includeFontPadding` 和自定义 View 去除边距的区别。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/802678f4071a4130bae3f7c6d527b283~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

从上到下是默认的 TextView 设置 `includeFontPadding` 的效果 自定义 View 的效果。可以看到 `includeFontPadding` 的效果确实是去除了边距，但是又没完全去除，会保留顶部和底部的一些间距。而自定义 View 的方式去除的比较明显。

另外有同学补充到，可以基于基线对齐，确实是这样的，如果使用 `ConstraintLayout` 的布局包裹 2 个不同大小的 `TextView` 确实可以使用基于基线对齐的方式！效果和比去除间距之后的底部对齐方式差不多。

如有更多的方法和方案欢迎评论区讨论。

完结！

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c37e63fa3c642ff9f3a5875a476ca6a~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)
