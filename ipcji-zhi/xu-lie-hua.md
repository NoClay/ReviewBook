## Serializable接口 {#serializable接口}

### 怎样实现接口？ {#怎样实现接口}

使用Serializable来实现序列化非常简单，只需要在类的声明中加入一个类似下面的标识即可自动实现默认的序列化过程。代码如下：

```java
package com.example.no_clay.messagertest.Data;


import java.io.Serializable;

/**
 * Created by no_clay on 2017/2/26.
 */

public class User implements Serializable{

    private static final long serialVersionUID = -2534337785069283741L;
    String name;
    String id;

    public User(String name, String id) {
        this.name = name;
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }



    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }
}
```

注意：serialVersionUID的生成可以使用Android Studio的插件搞定，插件安装：file-&gt; setting-&gt; plugins-&gt; search: serialVersion.然后直接在实现该接口的文件中使用alt + insert选择生成UID即可。

### 如何对对象进行序列化和反序列化？ {#如何对对象进行序列化和反序列化}

```java
public class User implements  Parcelable{

    String name;
    String id;

    public User(String name, String id) {
        this.name = name;
        this.id = id;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(name);
        dest.writeString(id);
    }

    protected User(Parcel in) {
        name = in.readString();
        id = in.readString();
    }


    public static final Creator<User> CREATOR = new Creator<User>() {
        @Override
        public User createFromParcel(Parcel in) {
            return new User(in);
        }

        @Override
        public User[] newArray(int size) {
            return new User[size];
        }
    };
}
```

注意：  
1. 序列化由writeToParcel来实现  
2. 反序列化由CREATOR来完成  
3. 内容描述由describeContents来实现，几乎所有的情况下这个方法都应该返回0，仅当当前对象中存在文件描述符的时候，返回1

## 对比 {#对比}

两个接口都可以实现序列化，但是相比来说Serializable是[Java](http://lib.csdn.net/base/javase)里边的实现序列化的接口，序列化和反序列化的过程中都需要大量的IO操作，而Parcelable是Android的序列化方式，因此更适用Android平台，他的缺点就是使用麻烦。我们首选的序列化方式应该是Parcelable，但是当遇到将对象序列化到存储设备中或者将对象序列化后通过网络传输建议是使用Serializable接口。

