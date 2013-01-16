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

## 建自定义change_form

根据你的APP和model，建立对应的目录: templates/admin/APP/MODEL/change_form.html .
**并且讲 templates 目录加入 settings.py 中的 TEMPLATE_DIRS**

change_form.html 中的内容：

    ```html
    {% extends "admin/change_form.html" %}
    {% load i18n admin_static admin_modify %}
    {% load url from future %}
    {% load admin_urls %}

    {% load custom_submit_line %}

    {% block submit_buttons_bottom %}{% custom_submit_line %}{% endblock %}
    ```

这里用了自定义的一个tag, **custom_submit_line** ，其实也就是简单的扩展了一下
django自身的 **contrib/admin/templatetags/admin_modify.py 中的 submit_row**


因为我们的目的是要修改 admin change_form 页面下面的一行 按钮，而这些按钮默认有
submit_row 这个tag来导入。所以我们自定义这个tag, 并且自定义 admin/submit_line.html

# 自定义tag

自定义 custom_submit_line tag, 如何自定义， [Django 文档](https://docs.djangoproject.com/en/1.4/howto/custom-template-tags/)中也讲的很明白，这里直接上代码

    ```python
    # cat YOURAPP/templatetags/custom_submit_line.py
    import os

    CURRENT_PATH = os.path.dirname(os.path.realpath(__file__))
    SUBMIT_LINE_HTML = os.path.normpath(
        os.path.join(CURRENT_PATH, '../../templates/admin/custom_submit_line.html')
    )

    from django import template

    register = template.Library()

    @register.inclusion_tag(SUBMIT_LINE_HTML, takes_context=True)
    def custom_submit_line(context):
        opts = context['opts']
        change = context['change']
        is_popup = context['is_popup']
        save_as = context['save_as']
        return {
            'onclick_attrib': (opts.get_ordered_objects() and change
                                and 'onclick="submitOrderForm();"' or ''),
            'show_delete_link': (not is_popup and context['has_delete_permission']
                                  and (change or context['show_delete'])),
            'show_save_as_new': not is_popup and change and save_as,
            'show_save_and_add_another': context['has_add_permission'] and
                                not is_popup and (not save_as or context['add']),
            'show_save_and_continue': not is_popup and context['has_change_permission'],
            'is_popup': is_popup,
            'show_save': True,
            'show_save_and_as_new': context['has_add_permission'] and
                                not is_popup and (not save_as or context['add']),
        }
    ```


在这个 tags 中，我们载入了自定义的 custom_submit_line.html，
正如上面所说，这个只是简单的在 admin/submit_line.html 中添加了一个 input


# 自定义submit_line.html

这个没什么好说的。就是把 admin/submit_line.html, 拷贝一份，然后加上自己的内容

    ```html
    {% load i18n %}
    <div class="submit-row">
    {% if show_save %}<input type="submit" value="{% trans 'Save' %}" class="default" name="_save" {{ onclick_attrib }}/>{% endif %}
    {% if show_delete_link %}<p class="deletelink-box"><a href="delete/" class="deletelink">{% trans "Delete" %}</a></p>{% endif %}
    {% if show_save_as_new %}<input type="submit" value="{% trans 'Save as new' %}" name="_saveasnew" {{ onclick_attrib }}/>{%endif%}
    {% if show_save_and_add_another %}<input type="submit" value="{% trans 'Save and add another' %}" name="_addanother" {{ onclick_attrib }} />{% endif %}
    {% if show_save_and_continue %}<input type="submit" value="{% trans 'Save and continue editing' %}" name="_continue" {{ onclick_attrib }}/>{% endif %}
    {% if show_save_and_as_new %}<input type="submit" value="将此内容作为新内容保存" name="_as_new" {{ onclick_attrib }}/>{% endif %} 
    </div>
    ```

上面最后一行是我新加的。


# 页面自定义完成

这时，在你对应的APP 的 增加/修改页面， 应该就会出现一个新的按钮：

![new button](http://i1297.photobucket.com/albums/ag23/yueyoum/bbb_zpsc607a564.png)


这样页面的自定义就完成了， 下一篇再讲如何写对应的功能

