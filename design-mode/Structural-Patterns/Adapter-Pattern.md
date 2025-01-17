### 一、适配器模式简介

##### 1.定义

适配器模式把一个类的接口变换成客户端所期待的另一种接口，从而使原本因接口不匹配而无法在一起工作的两个类能够在一起工作。

##### 2.定义阐述

适配器提供客户类需要的接口，适配器的实现就是把客户类的请求转化为对适配者的相应接口的调用。也就是说：当客户类调用适配器的方法时，在适配器类的内部将调用适配者类的方法，而这个过程对客户类是透明的，客户类并不直接访问适配者类。因此，适配器可以使由于接口不兼容而不能交互的类可以一起工作。这就是适配器模式的模式动机。

例如：　用电器做例子，笔记本电脑的插头一般都是三相的，即除了阳极、阴极外，还有一个地极。而有些地方的电源插座却只有两极，没有地极。电源插座与笔记本电脑的电源插头不匹配使得笔记本电脑无法使用。这时候一个三相到两相的转换器（适配器）就能解决此问题，而这正像是本模式所做的事情。

### 二、适配器模式结构

适配器模式有**类的适配器模式**和**对象的适配器模式**两种不同的形式。

##### 类适配器模式

类的适配器模式把适配的类的API转换成为目标类的API。

