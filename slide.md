# Perl入学式 in Yapc::Asia

## 今日の内容
  - モジュール入門
  - 起動引数
  - Youtube API

## Acme::Nyaa
- ターミナルから `nyaa.pl` を実行すると猫がしゃべっているように文章を整形する.

## おまじない
    #!/usr/bin/env perl
    use strict;
    use warnings;

- おまじないとして, 冒頭の3行を書くようにしよう
- `use strict` -> 厳密な書式を定めたり, 未定義の変数を警告するといった効果
- `use warnings` -> 望ましくない記述を警告してくれる効果
    - 以下, この資料のサンプルコードでは｢お約束｣を省略します. 書かれているものとして扱ってください

## モジュールのインストール
    $ cpanm Acme::Nyaa

- `cpanm` コマンドにより, 必要なモジュールをイントールする
- CPANというアーカイブから様々なモジュールを利用することができる

## どう使うの??
- [CPAN - Acme::Nyaa](http://search.cpan.org/~akxlix/Acme-Nyaa-0.0.7/lib/Acme/Nyaa.pm)
    - モジュールには **SYNOPSIS** という使い方が記載されているので, まずは **SYNOPSIS** を読みましょう

## SYNOPSIS
    use Acme::Nyaa;

    my $kijitora = Acme::Nyaa->new();
    my $nekotext = '猫がかわいい'
    print $kijitora->cat( \$nekotext );

- このコードを `nyaa.pl` に保存して実行しよう
    - `$ perl nyaa.pl` とすることで, 実行できます

## 解説
    use Acme::Nyaa; # モジュールの読み込み
    my $kijitora = Acme::Nyaa->new;
    my $nekotext = '猫がかわいい'
    print $kijitora->cat( \$nekotext ); # cat はメソッド(関数) で 引数にリファレンスをあたえる

- シジル(`$`,`@`,`%`) の前に`\(バックスラッシュ)`をつけると, その変数のリファレンスになる
- オブジェクトに関する説明は今回は省略します

## 練習問題
- `nyaa.pl` を改良して, 端末から文字を入力した文字列を猫が話しているような文字列に変換してください
    - STDIN
    - chomp

## 起動引数
    $ perl hoge.pl  foo bar baz
    $ perl ファイル名 起動引数

- perl 実行時のファイル名の後ろにある部分を **起動引数** と言う

## ARGV
    print "@ARGV"; # foo bar baz

- 起動引数はファイル内で`@ARGV`という特殊変数に格納されている
  - 特殊変数であるため`my`をつける必要はない

## fizzbuzz.pl
- 1 から `max` (起動時に定める)において, 数字が`3`で割り切れるなら`Fizz`, `5`で割り切れるなら`Buzz`, 両方で割り切れるなら`FizzBuzz` とする
- ハッシュリファレンスを使って, `Fizz`, `Buzz`, `FizzBuzz` をキーとして, 数値を分類してください
- 出力は `Data::Dumper` を使ってください
    - `print Dumper リファレンス`

# Web API 入門

## Web API
- API -> **A**pplication **P**rogramming **I**nterface
    - WebAPI にリクエストすることで, レスポンスを得ることができる

## Youtube Data API
-  [デベロッパーガイド](https://developers.google.com/youtube/2.0/developers_guide_protocol?hl=ja)

## web ブラウザで確認する
- [http://gdata.youtube.com/feeds/api/videos](http://gdata.youtube.com/feeds/api/videos)
  - web ブラウザで上記のアドレスにアクセスしてください

## 検索キーワード & レスポンスフォーマット
- [http://gdata.youtube.com/feeds/api/videos?q=YAPC::Asia&alt=json](http://gdata.youtube.com/feeds/api/videos?q=YAPC::Asia&alt=json)
  - 先程のURIに, `q=YAPC::Asia&alt=json` というクエリパラメータを組み合わせたものにアクセスします

## WebService::Simple
    $ cpanm WebService::Simple

- リクエストURIの構築, WebAPIにリクエスト, 結果をパースする, といった処理を行うモジュール

## WebService::Simple
    use WebService::Simple;
    use Data::Dumper;
    my $service = WebService::Simple->new(
        base_url => 'http://gdata.youtube.com/feeds/api/videos'
    );
    my $res = $service->get();
    my $ref = $res->parse_response();
    print Dumper $ref;

- 上記のスクリプトを実行してみてください

## 検索キーワード & レスポンスフォーマット
    use WebService::Simple;
    my $service = WebService::Simple->new(
        base_url        => 'http://gdata.youtube.com/feeds/api/videos',
        response_parser => 'JSON'
    );
    my $res = $service->get({ q => 'YAPC::Asia', alt => 'json' });
    my $ref = $res->parse_response();

## 動画の URL を出力する
    for my $entry ( @{$ref->{feed}{entry}} ) {
        my $url = $entry->{link}[0]{href};
        print "$url\n";
    }

- `$ref` というリファレンスから動画のURLを出力します

## 動画のタイトルを出力する
    for my $entry ( @{$ref->{feed}{entry}} ) {
        my $url   = $entry->{link}[0]{href};
        my $title = $entry->{'media$group'}{'media$title'}{'$t'};
        print "$title: $url\n";
    }

- `media$group` などにおける `$` は変数ではなくただの文字列であるため, キーを`''(シングルクォート)`でくくる必要がある

## 他の情報
    {
      "media\$category" => [
        {
          "\$t" => "Tech",
          label => "Science & Technology",
          scheme => "http://gdata.youtube.com/schemas/2007/categories.cat"
        }
      ],

- タイトル以外にも動画のカテゴリーがある

## 動画のカテゴリーの取得
    { => ハッシュ
      "media\$category" => [ => 配列
        { => ハッシュ
          "\$t" => "Tech",
          label => "Science & Technology",
          scheme => "http://gdata.youtube.com/schemas/2007/categories.cat"
        }
      ],

- `$entry->{'media$group'}{'media$category'}[0]{label}` とすることで取得できる

## 練習問題
- スクリプト実行時の引数から検索キーワードを渡せるように改良してください
- 余裕があれば, 複数のキーワードで検索できるようにしてください
  - 複数のキーワードから検索するには, クエリを`+`で連結する必要があるので, `join()` を使うといいですね

## 機能拡張
    $ perl youtube_api.pl --time this_week YAPC
      --time today
             this_week
             this_month
             all_time

- 上記のように **オプション** を使って, 指定した期間内にアップロードされた動画を取得するようにする

## Getopt::Long
    use Getopt::Long qw/:config posix_default no_ignore_case bundling auto_help/;

- `Getopt::Long` はコアモジュールであるため, 特別にインストールする必要はない
    - 上記の書き方はおまじないだと思ってください

## Getopt::Long
    my %opt; # => ハッシュ
    Getopt::Long::GetOptions(
        "f|foo=i" => \$opt{foo} # => 数値   --foo=256
        "v|var=s" => \$opt{var} # => 文字列 --var=booooon
        "a|all"   => \$opt{all} # => フラグ --all
    );
    print "$opt{foo}\n"; # => 256
    print "$opt{var}\n"; # => booooon
    print "$opt{all}\n"; # => 1

## Getopt::Long
    use Getopt::Long qw/:config posix_default no_ignore_case bundling auto_help/;

    my %opt;
    Getopt::Long::GetOptions(
        "t|time=s"    => \$opt{time},
        "o|orderby=s" => \$opt{orderby},
    );
    my $res = $service->get({ q => $query, time => $opt{time}, alt => 'json' });

- 実際に `Getopt::Long` を組み込みます
- コーディングができたら, 実際にオプションが適応されているか確認してください
    - `perl youtube_api.pl --time all_time YAPC::Asia`

## デフォルト値の設定
- オプションをつけれるようにはなったが, `perl youtube_api.pl hoge` と実行するとクエリが正しく生成できない (http://gdata.youtube.com/feeds/api/videos?alt=json&**time=**&q=hoge)
    - `--time` オプションがない場合は, `$opt{time}` を設定しておく必要がある

## デフォルト値の設定
    if ( !exists $opt{time} ) {
        $opt{time} = 'all_time';
    }

- `%opt` に `time` というキーがなければ, `all_time` という文字列を代入する処理

## exists
    my %hash = ( hoge => 'foo' );
    print exists( $hash{hoge} ); # => 1
    print exists( $hash{bar} ); # => undef

- ハッシュのキーが存在するかどうか確認するのに `exists` を使う
    - キーが存在するのであれば, `1`, そうでなければ `undef` が返される

## デフォルト値の設定
    $opt{time} ||= 'all_time';

- 論理演算子を用いて, このように書くこともできる

## 練習問題
    --orderby relevance (デフォルト値)
              published
              viewCount
              rating

- オプションとして, `orderby` を追加してください
- デフォルト値は `relevance` に設定してください

