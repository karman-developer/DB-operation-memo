VirtualboxにOracleLinuxとDBをインストールしてDBを使用する

VirtualBox 7.1.6

OracleLinux
NAME="Oracle Linux Server"
VERSION="8.10"
-cat /etc/os-release
-ネットワーク設定
--ブリッジ
-ip a

TeraTerm5
192.168...

作成済みユーザーにroot権限付与（※良くないかも）

パッケージ更新
-sudo dnf update -y

ツール
-sudo dnf install -y wget vim net-tools unzip

DBインストール手順OracleHomePageに記載
https://docs.oracle.com/en/database/oracle/oracle-database/21/xeinl/installation-guide.html

Win側でDBファイルダウンロード後LinuxへDBファイル転送
※Win側でOpenSSHクライアントは使用出来るようになっていた。（システム - オプション機能）
-scp oracle-database-xe-21c-1.0-1.ol8.x86_64.rpm oracle@192.168...:/tmp/

DBインストール
-dnf -y localinstall oracle-database-xe-21c-1.0-1.ol8.x86_64.rpm
-/etc/init.d/oracle-xe-21c configure

環境設定（ここ大事！）
-export ORACLE_SID=XE 
-export ORAENV_ASK=NO 
-. /opt/oracle/product/21c/dbhomeXE/bin/oraenv

ORACLE_HOME = [] ? /opt/oracle/product/21c/dbhomeXE
The Oracle base has been set to /opt/oracle

Connecting to Oracle Database XE
-cd $ORACLE_HOME/bin

sqlplus / as sysdba;
sqlplus LESSON/oracle@192.168...:1521/XEPDB1;

lsnrctl start;
lsnrctl stop;
lsnrctl status;

startup;
shutdown immediate;

SET AUTOCOMMIT OFF; 
SET AUTOCOMMIT ON;

---------------
備考
Oracle XE 21c の依存パッケージとは（GPTに聞く：OSインストール時には未インストール）
| パッケージ名             | 内容・役割                              |
| ------------------ | ---------------------------------- |
| `glibc`            | C言語ランタイムライブラリ（Linuxの基本）            |
| `libaio`           | 非同期I/Oを行うためのライブラリ（OracleのI/O処理に必要） |
| `net-tools`        | ネットワーク関連コマンド（ifconfigなど）           |
| `bc`               | 計算スクリプトで使われるツール                    |
| `binutils`         | バイナリ操作ツール（リンクやアセンブルなど）             |
| `compat-openssl10` | 古いOpenSSLバージョンとの互換性（Oracleが一部依存）   |
| `policycoreutils`  | SELinux関連の制御コマンド群                  |

Port5500使用中エラー（※Virtualbox特有の可能性有）
/etc/hostsにホスト名を明記する必要が或る。
192.168... OracleLinux8.myguest.virtualbox.org OracleLinux8

-----------------------------------------------

1.CDBとPDBの関係
CDB（Container Database） … Oracleの「器」。1つ以上の PDB を持つ。
PDB（Pluggable Database） … アプリやユーザーが使う「実際のデータベース」的な存在。
CDB
├── PDB1（例：XEPDB1）
├── PDB2（別の用途があれば）

2.表領域（Tablespace）の関係
表領域（USERS や TEMP など）は、CDBとPDBの両方に存在しますが、各PDBごとに独立しています。
つまり、PDBに接続して SELECT tablespace_name FROM dba_tablespaces; を実行すると、そのPDB内で使える表領域が表示されます。
SYSTEM    → データディクショナリ用
SYSAUX    → 補助辞書
UNDOTBS1  → UNDOデータ（CDBでは共有のUNDO表領域）
TEMP      → ソート用の一時表領域
USERS     → 通常ユーザーが使う表領域（あなたが使うことが多い）

本番で SQL*Plus が標準とされる理由
| 理由            | 説明                                           |
| ------------- | -------------------------------------------- |
| **信頼性が高い**    | GUIツールに比べてバグや誤動作のリスクが少なく、Oracle公式の基本操作ツールです。 |
| **スクリプト化が可能** | 操作をSQLスクリプトとして記録・実行できるため、作業の自動化や再現性が担保できます。  |
| **リモートでも軽量**  | SSH越しに軽量に操作でき、GUI不要でサーバ環境に最適です。              |
| **ログが残る**     | `spool` コマンドで作業ログをファイル出力し、監査や検証にも使えます。       |
| **トラブル対応が早い** | GUIが使えない障害時にも対応できる（GUIはDBが正常に動作していないと接続不可）   |
