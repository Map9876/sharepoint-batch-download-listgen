```
import requests
import xml.etree.ElementTree as ElementTree
from urllib.parse import urlparse, quote, unquote, parse_qs

ARIA2_INPUT_FILE = 'aria2-links.txt'
DLPATH_PRIFIX = 'Downloads/'

# 构造完整的URL
url = "https://lastsummer-my.sharepoint.com/:f:/g/personal/1125824247_lastsummer_onmicrosoft_com/Es1sg0jpeWxLnYTg_SHpUNgBBZSfTYGP5BA_wDwZmiu93A?e=hPj9ci" #  @param {type:"string"}
# 创建一个会话对象
session = requests.Session()

# 发送GET请求
response = session.get(url)

# 获取最终跳转的URL
final_url = response.url

# 解析最终跳转的URL以提取parent参数
parsed_url = urlparse(final_url)
query_params = parse_qs(parsed_url.query)
parent_param = query_params.get('id', [None])[0]

# 如果parent参数存在，构造新的链接
if parent_param:
    new_url = f"https://{parsed_url.netloc}{parent_param}"
    print(f"New URL: {new_url}")
else:
    print("No 'parent' parameter found in the URL.")

# 获取Set-Cookie头部信息
cookies = session.cookies.get('FedAuth')
print(f"Set-Cookie: {cookies}")

COOKIE_FEDAUTH = None
SHAREPOINT_ROOT = None

import os
from urllib.parse import urlparse, unquote

def download(url, save_folder):
    # 解析URL以获取文件名
    parsed_url = urlparse(url)
    file_name = unquote(os.path.basename(parsed_url.path))
    
    # 如果文件名是空的（例如，当URL指向一个目录时），则生成一个默认文件名
    if not file_name:
        file_name = 'downloaded_file'
    
    # 完整的保存路径
    save_path = os.path.join(save_folder, file_name)
    
    # 确保保存路径的目录存在
    os.makedirs(save_folder, exist_ok=True)
    
    # 发送GET请求并下载文件
    response = session.request("GET", url, stream=True)
    if response.status_code == 200:
        # 打开文件并写入内容
        with open(save_path, 'wb') as f:
            for chunk in response.iter_content(chunk_size=8192):
                if chunk:
                    f.write(chunk)
        print(f"Downloaded '{url}' to '{save_path}'")
    else:
        print(f"Failed to download '{url}'. Status code: {response.status_code}")
 
# 示例用法
# download('http://example.com/file.zip', '/path/to/save/file.zip')

def main():
    global COOKIE_FEDAUTH, SHAREPOINT_ROOT, SESSION, DLPATH_PRIFIX
    SHAREPOINT_ROOT = new_url
    SHAREPOINT_PATH = urlparse(SHAREPOINT_ROOT).path
    print(SHAREPOINT_ROOT, "root")
    """
    cookies = {
        'FedAuth': COOKIE_FEDAUTH
    }
    requests.utils.add_dict_to_cookiejar(session.cookies, cookies)
    """
    download_dir = "/content/"
    if download_dir != '':
        DLPATH_PRIFIX = download_dir.rstrip('/') + '/'
    else:
        print(f"Using {DLPATH_PRIFIX}")

    print("Fetching file list ...")
    resp = session.request("PROPFIND", SHAREPOINT_ROOT)
    try:
        xml_root = ElementTree.fromstring(resp.content.decode('utf8'))
    except ElementTree.ParseError:
        print(f"ERROR! Got Response <{resp.status_code}>: {resp.content.decode('utf8')}\nPossible invalid Real Path or FedAuth.")
        exit()

    print("Generating aria2 file list ...")
    fw = open(ARIA2_INPUT_FILE, 'w', encoding='utf-8')
    for e in xml_root.findall('.//{DAV:}href'):
        file_url = e.text
        if file_url.endswith('/'):
            continue
        url_info = urlparse(file_url)
        save_path = DLPATH_PRIFIX + "/".join(url_info.path.lstrip(SHAREPOINT_PATH).split('/')[:-1])
        save_path = unquote(save_path)
        encoded_url = url_info.geturl()
        fw.write(f'{encoded_url}\n  dir={save_path}\n\n')
        download(encoded_url, save_path)
    print(f'\nSucess!\nAria2 input file was saved to {ARIA2_INPUT_FILE}. Please run following command to start download:\n\naria2c --header="Cookie:FedAuth={cookies}" --input-file={ARIA2_INPUT_FILE} --max-concurrent-downloads=2 --max-connection-per-server=5 --save-session=session.txt --save-session-interval=30\n')

if __name__ == "__main__":
    main()
```
