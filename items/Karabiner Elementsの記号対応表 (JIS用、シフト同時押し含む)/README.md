Karabiner ElementsでComplex Modificationを書く際に、いつも**JISキーボードの記号とキーコードの対応関係**がわからなくなって、Karabiner EventViewerで都度調べてしまうので、チートシートとして表にまとめてみた。

特に**シフト同時押しの記号**は慣れないと相当にややこしく、なかなかシフト同時押しの場合の記載がある記事が見つからなかったので作ってみた。

（なお、表の書き方は、同様の内容をまとめてあるこちらのサイトを参考にした。 https://did2memo.net/2019/03/31/karabiner-elements-symbol-table/ ）

## 対応表

### シフトなし

:::note
【凡例】
「記号 (JIS)」：JISキーボードで入出力したい記号
「USでの見た目」： USキーボードで同じ場所にあるキーの見た目
「key_code」：設定すべきキーコード 
:::

|記号 (JIS)|（USでの見た目）[^1]|key_code|
|-|-|-|
|`-` (ハイフン)|`-`|hyphen|
|`,` (コンマ)|`,`|comma|
|`.` (ピリオド)|`.`|period|
|`/` (スラッシュ)|`/`|slash|
|`^` (キャレット)|`=` (イコール)|equal_sign|
|`¥` (円記号)||international3|
|`@` (アットマーク)|`[` (左角括弧)|open_bracket|
|`[` (左角括弧)|`]` (右角括弧)|close_bracket|
|`]` (右角括弧)|`\` (バックスラッシュ)|backslash|
|`;` (セミコロン)|`;`|semicolon|
|`:` (コロン)|`'` (クオート)|quote|
|`_` (アンダーバー)||international1|

### シフトあり

:::note
【凡例】
「記号」：JISキーボード上で入出力したい記号
「キーの見た目 (JIS)」：JISキーボード上でのキーストロークの見た目
「key_code」：設定すべきキーコード 
:::

※ key_code は見やすさ優先で表記。詳しくは注釈 [^2] に記載。

|記号|キーの見た目 (JIS)|key_code [^2]|
|-|-|-|
|`!` (感嘆符)|Shift + 1|Shift + 1|
|`"` (ダブルクオート)|Shift + 2|Shift + 2|
|`#` (シャープ)|Shift + 3|Shift + 3|
|`$` (ドル)|Shift + 4|Shift + 4|
|`%` (パーセント)|Shift + 5|Shift + 5|
|`&` (アンド)|Shift + 6|Shift + 6|
|`'` (クオート)|Shift + 7|Shift + 7|
|`(` (左括弧)|Shift + 8|Shift + 8|
|`)` (右括弧)|Shift + 9|Shift + 9|
|`~` (チルダ)|Shift + `^`|Shift + equal_sign [^3]|
| &#124; (縦棒)|Shift + `¥`|Shift + international3|
| &#096; (バッククオート)|Shift + `@`|Shift + open_bracket|
|`+` (プラス)|Shift + `;`|Shift + semicolon|
|`*` (アスタリスク)|Shift + `:`|Shift + quote|
|`{` (左波括弧)|Shift + `[`|Shift + close_bracket|
|`}` (右波括弧)|Shift + `]`|Shift + backslash|
|`<` (左山括弧)|Shift + `,`|Shift + comma|
|`>` (右山括弧)|Shift + `.`|Shift + period|
|`?` (疑問符)|Shift + `/`|Shift + slash|
|`_` (アンダーバー)|Shift + `_`|Shift + international1|

## その他のキー

その他のキーについては、特にJISに限ったことではないけれど、よく使うので全て記載してみた。

JISに特に関係するキーは、半角/全角、英数、かな。なかでも半角/全角キーは`grave_accent_and_tilde` [^8] であることだけでも知っておいてもいいかも。

### 文字・数字キー

|キー|key_code|
|-|-|
|`a`|`a`|
|`0`|`0`|

### テンキー

|キー|key_code|
|-|-|
|`1`|keypad_1|
|NumLock|keypad_num_lock|
|`/`|keypad_slash|
|`*`|keypad_asterisk|
|`-`|keypad_hyphen|
|`+`|keypad_plus|
|`.`|keypad_period|
|`,`|keypad_comma|
|(エンター)|keypad_enter|

### 特殊キー

