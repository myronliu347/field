# Field

<p align="center">
  <a href="https://www.npmjs.org/package/@alifd/field"><img src="https://img.shields.io/npm/v/@alifd/field.svg"></a>
  <a href="https://www.npmjs.org/package/@alifd/field"><img src="https://img.shields.io/npm/dm/@alifd/field.svg"></a>
  <a href="https://codecov.io/gh/alibaba-fusion/field"><img src="https://codecov.io/gh/alibaba-fusion/field/branch/master/graph/badge.svg"></a>
  <a href="https://travis-ci.com/alibaba-fusion/field"><img src="https://travis-ci.com/alibaba-fusion/field.svg?branch=master"></a>
  <a href="http://makeapullrequest.com"><img src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg"></a>
  <a href="https://github.com/alibaba-fusion/field/blob/master/LICENSE"><img src="https://img.shields.io/badge/license-MIT-brightgreen.svg"></a>
  <a href="https://github.com/semantic-release/semantic-release"><img src="https://img.shields.io/badge/%20%20%F0%9F%93%A6%F0%9F%9A%80-semantic--release-e10079.svg"></a>
</p>

- category: Components
- family: DataEntry
- chinese: 表单辅助工具
- cols: 1
- type: 表单

---

## 开发指南

### 何时使用

涉及到表单数据操作、校验的地方都可以用 Field 来管理数据。和组件关联后可以自动对表单数据进行回写、读取、校验。

### 使用注意

- 使用 Field `init` 过的组件, `value` `onChange` 必须放在 init 的第三个参数, 否则有可能被 init 覆盖。
- `Form`已经和`Field` 在`数据获取`和`自动校验提示`方面做了深度优化，建议在`Form`中使用`Field`, 请查看 Form demo。
- initValue 类似组件的 defaultValue 只有在组件第一次 render 的时候才生效(ajax 异步调用设置 initValue 可能已经错过了第一次 render)
- autoUnmount 默认打开的，如果需要保留会 `自动卸载的组件` 数据请关闭此项
- `parseName=true` 可以通过 `getValues` 获取到结构化的数据, 但是 getValue 还是必须传完整 key 值

### 基本使用

```
class Demo extends React.Component {
    field = new Field(this);    // 实例创建

    onClick = ()=>{
        console.log(this.field.getValue('name'));
    }
    render() {
        const init = this.field.init;

        // 注意：initValue只会在组件第一次初始化的时候被赋值，如果你是异步赋值请用setValue
        return <div>
            <Input {...init('name',{initValue:'first value'})} />
            <button onClick={this.onClick>获取数据</button>
        </div>
    }
}
```

### 更新数据

#### 事件更新

```
class Demo extends React.Component {
    field = new Field(this);

    onClick = ()=>{
        this.field.setValue('name', 'newvalue');    // 赋值会自动触发render
    }
    render() {
        const init = this.field.init;

        return <div>
            <Input {...init('name')} />
            <button onClick={this.onClick}>设置数据</button>
        </div>
    }
}
```

#### props 更新

```
class Demo extends React.Component {
    field = new Field(this);

    // 在组件挂载之前把数据设置进去(可以用initValue替代这种用法)
    componentWillMount() {
        this.field.setValue('name', 'init Name')
    }
    // 接收来自props的数据
    componentWillReceiveProps(nextProps) {
        this.field.setValue('name', nextProps.name)
    }
    render() {
        const init = this.field.init;

        return <div>
            <Input {...init('name')} />
        </div>
    }
}
```

#### ajax 更新

```
class Demo extends React.Component {
    field = new Field(this);

    onClick = ()=>{
        Ajax({
            url:'/demo.json',
            success:(json)=>{
                // 回调事件中赋值更新
                this.field.setValue('name',json.name);
            }
        });
    }
    render() {
        const init = this.field.init;

        return <div>
            <Input {...init('name')} />
            <button onClick={this.onClick}>设置数据</button>
        </div>
    }
}
```

#### onChange 更新监控

两种用法：

1. 统一管理

```
class Demo extends React.Component {
    field = new Field(this,{
        onChange:(name, value) => {
          switch(name) {
            case 'name1':
              this.field.setValue('name2','value set by name1');
              break;
            case 'name2':
              this.field.setValue('name1','value set by name2');
              break;
          }
        }
    });
    render() {
        const init = this.field.init;

        return <div>
          <Input {...init('name1')} />
          <Input {...init('name2')} />
        </div>
    }
}
```

2. 各自管理

