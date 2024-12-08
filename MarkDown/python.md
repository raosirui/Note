#### **1、文本文件 data.txt中保存一批实数，读文件，统计数据个数、最大数、最小数和平均值。**

```python
def process_data_file(file_path):
    try:
        # 读取文件内容
        with open(file_path, 'r') as file:
            # 将内容按空格或换行符分割，并转换为浮点数列表
            numbers = [float(num) for num in file.read().split()]
        
        # 统计相关信息
        count = len(numbers)
        max_num = max(numbers)
        min_num = min(numbers)
        avg_num = sum(numbers) / count if count > 0 else 0
        
        # 打印结果
        print(f"数据个数: {count}")
        print(f"最大数: {max_num}")
        print(f"最小数: {min_num}")
        print(f"平均值: {avg_num:.2f}")
        
    except FileNotFoundError:
        print(f"文件 {file_path} 未找到，请检查路径！")
    except ValueError:
        print("文件内容包含无法转换为数字的项，请检查数据格式！")

# 调用函数并传入文件路径
process_data_file('data.txt')
```





**2、从键盘中输入一批数据，以-9999 结束，统计数据个数、平均值，最大值、最小值，并将这些数据和统计结果写入文件 result.txt。**

```python
def process_input_and_write_to_file():
    try:
        # 提示用户输入数据
        print("请输入一批实数数据，以 -9999 结束：")
        data = []
        
        # 循环输入数据
        while True:
            user_input = input()
            try:
                num = float(user_input)
                if num == -9999:  # 结束条件
                    break
                data.append(num)
            except ValueError:
                print("无效输入，请输入实数或 -9999 结束。")
        
        # 统计信息
        count = len(data)
        max_num = max(data) if count > 0 else None
        min_num = min(data) if count > 0 else None
        avg_num = sum(data) / count if count > 0 else None
        
        # 将数据和统计结果写入文件
        with open("result.txt", "w") as file:
            file.write("输入数据：\n")
            file.write(" ".join(map(str, data)) + "\n\n")
            file.write(f"数据个数: {count}\n")
            file.write(f"最大值: {max_num}\n")
            file.write(f"最小值: {min_num}\n")
            file.write(f"平均值: {avg_num:.2f}" if avg_num is not None else "平均值: N/A")
        
        # 打印结果到控制台
        print("\n统计结果：")
        print(f"数据个数: {count}")
        print(f"最大值: {max_num}")
        print(f"最小值: {min_num}")
        print(f"平均值: {avg_num:.2f}" if avg_num is not None else "平均值: N/A")
        print("统计结果已保存至 result.txt 文件。")
        
    except Exception as e:
        print(f"程序发生错误：{e}")

# 调用函数
process_input_and_write_to_file()

```





**3、编写程序 file_read.py，读取文本文件并输出文件内容到控制台。文件名由命令行第一个参数给出。此程序的执行效果如下:**

**C:>python file read.py data.txt**

```python
import sys

def main():
    # 检查是否提供了命令行参数
    if len(sys.argv) < 2:
        print("用法: python file_read.py <文件名>")
        return
    
    # 获取文件名
    file_name = sys.argv[1]
    
    try:
        # 读取文件内容并输出到控制台
        with open(file_name, 'r') as file:
            content = file.read()
            print("文件内容如下：")
            print(content)
    except FileNotFoundError:
        print(f"错误: 文件 '{file_name}' 不存在，请检查文件名或路径！")
    except Exception as e:
        print(f"读取文件时发生错误: {e}")

if __name__ == "__main__":
    main()

```









**4、定义一个学生类。有下面的类属性:1 姓名;2 年龄;3 成绩(语文，数学英语)[每课成绩的类型为整数]。类方法:**

**1获取学生的姓名:get_name()返回类型:str;**

**2 获取学生的年龄:get_age()返回类型:int;**

**3返回3门科目中最高的分数。get_course()返回类型:int;**

**写好类以后，可以定义2个同学进行测试**
**wh = Student('林夕',20,[69,88,100])**
**zk= Student('李明',19,[99,86,98])**

