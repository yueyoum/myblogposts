Tags: python, django

# 我为什么用Django

 不得不说，刚开始学习Python Web的时候，由于Django名气大，
 所以就选择了学习Django。然后在工作中也是在用 Django。

 虽然前前后后接触/使用过 flask, web.py, tornado, bottle,
 但主力还是 Django.

 *  一站式web framework
 *  超强的Admin
 *  大量第三方程序/库的支持

第一点优劣参半，但第二点，也就是 **Admin** , 绝对值得用Django

# 遇到的问题

在项目中用 Django 的 Admin 做编辑器。有两个好处:

*   超快速的生成界面。
*   权限管理， 可以分配谁可以编辑，谁只能浏览

但也有缺点，比如超过三个表关联的时候，就无法在一个页面中编辑。
不过这不是今天要说的重点。

**重点是这个：**

有时候一个要编辑的条目有N种状态要选择，但每种状态要填写的字段又不一样，
以前只是简单的写到一个表中，使用大量的字段来兼容这些不同状态的值。
比如这样：

    ```python
    class SomeEditor(models.Model):
        mode = models.ForeignKey(SomeModes)
        mode_one_1 = models.IntegerField(default=0)
        mode_one_2 = models.IntegerField(default=0)
        mode_two_1 = models.IntegerField(default=0)
        mode_two_2 = models.IntegerField(default=0)
        ...
    ```

当然上面示例中也可以写成 `models.IntegerField(null=True, blank=True)`

页面出来的效果大概就是这样：
![showcase1](http://i1297.photobucket.com/albums/ag23/yueyoum/editor_showcase1_zps28c3c9fd.png)

然后你还得啪啪啪的给策划说，当你选择A的时候，要填哪几个方框，
选B的时候，填哪几个。。。

这种方式的缺点太多了

*   每添加一个新mode, 都得 alter table "xxx" add column
*   过一段时间，估计你这个编辑器的作者看到一大堆输入框都迷茫了

**蛋疼**...


# 较好的解决方法

今天又要做一个新的编辑器，同样遇到这个问题。
所以决定彻底解决。经过思考/设计后，下面的这种model关联方式可以很好的解决这种问题.

**直接取自真实项目, 但省略掉不相关代码**

    ```python
    class Condition(models.Model):
        """条件"""
        conditions = models.ManyToManyField(ConditionEnum, through='EnumValue')

        def __unicode__(self):
            cons = self.enumvalue_set.order_by('con_enum__id')
            text = map(
                lambda c: u'{0} {1} {2}'.format(
                    c.con_enum.zh_name,
                    c.target_id or '',
                    c.value
                ),
                cons
            )
            return u','.join(text)

        class Meta:
            verbose_name = u'条件'
            verbose_name_plural = u'条件'

    class ForEquip(models.Model):
        """装备"""
        TYPE = ( (6, u'武器'), (5, u'非武器') )
        # 在 Equipment 表中 武器的type范围是 6~11， 非武器是 2~5
        # 所以这里用 >5 来表示武器， <=5 来表示非武器
        condition = models.OneToOneField(Condition)
        equ_type = models.IntegerField(choices=TYPE, verbose_name=u'类型')
        lv_need = models.IntegerField(verbose_name=u'需求等级')

        class Meta:
            verbose_name = u'用于装备'
            verbose_name_plural = u'用于装备'


    class ForFormation(models.Model):
        """阵法"""
        TYPE = ( (1, u'防御型'), (2, u'攻击型') )
        condition = models.OneToOneField(Condition)
        for_type = models.IntegerField(choices=TYPE, verbose_name=u'类型')
        lv = models.IntegerField(verbose_name=u'等级')

        class Meta:
            verbose_name = u'用于阵法'
            verbose_name_plural = u'用于阵法'


    class ForSkill(models.Model):
        """技能"""
        condition = models.OneToOneField(Condition)
        lv = models.IntegerField(verbose_name=u'等级')
        lv_need = models.IntegerField(verbose_name=u'需求等级')

        class Meta:
            verbose_name = u'用于技能'
            verbose_name_plural = u'用于技能'
    ```


上面的 ForEquip, ForFormation, ForSkill是 Condition 的三个 mode, 
它们用 OneToOneField 关联在一起，

对应的admin.py 就这样写：


    ```python
    class EnumValueInline(admin.TabularInline):
        model = Condition.conditions.through

    class ForEquipInline(admin.StackedInline):
        model = ForEquip

    class ForFormationInline(admin.StackedInline):
        model = ForFormation

    class ForSkillInline(admin.StackedInline):
        model = ForSkill


    class ConditionAdmin(admin.ModelAdmin):
        list_display = ('Used_for', 'Details',)
        inlines = [ 
            ForEquipInline, ForFormationInline, ForSkillInline,
            EnumValueInline,
        ]   

        def Used_for(self, obj):
            try:
                return u'装备: {0}, 需求等级 {1}'.format(
                    obj.forequip.equ_type, obj.forequip.lv_need
                )   
            except:
                try:
                    return u'阵法: {0}, 等级 {1}'.format(
                        obj.forformation.for_type, obj.forformation.lv
                    )   
                except:
                    return u'技能: 等级 {0}, 需求等级 {1}'.format(
                        obj.forskill.lv, obj.forskill.lv_need
                    )   

        def Details(self, obj):
            return obj.__unicode__()


    admin.site.register(Condition, ConditionAdmin)
    ```


生成的Admin页面

![showcase2](http://i1297.photobucket.com/albums/ag23/yueyoum/editor_showcase2_zps8396be82.png)


这样就很清晰，到底要干什么，只用添加对应的表就可以。而且还是带有错误检测的。

这样设计表的好处：

*   作为程序员，以后要添加一个新的mode，只要在 models.py 中添加一个新表，并用OneToOne关联。
    然后执行 python manage.py syncdb 就行。方便快速
*   作为编辑人员，也会清楚的明白改怎样添加/修改

