关于ng取消右键默认行为问题，解决方法。

右键点击图谱出现详情页，但是却会出现浏览器的默认行为，即返回菜单。

![image-20190605143413848](/Users/ziqianwang/Library/Application Support/typora-user-images/image-20190605143413848.png)

![image-20190605143402523](/Users/ziqianwang/Library/Application Support/typora-user-images/image-20190605143402523.png)

但是在元素本身添加阻止默认`e.preventDefault();`并没有效果。

那么右键屏蔽的方法就是使用contextmenu事件。

```
(contextmenu)="nomenu($event)"

nomenu(event){
     // 主要是这个事件了，阻止默认右键菜单的弹出
    event.preventDefault();
}
```

注意要在该元素父级添加而不是本身。