![](https://upload-images.jianshu.io/upload_images/3985563-77548c0670a6bd50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  



在上图中可以看出，Adaptee类并没有sampleOperation2\(\)方法，而客户端则期待这个方法。为使客户端能够使用Adaptee类，提供一个中间环节，即类Adapter，把Adaptee的API与Target类的API衔接起来。Adapter与Adaptee是继承关系，这决定了这个适配器模式是类的：

模式所涉及的角色有：

●　　目标\(Target\)角色：这就是所期待得到的接口。注意：由于这里讨论的是类适配器模式，因此目标不可以是类。

●　　源\(Adapee\)角色：现在需要适配的接口。

●　　适配器\(Adaper\)角色：适配器类是本模式的核心。适配器把源接口转换成目标接口。显然，这一角色不可以是接口，而必须是具体类。

```java
public interface Target {
    /**
     * 这是源类Adaptee也有的方法
     */
    public void sampleOperation1(); 
    /**
     * 这是源类Adapteee没有的方法
     */
    public void sampleOperation2(); 
}
```

上面给出的是目标角色的源代码，这个角色是以一个JAVA接口的形式实现的。可以看出，这个接口声明了两个方法：sampleOperation1\(\)和sampleOperation2\(\)。而源角色Adaptee是一个具体类，它有一个sampleOperation1\(\)方法，但是没有sampleOperation2\(\)方法。

```java
public class Adaptee {

    public void sampleOperation1(){}

}
```

适配器角色Adapter扩展了Adaptee,同时又实现了目标\(Target\)接口。由于Adaptee没有提供sampleOperation2\(\)方法，而目标接口又要求这个方法，因此适配器角色Adapter实现了这个方法。

```java
public class Adapter extends Adaptee implements Target {
    /**
     * 由于源类Adaptee没有方法sampleOperation2()
     * 因此适配器补充上这个方法
     */
    @Override
    public void sampleOperation2() {
        //写相关的代码
    }

}
```

##### 对象适配器模式

与类的适配器模式一样，对象的适配器模式把被适配的类的API转换成为目标类的API，与类的适配器模式不同的是，对象的适配器模式不是使用继承关系连接到Adaptee类，而是使用委派关系连接到Adaptee类。

![](https://upload-images.jianshu.io/upload_images/3985563-202c3c1af705b68e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  



从上图可以看出，Adaptee类并没有sampleOperation2\(\)方法，而客户端则期待这个方法。为使客户端能够使用Adaptee类，需要提供一个包装\(Wrapper\)类Adapter。这个包装类包装了一个Adaptee的实例，从而此包装类能够把Adaptee的API与Target类的API衔接起来。Adapter与Adaptee是委派关系，这决定了适配器模式是对象的。

```java
public interface Target {
    /**
     * 这是源类Adaptee也有的方法
     */
    public void sampleOperation1(); 
    /**
     * 这是源类Adapteee没有的方法
     */
    public void sampleOperation2(); 
}
```

```java
public class Adaptee {

    public void sampleOperation1(){}

}
```

```java
public class Adapter {
    private Adaptee adaptee;

    public Adapter(Adaptee adaptee){
        this.adaptee = adaptee;
    }
    /**
     * 源类Adaptee有方法sampleOperation1
     * 因此适配器类直接委派即可
     */
    public void sampleOperation1(){
        this.adaptee.sampleOperation1();
    }
    /**
     * 源类Adaptee没有方法sampleOperation2
     * 因此由适配器类需要补充此方法
     */
    public void sampleOperation2(){
        //写相关的代码
    }
}
```

##### 时序图

![](https://upload-images.jianshu.io/upload_images/3985563-7d3c2c89d8d28c6a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  


### 三、类适配器和对象适配器的权衡

●　　类适配器使用对象继承的方式，是静态的定义方式；而对象适配器使用对象组合的方式，是动态组合的方式。

●　　对于类适配器，由于适配器直接继承了Adaptee，使得适配器不能和Adaptee的子类一起工作，因为继承是静态的关系，当适配器继承了Adaptee后，就不可能再去处理 Adaptee的子类了。

●　　对于对象适配器，一个适配器可以把多种不同的源适配到同一个目标。换言之，同一个适配器可以把源类和它的子类都适配到目标接口。因为对象适配器采用的是对象组合的关系，只要对象类型正确，是不是子类都无所谓。

●　 对于类适配器，适配器可以重定义Adaptee的部分行为，相当于子类覆盖父类的部分实现方法。

●　 对于对象适配器，要重定义Adaptee的行为比较困难，这种情况下，需要定义Adaptee的子类来实现重定义，然后让适配器组合子类。虽然重定义Adaptee的行为比较困难，但是想要增加一些新的行为则方便的很，而且新增加的行为可同时适用于所有的源。

●　　对于类适配器，仅仅引入了一个对象，并不需要额外的引用来间接得到Adaptee。

●　　对于对象适配器，需要额外的引用来间接得到Adaptee。

**建议尽量使用对象适配器的实现方式，多用合成/聚合、少用继承。当然，具体问题具体分析，根据需要来选用实现方式，最适合的才是最好的。**

### 四、缺省适配器

缺省适配\(Default Adapter\)模式为一个接口提供缺省实现，这样子类型可以从这个缺省实现进行扩展，而不必从原有接口进行扩展。

当不需要全部实现接口提供的方法时，可先设计一个抽象类实现接口，并为该接口中每个方法提供一个默认实现（空方法），那么该抽象类的子类可有选择地覆盖父类的某些方法来实现需求，它适用于一个接口不想使用其所有的方法的情况。

### 五、Java中适配器模式的使用

JDK1.1 之前提供的容器有 Arrays,Vector,Stack,Hashtable,Properties,BitSet，其中定义了一种访问群集内各元素的标准方式，称为 Enumeration（列举器）接口。

```java
Vector v=new Vector();
for (Enumeration enum =v.elements(); enum.hasMoreElements();) {
  Object o = enum.nextElement();
  processObject(o);
}
```

JDK1.2 版本中引入了 Iterator 接口，新版本的集合对（HashSet,HashMap,WeakHeahMap,ArrayList,TreeSet,TreeMap, LinkedList）是通过 Iterator 接口访问集合元素。

```java
List list=new ArrayList();
for(Iterator it=list.iterator();it.hasNext();){
   System.out.println(it.next());
}
```

这样，如果将老版本的程序运行在新的 Java 编译器上就会出错。因为 List 接口中已经没有 elements\(\)，而只有 iterator\(\) 了。那么如何将老版本的程序运行在新的 Java 编译器上呢? 如果不加修改，是肯定不行的，但是修改要遵循“开－闭”原则。我们可以用 Java 设计模式中的适配器模式解决这个问题。

```java
public class NewEnumeration implements Enumeration {
    Iterator it;

    public NewEnumeration(Iterator it) {
        this.it = it;
    }

    public boolean hasMoreElements() {
        return it.hasNext();
    }

    public Object nextElement() {
        return it.next();
    }

    public static void main(String[] args) {
        List list = new ArrayList();
        list.add("a");
        list.add("b");
        list.add("C");
        for (Enumeration e = new NewEnumeration(list.iterator()); e.hasMoreElements(); ) {
            System.out.println(e.nextElement());
        }
    }
}
```

NewEnumeration 是一个适配器类，通过它实现了从 Iterator 接口到 Enumeration 接口的适配，这样我们就可以使用老版本的代码来使用新的集合对象了。

### 六、Android中适配器模式的使用

在开发过程中,ListView的Adapter是我们最为常见的类型之一。一般的用法大致如下:

```java
 // 代码省略
 ListView myListView = (ListView) findViewById(listview_id);
// 设置适配器
 myListView.setAdapter(new MyAdapter(context,myDatas));

// 适配器
public class MyAdapter extends BaseAdapter {

    private LayoutInflater mInflater;
    List<String> mDatas;

    public MyAdapter(Context context, List<String> datas) {
        this.mInflater = LayoutInflater.from(context);
        mDatas = datas;
    }

    @Override
    public int getCount() {
        return mDatas.size();
    }

    @Override
    public String getItem(int pos) {
        return mDatas.get(pos);
    }

    @Override
    public long getItemId(int pos) {
        return pos;
    }

    // 解析、设置、缓存convertView以及相关内容
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder = null;
        // Item View的复用
        if (convertView == null) {
            holder = new ViewHolder();
            convertView = mInflater.inflate(R.layout.my_listview_item, null);
            // 获取title
            holder.title = (TextView) convertView.findViewById(R.id.title);
            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        holder.title.setText(mDatas.get(position));
        return convertView;
    }

}
```

我们知道，作为最重要的View，ListView需要能够显示各式各样的视图，每个人需要的显示效果各不相同，显示的数据类型、数量等也千变万化。那么如何隔离这种变化尤为重要。

Android的做法是增加一个Adapter层来应对变化，**将ListView需要的接口抽象到Adapter对象中**，**这样只要用户实现了Adapter的接口，ListView就可以按照用户设定的显示效果、数量、数据来显示特定的Item View。**

通过代理数据集来告知ListView数据的个数\( getCount函数 \)以及每个数据的类型\( getItem函数 \)，最重要的是要解决Item View的输出。Item View千变万化，但终究它都是View类型，Adapter统一将Item View输出为View \( getView函数 \)，这样就很好的应对了Item View的可变性。

那么ListView是如何通过Adapter模式 \( 不止Adapter模式 \)来运作的呢 ？我们一起来看一看。  
ListView继承自AbsListView，Adapter定义在AbsListView中，我们看一看这个类。

```java
public abstract class AbsListView extends AdapterView<ListAdapter>
        implements TextWatcher,
        ViewTreeObserver.OnGlobalLayoutListener, Filter.FilterListener,
        ViewTreeObserver.OnTouchModeChangeListener,
        RemoteViewsAdapter.RemoteAdapterConnectionCallback {

    ListAdapter mAdapter;

    // 关联到Window时调用的函数
    @Override
    protected void onAttachedToWindow() {
        super.onAttachedToWindow();
        // 代码省略
        // 给适配器注册一个观察者。
        if (mAdapter != null&&
        mDataSetObserver == null){
            mDataSetObserver = new AdapterDataSetObserver();
            mAdapter.registerDataSetObserver(mDataSetObserver);

            // Data may have changed while we were detached. Refresh.
            mDataChanged = true;
            mOldItemCount = mItemCount
            // 获取Item的数量,调用的是mAdapter的getCount方法
            mItemCount = mAdapter.getCount();
        }
        mIsAttached = true;
    }

    /**
     * 子类需要覆写layoutChildren()函数来布局child view,也就是Item View
     */
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);
        mInLayout = true;
        if (changed) {
            int childCount = getChildCount();
            for (int i = 0; i<childCount; i++) {
                getChildAt(i).forceLayout();
            }
            mRecycler.markChildrenDirty();
        }

        if (mFastScroller != null&&
        mItemCount != mOldItemCount){
            mFastScroller.onItemCountChanged(mOldItemCount, mItemCount);
        }
        // 布局Child View
        layoutChildren();
        mInLayout = false;

        mOverscrollMax = (b - t) / OVERSCROLL_LIMIT_DIVISOR;
    }

    // 获取一个Item View
    View obtainView(int position, boolean[] isScrap) {
        isScrap[0] = false;
        View scrapView;
        // 从缓存的Item View中获取,ListView的复用机制就在这里
        scrapView = mRecycler.getScrapView(position);

        View child;
        if (scrapView != null) {
            // 代码省略
            child = mAdapter.getView(position, scrapView, this);
            // 代码省略
        } else {
            child = mAdapter.getView(position, null, this);
            // 代码省略
        }

        return child;
    }
}
```

通过增加Adapter一层来将Item View的操作抽象起来，ListView等集合视图通过Adapter对象获得Item的个数、数据元素、Item View等，从而达到适配各种数据、各种Item视图的效果。

因为Item View和数据类型千变万化，Android的架构师们将这些变化的部分交给用户来处理，通过getCount、getItem、getView等几个方法抽象出来，也就是将Item View的构造过程交给用户来处理，灵活地运用了适配器模式，达到了无限适配、拥抱变化的目的。

### 七、适配器模式的优缺点

##### 适配器模式的优点

**更好的复用性**

系统需要使用现有的类，而此类的接口不符合系统的需要。那么通过适配器模式就可以让这些功能得到更好的复用。

**更好的扩展性**

在实现适配器功能的时候，可以调用自己开发的功能，从而自然地扩展系统的功能。

##### 适配器模式的缺点

过多的使用适配器，会让系统非常零乱，不易整体进行把握。比如，明明看到调用的是A接口，其实内部被适配成了B接口的实现，一个系统如果太多出现这种情况，无异于一场灾难。因此如果不是很有必要，可以不使用适配器，而是直接对系统进行重构。

