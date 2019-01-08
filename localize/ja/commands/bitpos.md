文字列内の1または0に設定された最初のビットの位置を返します。

最初のバイトの最上位ビットが位置0にあり、2番目のバイトの最上位ビットが位置8にあるというように、文字列を左から右へのビットの配列と見なして、位置が返されます。

The same bit position convention is followed by `GETBIT` and `SETBIT`.

By default, all the bytes contained in the string are examined.
It is possible to look for bits only in a specified interval passing the additional arguments *start* and *end* (it is possible to just pass *start*, the operation will assume that the end is the last byte of the string. However there are semantic differences as explained later). The range is interpreted as a range of bytes and not a range of bits, so `start=0` and `end=2` means to look at the first three bytes.

範囲を指定するために*start*と*end*が使用されている場合でも、ビット位置は常にビット0から始まる絶対値として返されることに注意してください。

`GETRANGE`コマンドの場合と同様に、文字列の末尾から始まるバイトにインデックスを付けるために、startとendに負の値を含めることができます。ここで、-1は最後のバイト、-2は最後から2番目です。

存在しないキーは空の文字列として扱われます。

@return

@integer-reply

コマンドは、要求に従って1または0に設定された最初のビットの位置を返します。

If we look for set bits (the bit argument is 1) and the string is empty or composed of just zero bytes, -1 is returned.

If we look for clear bits (the bit argument is 0) and the string only contains bit set to 1, the function returns the first bit not part of the string on the right. So if the string is three bytes set to the value `0xff` the command `BITPOS key 0` will return 24, since up to bit 23 all the bits are 1.

Basically, the function considers the right of the string as padded with zeros if you look for clear bits and specify no range or the *start* argument **only**.

However, this behavior changes if you are looking for clear bits and specify a range with both **start** and **end**. If no clear bit is found in the specified range, the function returns -1 as the user specified a clear range and there are no 0 bits in that range.

@examples

```cli
SET mykey "\xff\xf0\x00"
BITPOS mykey 0
SET mykey "\x00\xff\xf0"
BITPOS mykey 1 0
BITPOS mykey 1 2
set mykey "\x00\x00\x00"
BITPOS mykey 1
```
