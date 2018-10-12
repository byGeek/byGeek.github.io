---
title: WPF中遇到的一个关于RadioButton的数据绑定问题
date: 2018-10-11 14:00:12
tags:
- wpf
categories:
- coding
keywords:
- radiobutton
- data-binding
- wpf
description:
---



最近在重构代码的时候遇到一个WPF相关的问题，在使用MVVM pattern时，给WPF RadioButton建立绑定数据源时，理所当然的想到使用boolean类型。但是发生了一个奇怪的现象。废话不多说，直接上sample 代码。

<!--more-->

## 问题代码

完整sample代码在[github](https://github.com/byGeek/WPF_RadioButton/tree/master/src/RadioButtonTest2)上。

- TestClass.cs

  ```csharp
      public enum RW
      {
          Read, Write
      }
  
      public class TestClass
      {
  
          public RW ReadOrWrite { get; set; }
      }
  ```

- TestViewModel.cs

  ```csharp
  public class TestViewModel : ViewModelBase
      {
          private readonly TestClass testclass;
          public TestViewModel(TestClass tc)
          {
              testclass = tc;
          }
  
          public bool IsRead
          {
              get { return testclass.ReadOrWrite == RW.Read; }
              set
              {
                  testclass.ReadOrWrite = value ? RW.Read : RW.Write;
                  OnPropertyChanged("IsRead");
              }
          }
  
          public bool IsWrite
          {
              get { return testclass.ReadOrWrite == RW.Write; }
              set
              {
                  testclass.ReadOrWrite = value ? RW.Write : RW.Read;
                  OnPropertyChanged("IsWrite");
              }
          }
      }
  ```

- TestWindow.xaml

  ```xaml
      <StackPanel >
          <RadioButton GroupName="test" Content="read" IsChecked="{Binding IsRead, Mode=TwoWay}"/>
          <RadioButton GroupName="test" Content="write" IsChecked="{Binding IsWrite, Mode=TwoWay}"/>
      </StackPanel>
  ```

代码很简单，有两个window: MainWindowViewModel, TestWindow。TestWindow有两个RadioButton，分别建立双向绑定TestViewModel中的两个属性。在MainWindow中传递TestViewModel给TestWindow，作为其DataContext。同时在MainWindow有一个button，点击打开TestWindow。当点击button，然后关闭TestWindow，重复几遍，发现TestWindow中的RadioButton 来回**自动**切换状态！

第一次点击：

* [x] read
* [ ] write

第二次点击：

* [ ] read
* [x] write

第三次点击：

* [ ] read
* [ ] write

第四次点击:

* [x] read
* [ ] write

如此反复...



## 问题分析

为什么在“数据源不变”的情况下，RadioButton的状态会变化呢？这是一个很简单的数据绑定而已! 在TestViewModel中属性的getter和setting中打上断点发现：程序会"自动"调用两个属性的setter。在代码中没有直接给属性赋值，所以setter的调用肯定是WPF搞的鬼了。

会不会是因为两个RadioButton属于同一个Group，所以在选择一个的时候，WPF会自动将Group内的置为Uncheck状态？在xaml中去掉GroupName之后，果然没有出现这个问题了。

查看[Reference code](https://referencesource.microsoft.com/#PresentationFramework/src/Framework/System/Windows/Controls/RadioButton.cs,0582ad78f5047101):

```csharp
//file RadioButton.cs
[ThreadStatic] private static Hashtable _groupNameToElements;

protected override void OnChecked(RoutedEventArgs e)
        {
            // If RadioButton is checked we should uncheck the others in the same group
            UpdateRadioButtonGroup();
            base.OnChecked(e);
        }

private void UpdateRadioButtonGroup()
        {
            string groupName = GroupName;
            if (!string.IsNullOrEmpty(groupName))
            {
                Visual rootScope = KeyboardNavigation.GetVisualRoot(this);
                if (_groupNameToElements == null)
                    _groupNameToElements = new Hashtable(1);
                lock (_groupNameToElements)
                {
                    // Get all elements bound to this key and remove this element
                    ArrayList elements = (ArrayList)_groupNameToElements[groupName];
                    for (int i = 0; i < elements.Count; )
                    {
                        WeakReference weakReference = (WeakReference)elements[i];
                        RadioButton rb = weakReference.Target as RadioButton;
                        if (rb == null)
                        {
                            // Remove dead instances
                            elements.RemoveAt(i);
                        }
                        else
                        {
                            // Uncheck all checked RadioButtons different from the current one
                            if (rb != this && (rb.IsChecked == true) && rootScope == KeyboardNavigation.GetVisualRoot(rb))
                                rb.UncheckRadioButton();
                            i++;
                        }
                    }
                }
            }
            else // Logical parent should be the group
            {
                DependencyObject parent = this.Parent;
                if (parent != null)
                {
                    // Traverse logical children
                    IEnumerable children = LogicalTreeHelper.GetChildren(parent);
                    IEnumerator itor = children.GetEnumerator();
                    while (itor.MoveNext())
                    {
                        RadioButton rb = itor.Current as RadioButton;
                        if (rb != null && rb != this && string.IsNullOrEmpty(rb.GroupName) && (rb.IsChecked == true))
                            rb.UncheckRadioButton();
                    }
                }

            }
        }
```

看代码可知，RadioButton内部有一个static变量`_groupNameToElements`, 用来存储GroupName与RadioButton示例的对应关系。在RadioButton的状态改变时，会调用`UpdateRadioButtonGroup`函数。

```csharp
if (rb != null && rb != this && string.IsNullOrEmpty(rb.GroupName) && (rb.IsChecked == true))
                            rb.UncheckRadioButton();
```

料想应该是这段代码设置RadioButton的状态，然后通过binding调用了属性的setter。在TestWindow ShowDialog前后打断点，查看变量`_groupNameToElements`. 

第一次点击：

{% asset_img 1.1.png%}

{% asset_img 1.2.png %}

第二次点击：

{% asset_img 2.1.png%}

{% asset_img 2.2.png %}

第三次点击：

{% asset_img 3.1.png%}

{% asset_img 3.2.png %}

可以看到由于创建的RadioButton并没有马上被垃圾回收，还是残留在同一个Group中。所以在`UpdateRadioButtonGroup`函数中迭代`elements`时，执行

```csharp
rb.UncheckRadioButton();
```

同时由于在这几个RadioButton中TestViewModel是共享的(即DataContext为同一个对象)，所以通过setter会改变TestViewModel中的属性。而因为是双向绑定，又反过来作用与RadioButton的IsChecked状态。所以出现了上述的问题。



## 问题解决

至此应该很明了了，在RadioButton中使用绑定时，应该留个心眼，如果给每个RadioButton分别绑定一个属性的时候，需要注意这种情况。这种情况在xaml中直接给RadioButton去掉GroupName这个属性即可。我觉得更好的方法应该是使用其他的数据绑定方式。如stackoverflow中的一个问题，使用ListBox来模拟RadioButton.

[[Simple WPF RadioButton Binding?](https://stackoverflow.com/questions/1317891/simple-wpf-radiobutton-binding)]



## 延伸阅读

- [[How is XAML interpreted and executed at runtime?](https://stackoverflow.com/questions/20528909/how-is-xaml-interpreted-and-executed-at-runtime)]