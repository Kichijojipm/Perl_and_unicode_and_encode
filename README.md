# 吉祥寺.pmミニ「Perlと、Unicodeと、Encode」用のレジュメ

## 基礎知識

### 文字集合と、エンコーディングと、字形（フォント）

それぞれ別のもの。要注意！

### 参考文書

perlunitut.pod

perluniintro.pod

perlunicode.pod

perlunifaq.pod

## 最初の一歩

注）以降のソースコードは全てUTF-8で保存して下さい。

### 大原則

- Perlは、歴史的経緯で文字列は、latin-1(ISO/IEC 8859-1)だと仮定する。

        https://ja.wikipedia.org/wiki/ISO/IEC_8859-1

- latin-1以外を使う時は、入口でdecodeして、出口でencodeすればOK。

        use Encode qw/encode decode/;
    
        my $str = shift @ARGV // "吉祥寺.pm";
        my $decoded = decode("UTF-8", $str);
    
        say encode("UTF-8", $decoded) . ":length" . length($decoded);

Perlの内部表現（UTF-8）に変換されてから、処理される。

## Encodeモジュール

Encodeモジュールとは？

様々なencodingが存在する外の世界と、Perlの中のを繋ぐもの。

    +------------+                      +^^^^^^^^^^^^+
    |Perlの中の世界| <- Encodeモジュール -> |荒ぶる外の世界|
    +------------+                      +<><><><><><>+

Encodeモジュールを使うことで、latain-1以外のデータを正しく「文字列」として処理できる。

- latin-1以外のデータの長さが正しく計算できる。

        length("吉祥寺.pm"); # -> 6

- latin-1以外のデータでも正規表現が正しく動作する。

		if ( decode("utf8", "吉祥寺.pm２") =~ /\d/ ) {
	      print("matched!\n");
    	} else {
	      print("Not matched!\n");
	    }
	
	    # -> print "matched!"

- \d、\w、\sの挙動が変わるので要注意！

    - \dは[0-9]'以外'にもマッチする。 # 所謂全角数字も数値
    - \wは[0-9a-zA-Z_]'以外'にもマッチする。# 漢字もひらがなも文字
    - \wは0x20やtab'以外'にもマッチする。 # 所謂全角スペースも空白

それぞれの文字の種類を把握して使うことが重要！

## UTF8フラグ

変数の中身をPerlの内部でどのように管理しているのか、Devel::Peekモジュールで覗いてみましょう。

	use Encode qw/decode/;
	use Devel::Peek;

	my $str = "吉祥寺.pm";
	my $decoded = decode("UTF-8", $str);

	Dump($decoded);

実行すると以下の通り表示されます。

	SV = PV(0x7ffd040056f0) at 0x7ffd0402a4d0
	  REFCNT = 1
	  FLAGS = (POK,IsCOW,pPOK,UTF8)
	  PV = 0x7ffd03c3f260 "\345\220\211\347\245\245\345\257\272.pm"\0 [UTF8 "\x{5409}\x{7965}\x{5bfa}.pm"]
	  CUR = 12
	  LEN = 14
	  COW_REFCNT = 0

FLAGSとPerlの内部で文字列がUTF8で管理されていることが分かります（所謂UTF8フラグと呼ばれているものはこれ）。

ただし、このまま標準出力に出力しようとすると…

	use Encode qw/decode/;

	my $str = "吉祥寺.pm";
	my $decoded = decode("UTF-8", $str);

	print "$decoded\n";

一応、文字列はちゃんとターミナルに出力されますが、警告メッセージが出ます。

	Wide character in print at first.pl line 9.
	吉祥寺.pm

Perlの内部表現は、UTF-8をベースにしているし、UTF8というフラグで管理されていますが、あくまで「内部表現」なので、外の世界（＝ターミナル）に出力する時は明示的にエンコーディングを指定する必要が有ります。

	use Encode qw/encode decode/;

	my $str = "吉祥寺.pm";
	my $decoded = decode("UTF-8", $str);

	print encode("UTF-8", "$decoded\n");

これなら警告は出ません。

	吉祥寺.pm

decodeされた文字列は、あくまでPerlの内部表現（実装は変わるかもしれない）で管理されている事を覚えておきましょう。

## PerlIOレイヤー

