给BottomSheetDialog 设置默认高度，全屏显示

翻看了一下 BottomSheetBehavior 的源码

```java
	if (this.peekHeightAuto) {
            if (this.peekHeightMin == 0) {
                this.peekHeightMin = parent.getResources().getDimensionPixelSize(dimen.design_bottom_sheet_peek_height_min);
            }

            this.lastPeekHeight = Math.max(this.peekHeightMin, this.parentHeight - parent.getWidth() * 9 / 16);
        } else {
            this.lastPeekHeight = this.peekHeight;
        }
```

```java
        TypedArray a = context.obtainStyledAttributes(attrs, styleable.BottomSheetBehavior_Layout);
        TypedValue value = a.peekValue(styleable.BottomSheetBehavior_Layout_behavior_peekHeight);
        if (value != null && value.data == -1) {
            this.setPeekHeight(value.data);
        } else {
            this.setPeekHeight(a.getDimensionPixelSize(styleable.BottomSheetBehavior_Layout_behavior_peekHeight, -1));
        }

        this.setHideable(a.getBoolean(styleable.BottomSheetBehavior_Layout_behavior_hideable, false));
        this.setFitToContents(a.getBoolean(styleable.BottomSheetBehavior_Layout_behavior_fitToContents, true));
        this.setSkipCollapsed(a.getBoolean(styleable.BottomSheetBehavior_Layout_behavior_skipCollapsed, false));
```

### ~~设置默认高度（废弃）~~

PeekHeight 折叠状态，可以有3种方式设置。

1、设置 dimen  design_bottom_sheet_peek_height_min。 缺点 会影响所有bottom_sheet

2、在主题设置 behavior_peekHeight     缺点 每种bottom_sheet都要配置主题

```
<style name="Widget.Design.BottomSheet.Modal" parent="android:Widget">
    <item name="android:background">?android:attr/colorBackground</item>
    <item name="android:elevation" ns1:ignore="NewApi">
      @dimen/design_bottom_sheet_modal_elevation
    </item>
    <item name="behavior_peekHeight">auto</item>
    <item name="behavior_hideable">true</item>
    <item name="behavior_skipCollapsed">false</item>
  </style>
```

3、给layout xml中设置 minHeight    缺点 minHeight并不是实际高度

### 设置默认全屏

```
    private View wrapInBottomSheet(int layoutResId, View view, LayoutParams params) {
        FrameLayout container = (FrameLayout)View.inflate(this.getContext(), layout.design_bottom_sheet_dialog, (ViewGroup)null);
        CoordinatorLayout coordinator = (CoordinatorLayout)container.findViewById(id.coordinator);
        if (layoutResId != 0 && view == null) {
            view = this.getLayoutInflater().inflate(layoutResId, coordinator, false);
        }

        FrameLayout bottomSheet = (FrameLayout)coordinator.findViewById(id.design_bottom_sheet);
        this.behavior = BottomSheetBehavior.from(bottomSheet);
        this.behavior.setBottomSheetCallback(this.bottomSheetCallback);
        this.behavior.setHideable(this.cancelable);
        if (params == null) {
            bottomSheet.addView(view);
        } else {
            bottomSheet.addView(view, params);
        }
	............
    }
```

通过源码发现，拿到BottomSheetBehavior就可以很方便设置peekHight，设置state

```
BottomSheetBehavior mBehavior = BottomSheetBehavior.from((View)getView().getParent());
mBehavior.setPeekHeight(1000); //设置默认高度，折叠态

//设置默认全屏
mBehavior.setState(BottomSheetBehavior.STATE_EXPANDED) //设置为展开状态
mBehavior.setKipCollapsed(true)  //设置下滑跳过折叠态
```

设置默认全屏

```
给layout xml中设置 minHeight   (大于屏幕高度)
BottomSheetBehavior mBehavior = BottomSheetBehavior.from((View)getView().getParent());
mBehavior.skipCollapsed = true
mBehavior.state=BottomSheetBehavior.STATE_EXPANDED;
```
