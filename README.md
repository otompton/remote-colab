
# **README**

## **開発環境**
本リポジトリは下記のような構成にて開発しています。

またこのノートブックは、図のような開発環境を構築する手順を記載してます。
（編集中）

![代替テキスト](https://drive.google.com/uc?id=1Hy35Mdicyxx7Q3riCsfwoh7bHCrE-JJc)

## **設定方法**

# **Colabの初期設定**
`Kaggle`データセットのダウンロードはここから`GoogleDrive`上に行う。

ダウンロード後の分析作業は、各ノートブック`.ipynb`を作成して行う。

その際、`GoogleDrive`のマウントは各ノート上で実行する。

下記は、`KaggleAPI`の有効化、`GoogleDrive`のマウント化、`Tree`モジュールの追加、`Kaggle`データセットのダウンロード、githubとの連携～1st commitまでの実施手順。

## **`Kaggle API`のインストール**


```
!pip install kaggle
```

## **`Kaggle API`の有効化**

[Kaggle API with Colab](https://colab.research.google.com/drive/1eufc8aNCdjHbrBhuy7M7X6BGyzAyRbrF#scrollTo=5l1V_oxXsZ8l&forceEdit=true&sandboxMode=true)

下記実行前に、`kaggle.json`をあらかじめDLし、`GoogleDrive`に格納しておく。実行すると認証設定が呼び出され、許可すると`GoogleDrive`ディレクトリ内から`kaggle.json`ファイルが検索され、`root/.kaggle`以下に格納される。
元々のコードだと

`filename = "/content/.kaggle/kaggle.json"`

となっているが、API起動時に参照エラーが発生するため

`filename = "/root/.kaggle/kaggle.json"`

へ変更する事。



```
from googleapiclient.discovery import build
import io, os
from googleapiclient.http import MediaIoBaseDownload
from google.colab import auth

auth.authenticate_user()

drive_service = build('drive', 'v3')
results = drive_service.files().list(
        q="name = 'kaggle.json'", fields="files(id)").execute()
kaggle_api_key = results.get('files', [])

filename = "/root/.kaggle/kaggle.json"
os.makedirs(os.path.dirname(filename), exist_ok=True)

request = drive_service.files().get_media(fileId=kaggle_api_key[0]['id'])
fh = io.FileIO(filename, 'wb')
downloader = MediaIoBaseDownload(fh, request)
done = False
while done is False:
    status, done = downloader.next_chunk()
    print("Download %d%%." % int(status.progress() * 100))
os.chmod(filename, 600)
```

    Download 100%.


## **`GoogleDrive`のマウント**
先に`GoogleDrive`を`Colab上へ`マウントした場合、`Googledrive`上の`kaggle.json`内の記載が空白化する事象が発生したため、`KaggleAPI`導入後に実施


```
from google.colab import drive
drive.mount('/content/drive')
```

    Go to this URL in a browser: https://accounts.google.com/o/oauth2/auth?client_id=947318989803-6bn6qk8qdgf4n4g3pfee6491hc0brc4i.apps.googleusercontent.com&redirect_uri=urn%3aietf%3awg%3aoauth%3a2.0%3aoob&response_type=code&scope=email%20https%3a%2f%2fwww.googleapis.com%2fauth%2fdocs.test%20https%3a%2f%2fwww.googleapis.com%2fauth%2fdrive%20https%3a%2f%2fwww.googleapis.com%2fauth%2fdrive.photos.readonly%20https%3a%2f%2fwww.googleapis.com%2fauth%2fpeopleapi.readonly
    
    Enter your authorization code:
    ··········
    Mounted at /content/drive


## **`Tree`パッケージのインストール**

ディレクトリ構成の記載に便利なため導入


```
!apt-get install tree
```

## ディレクトリ構成

`My Drive`以下は、`GoogleDrive`のマウント先。データを保存することで、
`GoogleDrive`に同期される。

`Colab Notebooks`：Notebookの格納

`datasets`：各データセットの格納

`setting`：設定ファイルの格納


```
!tree -d
```

    .
    ├── Colab Notebooks
    ├── datasets
    │   ├── kaggle
    │   │   └── titanic
    │   └── ml100knock
    │       ├── 10_Questionnaire_analysis
    │       ├── 1_web_order
    │       ├── 2_Retail_data
    │       ├── 3_Customer_information
    │       ├── 4_Customer_behavior
    │       ├── 5_Customer_withdrawal
    │       ├── 6_Logistics_route
    │       ├── 7_Logistics_network
    │       ├── 8_Numerical_simulation
    │       └── 9_Potential_customer
    │           ├── img
    │           └── mov
    ├── imgs
    └── setting
        └── __pycache__
    
    20 directories


## **`Kaggle`データセットのダウンロード**

使用するデータセットは`./drive/My\ Drive/datasets/kaggle/{competition title}/`以下に格納

`> !kaggle competitions download -h`

```
usage: kaggle competitions download [-h] [-f FILE_NAME] [-p PATH] [-w] [-o]
                                    [-q]
                                    [competition]

optional arguments:
  -h, --help            show this help message and exit
  competition           Competition URL suffix (use "kaggle competitions list" to show options)
                        If empty, the default competition will be used (use "kaggle config set competition")"
  -f FILE_NAME, --file FILE_NAME
                        File name, all files downloaded if not provided
                        (use "kaggle competitions files -c <competition>" to show options)
  -p PATH, --path PATH  Folder where file(s) will be downloaded, defaults to current working directory
  -w, --wp              Download files to current working path
  -o, --force           Skip check whether local version of file is up to date, force file download
  -q, --quiet           Suppress printing information about the upload/download progress
```







```
!kaggle competitions download -c titanic -p ./drive/My\ Drive/githyb/datasets/kaggle/titanic/
```

## Githubとの連携
githubとcolabの連携は、[personal token](https://help.github.com/ja/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line)と[https URL](https://help.github.com/ja/github/using-git/which-remote-url-should-i-use#cloning-with-https-urls-recommended)を用いて行う。
[参考](https://towardsdatascience.com/google-drive-google-colab-github-dont-just-read-do-it-5554d5824228)

モジュール検索パスの追加


```
import sys
from os.path import join

REPO_NAME = 'remote-colab'
PROJECT_PATH = '/content/drive/My Drive/'+ REPO_NAME + '/'
sys.path.append(PROJECT_PATH)
```

設定ファイルのインポート


```
from setting import personal_setting as PS
# PS.email_address = {'your setting e-mail address'}
# PS.personal_token = {'your token'}
# PS.user_name = {'your name'}
```

Clone URL・プロジェクトディレクトリの作成


```
GIT_PATH = "https://" + PS.personal_token + "@github.com/" + PS.user_name + "/" + REPO_NAME + ".git"
print("GIT_PATH: ", GIT_PATH)

# プロジェクトディレクトリの作成
!mkdir "{PROJECT_PATH}"
!cd "{PROJECT_PATH}"
```

クローン


```
!git clone "{GIT_PATH}"
```

差分の更新・コミット・更新

`Shell`スクリプト上で`python`の変数`hoge`を利用する場合
`"{hoge}"`とすると利用できるみたい。便利。


```
!git add -A
!git config --global user.email "{PS.email_address}"
!git config --global user.name "{PS.user_name}"
```

アクセストークンが`commit`するファイル内に含まれる場合、`github`が検知して
アクセストークンが無効化される。

そのため、`.gitignore`でファイルを追跡しないように設定する。



```
# .gitignore
setting/*.py # personal_settingを記述しているため、追跡から除外
*.json
*.csv
.git/* #.git/config内に同様の内容が含まれるため、除外（デフォルトで除外される？）
```




```
!git commit -m 'some fixes'
```


```
!git push origin master
```


```
!sudo apt-get install pandoc
```

    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    pandoc is already the newest version (1.19.2.4~dfsg-1build4).
    pandoc set to manually installed.
    The following package was automatically installed and is no longer required:
      libnvidia-common-430
    Use 'sudo apt autoremove' to remove it.
    0 upgraded, 0 newly installed, 0 to remove and 7 not upgraded.

