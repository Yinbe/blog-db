用 export default 统一导出

```
export default {
    form:  {
        card: '6216 2610 0000 0073 28',
        bank: '',
        mobile: '132 3757 4799',
        code: '',
    },
    dialogFormVisible: false,
    cardListVisible: false,
    addCardVisible: false,
}
```

相当于 生成一个 default 对象，包含了{} 里的参数方法

导入时

```
然后在引用页面用一个变量接住
import module1 from "./xx"   对象导入
let {form, dialogFormVisible, cardListVisible, addCardVisible} = module1  按需导入
```

用 export 导出

```
export function checkMobile(rule, value, callback) {
        if (!(/^\d{11}$/.test(value))) {
            callback(new Error('手机号码格式不正确，请重新输入'));
        } else {
            callback();
        }
    }

export let a = 1
```

导出引用文件内解构导入

无参数函数直接调用，不用加()

```
const isFF = () => getBrowserType === 'FF'   箭头函数只有一行时会自动return
```
