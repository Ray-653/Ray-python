# Ray-python
python学习
from tkinter import *
from tkinter.messagebox import *
from tkinter import ttk
import os
import matplotlib.pyplot as plt
import matplotlib as mpl
import numpy as np

file_name = "student.txt"
id = ''

# 创建窗口
root = Tk()

# 设置窗口大小和位置
root.geometry('700x600+300+50')

root.title('学生管理系统')

Label(root, text="学号：").place(relx=0, rely=0.05, relwidth=0.1)
Label(root, text="姓名：").place(relx=0.5, rely=0.05, relwidth=0.1)
Label(root, text="Python：").place(relx=0, rely=0.1, relwidth=0.1)
Label(root, text="C语言：").place(relx=0.5, rely=0.1, relwidth=0.1)

stu_id = StringVar()
name = StringVar()
python = StringVar()
c = StringVar()

Entry(root, textvariable=stu_id).place(relx=0.1, rely=0.05, relwidth=0.37, height=25)
Entry(root, textvariable=name).place(relx=0.6, rely=0.05, relwidth=0.37, height=25)

Entry(root, textvariable=python).place(relx=0.1, rely=0.1, relwidth=0.37, height=25)
Entry(root, textvariable=c).place(relx=0.6, rely=0.1, relwidth=0.37, height=25)

Label(root, text='学生信息管理', bg='white', fg='red', font=('宋体', 15)).pack(side=TOP, fill='x')


# 显示学生信息
def show_info():
    index = 0
    x = treeview.get_children()
    for item in x:
        treeview.delete(item)
    if os.path.exists(file_name):
        with open(file_name, 'r', encoding='utf8') as file_text:
            file_list = file_text.readlines()
    else:
        file_list = []
    for item in file_list:
        item = tuple(dict(eval(item)).values())
        treeview.insert("", index, values=item)
        index += 1


# 添加学生信息
def add_info():
    if len(stu_id.get()) * len(name.get()) * len(python.get()) * len(c.get()) == 0:
        showerror(title='提示', message='输入不能为空')
    else:
        if os.path.exists(file_name):
            with open(file_name, 'r', encoding='utf8') as file_text:
                file_list = file_text.readlines()
            file_list = [dict(eval(x))['id'] for x in file_list]
        else:
            file_list = []
        x = treeview.get_children()
        for item in x:
            treeview.delete(item)
        stu_dict = dict()
        if stu_id.get() not in file_list:
            stu_dict['id'] = stu_id.get()
            stu_dict['name'] = name.get()
            try:
                stu_dict['Python'] = int(python.get())
                stu_dict['C语言'] = int(c.get())
                if (stu_dict['Python'] * stu_dict['C语言'] < 0):
                    raise ValueError
            except ValueError:
                showerror("错误", "成绩输入非法")
                return

            with open(file_name, 'a+', encoding='utf8') as file_text:
                file_text.write(str(stu_dict) + '\n')
            show_info()

        else:
            showerror("错误", "学号重复")
            show_info()


# 修改学生信息
def modify_info():
    not_find = True
    if len(stu_id.get()) * len(name.get()) * len(python.get()) * len(c.get()) == 0:
        showinfo("提示", "信息未填写完整!")
    elif os.path.exists(file_name):
        with open(file_name, 'r', encoding='utf8') as file_old:
            file_list = file_old.readlines()
        with open(file_name, 'w', encoding='utf8') as file_new:
            for stu in file_list:
                stu = dict(eval(stu))
                if stu['id'] == stu_id.get():
                    try:
                        stu['Python'] = int(python.get())
                        stu['C语言'] = int(c.get())
                        stu['id'] = stu_id.get()
                        stu['name'] = name.get()
                        if (stu['Python'] * stu['C语言'] < 0):
                            raise ValueError
                    except ValueError:
                        showerror("错误", "成绩输入非法")
                    not_find = False
                file_new.write(str(stu) + '\n')
            showinfo("提示", "修改成功!")
            if not_find:
                showerror("提示", "没有找到该学生")
    else:
        showerror("错误", "文件不存在")
    show_info()


# 删除学生信息
def del_info():
    if os.path.exists(file_name):
        with open(file_name, 'r', encoding='utf-8') as file_old:
            stu_old = file_old.readlines()
    else:
        stu_old = []
    is_find = False
    if stu_old:
        # 删除学生
        with open(file_name, 'w', encoding='utf-8') as file_new:
            for stu in stu_old:
                d = dict(eval(stu))
                if (d['id'] != id):
                    file_new.write(str(d) + "\n")
                elif askokcancel("提示", "你确定要删除该学生吗？"):
                    is_find = True
            if is_find:
                showinfo("提示", "删除成功!")

    else:
        showerror("错误", "没有找到该学生")
    show_info()


def search_info():
    not_find = True
    x = treeview.get_children()
    for item in x:
        treeview.delete(item)
    id = stu_id.get()
    if os.path.exists(file_name):
        with open(file_name, 'r', encoding='utf8') as file_text:
            file_list = file_text.readlines()
    else:
        file_list = []
    for stu in file_list:
        stu_dict = dict(eval(stu))
        if stu_dict['id'] == id:
            stu = tuple(x for x in stu_dict.values())
            treeview.insert("", 0, values=stu)
            not_find = False
    if not_find:
        showerror("提示", "未查询到该学生信息")