```python
class Student:
    def __init__(self, name: str, age: int, scores: list[int]):

        self.name = name
        self.age = age
        self.scores = scores

    def get_name(self) -> str:
        """获取学生姓名"""
        return self.name

    def get_age(self) -> int:
        """获取学生年龄"""
        return self.age

    def get_course(self) -> int:
        """获取3门科目中最高的分数"""
        return max(self.scores)

# 测试代码
if __name__ == "__main__":
    # 定义两个学生
    wh = Student('林夕', 20, [69, 88, 100])
    zk = Student('李明', 19, [99, 86, 98])
    
    # 测试第一个学生
    print(f"学生姓名: {wh.get_name()}")
    print(f"学生年龄: {wh.get_age()}")
    print(f"最高分: {wh.get_course()}")

    print()  # 分隔行

    # 测试第二个学生
    print(f"学生姓名: {zk.get_name()}")
    print(f"学生年龄: {zk.get_age()}")
    print(f"最高分: {zk.get_course()}")

```







**5、自定义一个成绩计算类，计算课程 python、C、SQL三门课程的总成绩和平均成绩。某位同学python、C、SQL三门课程成绩为98、88 和 84 分，打印三门课程总SQL 三门课程及平均成绩。**

```python
class GradeCalculator:
    def __init__(self, python: int, c: int, sql: int):
        """
        初始化成绩计算类。
        :param python: Python 课程成绩
        :param c: C 课程成绩
        :param sql: SQL 课程成绩
        """
        self.python = python
        self.c = c
        self.sql = sql

    def calculate_total(self) -> int:
        """计算总成绩"""
        return self.python + self.c + self.sql

    def calculate_average(self) -> float:
        """计算平均成绩"""
        return self.calculate_total() / 3

# 测试代码
if __name__ == "__main__":
    # 某位同学的成绩
    student = GradeCalculator(python=98, c=88, sql=84)
    
    # 计算总成绩和平均成绩
    total = student.calculate_total()
    average = student.calculate_average()

    # 打印结果
    print(f"Python、C 和 SQL 三门课程的总成绩: {total}")
    print(f"Python、C 和 SQL 三门课程的平均成绩: {average:.2f}")

```







**6、自定义一个成绩计算类，计算课程python、C、SQL三门课程的总成绩和平均成绩。某位同学python、C、SQL三门课程成绩为98、88和84分，打印三门课程总成绩及平均成绩。**

```python
class GradeCalculator:
    def __init__(self, python: int, c: int, sql: int):
        """
        初始化成绩计算类。
        :param python: Python 课程成绩
        :param c: C 课程成绩
        :param sql: SQL 课程成绩
        """
        self.python = python
        self.c = c
        self.sql = sql

    def calculate_total(self) -> int:
        """计算总成绩"""
        return self.python + self.c + self.sql

    def calculate_average(self) -> float:
        """计算平均成绩"""
        return self.calculate_total() / 3

# 测试代码
if __name__ == "__main__":
    # 某位同学的成绩
    student = GradeCalculator(python=98, c=88, sql=84)
    
    # 计算总成绩和平均成绩
    total = student.calculate_total()
    average = student.calculate_average()

    # 打印结果
    print(f"Python、C 和 SQL 三门课程的总成绩: {total}")
    print(f"Python、C 和 SQL 三门课程的平均成绩: {average:.2f}")

```









**7、计算机中已经安装了MySQL并创建了数据库MyDB_01，数据库的基本信息为：host="localhost", port=3306, charset="utf8", user="root", passwd="123456", database="MyDB_01"，数据库MyDB_01中已经创建了数据表user，其中表中字段包括uid, name, age, birthday, salary, note，请用Python编程实现向该数据表中插入一条数据(0009,'李明_01', 50,'1978-09-15', 19000.0,'python老师 1号')，并显示数据库更新影响的数据行数，程序中要有异常处理机制。**

```python
import pymysql

def insert_user():
    # 数据库连接信息
    db_config = {
        "host": "localhost",
        "port": 3306,
        "user": "root",
        "password": "123456",
        "database": "MyDB_01",
        "charset": "utf8"
    }

    # 插入数据的 SQL 语句
    insert_query = """
    INSERT INTO user (uid, name, age, birthday, salary, note)
    VALUES (%s, %s, %s, %s, %s, %s)
    """
    data = ('0009', '李明_01', 50, '1978-09-15', 19000.0, 'python老师 1号')

    try:
        # 连接数据库
        connection = pymysql.connect(**db_config)
        with connection.cursor() as cursor:
            # 执行插入操作
            rows_affected = cursor.execute(insert_query, data)
            # 提交事务
            connection.commit()
            print(f"插入成功！影响的数据行数: {rows_affected}")
    except pymysql.MySQLError as e:
        # 捕获数据库相关的异常
        print(f"数据库操作失败: {e}")
    except Exception as e:
        # 捕获其他异常
        print(f"发生错误: {e}")
    finally:
        # 确保连接关闭
        if 'connection' in locals() and connection:
            connection.close()

# 调用函数
insert_user()

```









