# 吉祥寺.pmミニ「Perlと、Unicodeと、Encode」用のレジュメ

## 基礎知識

### 文字集合と、文字コードと、字形（フォント）

### 参考文書

perlunitut.pod

perluniintro.pod

perlunicode.pod

perlunifaq.pod


## 最初の一歩

### 大原則

decodeして、encodeする。

    use Encode qw/encode decode/;
    
    my $str = shift @ARGV // "吉祥寺.pm";
    my $decoded = decode("UTF-8", $str);
    
    say encode("UTF-8", $decoded) . ":length" . length($decoded);

## Encodeモジュール

Encodeモジュールの基本的な使い方と、何をしているのか？

## PerlIOレイヤー

PerlIOレイヤーは何をしているのか？

PerlIOレイヤー or Encode?

## 気を付けるところ

### 自動アップグレード

盛大に文字化け

### モジュールの挙動を把握する

- Text::CSV::Encode
- Text::xSlate

### ARGVの扱い

自分でデコード

### ファイル名の扱い

特にOS XのNFDと、WindowsのLong File Name

### Test::MoreとTAPと日本語

TAPに日本語で出力することはできる。
proveは、標準入力から受け取る。文字コードは？

### 環境変数

自前でデコード

## 便利なモジュール

いくつかの便利なモジュールの紹介。

- Encode::Guess
- Data::Dumper::AutoEncode
- Encode::Local
- Win32::Unicode

## 結論

その文字列がどこから来たものなのか、何者なのか把握しないと結局化ける。

穴が無い事を一つ一つ確かめるのは結構厳しい。

モジュールの対応状況もきちんと調べないとダメ。