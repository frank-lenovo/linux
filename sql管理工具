import os
import shutil
import tkinter as tk
from tkinter import filedialog, scrolledtext, messagebox
import random
import hashlib
import logging

# 设置日志记录
logging.basicConfig(filename='app.log', level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

def file_hash(filepath):
    """计算文件的 MD5 哈希值"""
    if not os.path.isfile(filepath):  # 检查文件是否存在
        logging.error(f"File not found: {filepath}")
        return None
    hash_md5 = hashlib.md5()
    with open(filepath, "rb") as f:
        for chunk in iter(lambda: f.read(4096), b""):
            hash_md5.update(chunk)
    return hash_md5.hexdigest()

def read_keywords(file_path):
    """读取关键字文件内容"""
    if not os.path.isfile(file_path):  # 检查文件是否存在
        logging.error(f"Keyword file not found: {file_path}")
        return []
    try:
        with open(file_path, 'r') as file:
            return [line.strip().lower() for line in file]
    except FileNotFoundError:
        logging.error(f"File not found: {file_path}")
        return []

def create_directories(root_dir, subdir_names):
    """创建目录和提示消息"""
    directories_created = False
    if not os.path.exists(root_dir):
        os.makedirs(root_dir)
        directories_created = True
    for subdir in subdir_names:
        subdir_path = os.path.join(root_dir, subdir)
        if not os.path.exists(subdir_path):
            os.makedirs(subdir_path)
            directories_created = True
    if directories_created:
        messagebox.showinfo("提示", "一些目录不存在，已创建。")

def copy_complete_message(copied_files_count, duplicate_files_count):
    """显示复制完成的消息"""
    message = "复制完成:\n"
    message += f"共找到 SQL 文件: {sum(copied_files_count.values()) + duplicate_files_count} 个\n"
    for category, count in copied_files_count.items():
        message += f"类别 {category} 中复制的文件数量: {count}\n"
    message += f"重复文件数量: {duplicate_files_count}"

    # 在文本输出区域显示结果
    text_output.insert(tk.END, message + "\n")

    # 显示提示框内容
    messagebox.showinfo("复制完成", message)

def browse_directory():
    """使用文件对话框选择根目录"""
    selected_dir = filedialog.askdirectory()
    if selected_dir:
        entry_root_dir.delete(0, tk.END)
        entry_root_dir.insert(0, selected_dir)


def run_script():
    """运行脚本的主要逻辑"""
    try:
        # 获取界面输入的路径
        root_dir = entry_root_dir.get()
        a_class_folder = entry_a_class.get()
        b_class_folder = entry_b_class.get()
        c_class_folder = entry_c_class.get()
        o_class_folder = entry_o_class.get()

        # 创建不存在的目录
        create_directories(root_dir, [a_class_folder, b_class_folder, c_class_folder, o_class_folder])

        target_subdir_names = entry_subdir_names.get().lower().split(",")  # 子目录关键字列表
        exclude_keywords = [kw for kw in entry_exclude_keyword.get().lower().split(",") if kw]

        # 读取关键字
        keywords_a = read_keywords(os.path.join(root_dir, 'keywords_a.txt'))
        keywords_b = read_keywords(os.path.join(root_dir, 'keywords_b.txt'))
        keywords_c = read_keywords(os.path.join(root_dir, 'keywords_c.txt'))

        #text_output.delete(1.0, tk.END)  # 清空输出区域

        copied_files_count = {"A": 0, "B": 0, "C": 0, "O": 0}
        duplicate_files_count = 0

        # 遍历目录复制文件
        for root, dirs, files in os.walk(root_dir):
            # 检查当前目录是否包含子目录关键字
            if not any(subdir_keyword in root.lower() for subdir_keyword in target_subdir_names):
                continue  # 如果不包含，则跳过当前目录

            for file in files:
                file_path = os.path.join(root, file)
                if file_path.endswith('.sql'):
                    file_lower = file_path.lower()

                    # 检查是否打印每个找到的 SQL 文件的明细
                    if var_print_details.get():
                        text_output.insert(tk.END, f"找到 SQL 文件: {file_path}\n")

                    if any(exclude_keyword in file_lower for exclude_keyword in exclude_keywords):
                        # 如果文件包含排除关键字，则根据设置打印信息并跳过处理
                        if var_print_exclude_details.get():
                            text_output.insert(tk.END, f"文件被排除（含排除关键字）: {file_path}\n")
                        continue

                    if not any(exclude_keyword in file_lower for exclude_keyword in exclude_keywords):
                        for category, keywords in [("A", keywords_a), ("B", keywords_b), ("C", keywords_c)]:
                            if any(keyword in file_lower for keyword in keywords):
                                target_folder = eval(f"{category.lower()}_class_folder")
                                target_file_path = os.path.join(target_folder, os.path.basename(file))
                                if not os.path.exists(target_file_path):
                                    shutil.copy(file_path, target_file_path)
                                    copied_files_count[category] += 1
                                    text_output.insert(tk.END, f"复制到 {category} 类: {target_file_path}\n")
                                else:
                                    text_output.insert(tk.END, f"文件已存在，跳过: {target_file_path}\n")
                                break
                        else:  # 不符合任何关键字，复制到O类目录
                            target_folder = o_class_folder
                            target_file_path = os.path.join(target_folder, os.path.basename(file))
                            if not os.path.exists(target_file_path):
                                shutil.copy(file_path, target_file_path)
                                copied_files_count["O"] += 1
                                text_output.insert(tk.END, f"复制到 O 类: {target_file_path}\n")
                            else:
                                text_output.insert(tk.END, f"文件已存在，跳过: {target_file_path}\n")

        copy_complete_message(copied_files_count, duplicate_files_count)

    except Exception as e:
        messagebox.showerror("运行错误", f"运行脚本时出现错误: {e}")
        logging.error(f"运行脚本时出现错误: {e}")


def clear_all():
    """清除所有输入框和输出区域"""
    text_output.delete(1.0, tk.END)

def view_log():
    """打开日志文件"""
    os.system("notepad.exe app.log")  # 在 Windows 上打开日志文件

# 创建主窗口
root = tk.Tk()
root.title("SQL 文件管理器（keywords文件默认在根目录）示例：keywords_a.txt")
root.geometry("800x600")  # 设置窗口初始大小

# 创建输入框和标签
entry_root_dir = tk.Entry(root, width=50)
entry_root_dir.insert(0, 'C:/Users/yx2.0/Destop/tmp/')
entry_subdir_names = tk.Entry(root, width=50)
entry_subdir_names.insert(0, "全网,青海")
entry_exclude_keyword = tk.Entry(root, width=50)
entry_exclude_keyword.insert(0, "回滚,回退")
entry_a_class = tk.Entry(root, width=50)
entry_a_class.insert(0, 'C:/Users/yx2.0/Desktop/tmp/1')
entry_b_class = tk.Entry(root, width=50)
entry_b_class.insert(0, 'C:/Users/yx2.0/Desktop/tmp/2')
entry_c_class = tk.Entry(root, width=50)
entry_c_class.insert(0, 'C:/Users/yx2.0/Desktop/tmp/3')
entry_o_class = tk.Entry(root, width=50)
entry_o_class.insert(0, 'C:/Users/yx2.0/Desktop/tmp/o')

# 布局
labels = ["根目录:", "子目录（省份）:", "排除关键字:", "A类目录:", "B类目录:", "C类目录:", "O类目录:"]
entries = [entry_root_dir, entry_subdir_names, entry_exclude_keyword, entry_a_class, entry_b_class, entry_c_class, entry_o_class]
for i, (label, entry) in enumerate(zip(labels, entries)):
    tk.Label(root, text=label).grid(row=i, column=0, sticky="e")
    entry.grid(row=i, column=1)

# 添加浏览按钮
browse_button = tk.Button(root, text="浏览", command=browse_directory)
browse_button.grid(row=0, column=2)

# 设置是否打印明细复选框
var_print_details = tk.BooleanVar(value=False)
check_print_details = tk.Checkbutton(root, text="打印每个找到的 SQL 文件的明细", variable=var_print_details)
check_print_details.grid(row=7, column=1, sticky="w")

# 设置是否打印包含排除关键字的文件信息
var_print_exclude_details = tk.BooleanVar(value=False)
check_print_exclude_details = tk.Checkbutton(root, text="打印包含排除关键字的文件信息", variable=var_print_exclude_details)
check_print_exclude_details.grid(row=8, column=1, sticky="w")

# 运行按钮
run_button = tk.Button(root, text="运行", command=run_script)
run_button.grid(row=8, column=2)

# 清除按钮
clear_button = tk.Button(root, text="清除", command=clear_all)
clear_button.grid(row=8, column=3)

# 查看日志按钮
log_button = tk.Button(root, text="查看日志", command=view_log)
log_button.grid(row=8, column=4)

# 输出区域
text_output = scrolledtext.ScrolledText(root, width=100, height=20)
text_output.grid(row=9, column=0, columnspan=4, padx=5, pady=5, sticky="nsew")

# 配置行和列的权重
root.grid_rowconfigure(9, weight=1)
root.grid_columnconfigure(0, weight=1)

root.mainloop()
