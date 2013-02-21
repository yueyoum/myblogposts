Tags: python, django

Django Admin中的Models是根据 verbose_name_plural 来排序的，
这往往与我们想要的排序方式不一致。

以前就注意到了这个问题，在今天终于忍不住去看了下源码，
做了个[pull request](https://github.com/django/django/pull/745)给Django.


以后就可以这样来指定Models的顺序了：

    ```python
    admin.site.register(ModelOne, ModelOneAdmin, order_sequence=2)
    admin.site.register(ModelTwo, ModelTwoAdmin, order_sequence=1)
    ```

这样 ModelTwo 就会显示在 ModelOne 的上方.

## 下面是我发表的 pull request discussion

Hi,  I using admin page in my daily work.

But, Django admin/app index page **not** ordering the models what I want.

So, I add an additional argument `order_sequence` in the `register` method

```python
admin.site.register(ModelOne, ModelOneAdmin, order_sequence=2)
admin.site.register(ModelTwo, ModelTwoAdmin, order_sequence=1)
```

#### This will be useful when edit multi-related-models, Put this models together, It's More convenient for edit

Details see the diff.

And also the docs.

![docs](http://i1297.photobucket.com/albums/ag23/yueyoum/django-docs_zpsc8950ce8.png)


## Below shows the difference in my project

### The Default Admin

![default-admin](http://i1297.photobucket.com/albums/ag23/yueyoum/django-admin1_zpsd1146cc0.png)


### The Custom order Admin

![custom](http://i1297.photobucket.com/albums/ag23/yueyoum/django-admin2_zpsa8d0751f.png)

