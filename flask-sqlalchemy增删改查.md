#创建学生模型类
	from flask_sqlalchemy import SQLAlchemy
	
	
	db = SQLAlchemy()
	
	class Student(db.Model):
	    id = db.Column(db.Integer, primary_key=True, autoincrement=True)
	    s_name = db.Column(db.String(10), unique=True, nullable=False)
	    gender = db.Column(db.Boolean, default=1)
	    grade_id = db.Column(db.Integer, db.ForeignKey('grade.id'),nullable=True)
	
	
	    # __tablename__用于指定模型映射的表名，如果没有表名，则数据库中的表名为模型类名的小写
	    def save(self):
	        db.session.add(self)
	        db.session.commit()

#在应用app views.py中创建蓝图
	from flask import Blueprint, render_template, redirect, url_for, request

	blue = Blueprint('user',__name__)
#在工程目录下创建manage.py并注册蓝图
	from flask import Flask
	app = Flask(__name__)
	app.register_blueprint(blueprint=blue,url_prefix='/app')


#flask框架中对数据的增删改查
	#添加
	@blue.route('/add_stu/', methods=['POST'])
	def add_stu():
	    # 插入数据
	    stu = Student()
	    stu.s_name = '表妹'
	    stu.save()
	    return '创建学生成功'

	#删除	
	@blue.route('/del_stu/', methods=['DELETE'])
	def del_stu():
	    # 删除数据
	    stu = Student.query.filter(Student.s_name == '小明').first()
	    db.session.delete(stu)
	    db.session.commit()
	    return '删除学生成功'

	#更新
	@blue.route('/update_stu/',methods=['PATCH'])
	def update_stu():
	    # stu = Student.query.filter(Student.s_name == '小明').first()
	    # stu.s_name = '猴子'
	    # stu.save()
	    # 第二种，使用update
	    stu = Student.query.filter(Student.s_name == '猪八戒').update({'s_name':'小张'})
	    stu.commit()
	    return '更新学生成功'


	#修改
	@blue.route('/select_stu/', methods=['GET'])
	def select_stu():
	    # filter(模型名.字段 == '值')
	    stu = Student.query.filter_by(s_name='小明').first()
	    # all():查询所有的结果，返回结果的列表
	    # first(): 返回对象
	    # 注意不要写all().first()
	
	    # 查询id为2的学生，使用get()方法,
	    stu = Student.query.filter(Student.id == 2).first()
	    # get(): 获取主键所在行的对象，获取不到，返回空。
	    stu = Student.query.get(2)
	
	    # order_by()排序
	    # 降序: -id、id desc
	    # 升序: id、 id asc
	    stu = Student.query.order_by('-id')
	
	    # 使用运算符
	    # Django:filter(s_name__contains='111')
	    # Flask的SQLALchemy中: filter(模型名.s_name.contains(''))
	
	
	    # 例子:模糊查询姓名中包含'小'的学生背景
	    stu = Student.query.filter(Student.s_name.contains('小')).all()
	
	
	    # startswith, endswith, like _和%, contains
	    stus = Student.query.filter(Student.s_name.startswith('小')).all()
	    stus = Student.query.filter(Student.s_name.endswith('明')).all()
	
	    # 查询姓名中包含'李'的学生信息，使用like, _和%
	    stus = Student.query.filter(Student.s_name.like('%明%')).all()
	    stus = Student.query.filter(Student.s_name.like('_明')).all()
	
	    # in_(): 查询某个范围之内的对象
	    stus = Student.query.filter(Student.id.in_([1,2,3,4,5])).all()
	
	
	
	    # 查询id大于5的学生信息,__gt__
	    stus = Student.query.filter(Student.id.__gt__(5)).all()
	    stus = Student.query.filter(Student.id > 5).all()
	
	    # 分页
	    page = request.args.get('page',1)
	    paginate = Student.query.paginate(int(page),3)
	    stus = paginate.items
	    # return render_template('stus.html',stus=stus,paginate=paginate)
	
	    # 例子，查询性别男的，且姓名中包含'小'的学生信息
	    stus = Student.query.filter(Student.gender == 1, Student.s_name.contains('小')).all()
	    # and_ 且，not_ 非，or_ 或
	    stus = Student.query.filter(or_(Student.gender == 1, Student.s_name.contains('小'))).all()
	    stus_names = [stu.s_name for stu in stus ]
	    return str(stus_names)






	