# 排序
def treeview_sort_column(tv, col, reverse):  # Treeview、列名、排列方式
    l = [(tv.set(k, col), k) for k in tv.get_children('')]
    l.sort(reverse=reverse)  # 排序方式
    # rearrange items in sorted positions
    for index, (val, k) in enumerate(l):  # 根据排序后索引移动
        tv.move(k, '', index)
    tv.heading(col, command=lambda: treeview_sort_column(tv, col, not reverse))  # 重写标题，使之成为再点倒序的标题


def statistics_info():
    print("————————数据统计—————————")
    stu_list = []
    if os.path.exists(file_name):
        with open('student.txt', 'r', encoding='utf-8') as file:
            stu_old = file.readlines()
        for stu in stu_old:
            stu = dict(eval(stu))
            stu_list.append(stu)

        print("一共有%d名学生" % len(stu_list))
        # 统计最高分、最低分、平均分[(13,56),(45,98)]
        stu_score_tuple = [(x['Python'], x['C语言']) for x in stu_list]
        python_min = min(x[0] for x in stu_score_tuple)
        python_max = max(x[0] for x in stu_score_tuple)
        c_min = min(x[1] for x in stu_score_tuple)
        c_max = max(x[1] for x in stu_score_tuple)
        python_avg = sum(x[0] for x in stu_score_tuple) / len(stu_list)
        c_avg = sum(x[1] for x in stu_score_tuple) / len(stu_list)
        print(f"\t\t{'最高分':<8s}\t", end='')
        print(f"{'最低分':<8s}\t", end='')
        print(f"{'平均分':<8s}")
        print(f"{'Python':<8s}", end='')
        print(f"{python_max:<8.1f}\t", end='')
        print(f"{python_min:<8.1f}\t", end='')
        print(f"{python_avg:<8.1f}")
        print(f"{'C':<8s}", end='')
        print(f"{c_max:<8.1f}\t", end='')
        print(f"{c_min:<8.1f}\t", end='')
        print(f"{c_avg:<8.1f}")

        # 柱状图
        # 使图形中的中文正常编码显示
        mpl.rcParams["font.sans-serif"] = ["SimHei"]
        # 每个柱子下标的索引
        x = np.arange(len(stu_list))

        y = [x['Python'] for x in stu_list]
        y1 = [x['C语言'] for x in stu_list]

        # 柱子的宽度
        bar_width = 0.35
        tick_label = [x['name'] for x in stu_list]

        # 绘制柱状图并设置其各项属性
        plt.bar(x, y, bar_width, align="center", color="c", label="Python", alpha=0.5)
        plt.bar(x + bar_width, y1, bar_width, color="b", align="center", label="C语言", alpha=0.5)

        plt.title('学生成绩统计表')
        plt.xlabel("姓名")
        plt.ylabel("成绩")

        plt.xticks(x + bar_width / 2, tick_label)
        plt.yticks(np.arange(0, 101, 20))

        # 添加图例
        plt.legend(loc="upper left")

        plt.show()
    else:
        showinfo("提示", "无任何学生信息")


def treeview_click(event):  # 单击
    global id
    for item in treeview.selection():
        item_text = treeview.item(item, "values")
        id = item_text[0]
        stu_id.set(item_text[0])
        name.set(item_text[1])
        python.set(item_text[2])
        c.set(item_text[3])


Button(root, text="显示所有信息", command=show_info).place(relx=0.05, rely=0.2, width=80)
Button(root, text="添加学生信息", command=add_info).place(relx=0.20, rely=0.2, width=80)
Button(root, text="删除学生信息", command=del_info).place(relx=0.35, rely=0.2, width=80)
Button(root, text="修改学生信息", command=modify_info).place(relx=0.50, rely=0.2, width=80)
Button(root, text="统计学生信息", command=statistics_info).place(relx=0.65, rely=0.2, width=80)
Button(root, text="查询学生信息", command=search_info).place(relx=0.80, rely=0.2, width=80)

treeview = ttk.Treeview(root, show='headings', column=('stu_id', 'name', 'python', 'c'))
treeview.column('stu_id', width=150, anchor="center")
treeview.column('name', width=150, anchor="center")
treeview.column('python', width=150, anchor="center")
treeview.column('c', width=150, anchor="center")

treeview.heading('stu_id', text='学号', command=lambda: treeview_sort_column(treeview, 'stu_id', False))
treeview.heading('name', text='姓名', command=lambda: treeview_sort_column(treeview, 'name', False))
treeview.heading('python', text='Python', command=lambda: treeview_sort_column(treeview, 'python', False))
treeview.heading('c', text='C语言', command=lambda: treeview_sort_column(treeview, 'c', False))

treeview.place(rely=0.3, relwidth=1)
treeview.bind('<ButtonRelease-1>', treeview_click)  # 绑定单击离开事件===========

root.mainloop()
