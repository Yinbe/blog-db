Jetpack Compose和Android View的相互嵌套，对于Compose来说, 和Android View的结合是无缝的。

新界面想用Compose, Compose支持不了的, 用Android View。已有Android View界面的一部分想用Compose, 可以。有的UI效果想复用之前的, 可以直接拿来内嵌。

```
    buildFeatures {
        compose true
        viewBinding true
    }
```

```
dependencies {
    implementation "androidx.compose.ui:ui-viewbinding:1.1.0"
}  
```

jetpack_compose.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <TextView
        android:text="我是xml View"
        android:layout_gravity="center"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
    <androidx.compose.ui.platform.ComposeView
        android:id="@+id/compose_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>
    <FrameLayout
        android:id="@+id/fl_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</LinearLayout>
```

```
@Preview(showBackground = true)
@Composable
fun DefaultPreview11() {
    JetpackTheme {
        Column() {
            val view = View.inflate(LocalContext.current,R.layout.jetpack_compose,null)
            AndroidView(factory = {view}){
                val mComposeView = it.findViewById<ComposeView>(R.id.compose_view)
            }
            AndroidViewBinding(JetpackComposeBinding::inflate){
                composeView.setContent { 
                    Text(text = "我是AndroidView里面的JetpackCompose")
                }

            }
        }
    }
}
```