```
class Demo extends React.Component {
    render() {
        const init = this.field.init;

        return <div>
          <Input {...init('name1',{
              props:{
                onChange:(v)=>{
                  this.field.setValue('name2','value set by name1');
                }
              }
            })} />
          <Input {...init('name2',{
              props:{
                onChange:(v)=>{
                  this.field.setValue('name1','value set by name2');
                }
              }
            })} />
        </div>
    }
}
```

详细请查看 demo 演示 `关联控制`

## API

### 初始化

```
let myfield = new Field(this [,options]);
```

| 参数    | 说明                       | 类型            | 可选值   | 默认值 |
| ------- | -------------------------- | --------------- | -------- | ------ |
| this    | 传入调用 class 的 this     | React.Component | 必须设置 |        |
| options | 一些事件配置, 详细参数如下 | Object          | 非必须   |        |

`options` 配置项

| 参数               | 说明                                                                                                                                    | 类型                 | 默认值 |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------- | -------------------- | ------ |
| onChange           | 所有组件的 change 都会到达这里[setValue 不会触发该函数]                                                                                 | Function(name,value) |        |
| parseName          | 是否翻译`init(name)`中的`name`(getValues 会把带`.`的字符串转换成对象)                                                                   | Boolean              | false  |
| forceUpdate        | 仅建议 PureComponent 的组件打开此强制刷新功能，会带来性能问题(500 个组件为例：打开的时候 render 花费 700ms, 关闭时候 render 花费 400ms) | Boolean              | false  |
| scrollToFirstError | field.validate 的时候滚动到第一个出错的组件, 如果是整数会进行偏移                                                                       | Boolean/Number       | true   |
| autoUnmount        | 自动删除 Unmout 元素，如果想保留数据可以设置为 false                                                                                    | Boolean              | true   |
| autoValidate       | 是否修改数据的时候就自动触发校验, 设为 false 后只能通过 validate() 来触发校验                                                           | Boolean              | true   |
| values             | 初始化数据                                                                                                                              | Object               |        |
| onValuesChange     | 当 values 发生改变时回调(setValue/setValues 也会触发此回调)                                                                            | Function(values)     |        |

#### API 接口

`new`之后的对象提供的 api 接口 （例：`myfield.getValues()`）(`set` 开头的 api 函数不要在 render 里面操作, 可能会触发死循环)

| 参数             | 说明                                                                     | 类型                                                                               | 可选值                         | 默认值 |
| ---------------- | ------------------------------------------------------------------------ | ---------------------------------------------------------------------------------- | ------------------------------ | ------ |
| init             | 初始化每个组件，[详细参数如下](#init)）                                  | Function(name:String, option:Object)                                               |                                |        |
| getValues        | 获取一组输入控件的值，如不传入参数，则获取全部组件的值                   | Function([names: String[]])                                                        |                                |        |
| getValue         | 获取单个输入控件的值                                                     | Function(name: String)                                                             |                                |        |
| setValues        | 设置一组输入控件的值（会触发 render，请遵循 react 时机使用)              | Function(obj: Object)                                                              |                                |        |
| setValue         | 设置单个输入控件的值 （会触发 render，请遵循 react 时机使用)             | Function(name: String, value)                                                      |                                |        |
| validate         | 校验并获取一组输入域的值与 Error                                         | Function([names: String[]], [options: Object], callback: Function(errors, values)) |                                |        |
| getError         | 获取单个输入控件的 Error                                                 | Function(name: String)                                                             |                                |        |
| getErrors        | 获取一组输入控件的 Error                                                 | Function([name: String])                                                           |                                |        |
| setError         | 设置单个输入控件的 Error                                                 | Function(name: String, errors:String/Array[String])                                |                                |        |
| setErrors        | 设置一组输入控件的 Error                                                 | Function(obj: Object)                                                              |                                |        |
| reset            | 重置一组输入控件的值、清空校验                                           | Function([names: String[]])                                                        |                                |        |
| resetToDefault   | 重置一组输入控件的值为默认值                                             | Function([names: String[]])                                                        |                                |        |
| getState         | 判断校验状态                                                             | Function(name: String)                                                             | 'error' 'success' 'loading' '' | ''     |
| getNames         | 获取所有组件的 key                                                       | Function()                                                                         |                                |        |
| remove           | 删除某一个或者一组控件的数据，删除后与之相关的 validate/value 都会被清空 | Function(name: String/String[])                                                    |                                |        |
| addArrayValue    | 添加 name 是数组格式的数据, 并且自动处理其他 name 的数组错位问题         | Function(key: String, index: Number, value1, value2, ...)                          |                                |        |
| deleteArrayValue | 删除 name 是数组格式的数据, 并且自动处理其他 name 的数组错位问题         | Function(key: String, index: Number, howmany)                                      |                                |        |

