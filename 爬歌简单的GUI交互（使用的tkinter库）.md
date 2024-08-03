# 爬虫GUI程序更新迭代过程（使用的tkinter库）

------

## 榕哥的练习代码

```python
import requests
import re
import base64
import os
import urllib.parse

# 起始页码是1
# index = 1
# 起始爬取的网页
start_url = 'https://hifini.com/index-'
# + str(index) + ".htm"

# 拼接的网页
base_url = "https://hifini.com/"
# 伪装
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36',
    'Cookie': 'bbs_sid=vv2trfih02c6nsmtnjg8ajksh2'
}


# 获得列表页数据
def get_data(url):
    r = requests.get(url, headers=headers)
    if r.status_code == 200:
        data = r.text

        # print("请求成功")
        # print(data)
        return data
    else:
        print("请求失败!")


# 获取详情页数据
def get_song_data(song_url):
    return get_data(song_url)


# 解析数据
def parse_data(data):
    # 解析歌曲名字和链接
    z1 = '<li\sclass="media\sthread\stap\s\s".*?<div\sclass="subject\sbreak-all">.*?<a\shref="(.*?)">(.*?)</a>'
    result = re.findall(z1, data, re.S)
    songs_dict = {}
    # 解析列表有多少页
    parse_page = '<a\s+href="index-.*?>\.\.\.(\d+)</a>'
    # 获取到了所有页码
    page = re.search(parse_page, data, re.S)
    # print(page.group(1))
    # x = 0
    # # 获取多少个歌曲
    # max_song = 15
    if result:
        for i in result:
            # print(i)
            # if x >= max_song:
            #     break
            song_url = base_url + i[0]
            name = i[1]
            # print(song_url, name)
            # 调用函数 获得详情页信息
            song_data = get_song_data(song_url)
            if song_data:
                # print(song_data)
                print("歌曲详情页捕获成功")
                z2 = "music:\s\[.*?url:\s'(.*?)'\s\+\sgenerateParam\('(.*?)'"
                song_result = re.findall(z2, song_data, re.S)
                # [0]

                # x += 1
                if song_result:
                    print('歌曲链接捕获成功')
                    for j in song_result:
                        song_first_link = j[0]
                        p = generate_param(j[1])
                        song_all_link = base_url + song_first_link + p
                        print(f"歌曲名称为：{name}")
                        print("链接全称为", song_all_link)
                        song = requests.get(song_all_link, headers=headers)
                        print("去往歌曲链接的请求状态:", song.status_code)
                        song_data_bytes = song.content
                        # print(song_data_bytes)
                        songs_dict[name] = song_data_bytes
                else:
                    print(300 * "=")
                    print("Error：歌曲链接捕获失败！")
                    print(f"详情页链接为：{song_url}")
                    print(300 * "=")

    else:
        print("解析数据失败 ")

    return songs_dict


# 保存数据
def save_data(songs_dict):
    if not os.path.exists("周杰伦歌曲"):
        os.makedirs("周杰伦歌曲")
    for name, song_data_bytes in songs_dict.items():
        print(f"存储歌曲名称: {name}")  # 键名
        # print(f"歌曲数据: {song_data_bytes}")  # 键值
        # print(type(name))
        song_name = re.sub('[\/:*?<>|]', "-", name)
        with open('周杰伦歌曲\{}.m4a'.format(song_name), 'wb') as f:
            f.write(song_data_bytes)
        print("存储成功")


def get_page_data(url, start_page, end_page):
    if 'search' not in url:
        for i in range(start_page, end_page+1):
            all_url = url + str(i) + ".htm"
            # print(all_url)
            print("抓取的页码为：", i)
            data = get_data(all_url)
            songs_dict = parse_data(data)
            all_keys = songs_dict.keys()
            save_data(songs_dict)
            print("所有歌曲名称:", list(all_keys))

    else:
        for i in range(start_page, end_page+1):
            all_url = url + "1-" + str(i) + ".htm"
            # print(all_url)
            print("抓取的页码为：", i)
            data = get_data(all_url)
            songs_dict = parse_data(data)
            all_keys = songs_dict.keys()
            save_data(songs_dict)
            print("所有歌曲名称:", list(all_keys))


def get_search_data(content):
    url_part1 = escape_chinese(content)
    # 拼接网址
    url = base_url + "search-" + url_part1 + "-"
    print(url)
    return url


'''
下面的函数：
    1.对网站加密的处理
    2.对字符转义的处理
'''


# 1.解密处理
def xor_encrypt(data, key):
    out_text = ''
    j = 0
    for i in range(len(data)):
        if j == len(key):
            j = 0
        out_text += chr(ord(data[i]) ^ ord(key[j]))
        j += 1
    return out_text


def base32_encode(data):
    return base64.b32encode(data.encode()).decode()


def generate_param(data):
    key = '95wwwHiFiNicom27'
    out_text = xor_encrypt(data, key)
    return base32_encode(out_text)


# 对搜索内容的处理
def escape_chinese(text):
    result = []
    for char in text:
        if '\u4e00' <= char <= '\u9fff':  # 检查是否是汉字
            # 将汉字字符转换为 URL 编码形式
            result.append(urllib.parse.quote(char.encode('utf-8')))
        else:
            # 英文字符直接添加
            result.append(char)

    result = ''.join(result)
    result = re.sub('%', '_', result)
    return result


if __name__ == '__main__':
    #     get_page_data(start_url, 2, 5)
    #     content1 = "周杰伦"
    #     content2 = "star"
    #     result1 = escape_chinese(content1)
    #     print(result1)
    search_content = "周杰伦"
    url = get_search_data(search_content)
    get_page_data(url, 1, 2)
```

