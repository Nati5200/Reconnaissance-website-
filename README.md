import requests
from bs4 import BeautifulSoup
import os
from urllib.parse import urljoin, urlparse, unquote

# הגדרת headers כדי להיראות כמו דפדפן
HEADERS = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64)"
}


def sanitize_filename(name):
    return "".join(c for c in name if c.isalnum() or c in (' ', '.', '_', '-')).rstrip()


def giveme_links_and_download_files(url):
    try:
        response = requests.get(url, headers=HEADERS, timeout=10)
        response.raise_for_status()
    except requests.RequestException as e:
        print(f"[!] Failed to load the page: {e}")
        return

    soup = BeautifulSoup(response.text, 'html.parser')
    my_links = set()

    # חיפוש בכל האלמנטים עם קישורים
    for tag in soup.find_all(['a', 'img', 'link'], href=True) + soup.find_all('img', src=True):
        src_attr = tag.get('href') or tag.get('src')
        full_url = urljoin(url, src_attr)
        my_links.add(full_url)

    print(f"\n[+] Found {len(my_links)} links.")

    # נתיב תיקייה לשמירה
    downloads_folder = os.path.expanduser("C:\\Users\\nati\\Desktop\\don")
    os.makedirs(downloads_folder, exist_ok=True)

    # סוגי קבצים שרוצים להוריד
    downloadable_extensions = ('.pdf', '.jpg', '.jpeg', '.png', '.gif', '.bmp', '.docx', '.zip', '.rar')

    for link in my_links:
        parsed_url = urlparse(link)
        file_name = os.path.basename(parsed_url.path)
        file_name = unquote(file_name)
        file_name = sanitize_filename(file_name)

        if not file_name or not file_name.lower().endswith(downloadable_extensions):
            continue  # דלג על קבצים לא רלוונטיים

        file_path = os.path.join(downloads_folder, file_name)

        try:
            print(f"[>] Downloading: {link}")
            file_response = requests.get(link, headers=HEADERS, timeout=10)
            file_response.raise_for_status()

            with open(file_path, 'wb') as f:
                f.write(file_response.content)
            print(f"[✓] Saved to: {file_path}\n")
        except requests.RequestException as e:
            print(f"[!] Failed to download {link}: {e}")


if __name__ == "__main__":
    url = input("Enter the URL to scan and download files: ").strip()
    if not url.startswith("http"):
        url = "http://" + url
    giveme_links_and_download_files(url)
