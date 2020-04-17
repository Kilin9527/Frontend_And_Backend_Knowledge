<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [@ViewChild/@ViewChildren](#viewchildviewchildren)
- [ElementRef](#elementref)
- [TemplateRef](#templateref)
- [ViewRef](#viewref)
- [ViewContainerRef](#viewcontainerref)

<!-- /code_chunk_output -->

## @ViewChild/@ViewChildren
用于从 Component 和 Directive 中获取匹配的元素，且只能在 Component 和 Directive 中使用。

## ElementRef
通过 ElementRef 可以在应用层直接操作 dom，但会导致应用层和渲染层的强耦合。

主要有如下三种用途：
* 作为组件的属性，可以获取指定html子元素。
```Typescript
export class someComponent {
    @ViewChild('subElementName', {read: false, static: false})
    subElement: ElementRef;
}
```

* 作为组件的依赖注入，可以获取当前组件的html元素。
```Typescript
export class someComponent {
    constructor(private elementRef: ElementRef) {}
}
```

* 作为指令的依赖注入，可以获取宿主的html元素。
```Typescript
export class someDirective {
    constructor(private elementRef: ElementRef) {}
}
```

## TemplateRef
用户可以在 html 中用 template 标签声明模板元素，浏览器可以解析 template 内容，创建 dom 树，但不会渲染它。
template 可以在 dom 中直接操作，但是要用 template.content 对象，例如：
```Typescript
var template = document.getElementById('templateId');
var h1 = template.content.querySelector('h1');
var p = template.content.querySelector('p');
```

templateRef 是对 ng-template 的引用，templateRef.elementRef 是对模板的宿主的引用。
```Typescript
@Component({
  selector: 'example',
  template: `
      <ng-template #templateId>
          <span>I am span in template</span>
      </ng-template>
  `,
})
export class SampleComponent implements AfterViewInit {
  @ViewChild("templateId") tpl: TemplateRef<any>;
  
  ngAfterViewInit() {
    let elementRef = this.tpl.elementRef;
  }
}
```

## ViewRef
ViewRef 表示一个 Angular 视图，Angular 视图分两种：
* 嵌入视图(Embedded View)，由 Template 提供。
```Typescript
@Component({
  selector: 'example',
  template: `
      <ng-template #templateId>
          <span>I am span in template</span>
      </ng-template>
  `,
})
export class SampleComponent implements AfterViewInit {
  @ViewChild("templateId") tpl: TemplateRef<any>;
  
  ngAfterViewInit() {
    let view = this.tpl.createEmbeddedView(null);
  }
}
```

* 宿主视图(Host View)，由 Component 提供。宿主视图是在组件动态实例化时创建的，一个动态组件（dynamic component）可以通过 ComponentFactoryResolver 创建。
```Typescript
export class SampleComponent implements   AfterViewInit {
  constructor(private injector: Injector,
    private r: ComponentFactoryResolver) {
      let factory = this.r.resolveComponentFactory  (someComponent);
      let componentRef = factory.create(injector);
      let view = componentRef.hostView;
  }
}
```

## ViewContainerRef
视图一旦被创建，接下来就需要插入 dom 树，通过 ViewContainer 可以实现这个功能。

ViewContainer 可以挂载一个或多个视图的容器，任何元素都可以作为视图容器，它并不是插入到元素的内部，而是追加到元素的后面。

操作视图：
```Typescript
class ViewContainerRef {
  element: ElementRef
  length: number  
  // 通过组件工厂对象创建视图
  createComponent(componentFactory...):  ComponentRef<C>
  // 通过模板引用对象创建视图
  createEmbeddedView(templateRef...):  EmbeddedViewRef<C>
  ...
  // 清空视图
  clear() : void
  // 插入视图
  insert(viewRef: ViewRef, index?: number) :  ViewRef
  // 获取视图
  get(index: number) : ViewRef
  // 获取视图序号
  indexOf(viewRef: ViewRef) : number
  // 移除视图
  detach(index?: number) : ViewRef
  // 移动视图
  move(viewRef: ViewRef, currentIndex: number) :  ViewRef
}
```