標準入出力や、ファイルアクセスに対して、様々な処理を追加できるのがPerlIOレイヤー。

詳しくは、perlIOモジュールのpodを参照。

binmodeでPerl IOレイヤーをセットして標準入出力の挙動を変えると、アプリケーション全体に影響するので、要注意（明示的にencode/decodeした方が良い）。

ファイルの入出力なら、open時に明示的に指定するのでOK。

    open my $fh, "<:encoding(utf8)", $path;

## utf8プラグマ

    use utf8;

UTF-8でソースコードが保存されていることをPerlに伝える。最初からUTF8フラグが付与される。

	use utf8;
	use Devel::Peek;

	my $str = "吉祥寺.pm";

	Dump($str);

実行結果は、以下の通りとなる(Encodeモジュールは使っていない)。

	SV = PV(0x7ff863805260) at 0x7ff86382a4d0
	  REFCNT = 1
	  FLAGS = (POK,IsCOW,pPOK,UTF8)
	  PV = 0x7ff863500b10 "\345\220\211\347\245\245\345\257\272.pm"\0 [UTF8 "\x{5409}\x{7965}\x{5bfa}.pm"]
	  CUR = 12
	  LEN = 14
	  COW_REFCNT = 1

## 気を付けるところ

### is_utf8メソッド

	use Encode qw/is_utf8/;
	my $str = "Kichijoji.pm";

	if (is_utf8($str)) {
	    print "flagged\n";
	} else {
	    print "not flagged\n";
	}

実行すると…

    not flagged

普通のアプリケーションで、`is_utf8`メソッドを使う場面はほぼ無い。必要だと思ったら文字列周りの設計がおかしい可能性大。

### 自動アップグレード

	use Encode qw/encode decode/;

	my $str1 = "武蔵野市";
	my $str2 = "吉祥寺";

	my $decoded = decode("UTF-8", $str2);
	my $address = $str1 . $decoded;

	print encode("UTF-8", $address) . "\n";

実行すると…

	æ­¦èµéå¸吉祥寺

flaggedなデータと、flaggedじゃないデータを連結すると、勝手にflaggedじゃないデータをlatin-1と見なして、decode("latain-1", flaggedじゃないデータ)が実行される。->盛大に文字化けする！

### Unicodeにもバージョンが有ること

    https://ja.wikipedia.org/wiki/Unicode
    
どのPerlがどのUnicodeのバージョンに対応しているかは、perldeltaを参照。絵文字関連は要注意！

### 標準入出力経由でデータをやりとりする

同じPerlのプログラムの中でも、forkして他のPerlのプロセスの標準出力をトラップして、インプットにする様なプログラムを、ポータビリティの有る方法で実装するのは難しい。

Test::MoreがTAPに出力した結果をproveがpipe経由で受け取るとか。

## モジュールの挙動を把握する

残念ながら統一的な挙動／メソッドインタフェースは存在しないので、モジュールのpod見て、一つ一つ確認。

### テキスト処理

- Text::CSV::Encode
- Text::xSlate
- JSON

### Webアプリケーション

- LWP
- Plack

#### テスト

- Test::More and TAP

binmodeを使わないと、日本語でTAP出力は不可。ただし、出力はできるが、proveコマンドが標準入力経由でTAPの結果を受け取るので、要注意！

#### DB

- ORM and DBI

#### モジュール

- ExtUtils::MakeMaker


### ARGVの扱い

自分でデコードすること。

### ファイル名の扱い

ファイルパスは、UTF-8であるとは限らない。

特にOS XのNFDと、WindowsのLong File Name

### Test::MoreとTAPと日本語

TAPに日本語で出力することはできる。
proveは、標準入力から受け取る。文字コードは？

### 環境変数

自前でデコードする。

## 便利なモジュール

いくつかの便利なモジュールの紹介。

- Encode::Guess
- Data::Dumper::AutoEncode
- Encode::Local
- Win32::Unicode
- Unicode::Japanese::JA

## 結論

Perlが文字列として扱えるのはlatin-1か、UTF-8（っぽい内部表現用のエンコーディング）。

その文字列がどこから来たものなのか、何者なのか把握しないと文字化けが起きる。

モジュールの対応状況もきちんと調べないとダメ。