## 初版实现

初始版本主要实现了基本的歌曲爬取功能，核心代码如下：

```python
import tkinter as tk
from tkinter import ttk, messagebox
import requests
import re
import base64
import os
import urllib.parse

# 起始页码是1
start_url = 'https://hifini.com/index-'
base_url = "https://hifini.com/"
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36',
    'Cookie': 'bbs_sid=vv2trfih02c6nsmtnjg8ajksh2'
}

def get_data(url):
    r = requests.get(url, headers=headers)
    if r.status_code == 200:
        return r.text
    else:
        print("请求失败!")

def get_song_data(song_url):
    return get_data(song_url)

def parse_data(data):
    z1 = '<li\sclass="media\sthread\stap\s\s".*?<div\sclass="subject\sbreak-all">.*?<a\shref="(.*?)">(.*?)</a>'
    result = re.findall(z1, data, re.S)
    songs_dict = {}
    parse_page = '<a\s+href="index-.*?>\.\.\.(\d+)</a>'
    page = re.search(parse_page, data, re.S)
    if result:
        for i in result:
            song_url = base_url + i[0]
            name = i[1]
            song_data = get_song_data(song_url)
            if song_data:
                z2 = "music:\s\[.*?url:\s'(.*?)'\s\+\sgenerateParam\('(.*?)'"
                song_result = re.findall(z2, song_data, re.S)
                if song_result:
                    for j in song_result:
                        song_first_link = j[0]
                        p = generate_param(j[1])
                        song_all_link = base_url + song_first_link + p
                        song = requests.get(song_all_link, headers=headers)
                        song_data_bytes = song.content
                        songs_dict[name] = song_data_bytes
                else:
                    print("Error：歌曲链接捕获失败！")
                    print(f"详情页链接为：{song_url}")
    else:
        print("解析数据失败 ")
    return songs_dict

def save_data(songs_dict):
    if not os.path.exists("周杰伦歌曲"):
        os.makedirs("周杰伦歌曲")
    for name, song_data_bytes in songs_dict.items():
        song_name = re.sub('[\/:*?<>|]', "-", name)
        with open(f'周杰伦歌曲/{song_name}.mp4', 'wb') as f:
            f.write(song_data_bytes)
        print("存储成功")

def get_page_data(url, start_page, end_page):
    for i in range(start_page, end_page+1):
        all_url = url + str(i) + ".htm"
        data = get_data(all_url)
        songs_dict = parse_data(data)
        save_data(songs_dict)
        print("所有歌曲名称:", list(songs_dict.keys()))

def get_search_data(content):
    url_part1 = escape_chinese(content)
    url = base_url + "search-" + url_part1 + "-"
    return url

def xor_encrypt(data, key):
    out_text = ''
    j = 0
    for i in range(len(data)):
        if j == len(key):
            j = 0
        out_text += chr(ord(data[i]) ^ ord(key[j]))
        j += 1
    return out_text

def base32_encode(data):
    return base64.b32encode(data.encode()).decode()

def generate_param(data):
    key = '95wwwHiFiNicom27'
    out_text = xor_encrypt(data, key)
    return base32_encode(out_text)

def escape_chinese(text):
    result = []
    for char in text:
        if '\u4e00' <= char <= '\u9fff':  # 检查是否是汉字
            result.append(urllib.parse.quote(char.encode('utf-8')))
        else:
            result.append(char)
    result = ''.join(result)
    result = re.sub('%', '_', result)
    return result

# 创建GUI
class App:
    def __init__(self, root):
        self.root = root
        self.root.title("歌曲爬虫")
        self.create_widgets()

    def create_widgets(self):
        self.label = tk.Label(self.root, text="输入歌手名称:")
        self.label.pack(pady=5)

        self.entry = tk.Entry(self.root, width=30)
        self.entry.pack(pady=5)

        self.mode_label = tk.Label(self.root, text="选择模式:")
        self.mode_label.pack(pady=5)

        self.mode = ttk.Combobox(self.root, values=["歌手模式", "随机模式"])
        self.mode.pack(pady=5)
        self.mode.current(0)

        self.start_button = tk.Button(self.root, text="开始爬取", command=self.start_scraping)
        self.start_button.pack(pady=20)

    def start_scraping(self):
        singer = self.entry.get()
        mode = self.mode.get()
        if singer:
            url = get_search_data(singer)
            if mode == "歌手模式":
                get_page_data(url, 1, 2)
                messagebox.showinfo("完成", "歌手模式爬取完成！")
            elif mode == "随机模式":
                get_page_data(start_url, 1, 2)
                messagebox.showinfo("完成", "随机模式爬取完成！")
        else:
            messagebox.showwarning("警告", "请输入歌手名称！")

if __name__ == "__main__":
    root = tk.Tk()
    app = App(root)
    root.mainloop()

```

