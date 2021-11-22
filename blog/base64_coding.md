# Base64 Coding

Base64 要求把每三个 8Bit 的字节转换为四个 6Bit 的字节（3\*8 = 4\*6 = 24），然后把 6Bit 再添两位高位 0，组成四个 8Bit 的字节，也就是说，转换后的字符串理论上将要比原来的长1/3。

原文的字节数量应该是3的倍数，如果这个条件不能满足的话，具体的解决办法是这样的：原文剩余的字节根据编码规则继续单独转(1变2，2变3；不够的位数用0补全)，再用`=`号补满4个字节。

关于这个编码的规则：
1. 把3个字节变成4个字节。
2. 每76个字符加一个换行符。
3. 最后的结束符也要处理。

~~~ c
void b64_encode(const uint8_t *src, uint32_t src_len, uint8_t *dst)
{
    /* base64 index table */
    static const uint8_t b64[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
    uint32_t i, j, a, b, c;

    for (i = j = 0; i < src_len; i += 3) {
        a = src[i];
        b = i + 1 >= src_len ? 0 : src[i + 1];
        c = i + 2 >= src_len ? 0 : src[i + 2];

        dst[j++] = b64[a >> 2];
        dst[j++] = b64[((a & 3) << 4) | (b >> 4)];
        if (i + 1 < src_len) {
            dst[j++] = b64[(b & 0x0F) << 2 | (c >> 6)];
        }
        if (i + 2 < src_len) {
            dst[j++] = b64[c & 0x3F];
        }
    }
    while (j % 4 != 0) {
        dst[j++] = '=';
    }
    dst[j++] = '\0';
}
~~~

