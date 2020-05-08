# aribb25 library

*Current Version:* 0.2.8

Basic implementation of the ARIB STD-B25 public standard.

Forked from libarib25 and renamed to aribb25.
Modified for VLC Media Player integration and cross platform
builds.

----
## About STD-B25

ARIB [STD-B25](http://www.arib.or.jp/english/html/overview/doc/6-STD-B25v5_0-E1.pdf) is a 
specification of conditional access system for digital broadcasting.

This standard covers three parts:

* Conditional Access System

* Conditional Playback System

* Content Protection System

## This library

This implementation currently only allows playback of scrambled streams according to the provided conditional access rights.




The following sections are approximative English translations from Japanese.
See README.jp.txt for original version.

## Initial release background

With the end of analog TV in Japan in July 2011 a wish for cheap Digital TV
receivers emerged.

Although, the volontary introduced complexity in the ARIB standard makes it really
hard to understand, induces higher development costs for device manufacturers
and then nullifies the chances of having low cost receivers on the market.

For that reason, this library gathers most of the necessary specification
into a comprehensible code that can be used as a starting point.

To stay within the initial objective, the binary only distribution won't
be made.

## Scope of this implementation

The Conditional Access system (CA) accordingly to the associated B-CAS Card will
decrypt TS streams using the ECM table 0x82 and EMM table 0x84.
EMM table 0x85 messages processing are to be done.

## Operating system and environments

Conditional Access Cards can be read through any ISO-7816
compliant IC card reader.

Known and working card readers are:
 * Hitachi / Maxell HX-520UJ (Windows Only)
 * NTT SCR-3310 eTax reader

## Source code licence

 * This code is released under the ISC Licence terms
 * The user of source code is fully responsible for any arisen problem, misuse,
   and patent that could be claimed in the country of usage.

## Structure of the library

* arib_std_b25.h/c

Contains main definitions linking the MPEG-2 TS parser module,
the CA system module and the MULTI2 crypto module.

* ts_section_parser.h/c

Custom MPEG-2 TS parser

* b_cas_card.h/c

Conditional Access System (B-CAS Card) resource control and
interface.

* multi2.h/c

MULTI2 crypto module

* td.c

MPEG-2 TS stream based decryption test program using PAT/PMT/ECM and
the Conditial Access elements.

The number of MULTI2 rounds can be set through command line options.
Defaults to 4 rounds, but can exists up to 32. Although, this
parameter being usually confidential, it has to be guessed.

## Usage guidelines

### Initialization

* By creating a new instance of the B_CAS_CARD module, the
BCAS Card will be initialized.

On Windows systems, this is done through the smartcard API.
On other platforms, this is done though the pcsclite library.

Registering with the CA System, the BCAS Card module will receive
a 64 Bytes key resulting into 8 Bytes after CBC chaining.

* The application then registers the BCAS Card module against
an instance of the ARIB_STD_B25 module.

### Data processing time

By feeding data to the ARIB_STD_B25 module the data will be
processed as following.

* The TS splitter module will find out packet size (common
values being 188, 192 and 204 bytes) by buffering input
data. If it cannot be determinated after 8KBytes of data,
an error will be issued.

* If PAT hasn't be discovered yet, more data will be buffered
up to 16MBytes before throwing an error. If it has been 
successfully discovered, PID will be set up.

* Buffering will continue until the matching PMT will be fully
known or 2 section will be received. Buffering limit for this
purpose being 32MBytes. On success, the presence of ECM will
be verified against the corresponding program.

* Each received ECM will be then submited to the B_CAS_CARD module
and will retrieve the matching key for the MULTI2 module to be able
to compute decryption key.

* If the TS packet is encrypted, packet will be sent to the MULTI2
decryption module buffer. Otherwise, it will be directly sent to
the output buffer.

* After detecting the CAT section, EMM will be validated.

* After full reception of EMM, the B-CAS Card ID will be matched and
processing done by the card.

* On ECM renewal, the B_CAS_CARD will register the new key to the 
MULTI2 module.

* On PMT change, the decryptor will return 4.

* On PAT change, the process will abort and return 3.

## Get the source and build it

If compiling from the packaged source, unpack the tarball and change to the
resulting directory.

If compiling from a checked out repository, please make sure you've got the cloned too (use `git clone  https://github.com/shirow-github/libarib25.git`)

Then run following commands to compile and install libaribb25:

```bash
$ ./bootstrap
$ ./configure
$ make
$ sudo make install
$ sudo /sbin/ldconfig

* When you use the libpcsclitebcas library,
$ ./configure pcsclite_CFLAGS="-I/usr/local/include/libpcsclite-bcas -I/usr/include/PCSC -pthread" \
        pcsclite_LIBS="-L/usr/lib -lpcsclitebcas"
```

## Changelog

* 2020,05/02 - ver. 0.2.8

  A minor bug has been fixed.  

* 2016,02/18 - ver. 0.2.7

  Minor release to update documentation and fix compilation on OSX.

* 2014,10/21 - ver. 0.2.6

  Migration of the build system to autoconf.  
  Allow setting packet size to skip discovery process.  
  Various fixes and platform compatibility fixes.

* 2012,02/13 - ver. 0.2.5

  WOWOW でノンスクランブル <-> スクランブル切り替え後に復号が  
  行われないことがあるバグを修正

  http://www.marumo.ne.jp/db2012_2.htm#13 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.2.5.lzh

* 2009,04/19 - ver. 0.2.4

  終端パケットが野良パケット (PMT に記載されていない PID の  
  パケット) だった場合に、ECM が 1 つだけでも復号が行われない  
  バグを修正

  transport_error_indicator が立っている場合はパケット処理を  
  行わず、そのまま素通しするように変更

  http://www.marumo.ne.jp/db2009_4.htm#19 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.2.4.lzh

* 2008,12/30 - ver. 0.2.3

  CA_descriptor の解釈を行う際に CA_system_id が B-CAS カード  
  から取得したものと一致するか確認を行うように変更

  http://www.marumo.ne.jp/db2008_c.htm#30 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.2.3.lzh

* 2008,11/10 - ver. 0.2.2

  修正ユリウス日から年月日への変換処理をより正確なものへ変更

  TS パケットサイズの特定方法を変更

  http://www.marumo.ne.jp/db2008_b.htm#10 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.2.2.lzh

* 2008,04/09 - ver. 0.2.1

  PAT 更新時に復号漏れが発生していたバグを修正  
  (ver. 0.2.0 でのエンバグ)

  野良 PID (PMT に記載されていないストリーム) が存在した場合  
  TS 内の ECM がひとつだけならば、その ECM で復号する形に変更

  EMM の B-CAS カードへの送信をオプションで選択可能に変更 (-m)  
  進捗状況の表示をオプションで選択可能に変更 (-v)  
  通電制御情報 (EMM受信用) を表示するオプションを追加 (-p)

  http://www.marumo.ne.jp/db2008_4.htm#9 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.2.1.lzh

* 2008,04/06 - ver. 0.2.0

  EMM 対応  
  利用中の B-CAS カード ID 向けの EMM を検出した場合、EMM を  
  B-CAS カードに渡す処理を追加

  ECM 処理の際に未契約応答が返された場合、処理負荷軽減の為、  
  以降、その PID の ECM を B-CAS カードで処理しないように変  
  更 (EMM を処理した場合は再び ECM を処理するように戻す)

  進捗を nn.nn% の書式で標準エラー出力に表示するように変更  

  http://www.marumo.ne.jp/db2008_4.htm#6 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.2.0.lzh

* 2008,03/31 - ver. 0.1.9

  MULTI2 モジュールのインスタンスが未作製の状況で、MULTI2 の  
  機能を呼び出して例外を発生させることがあったバグを修正

  #パッチを提供してくれた方に感謝

  http://www.marumo.ne.jp/db2008_3.htm#31 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.1.9.lzh

* 2008,03/24 - ver. 0.1.8

  -s オプション (NULL パケットの削除) を追加  
  -s 1 で NULL パケットを出力ファイルには保存しなくなる  
  デフォルトは -s 0 の NULL パケット保持

  http://www.marumo.ne.jp/db2008_3.htm#24 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.1.8.lzh

* 2008,03/17 - ver. 0.1.7

  arib_std_b25.h に「extern "C" {」を閉じるコードがなかった問題  
  (C++ コードから利用する場合にコンパイルエラーを発生させる) を  
  修正

  TS パケットの中途でストリームが切り替わるケースで問題が発生し  
  にくくなるように、arib_std_b25.c 内のコードを修正

  http://www.marumo.ne.jp/db2008_3.htm#17 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.1.7.lzh

* 2008,03/16 - ver. 0.1.6

  PMT 更新の際、ECM 関連の状況が変更 (スクランブル - ノンスク  
  ランブルの切り替えや、ECM PID の変更等) が行われても、それが  
  反映されていなかった問題を修正

  http://www.marumo.ne.jp/db2008_3.htm#16 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.1.6.lzh

* 2008,02/14

  readme.txt (このファイル) を修正  
  ソースコードのライセンスについての記述を追加

* 2008,02/12 - ver. 0.1.5

  PMT の更新に伴い、どのプログラムにも所属しなくなった PID (スト  
  リーム) でパケットが送信され続けた場合、そのパケットの復号が  
  できなくなっていた問題を修正

  http://www.marumo.ne.jp/db2008_2.htm#12 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.1.5.lzh

* 2008,02/02 - ver. 0.1.4

  ver. 0.1.3 での PMT 処理方法変更に問題があり、PMT が更新された  
  場合、それ以降で正常な処理が行えなくなっていたバグを修正

  B-CAS カードとの通信でエラーが発生した場合のリトライ処理が機能  
  していなかったバグを修正

  http://www.marumo.ne.jp/db2008_2.htm#2 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.1.4.lzh

* 2008,02/01 - ver. 0.1.3

  有料放送等で未契約状態の B-CAS カードを使った際に、鍵が取得で  
  きていないにもかかわらず、間違った鍵で復号をしていた問題に対処

  鍵が取得できなかった ECM に関連付けられたストリームでは復号を  
  行わず、スクランブルフラグを残したまま入力を素通しする形に変更  
  鍵が取得できない ECM が存在する場合、終了時にチャネル番号と  
  B-CAS カードから取得できたエラー番号を警告メッセージとして表示  
  する形に変更

  暗号化されていないプログラムで例外を発生させていたバグを修正

  http://www.marumo.ne.jp/db2008_2.htm#1 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.1.3.lzh

* 2008,01/11 - ver. 0.1.2

  デジタル BS 放送等で、PAT に登録されているのに、ストリーム内で  
  PMT が一切出現しないことがある場合に対応

  PMT 内の記述子領域 2 に CA_descriptor が存在する場合に対応する  
  ため arib_std_b25.c 内部での処理構造を変更

  別プログラムと同時実行するためにスマートカードの排他制御指定を  
  変更

  http://www.marumo.ne.jp/db2008_1.htm#11 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.1.2.lzh

* 2008,01/07 - ver. 0.1.1

  セクション (PAT/PMT/ECM 等) が複数の TS パケットに分割されている  
  場合に、正常に処理できなかったり、例外を発生をさせることがある  
  バグを修正

  http://www.marumo.ne.jp/db2008_1.htm#7 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.1.1.lzh

* 2007,11/25 - ver. 0.1.0

  公開

  http://www.marumo.ne.jp/db2007_b.htm#25 又は  
  http://www.marumo.ne.jp/junk/arib_std_b25-0.1.0.lzh