## 第二版改进

在第二个版本中，添加了更多的错误处理和日志功能，同时优化了界面布局。

```python
import tkinter as tk
from tkinter import ttk, messagebox
import requests
import re
import base64
import os
import urllib.parse

# 起始页码是1
start_url = 'https://hifini.com/index-'
base_url = "https://hifini.com/"
headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36',
    'Cookie': 'bbs_sid=vv2trfih02c6nsmtnjg8ajksh2'
}

def get_data(url):
    r = requests.get(url, headers=headers)
    if r.status_code == 200:
        return r.text
    else:
        return None

def get_song_data(song_url):
    return get_data(song_url)

def parse_data(data, app):
    z1 = '<li\sclass="media\sthread\stap\s\s".*?<div\sclass="subject\sbreak-all">.*?<a\shref="(.*?)">(.*?)</a>'
    result = re.findall(z1, data, re.S)
    songs_dict = {}
    if result:
        for i in result:
            song_url = base_url + i[0]
            name = i[1]
            song_data = get_song_data(song_url)
            if song_data:
                z2 = "music:\s\[.*?url:\s'(.*?)'\s\+\sgenerateParam\('(.*?)'"
                song_result = re.findall(z2, song_data, re.S)
                if song_result:
                    for j in song_result:
                        song_first_link = j[0]
                        p = generate_param(j[1])
                        song_all_link = base_url + song_first_link + p
                        song = requests.get(song_all_link, headers=headers)
                        song_data_bytes = song.content
                        songs_dict[name] = song_data_bytes
                        app.update_log(f"捕获到歌曲: {name}")
                else:
                    app.update_log(f"Error：歌曲链接捕获失败！详情页链接为：{song_url}")
            else:
                app.update_log(f"Error：无法获取歌曲详情页数据！详情页链接为：{song_url}")
    else:
        app.update_log("解析数据失败")
    return songs_dict

def save_data(songs_dict, app):
    if not os.path.exists("周杰伦歌曲"):
        os.makedirs("周杰伦歌曲")
    for name, song_data_bytes in songs_dict.items():
        song_name = re.sub('[\/:*?<>|]', "-", name)
        with open(f'周杰伦歌曲/{song_name}.mp4', 'wb') as f:
            f.write(song_data_bytes)
        app.update_log(f"存储成功: {name}")

def get_page_data(url, start_page, end_page, app):
    for i in range(start_page, end_page + 1):
        all_url = url + str(i) + ".htm"
        data = get_data(all_url)
        if data:
            songs_dict = parse_data(data, app)
            save_data(songs_dict, app)
        else:
            app.update_log(f"请求失败：{all_url}")

def get_search_data(content):
    url_part1 = escape_chinese(content)
    return base_url + "search-" + url_part1 + "-"

def xor_encrypt(data, key):
    out_text = ''
    j = 0
    for i in range(len(data)):
        if j == len(key):
            j = 0
        out_text += chr(ord(data[i]) ^ ord(key[j]))
        j += 1
    return out_text

def base32_encode(data):
    return base64.b32encode(data.encode()).decode()

def generate_param(data):
    key = '95wwwHiFiNicom27'
    out_text = xor_encrypt(data, key)
    return base32_encode(out_text)

def escape_chinese(text):
    result = []
    for char in text:
        if '\u4e00' <= char <= '\u9fff':  # 检查是否是汉字
            result.append(urllib.parse.quote(char.encode('utf-8')))
        else:
            result.append(char)
    result = ''.join(result)
    result = re.sub('%', '_', result)
    return result

# 创建GUI
class App:
    def __init__(self, root):
        self.root = root
        self.root.title("歌曲爬虫")
        self.create_widgets()

    def create_widgets(self):
        self.label = tk.Label(self.root, text="输入歌手名称:")
        self.label.pack(pady=5)

        self.entry = tk.Entry(self.root, width=30)
        self.entry.pack(pady=5)

        self.mode_label = tk.Label(self.root, text="选择模式:")
        self.mode_label.pack(pady=5)

        self.mode = ttk.Combobox(self.root, values=["歌手模式", "随机模式"])
        self.mode.pack(pady=5)
        self.mode.current(0)

        self.start_button = tk.Button(self.root, text="开始爬取", command=self.start_scraping)
        self.start_button.pack(pady=20)

        self.log_text = tk.Text(self.root, height=15, width=60)
        self.log_text.pack(pady=10)

    def start_scraping(self):
        singer = self.entry.get()
        mode = self.mode.get()
        if singer:
            url = get_search_data(singer)
            if mode == "歌手模式":
                self.log_text.delete(1.0, tk.END)
                get_page_data(url, 1, 2, self)
                messagebox.showinfo("完成", "歌手模式爬取完成！")
            elif mode == "随机模式":
                self.log_text.delete(1.0, tk.END)
                get_page_data(start_url, 1, 2, self)
                messagebox.showinfo("完成", "随机模式爬取完成！")
        else:
            messagebox.showwarning("警告", "请输入歌手名称！")

    def update_log(self, message):
        self.log_text.insert(tk.END, message + '\n')
        self.log_text.see(tk.END)

if __name__ == "__main__":
    root = tk.Tk()
    app = App(root)
    root.mainloop()
```

