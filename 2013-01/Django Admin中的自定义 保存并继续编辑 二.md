Tags: python django

在 上一篇 中分析了需求，以及要达到此需求的方法。
并且完成了自定义页面，加入了需要的元素。

现在就开始来完成对应的功能。

# 简单Model

    ```python
    class SimpleModel(models.Model):
        field_one = models.IntegerField()
        field_two = models.CharField(max_length=32)
    ```

这种是最简单的情况，没有其他表与之关联。
所以其对应的admin.py也很简单

    ```python
    class SimpleModelAdmin(admin.ModelAdmin):
        def save_model(self, request, obj, form, change):
            if change and request.POST.get('_as_new', None) is not None:
                max_id = SimpleModel.objects.all().order_by("-id")[0].id
                obj.id = max_id + 1
                # yes, you can use this: obj.id = None
            obj.save()
    ```

**\_as_new** 是我们自定义的 custom_submit_line.html 中写的新增的 input 的 name

所以，上面的判断就是 **在修改一条记录的时候，点击了我们自定义的按钮**
然后把这个条目在数据库中做一个复制。

因为我们将其ID设置为表中最大ID+1，或者None，然后在保存的时候会判断此obj的ID
不存在，就会自动为我们新建一条记录。我们的目的就达到了。然后就可以在这个新条目上
继续修改，然后点击保存。


# 复杂Model

如果admin中 有了 inline model, 那么情况就变得复杂了

    ```python
    class ComplexModelAdmin(admin.ModelAdmin):
        inlines = [RealtedModelInline, ]
    ```

比如图中这种情况：

![ComplexModelAdmin](http://i1297.photobucket.com/albums/ag23/yueyoum/ccc_zpsdd744049.png)


这时不能还像上面那样简单的用 save_model, 因为 只会保存 自身modele的信息，而关联表却没有
对应的保存下来。

虽然 可以重写 **save_formset** 方法， 但我并没用这个方法， 而是继续在 save_model 中
添加了额外的操作来保存关联表.

    
    ```python
    class ComplexModelAdmin(admin.ModelAdmin):
        inlines = [RelatedModelInline, ]

        def save_model(self, request, obj, form, change):
            # 记录下原始ID， 后面会更改obj的ID
            original_id = obj.id

            if change and request.POST.get('_as_new', None) is not None:
                max_id = ComplexModel.objects.all().order_by("-id")[0].id
                obj.id = max_id+1

            obj.save()

            # 上面只是将obj自身进行了复制保存，与它相关联的记录还没有相应的复制
            # 下面就是关联条目的处理
            if change and request.POST.get('_as_new', None) is not None:
                original_obj = ComplexModel.objects.get(pk=original_id)
                original_related = original_obj.related_set.all()
                for r in original_related:
                    # 对每个关联条目，进行复制
                    new_r = Related.objects.get(pk=r.id)
                    new_r.id = None
                    new_r.COMPLEX = obj
                    new_r.save()
    ```

上面都是伪代码， 对于关联条目的思路就是清理ID（这样就会复制保存），
然后把关联 COMPLEX 替换成 新obj. 这样就完成了 自身和关联条目 的完整复制

