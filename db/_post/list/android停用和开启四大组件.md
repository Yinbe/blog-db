有时组件只用一次，然后就用不到了。比如监听(receiver)第一次开机启动后，获得了系统信息后，以后就不用再启动该服务。这时候就可以考虑把监听(receiver)给关掉。这个时候可以考虑使用系统提供的setComponentEnabledSetting(ComponentName componentName,int newState,int flags)

componentName:组件名
newState:组件新状态：

> 不可用状态：COMPONENT_ENABLED_STATE_DISABLED
> 可用状态：COMPONENT_ENABLED_STATE_ENABLED
> 默认状态：COMPONENT_ENABLED_STATE_DEFAULT

flags:行为标签，值可以是DONT_KILL_APP或者0。 0说明杀死包含该组件的app

实例1：禁止开机启动的Receiver

```
final ComponentName receiver = new ComponentName(context,需要禁止的receiver); 
final PackageManager pm = context.getPackageManager(); 
pm.setComponentEnabledSetting(receiver,PackageManager.COMPONENT_ENABLED_STATE_DISABLED,PackageManager.DONT_KILL_APP); 　}
```

实例2：隐藏和显示应用图标

```
PackageManager packageManager = getPackageManager();
ComponentName componentName = new ComponentName(this,FirstActivity.class);
int res = packageManager.getComponentEnabledSetting(componentName);
if (res == PackageManager.COMPONENT_ENABLED_STATE_DEFAULT|| res == PackageManager.COMPONENT_ENABLED_STATE_ENABLED) {
      // 隐藏应用图标
      packageManager.setComponentEnabledSetting(componentName, PackageManager.COMPONENT_ENABLED_STATE_DISABLED,PackageManager.DONT_KILL_APP);
} else {
      // 显示应用图标
      packageManager.setComponentEnabledSetting(componentName,PackageManager.COMPONENT_ENABLED_STATE_DEFAULT,PackageManager.DONT_KILL_APP);
}
```