## 最终版优化

最终版进一步优化了爬虫性能，增加了更多的日志信息，并简化了代码结构。

```python
import requests
import re
import base64
import os
import tkinter as tk
from tkinter import ttk, scrolledtext, messagebox
import urllib.parse

start_url = 'https://hifini.com'
base_url = "https://hifini.com/"
headers = {
    'user-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36',
    'cookie': 'bbs_sid=vv2trfih02c6nsmtnjg8ajksh2'
}


def get_data(url):
    r = requests.get(url, headers=headers)
    if r.status_code == 200:
        data = r.text
        log_message("请求成功")
        return data
    else:
        log_message("请求失败!")
        return None


def get_song_data(song_url):
    return get_data(song_url)


def parse_data(data):
    z1 = '<li\sclass="media\sthread\stap\s\s".*?<div\sclass="subject\sbreak-all">.*?<a\shref="(.*?)">(.*?)</a>'
    result = re.findall(z1, data, re.S)
    songs_dict = {}
    if result:
        for i in result:
            song_url = base_url + i[0]
            name = i[1]
            song_data = get_song_data(song_url)
            if song_data:
                z2 = "music:\s\[.*?url:\s'(.*?)'\s\+\sgenerateParam\('(.*?)'"
                song_result = re.findall(z2, song_data, re.S)
                if song_result:
                    for j in song_result:
                        song_first_link = j[0]
                        p = generate_param(j[1])
                        song_all_link = base_url + song_first_link + p
                        log_message(f"歌曲名称为：{name}")
                        log_message(f"链接全称为：{song_all_link}")
                        song = requests.get(song_all_link, headers=headers)
                        log_message(f"去往歌曲链接的请求状态: {song.status_code}")
                        song_data_bytes = song.content
                        songs_dict[name] = song_data_bytes
    else:
        log_message("解析数据失败")
    return songs_dict


def save_data(songs_dict, mode, singer_name=None):
    folder_name = "音乐/随机模式" if mode == "随机模式" else f"音乐/歌手模式/{singer_name}"
    if not os.path.exists(folder_name):
        os.makedirs(folder_name)
    for name, song_data_bytes in songs_dict.items():
        log_message(f"存储歌曲名称: {name}")
        song_name = re.sub('[\/:*?<>|]', "-", name)
        with open(f'{folder_name}/{song_name}.m4a', 'wb') as f:
            f.write(song_data_bytes)
        log_message("存储成功")


def xor_encrypt(data, key):
    out_text = ''
    j = 0
    for i in range(len(data)):
        if j == len(key):
            j = 0
        out_text += chr(ord(data[i]) ^ ord(key[j]))
        j += 1
    return out_text


def base32_encode(data):
    return base64.b32encode(data.encode()).decode()


def generate_param(data):
    key = '95wwwHiFiNicom27'
    out_text = xor_encrypt(data, key)
    return base32_encode(out_text)


def log_message(message):
    log_text.insert(tk.END, message + "\n")
    log_text.see(tk.END)


def get_search_data(content):
    url_part1 = escape_chinese(content)
    url = f"{base_url}search-{url_part1}-1.htm"
    return url


def escape_chinese(text):
    result = []
    for char in text:
        if '\u4e00' <= char <= '\u9fff':  # 检查是否是汉字
            result.append(urllib.parse.quote(char.encode('utf-8')))
        else:
            result.append(char)
    result = ''.join(result)
    result = re.sub('%', '_', result)
    return result


def start_scraping():
    mode = mode_var.get()
    if mode == "歌手模式":
        singer_name = singer_name_entry.get().strip()
        if not singer_name:
            messagebox.showwarning("输入错误", "请填写歌手名称")
            return
        search_url = get_search_data(singer_name)
    else:
        singer_name = None
        search_url = start_url

    log_text.delete(1.0, tk.END)

    data = get_data(search_url)
    if data:
        songs_dict = parse_data(data)
        save_data(songs_dict, mode, singer_name)


def on_mode_change(*args):
    if mode_var.get() == "歌手模式":
        singer_name_label.grid(row=1, column=0, padx=5, pady=5, sticky="w")
        singer_name_entry.grid(row=1, column=1, padx=5, pady=5, sticky="w")
    else:
        singer_name_label.grid_remove()
        singer_name_entry.grid_remove()


root = tk.Tk()
root.title("歌曲爬虫")

main_frame = ttk.Frame(root, padding="10")
main_frame.grid(row=0, column=0, sticky=(tk.W, tk.E, tk.N, tk.S))

mode_var = tk.StringVar()
mode_var.set("歌手模式")
mode_var.trace_add("write", on_mode_change)

mode_label = ttk.Label(main_frame, text="模式选择:")
mode_label.grid(row=0, column=0, padx=5, pady=5, sticky="w")

mode_combobox = ttk.Combobox(main_frame, textvariable=mode_var, values=["歌手模式", "随机模式"], state="readonly")
mode_combobox.grid(row=0, column=1, padx=5, pady=5, sticky="w")

singer_name_label = ttk.Label(main_frame, text="歌手名称:")
singer_name_label.grid(row=1, column=0, padx=5, pady=5, sticky="w")

singer_name_entry = ttk.Entry(main_frame)
singer_name_entry.grid(row=1, column=1, padx=5, pady=5, sticky="w")

start_button = ttk.Button(main_frame, text="开始爬取", command=start_scraping)
start_button.grid(row=2, column=0, columnspan=2, pady=10)

log_text = scrolledtext.ScrolledText(main_frame, width=60, height=20)
log_text.grid(row=3, column=0, columnspan=2, pady=10)

root.mainloop()
```

## 最终总结

通过三个版本的迭代，程序从最初的功能实现到逐步优化，提升了用户体验。每个版本都在前一个版本的基础上做出了改进，使得用户界面更友好。

**待解决的问题**：

1. **不能回车确认**：目前只能通过鼠标点击按钮进行操作，无法使用回车键确认。
2. **GUI界面和终端同时出现**：在启动程序时，GUI界面和终端窗口会同时出现，影响用户体验。
3. **爬取过程卡顿**：爬取数据时，程序运行速度较慢，用户体验不佳。
4. **不能创建文件夹和下载**：在某些情况下，程序未能成功创建文件夹或下载文件。