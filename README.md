# qmk-romaji
QMK-Firmware, Japanese Romaji input using simultaneous keying
- maintainer: Bottilabo
- original repository: https://github.com/bottilabo/qmk-romaji

QMK Firmware で、同時打鍵ローマ字入力を行います。QWERTY配列では同時打鍵は難しいでしょう。最初から同時打鍵ローマ字入力を行うために設計された[Harmony配列](https://github.com/bottilabo/harmony-keyboard-layout)との組み合わせをおすすめします。


## 同時打鍵変換規則
スペースキーを`<SPC>`と表記し、同時打鍵ごとの区切りを「,」で表します。

- 基本的には、子音１つ[KSTNHMYRW] + 母音１つ以上[AIUEO]の組み合わせです
- y は「や」行ですが、他の子音と組み合わせると　kya -> きゃ　のように開拗音になります。
- 母音が同時に押された場合は、必ずaoueiの順番で出力されます。同時打鍵では「iu」の順番では出力できないので、その場合は同時打鍵ではなく２回に分けて入力する必要があります。
- SPACEキーの同時打鍵で、最後に「nn」が追加されます。「kio`<SPC>`」の同時打鍵で「koinn」「こいん」になります。
- SHIFTキーの同時打鍵で、「KSTHF」は「GZDBP」になり、「かさたはふぁ」は「がざだばぱ」になります。ただし、単打でBS，組み合わせでSHIFTとなるSFT_T(KC_BSPC)のようなシフトキーの設定の場合は動作しません。

### 例

- わんちゃん `WA<SPC>,TYA<SPC>`
- さいかい `SAI,KAI`
- ほうたい `HUO,TAI`
- いっしゅん `I,S,SUY<SPC>`
- しょうきん `SUOY,KI<SPC>`


# QMK Firmware での組み込み方

## config.h
同時打鍵と判定する時間（ミリ秒）を定義してください。

```
#define ROMAJI_TERM 75
```

## keymap.c
enum custom_keycodes に以下を追加してください。
- キーにINC_PARAM,DEC_PARAMを割り当てると、動的に同時打鍵と判定する時間を調整できます。
- キーにEISU,ROMAJIを割り当てると、GUIでの入力モードの切替と同時打鍵ローマ字入力モードを一度に切り替えることができます。

```
  EISU,
  ROMAJI,
  REPORT_PARAM,
  INC_PARAM,
  DEC_PARAM,
```

上の定義の後で、
```
#include "romaji.c"
```
して、最後に　process_record_user　に以下のように追加してください。
```
bool process_record_user(uint16_t keycode, keyrecord_t *record) {
  if( ! process_romaji(keycode,record) )
    return false;
  
  switch (keycode) {
    // ......
    // 既存のコード
    // ......
    case EISU:
      if (record->event.pressed) {
        romaji_reset();
        _romaji_mode = false;

        if(keymap_config.swap_lalt_lgui==false){
          register_code(KC_LANG2);
        }else{
          SEND_STRING(SS_LALT("`"));
        }
      } else {
        unregister_code(KC_LANG2);
      }
      return false;
    case ROMAJI:
      if (record->event.pressed) {
        romaji_reset();
        _romaji_mode = true;

        if(keymap_config.swap_lalt_lgui==false){
          register_code(KC_LANG1);
        }else{
          SEND_STRING(SS_LALT("`"));
        }
      } else {
        unregister_code(KC_LANG1);
      }
      return false;
  }
  return true;
}
```

これだけで同時打鍵ローマ字入力ができるようになるはずですが、プログラムのサイズが増えてしまうため、液晶やLEDなどの機能を削らなければならないかもしれません。

# Windowsでの漢字入力モードの切替

MacOSではこのままで動作しますが、WindowsではKC_LANG1,KC_LANG2を、「変換キー」「無変換キー」にAutoHotKeyを使って変換してやるとうまくいきます。  
  
[d-yoshi's blog ErgoDoxのキーマップを考える](https://d-yoshi.github.io/keyboard/ergodox/2017/03/21/my-ergodox-keymap.html)
```
;KC_LANG1 -> 変換、KC_LANG2 -> 無変換
sc071 Up::Send,{vk1Dsc07B}
sc072 Up::Send,{vk1Csc079}
```



