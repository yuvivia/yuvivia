import datetime
import sqlite3
from sqlite3 import Error
from typing import List, Any
import mysql.connector

import pymysql
import tkinter as tk

from sqlalchemy.testing import db

# 连接到 MySQL 数据库
conn = mysql.connector.connect(
    host="localhost",
    user="root",
    password="123456",
    database="library_manage_system"
)

cursor = conn.cursor()

tables = ['user_info', 'manager_info', 'book_list', 'borrow_info']
for table in tables:
    cursor.execute(f"SHOW TABLES LIKE '{table}'")
    if not cursor.fetchone():
        if table == 'user_info':
            cursor.execute('''CREATE TABLE IF NOT EXISTS user_info
                              (user_id VARCHAR(50), user_name VARCHAR(50), user_password VARCHAR(50))''')
        elif table == 'manager_info':
            cursor.execute('''CREATE TABLE IF NOT EXISTS manager_info
                              (manager_id VARCHAR(100), manager_name VARCHAR(12), manager_password VARCHAR(20))''')
        elif table == 'book_list':
            cursor.execute('''CREATE TABLE IF NOT EXISTS book_list
                              (bookid VARCHAR(12), ISBN VARCHAR(20), bookType VARCHAR(10),
                              author VARCHAR(15), publish_name VARCHAR(50), keyword VARCHAR(20),
                              index_ VARCHAR(20), topic VARCHAR(20))''')
        elif table == 'borrow_info':
            cursor.execute('''CREATE TABLE IF NOT EXISTS borrow_info
                              (book_id VARCHAR(12), user_id VARCHAR(20), book_name VARCHAR(10),
                              author VARCHAR(15), borrow_date VARCHAR(50), return_date VARCHAR(20))''')

def login():
    s = 0
    while True:
        username = input("请输入用户名：").strip()
        password = input("请输入密码：").strip()

        # 管理员登录
        if username == "admin" and password == "123":
            print("管理员登录成功！")
            admin_module()  # 调用管理员模块函数
            break

        # 用户登录
        elif username != "admin":
            cursor.execute("select * from user_info where user_id=%s and user_password =%s",(username,password))
            if cursor.fetchone():
                print("用户登录成功")
                user_module()
                break

        else:
            print("用户名或密码错误！")
            s += 1
            if s < 3:
                print(f"您还有{3 - s}次机会尝试")
            else:
                print("已达到最大尝试次数，退出登录程序。")
                break