**8、基于套接字（socket）实现TCP通信程序的服务器端和客户机端的Python程序设计。其中服务器端的IP地址为：172.26.3.166，端口为8000。要求服务器端接收到客户机端的数据后，将该数据回送到客户机端，接收到空数据时停止通信。（其中实例化一个socket对象时，参数AF_INET表示该socket网络成使用IP协议，参数SOCK_STREAM表示该socket传输层使用TCP协议）**



**服务端代码：**

```python
import socket

def start_server():
    server_ip = "172.26.3.166"
    server_port = 8000

    # 创建一个 TCP 套接字
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as server_socket:
        # 绑定 IP 和端口
        server_socket.bind((server_ip, server_port))
        # 启动监听
        server_socket.listen(5)
        print(f"服务器已启动，正在监听 {server_ip}:{server_port} ...")

        # 等待客户端连接
        while True:
            client_socket, client_address = server_socket.accept()
            print(f"客户端已连接: {client_address}")

            # 接收和发送数据
            with client_socket:
                while True:
                    data = client_socket.recv(1024)  # 每次接收最大 1024 字节
                    if not data:  # 如果收到空数据，结束通信
                        print("接收到空数据，关闭连接。")
                        break
                    print(f"收到来自客户端的数据: {data.decode()}")
                    client_socket.sendall(data)  # 将数据回送给客户端

if __name__ == "__main__":
    start_server()

```

**客户端代码：**

```python
import socket

def start_client():
    server_ip = "172.26.3.166"
    server_port = 8000

    # 创建一个 TCP 套接字
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client_socket:
        # 连接到服务器
        client_socket.connect((server_ip, server_port))
        print(f"已连接到服务器 {server_ip}:{server_port}")

        while True:
            # 从用户输入获取数据
            message = input("请输入发送给服务器的消息 (输入空行退出): ")
            if not message:  # 如果输入为空，则退出
                print("发送空数据，关闭连接。")
                break
            client_socket.sendall(message.encode())  # 发送数据
            data = client_socket.recv(1024)  # 接收服务器回送的数据
            print(f"收到来自服务器的回送数据: {data.decode()}")

if __name__ == "__main__":
    start_client()

```







**9、自定义一个timer函数，参数为interval，输出3次当前线程名及时间，每次随机睡眠interval秒。设计Python程序，主函数调用time函数2次，参数值分别为5和6，计算单线程串行执行的时间耗费，接着计算多线程并行执行的时间耗费。（线程threading的get_ident()方法可以获取当前线程的标识符）**

```python
import threading
import time
import random

def timer(interval):
    """自定义 timer 函数，输出当前线程名及时间，随机睡眠 interval 秒"""
    thread_name = threading.current_thread().name
    for i in range(3):
        print(f"线程 {thread_name} | 时间: {time.strftime('%Y-%m-%d %H:%M:%S')} | 第 {i+1} 次")
        sleep_time = random.uniform(1, interval)  # 随机睡眠时间
        time.sleep(sleep_time)

def main():
    # 记录单线程串行执行时间
    start_time = time.time()
    print("开始单线程串行执行...")
    timer(5)
    timer(6)
    end_time = time.time()
    print(f"单线程串行执行时间耗费: {end_time - start_time:.2f} 秒\n")

    # 记录多线程并行执行时间
    start_time = time.time()
    print("开始多线程并行执行...")
    thread1 = threading.Thread(target=timer, args=(5,), name="Timer-5")
    thread2 = threading.Thread(target=timer, args=(6,), name="Timer-6")
    thread1.start()
    thread2.start()
    thread1.join()
    thread2.join()
    end_time = time.time()
    print(f"多线程并行执行时间耗费: {end_time - start_time:.2f} 秒")

if __name__ == "__main__":
    main()

```









