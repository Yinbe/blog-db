AndroidManifest.xml  与  application

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
         xmlns:tools="http://schemas.android.com/tools"
         package="com.**">
   <application 
		android:name="com.**.App"
                android:allowBackup="true"
                android:icon="@mipmap/app_icon"
                android:label="@string/app_name"
                android:supportsRtl="true"
                android:theme="@style/AppTheme"
                tools:replace="android:name">
   </application>
</manifest>

```

子模块之间的多个AndroidManifest.xml会在打包时合并，而application `android:name="com.**.App"`

全局注册到Manifest只能有一个，不然会报 `Manifest merger failed : Attribute application@name ` 使用

`tools:replace="android:name"` 是直接替换掉。所以主工程需要继承模块的application，并使用

`tools:replace="android:name"`  替换掉模块Manifest里的注册。