def add_book():
     book_id=(input("请输入图书ID："))
     book_name=(input("请输入图书名称："))
     book_author=(input("请输入图书作者："))
     book_publisher=(input("请输入图书出版社："))
     book_type=(input("请输入图书类型："))
     book_remind=(input("请输入共几本书："))
     book_lend=(input("请输入可借本数："))
     cursor.execute(
         "INSERT INTO book_list (bookid, bookType, author, publish_name, keyword, index_, topic) VALUES (%s, %s, %s, %s, %s, %s, %s)",
         (book_id, book_type, book_author, book_publisher, "", "", ""))
     db.commit()
     print("于" + datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S") + "图书添加成功！")

def admin_module():
    print("欢迎进入管理员模块！这里是管理员可以执行的操作...")
    # 该模块用于管理图书信息，包括添加、删除、修改、查询等功能。
    # 功能包括：
    # 1. 添加图书：输入图书信息，将图书信息写入文件中。
    # 2. 删除图书：输入图书ID，从文件中删除该图书信息。
    # 3. 修改图书：输入图书ID，修改图书信息，并将修改后的信息写入文件中。

    book_id = []
    book_type = []
    book_name = []
    book_author = []
    book_publisher = []
    book_remind = []
    book_lend = []

    user_id=[]
    user_name: list[Any]=[]
    user_password=[]

    with open('book_data.txt', 'r', encoding='utf-8') as f:
        for lines in f:
            parts = lines.strip().split(',')
            book_id.append(parts[0])
            book_type.append(parts[4])
            book_name.append(parts[1])
            book_author.append(parts[2])
            book_publisher.append(parts[3])
            book_remind.append(parts[5])
            book_lend.append(parts[6])

    # 1. 添加新的图书

    # 添加已经存在的图书
    def add_book_exist():
        print("添加已经存在的图书")
        id = input("请输入图书编号：")
        book_found = False

        # 1. 读取原始文件数据
        with open('book_data.txt', 'r', encoding='utf-8') as f:
            lines = f.readlines()

        # 2. 更新指定图书的数据
        for i, line in enumerate(lines):
            parts = line.strip().split(',')
            if id == parts[0]:
                lend_int = int(parts[6]) if parts[6].isdigit() else 0
                if lend_int > 0:
                    book_remind = str(int(parts[5]) + 1)
                    book_lend = str(int(parts[6]) + 1)
                    lines[i] = ','.join(parts[:4] + [parts[4]] + [book_remind, book_lend]) + '\n'
                    book_found = True
                else:
                    print("无法添加图书！")
                    break

        if not book_found:
            print("图书编号不存在！")
        else:
            # 3. 写回更新后的数据
            with open('book_data.txt', 'w', encoding='utf-8') as f:
                f.writelines(lines)
            print("于" + datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S") + "图书数量增加成功！")

    # 2. 删除图书
    def del_book():

        book_id_del = input("请输入要删除的图书ID：")
        if book_id_del in book_id:
            index = book_id.index(book_id_del)
            book_id.pop(index)
            book_name.pop(index)
            book_author.pop(index)
            book_publisher.pop(index)
            book_type.pop(index)
            book_remind.pop(index)
            book_lend.pop(index)
            with open('book_data.txt', 'w', encoding='utf-8') as f:
                for i in range(len(book_id)):
                    f.write(book_id[i] + ',' + book_name[i] + ',' + book_author[i] + ',' + book_publisher[i] + ',' +
                            book_type[i] + ',' + book_remind[i] + ',' + book_lend[i] + '\n')
            print("图书删除成功！")
            return True
        else:
            print("图书ID不存在！")
            return False

    # 3. 修改图书
    def modify_book():
        book_id_modify = input("请输入要修改的图书ID：")
        cursor.execute("select *from book_list where bookid =%s",(book_id_modify,))
        row=cursor.fetchone()
        if row:
            new_book_name=input("请输入新的图书名称:")
            new_book_author=input("请输入新的图书作者：")
            new_book_publisher=input("请输入新的图书出版社:")
            new_book_type=input("请输入新的图书类型")
            cursor.execute("""update book_list set bookType =%s,author=%s,publish_name=%s,topic=%s where bookid=%s""",(new_book_type,new_book_author,new_book_publisher,new_book_name,book_id_modify))
            db.commit()
            print("图书修改成功！")
        else:
            print("图书ID不存在！")

    # 4. 查询图书
def query_book():
    book_id_query = input("请输入要查询的图书ID：")
    cursor.execute("select *from book_list where bookid=%s",(book_id_query,))
    row=cursor.fetchone()
    if row:
        print("图书ID：", row[0])
        print("图书名称：", row[2])
        print("图书作者：", row[3])
        print("图书出版社：", row[4])
        print("图书类型：", row[1])
        print("共几本书：", "")
        print("可借本数：", "")
    else:
        print("图书ID不存在！")


    # 5. 添加新的用户
def add_user():
    user_id=input("请输入用户ID：")
    user_name=input("请输入用户名字：")
    user_password=input("请输入用户的密码:")
    cursor.execute("INSERT INTO user_info (user_id, user_name, user_password) VALUES (%s, %s, %s)",
                       (user_id, user_name, user_password))
    db.commit()

    print("于" + datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S") + "用户添加成功！")

    #add_user()


    #删除用户
def del_user():
    user_id_del = input("请输入要删除的用户ID：")
    cursor.execute("DELETE FROM user_info where user_id=%s",(user_id_del,))
    db.commit()
    if cursor.rowcount > 0:
        print("用户删除成功！")
    else:
        print("用户ID不存在！")

    #修改用户信息

def modify_user():
    user_id_modify = input("请输入要修改的用户ID：")
    cursor.execute("delete from user_info where user_id=%s",(user_id_modify,))
    db.commit()
    if cursor.rowcount>0:
         print("用户删除成功！")
    else:
        print("用户ID不存在！")


def modify_book():
    pass


def del_book():
    pass


def main():
    while True:
        print("欢迎使用图书管理系统！")
        print("1. 添加图书")
        print("2. 删除图书")
        print("3. 修改图书")
        print("4. 查询图书")
        print("5. 添加用户")
        print("6. 删除用户")
        print("7. 修改用户")
        print("0. 退出系统")
        choice = input("请输入您的选择：")
        if choice == '1':
            add_book()
        elif choice == '2':
             del_book()
        elif choice == '3':
            modify_book()
        elif choice == '4':
            query_book()
        elif choice=='5':
             add_user()
        elif choice=='6':
            del_user()
        elif choice=='7':
            modify_user()
        elif choice == '0':
            print("欢迎下次使用！")
            break
        else:
            print("输入错误，请重新输入！")

if __name__ == '__main__':
    main()

modify_book
def user_module():
    print("欢迎进入用户模块！这里是用户可以执行的操作...")
    book_id = []
    book_type = []
    book_name = []
    book_author = []
    book_publisher = []
    book_remind = []
    book_lend = []
    with open('book_data.txt', 'r', encoding='utf-8') as f:
        for lines in f:
            parts = lines.strip().split(',')
            book_id.append(parts[0])
            book_type.append(parts[4])
            book_name.append(parts[1])
            book_author.append(parts[2])
            book_publisher.append(parts[3])
            book_remind.append(parts[5])
            book_lend.append(parts[6])

    # 图书类型查询
    def user_search_by_type():
        while True:
            print("*l图书类型查询")
            print("1. 文学 2. 数学 3. 计算机4. 医学")
            type_name = input("请输入图书类型：")
            for i in range(len(book_type)):
                if type_name == book_type[i]:
                    print("图书编号：", book_id[i], "图书名称：", book_name[i], "图书作者：", book_author[i],
                          "图书出版社：",
                          book_publisher[i], "图书剩余数量：", book_remind[i], "图书已借数量：", book_lend[i])
                    print("-" * 150)
            print("是否继续查询图书类型？y/n")
            choice = input()
            if choice == 'n':
                break

    def user_search_by_name():
        while True:
            print("图书名称查询")
            name = input("请输入图书名称：")
            for i in range(len(book_name)):
                if name in book_name[i]:
                    print("图书编号：", book_id[i], "图书名称：", book_name[i], "图书作者：", book_author[i],
                          "图书出版社：",
                          book_publisher[i], "图书剩余数量：", book_remind[i], "图书已借数量：", book_lend[i])
                    print("-" * 150)
            print("是否继续查询图书名称？y/n")
            choice = input()
            if choice == 'n':
                break

    def user_search_by_author():
        while True:
            print("图书作者查询")
            author = input("请输入图书作者：")
            for i in range(len(book_author)):
                if author in book_author[i]:
                    print("图书编号：", book_id[i], "图书名称：", book_name[i], "图书作者：", book_author[i],
                          "图书出版社：",
                          book_publisher[i], "图书剩余数量：", book_remind[i], "图书已借数量：", book_lend[i])
                    print("-" * 150)
            print("是否继续查询图书作者？y/n")
            choice = input()
            if choice == 'n':
                break

    def user_search_by_id():
        while True:
            print("图书编号查询")
            id = input("请输入图书编号：")
            for i in range(len(book_id)):
                if id == book_id[i]:
                    print("图书编号：", book_id[i], "图书名称：", book_name[i], "图书作者：", book_author[i],
                          "图书出版社：",
                          book_publisher[i], "图书剩余数量：", book_remind[i], "图书已借数量：", book_lend[i])
                    print("-" * 150)

    # 用户是否借阅图书
    def user_borrow_book():
        while True:
            print("借阅图书")
            id = input("请输入图书编号：")
            for i in range(len(book_id)):
                if id == book_id[i]:
                    # 将库存从字符串转换为整数
                    remind_int = int(book_remind[i]) if book_remind[i].isdigit() else 0

                    if remind_int > 0:  # 检查转换后的库存是否足够
                        book_remind[i] = str(remind_int - 1)  # 更新库存为新的字符串值
                        book_lend[i] = str(int(book_lend[i]) - 1)  # 增加借出数量，确保借出数量也是字符串类型
                        print("图书借阅成功！")
                        print("图书编号：", book_id[i], "图书名称：", book_name[i], "图书作者：", book_author[i],
                              "图书出版社：",
                              book_publisher[i], "图书剩余数量：", book_remind[i], "图书可借数量：", book_lend[i])
                        print("-" * 150)
                        break
                    else:
                        print("图书借阅失败！图书库存不足！")
                        break
            print("是否继续借阅图书？y/n")
            choice = input()
            if choice == 'n':
                break
            else:
                remind_int = int(book_remind[i]) if book_remind[i].isdigit() else 0
                if remind_int > 0:  # 检查转换后的库存是否足够
                    book_remind[i] = str(remind_int)  # 更新库存为新的字符串值
                    book_lend[i] = str(int(book_lend[i]))  # 增加借出数量，确保借出数量也是字符串类型
                    print("图书续借阅成功！")
                    print("图书编号：", book_id[i], "图书名称：", book_name[i], "图书作者：", book_author[i],
                          "图书出版社：",
                          book_publisher[i], "图书剩余数量：", book_remind[i], "图书可借数量：", book_lend[i])
                    print("-" * 150)
                    break

    # 用户是否归还图书
    def user_return_book():
        while True:
            print("归还图书")
            id = input("请输入图书编号：")
            for i in range(len(book_id)):
                if id == book_id[i]:
                    # 将库存从字符串转换为整数
                    lend_int = int(book_lend[i]) if book_lend[i].isdigit() else 0
                    if lend_int > 0:  # 检查转换后的借出数量是否足够
                        book_remind[i] = str(int(book_remind[i]) + 1)
                        book_lend[i] = str(int(book_lend[i]) + 1)
                        print("图书归还 成功！")
                        print("图书编号：", book_id[i], "图书名称：", book_name[i], "图书作者：", book_author[i],
                              "图书出版社：",
                              book_publisher[i], "图书剩余数量：", book_remind[i], "图书可借数量：", book_lend[i])
                        # 归还后，生成一个归还时间
                        return_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                        print("归还时间：", return_time)
                        print("-" * 150)
                        break
                    else:
                        print("图书归还失败！图书未被借阅！")
                        break
            print("是否继续归还图书？y/n")
            choice = input()
            if choice == 'n':
                break

    # 查看用户借阅状态
    def user_borrow_status():
        while True:
            print("查看借阅状态")
            id = input("请输入图书编号：")
            for i in range(len(book_id)):
                if id == book_id[i]:
                    lend_int = int(book_lend[i]) if book_lend[i].isdigit() else 0
                    if lend_int > 0:  # 检查图书是否已被借出
                        print("图书编号：", book_id[i], "图书名称：", book_name[i], "图书作者：", book_author[i],
                              "图书出版社：",
                              book_publisher[i], "图书剩余数量：", book_remind[i], "图书已借数量：", book_lend[i])
                        # 借出状态，生成一个借出时间
                        borrow_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                        print("借出时间：", borrow_time)
                        print("-" * 150)
                        break
                    else:
                        print("图书未被借阅！")
                        break
            print("是否继续查看借阅状态？y/n")
            choice = input()
            if choice == 'n':
                break

    def user_renew_book():
        while True:
            print("续借图书")
            id = input("请输入图书编号：")
            for i in range(len(book_id)):
                if id == book_id[i]:
                    # 将库存从字符串转换为整数
                    lend_int = int(book_lend[i]) if book_lend[i].isdigit() else 0
                    if lend_int > 0:  # 检查转换后的借出数量是否足够
                        book_remind[i] = str(int(book_remind[i]))
                        book_lend[i] = str(int(book_lend[i]))
                        print("图书续借 成功！")
                        print("图书编号：", book_id[i], "图书名称：", book_name[i], "图书作者：", book_author[i],
                              "图书出版社：",
                              book_publisher[i], "图书剩余数量：", book_remind[i], "图书可借数量：", book_lend[i])
                        # 续借后，生成一个续借时间
                        renew_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                        print("续借时间：", renew_time)
                        print("-" * 150)
                        break
                    else:
                        print("图书续借失败！图书已经被借阅！")
                        break

            print("是否继续续借图书？y/n")
            choice = input()
            if choice == 'n':
                break

    def main():
        search_options = {
            "1": user_search_by_type,
            "2": user_search_by_name,
            "3": user_search_by_author,
            "4": user_search_by_id,
        }

        while True:
            print("请选择查询方式:")
            print("1. 图书类型")
            print("2. 图书名称")
            print("3. 图书作者")
            print("4. 图书编号")
            print("5. 退出")
            option = input("请输入选项：")

            if option in search_options:
                search_options[option]()
            elif option == "5":
                print("已退出查找程序")
                break
            else:
                print("无效的选项，请重新输入")

    def main_b():
        user_borrow_lend_options = {
            "1": user_borrow_book,
            "2": user_return_book,
            "3": user_renew_book,
            "4": user_borrow_status
        }
        while True:
            print("请选择借阅/归还图书/续借图书方式:")
            print("1. 借阅图书")
            print("2. 归还图书")
            print("3. 续借图书")
            print("4. 查看借阅状态")
            print("5. 退出")
            option = input("请输入选项：")

            if option in user_borrow_lend_options:
                user_borrow_lend_options[option]()
            elif option == "5":
                print("已退出借阅/归还程序")
                break
            else:
                print("无效的选项，请重新输入")

    # 将主程序放在最后
    if __name__ == "__main__":
        main()
    if __name__ == "__main__":
        main_b()


# 调用登录函数开始程序
login()

# 关闭连接
cursor.close()
conn.close()