|キー|key_code|
|-|-|
|英数|japanese_eisuu (または lang2)|
|かな|japanese_kana (または lang1)|
|Delete or Backspace|delete_or_backspace|
|Return or Enter (&#x21a9;)|return_or_enter|
|スペース|spacebar|
|タブ|tab|
|ESC|escape|
|Command (&#x2318;)|left_command [^4]|
|Shift (&#x21E7;)|left_shift [^4]|
|Control (&#x2303;)|left_control [^4]|
|Option (&#x2325;)|left_option [^4]|
|CapsLack (&#x21EA;)|caps_lock|
|fn|fn|
|上矢印|up_arrow [^5]|
|F1|f1 [^6]|
|半角/全角|grave_accent_and_tilde [^8]|
|PrintScreen|print_screen|
|ScrollLock|scroll_lock|
|Pause|pause|
|Insert|insert|
|Home|home|
|End|end|
|(無効にする)|vk_none|

### 機能キー

|キー|key_code|
|-|-|
|明るさ調整 下げる|display_brightness_decrement|
|明るさ調整 上げる|display_brightness_increment|
|音声コントロール|dictation|
|早戻し|rewind|
|再生・一時停止|play_or_pause|
|早送り|fast_forward|
|消音|mute|
|音量 下げる|volume_decrement|
|音量 上げる|volume_increment|
|取り出し|eject|
|ミッションコントロール|mission_control|
|スポットライト|spotlight|
|Launchpad|launchpad|
|ダッシュボード|dashboard|

その他のキーについては、https://github.com/pqrs-org/Karabiner-Elements/issues/925 などを参考。

## 使用例

以下、JISキーボードにおけるComplex Modificationの使用例 [^7]。

- Shift + `[`（左角括弧） を `^`（キャレット）に置換（Programmer's Dvorak配列の例）

```json
{
    "type": "basic",
    "from": {
        "key_code": "close_bracket",
        "modifiers": {
            "mandatory": [
                "left_shift"
            ]
        }
    },
    "to": [
        {
            "key_code": "equal_sign"
        }
    ]
}
```

- 日本語入力中に `:`（コロン）をバックスペースに置換（新下駄配列の例）

```json
{
    "type": "basic",
    "conditions": [
        {"type": "input_source_if", "input_sources": [{"language": "ja"}]}
    ],
    "from": {"key_code": "quote"},
    "to": [{"key_code": "delete_or_backspace", "repeat": true}]
}
```

- 日本語入力中に `S` + `;`（セミコロン） を `そ`（s,o）に置換（新下駄配列の例）

```json
{
    "type": "basic",
    "conditions": [
        {"type": "input_source_if", "input_sources": [{"language": "ja"}]}
    ],
    "from": {"simultaneous": [{"key_code": "s"}, {"key_code": "semicolon"}]},
    "to": [{"key_code": "s"}, {"key_code": "o", "repeat": false}]
}
```

[^1]: JISキーボードで同場所にあるキーの配置は、[こちらの図](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/44103/87b1ae53-3b02-474f-022c-e80086f6e06c.png)が参考になる（http://www.nagasaki-gaigo.ac.jp/toguchi/pc/multilingual/keyboard_us_jis.htm より引用）。

[^2]: key_code に例えば `Shift + open_bracket` と記載がある場合は、 `{"key_code": "open_bracket", "modifiers": ["left_shift"]}` であることを示す。

[^3]: `~` (チルダ)を入出力するときのJISキーボード上の見た目は `Shift + ^` だが、`^`と同位置にあるUSキーボードの見た目が`=` (イコール)であるため、前項の「シフトなし」の表と照らし合わせて、キーコードは`Shift + equal_sign`となる。この点がとてもややこしい。

[^4]: 右は `right_`

[^5]: 右は `right_`、上は `up_`、下は `down_`

[^6]: 他は `f2`、`f3` など。明るさ調整や音量キーは別記。

[^7]: 実際には設定ファイルのrules > manipulators以下に記載。参考：https://qiita.com/s-show/items/a1fd228b04801477729c

[^8]: 半角/全角キーのUSキーボードでの見た目はバッククオート（ &#096; ）。グレイヴ・アクセントともいい、USではチルダ（`~`）を入力するためのキーでもあるので、キーコードは `grave_accent_and_tilde`。でも、[グレイヴ・アクセント（アクサン・グラーヴ）としての用法](https://ja.wikipedia.org/wiki/%E3%82%B0%E3%83%AC%E3%82%A4%E3%83%B4%E3%83%BB%E3%82%A2%E3%82%AF%E3%82%BB%E3%83%B3%E3%83%88)は日本語入力にもコーディングにも使わないので、個人的には名称がちょっと意外だった。欧米圏だと馴染みがありそう。
