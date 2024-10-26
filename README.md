
`Streamlit`不仅让创建单页应用变得易如反掌，更通过一系列创新特性，支持构建**多页面应用**，极大地丰富了用户体验和数据探索的可能性。


随着我们`Streamlit App`的功能逐渐增多之后，单个页面展示过多信息，使用不便，


通过多页面可以将功能相关的部分组织在一起，形成逻辑清晰的多个页面，使用户能够轻松地与不同的功能模块进行交互。


从代码方面来看，多页面应用将不同的功能模块拆分成独立的页面，每个页面可以有自己的代码逻辑和数据流。


这有助于实现代码的模块化，使代码结构更加清晰、易于管理。


从运行性能上来看，多页面应用可以加快页面的加载速度，因为用户只需加载当前所需页面的内容，而无需加载整个应用的全部内容。


此外，对于复杂的应用来说，多页面应用更容易实现功能的迭代和扩展。


随着应用的发展，可以逐步添加新的页面和功能模块，而无需对现有页面进行大规模修改。


本篇主要介绍构建一个`Streamlit`的多页面应用需要掌握的基本知识。


# 1\. 多页应用的文件结构


在`Streamlit`多页面应用中，文件和文件夹的布局对于项目的组织、管理和维护至关重要。


下面是一个推荐的布局方式：



```
my_app/  
├── app.py  # 主应用文件，负责启动应用和配置路由
├── pages/  
│   ├── __init__.py  # 可选，用于将pages文件夹作为Python包处理  
│   ├── page1.py  
│   ├── page2.py  
│   └── ...  # 其他页面文件  
├── session_state.py  # Session State管理类文件
└── common.py  # 共通函数

```

扩展功能时，在`pages`文件夹下添加新的**py文件**即可。


其中`session_state.py`和`common.py`不是必需的，当应用的`session`管理变得复杂，或者共通函数比较多时才需要单独用文件管理。


对于简单的多页面应用，一般只需要上面的`app.py`，`page1.py`和`page2.py`就够了。


# 2\. 多页应用的导航


在`Streamlit`中，使用`st.navigation`，可以帮助我们轻松地创建动态导航菜单。


比如，以`app.py`，`page1.py`和`page2.py`为例，创建一个多页面应用。



```
# app.py

import streamlit as st

page1 = st.Page("pages/page1.py", title="页面1")
page2 = st.Page("pages/page2.py", title="页面2")

pg = st.navigation([page1, page2])
pg.run()

```


```
# page1.py

import streamlit as st

st.header("这是页面 1")

```


```
# page2.py

import streamlit as st

st.header("这是页面 2")

```

通过`streamlit run app.py` 启动之后，一个带有导航的简单多页面应用就完成了。


![](https://img2024.cnblogs.com/blog/83005/202410/83005-20241025120218662-735147877.gif)


通过侧边栏中的菜单，可以自由切换页面。


除了通过`app.py`生成的菜单来切换页面，`Streamlit`中还提供了`st.switch_page`方法，


可以在一个页面中导航到其他页面。


比如，可以在`page1.py`和`page2.py`中添加一个互相导航的按钮。



```
# page1.py

import streamlit as st

st.header("这是页面 1")

if st.button("GoTo Page 2"):
    st.switch_page("pages/page2.py")

```


```
# page2.py

import streamlit as st

st.header("这是页面 2")

if st.button("GoTo Page 1"):
    st.switch_page("pages/page1.py")

```

![](https://img2024.cnblogs.com/blog/83005/202410/83005-20241025120218750-1240632933.gif)


# 3\. 多页之间共享数据


最后，介绍下如何在不同的页面直接共享数据，这样就可以让不同页面的功能联动起来。


Streamlit多页面之间共享数据有几个方案可以实现，


第一个方案是使用**全局变量**，


但是这种方法存在一些问题，比如如并发访问时的数据不一致性和难以调试等。


因此，一般**不推荐**使用全局变量来共享数据。


第二个方案是**使用外部存储**，比如将共享的数据保存在文件或者数据库中，这种方案适用于需要比较大型的应用，或者需要持久化存储的应用场景。


如果你的应用规模不大，并且不需要持久化存储，那么用这个方案显得有些笨重。


最后一个方案就是`Session State`，这是`Streamlit`提供的一种机制，特别适合在不同页面之间传递和保存状态数据。


下面构造一个模拟的示例，演示如何在不同的页面间共享数据。


首先在`page1.py`中，我们可以选择数据集，


然后在`page2.py`中，会自动根据我们选择的数据集开始分析。



```
# page1.py

import streamlit as st

st.header("这是页面 1")

if st.button("GoTo Page 2"):
    st.switch_page("pages/page2.py")

datalist = ("", "人口数据", "环境数据", "交易数据")

if "dataset" not in st.session_state:
    option = st.selectbox(
        "请选择数据集",
        datalist,
    )
else:
    option = st.session_state.dataset
    option = st.selectbox(
        "请选择数据集",
        datalist,
        index=datalist.index(option),
    )


if option == "":
    st.write("当前尚未选择数据集")
else:
    st.write("你当前选择的是: 【", option, "】")

st.session_state.dataset = option

```

`page1.py`中将选择数据集名称保存到`Session State`中。



```
# page2.py

import streamlit as st

st.header("这是页面 2")

if st.button("GoTo Page 1"):
    st.switch_page("pages/page1.py")

if "dataset" not in st.session_state or st.session_state.dataset == "":
    st.write("当前尚未选择数据集")
else:
    st.write("开始分析数据集: 【", st.session_state.dataset, "】")

```

`page2.py`直接从`Session State`中读取数据集的名称。


![](https://img2024.cnblogs.com/blog/83005/202410/83005-20241025120218430-686422022.gif)


 本博客参考[豆荚加速器](https://baitenghuo.com) 。转载请注明出处！
