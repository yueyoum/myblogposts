Tags: python, django

看这两个model:

    ```python
    class ModelOne(models.Model):
        id = models.IntegerField(primary_key=True)
        # other files... 

    class ModelTwo(models.Model):
        # 没有指定primary_key
        # other files... 
    ```

在 Admin 页面中， ModelOne 是可以使用 **保存并继续编辑** 这个功能的。
因为它的ID在admin页面中是可编辑的。而 ModelTwo 却无法使用这个功能。

把 ModelOne 的主键暴露出来，是因为在填写 ModelOne 的时候，
也要指定ID。 但对于 ModelTwo，比如不关心主键，或者是一个ForeignKey or
ManyToMany 关联表。不需要暴露出来，暴露出来也不方便填写。


所以这里问题就来了。

# 如何方便的新加条目

如果是上面 ModelOne 那种形式， 直接在 admin 页面中修改主键ID，
然后点击 保存并继续编辑 按钮。这样就会插入一条新记录。

但是对于 ModelTwo 这样的，无法编辑主键， 当你单击了 保存并继续编辑 后，
是直接改变的原来的记录，并没有新加。
因为这条记录的ID没变，django认为你是在修改，而不是新增。

那么如何让 ModelTwo 能达到 **保存并继续编辑**， 也就是根据已有条目，
稍作修改后，新增条目呢？

因为要是像图中这么多的数据，每新增一个，都让策划重新填一边，那就只能蛋疼了……

![meshfields](http://i1297.photobucket.com/albums/ag23/yueyoum/aaa_zpsa5043b8a.png)

我想到两个容易实现的方法：

1.  给change_list添加一个新add按钮，点击此按钮，就可以新增。
    但并不是默认的新增那样，所有域都是空白的，从这个新add按钮
    进去，所以的域会自动用上一次编辑的页面字段来填充。
    这样，只要修改相应的地方，点击保存即可

2.  在change_form页面，增加一个按钮，当点击此按钮时，将此条目的主键ID更换，
    再保存。这样就建立的一个新条目

不过最后，我是按照第二条方法来做的。因为比第一条简单。

# 自定义change_form

既然要在 change_form 中添加按钮，那么肯定要自定义change_form。

自定义change_form，在 [Django 文档](https://docs.djangoproject.com/en/1.4/ref/contrib/admin/)
中有说明。只要照做就可以。
