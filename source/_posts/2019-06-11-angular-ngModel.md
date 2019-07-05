---
title: angular-ngModel
date: 2019-06-11 14:23:38
tags:
  - angular
categories:
  - angular
---
> 引用 https://segmentfault.com/a/1190000011733166


### ngModel的实现

#### ControlValueAccessor
> controlValueAccessor时一个连接表达模型和视图的接口，自定义的表达空间必须实现这个接口，他的作用是  
- 把form模型的值映射到视图中
- 当视图发生变化时，通知form directives或form controls

Angular 引入这个接口的原因是，不同的输入控件数据更新方式是不一样的。例如，对于我们常用的文本输入框来说，我们是设置它的 value 值，而对于复选框 (checkbox) 我们是设置它的 checked 属性。实际上，不同类型的输入控件都有一个 ControlValueAccessor，用来更新视图  
<!-- more -->
`angular`中常见的`ControlValueAccesor`有：
- `DefaultValueAccessor` 用于text和textarea类型的输入控件
- `SelectControlValueAccessor` 用于 select 选择控件
- `CheckboxControlValueAccessor` 用于 checkbox 复选控件

** 实现ControlValueAccessor接口 **
```
export interface ControlValueAccessor {
    writeValue(obj:any):void;
    registerOnchange(fn:any):void;
    registerOnTouched(fn:any):void;
    setDisabledState?(isDisabled:boolean):void;
}
```
- `writeValue(obj:any)`用于将模型中的新值写入视图或DOM属性中，即`model->view`
- `registerOnchange(fn:any)` 设置当控件接收到change事件后调用的函数，可以用来通知外部，组件已经发生变化，即`view-model`
- `registerOnTouched(fn:any)` 设置当控件接收到 touched 事件后，调用的函数
- `setDisabledState?(isDisabled:boolean)` 当控件状态变成disabled或从disabled状态变化成enable状态时会调用该函数。该函数会根据参数值，启用或禁用指定的dom元素  

#### 注册成为表单控件
- `NG_VALUE_ACCESSOR`： token类型为`controlValueAccessor`，将控件本身注册到框架成为一个可以让表单访问其值的控件
- `NG_VALIDATORS`: 将控件注册成为一个可以让表单得到其验证状态的控件，`NG_VALIDATORS`的token类型为function或validator，配合useExisting,可以让控件只暴露对应的function或Validator的validate方法。针对token为Validator类型来说，控件实现了validate方法就可以实现表单控件验证  
- `forwardRef`: 向前引用，允许我们引用一个尚未定义的对象  
- `multi`: 设为true，表示这个token对应多个依赖项，使用相同的token去获取依赖项的时候，获取的是已注册的依赖对象列表。如果不设置multi为true，那么对于相同token的提供商来说，后定义的提供商会覆盖前面已定义的提供商  

步骤一： 创建Token为NG_VALUE_ACCESSOR的提供商
```
@Component({
    selector: 'exe-counter',
    ...
    providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => CounterComponent ),
      multi: true
    }
  ]
})

```
步骤二：创建Token为NG_VALIDATORS的表单控件验证器
```
@Component({
    selector: 'exe-counter',
    ...
    providers: [
    {
      provide: NG_VALUE_ACCESSOR,
      useExisting: forwardRef(() => CounterComponent ),
      multi: true
    },
    {
      provide: NG_VALIDATORS,
      useValue: validateCounterRange,
      multi: true
    }
  ]
```
CounterComponent 组件的完整代码如下：
```
import { Component, Input, forwardRef } from '@angular/core';
import {
    ControlValueAccessor, NG_VALUE_ACCESSOR, NG_VALIDATORS,
    AbstractControl, ValidatorFn, ValidationErrors, FormControl
} from '@angular/forms';

export const EXE_COUNTER_VALUE_ACCESSOR: any = {
    provide: NG_VALUE_ACCESSOR,
    useExisting: forwardRef(() => CounterComponent),
    multi: true
};

export const validateCounterRange: ValidatorFn = (control: AbstractControl): 
  ValidationErrors => {
    return (control.value > 10 || control.value < 0) ?
        { 'rangeError': { current: control.value, max: 10, min: 0 } } : null;
};

export const EXE_COUNTER_VALIDATOR = {
    provide: NG_VALIDATORS,
    useValue: validateCounterRange,
    multi: true
};

@Component({
    selector: 'exe-counter',
    template: `
    <div>
      <p>当前值: {{ count }}</p>
      <button (click)="increment()"> + </button>
      <button (click)="decrement()"> - </button>
    </div>
    `,
    providers: [EXE_COUNTER_VALUE_ACCESSOR, EXE_COUNTER_VALIDATOR]
})
export class CounterComponent implements ControlValueAccessor {
    @Input() _count: number = 0;

    get count() {
        return this._count;
    }

    set count(value: number) {
        this._count = value;
        this.propagateChange(this._count);
    }

    propagateChange = (_: any) => { };

    writeValue(value: any) {
        if (value) {
            this.count = value;
        }
    }

    registerOnChange(fn: any) {
        this.propagateChange = fn;
    }

    registerOnTouched(fn: any) { }

    increment() {
        this.count++;
    }

    decrement() {
        this.count--;
    }
}
```