#### init

```
init(name, options, props)
```

| 参数                      | 说明                                                                                      | 类型                                                       | 可选值 | 默认值     |
| ------------------------- | ----------------------------------------------------------------------------------------- | ---------------------------------------------------------- | ------ | ---------- |
| id                        | 自定义的表单域 id，如不提供则使用 name 替代                                               | String                                                     |        |            |
| name                      | 必填输入控件唯一标志                                                                      | String                                                     |        |            |
| options.valueName         | 组件值的属性名称，如 Checkbox 的是 `checked`，Input 是 `value`                            | String                                                     |        | 'value'    |
| options.initValue         | 组件初始值(组件第一次 render 的时候才会读取，后面再修改此值无效),类似 defaultValue        | any                                                        |        |            |
| options.trigger           | 触发数据变化的事件名称                                                                    | String                                                     |        | 'onChange' |
| options.rules             | 校验规则                                                                                  | Array/Object                                               |        |            |  |
| options.getValueFormatter | 自定义从组件获取 `value` 的方式，详细用法查看 demo `自定义数据获取`                       | Function(value,...args) 参数顺序和组件的 onChange 完全一致 |        |            |  |
| options.setValueFormatter | 自定义转换 `value` 为组件需要的数据 ，详细用法查看 demo `自定义数据获取`                  | Function(value)                                            |        |            |  |
| props                     | 组件自定义的事件可以写在这里                                                              | Object                                                     |        |            |  |
| autoValidate              | 是否修改数据的时候自动触发校验单个组件的校验, 设为 false 后只能通过 validate() 来触发校验 | Boolean                                                    | true   |

返回值

```
{id,value,onChange}
```

#### rules

```
{
    rules:[{ required: true }]
}
```

多个 rule

```
{
    rules:[{required:true,trigger:'onBlur'},{pattern:/abcd/,message:'abcd不能缺'},{validator:(rule, value, callback)=>{callback('出错了')}}]
}
```

| 参数      | 说明                                                                  | 类型                          | 可选值                  | 使用类型                             |
| --------- | --------------------------------------------------------------------- | ----------------------------- | ----------------------- | ------------------------------------ |
| required  | 不能为空                                                              | Boolean                       | true                    | `undefined/null/“”/[]` 会触发此规则) |
| pattern   | 校验正则表达式                                                        | 正则                          |                         |                                      |
| minLength | 字符串最小长度 / 数组最小个数                                         | Number                        |                         | String/Number/Array                  |
| maxLength | 字符串最大长度 / 数组最大个数                                         | Number                        |                         | String/Number/Array                  |
| length    | 字符串精确长度 / 数组精确个数                                         | Number                        |                         | String/Number/Array                  |
| min       | 最小值                                                                | Number                        |                         | String/Number                        |
| max       | 最大值                                                                | Number                        |                         | String/Number                        |
| format    | 对常用 pattern 的总结                                                 | String                        | url、email、tel、number | String                               |
| validator | 自定义校验,(校验成功的时候不要忘记执行 `callback()`,否则会校验不返回) | Function(rule,value,callback) |                         |                                      |
| trigger   | 触发校验的事件名称                                                    | String/Array                  | onChange/onBlur/...     | onChange                             |
| message   | 出错时候信息                                                          | String                        |                         |                                      |

## 自定义组件接入 Field 标准

- 支持受控模式(value+onChange) `必须`

  - value 控制组件数据展现
  - onChange 组件发生变化时候的回调函数（第一个参数可以给到 value)

- 一次完整操作抛一次 onChange 事件 `建议`
  比如有 Process 表示进展中的状态，建议增加 API `onProcess`；如果有 Start 表示启动状态，建议增加 API `onStart`

- `value={undefined}`的时候清空数据, field 的 reset 函数会给所有组件下发 undefined 数据 `建议`

```
componentWillReceiveProps(nextProps) {
    if ('value' in nextProps ) {
        this.setState({
           value: nextProps.value === undefined? []: nextProps.value   //  设置组件的被清空后的数值
        })
    }
}
```

## 已知问题

- 为何手动调用`this.field.validate`的时候进不了回调函数？ 答: 是不是自定义了 validator 方法,确保`callback`在任何分支下都能被执行到。
