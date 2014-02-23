##BCD编码：
<pre>
最常用的BCD编码，就是使用"0"至"9"这十个数值的二进码来表示。这种编码方式，在称之为“8421码”
（日常所说的BCD码大都是指8421BCD码形式）。除此以外，对应不同需求，各人亦开发了不同的编码方法，
以适应不同的需求。这些编码，大致可以分成有权码和无权码两种：
有权BCD码，如：8421(最常用)、2421、5421…
无权BCD码，如：余3码、格雷码…（注意：格雷码并不是BCD码）

BCD码也叫8421码就是将十进制的数以8421的形式展开成二进制，大家知道十进制是0～9十个数组成，
这十个数每个数都有自己的8421码：
0=0000
1=0001
2=0010
3=0011
4=0100
5=0101
6=0110
7=0111
8=1000
9=1001
举个例子：
321的8421码就是
3 2 1
0011 / 0010 / 0001
原因: 0 0 1 1
=8x0+4x0+1x2+1x1
=3
0 0 1 0　=8x0+4x0+2x1+1x0
=2.
0 0 0 1
=8x0+4x0+2x0+1x1
=1
具体:
BCD码是四位二进制码, 也就是将十进制的数字转化为二进制, 但是和普通的转化有一点不同, 
每一个十进制的数字0-9都对应着一个四位的二进制码,对应关系如下: 十进制0 对应 二进制0000 ;
十进制1 对应二进制0001 ....... 9 1001 接下来的10就有两个上述的码来表示 10 表示为00010000
 也就是BCD码是遇见1001就产生进位,不象普通的二进制码,到1111才产生进位10000。
</pre>
 
##ASCII码，参见百度百科


##ASCII码和BCD码互转
```java
public static void main(String[] args) throws IOException {  
        // TODO Auto-generated method stub  
        String s = "AD6DEC74E223517170487C238A7527B0DD0CA1C684FC13F666473E7C08323F9F66473E7C08323F9F66473E7C08323F9F66473E7C08323F9F66473E7 C08323F9F66473E7C08323F9F0000000000000000";  
        byte[] bcd = ASCII_To_BCD(s.getBytes(), s.length());  
        String s1 = bcd2Str(bcd);  
          
    }  
  
    private static byte asc_to_bcd(byte asc) {  
        byte bcd;  
  
        if ((asc >= '0') && (asc <= '9'))  
            bcd = (byte) (asc - '0');  
        else if ((asc >= 'A') && (asc <= 'F'))  
            bcd = (byte) (asc - 'A' + 10);  
        else if ((asc >= 'a') && (asc <= 'f'))  
            bcd = (byte) (asc - 'a' + 10);  
        else  
            bcd = (byte) (asc - 48);  
        return bcd;  
    }  
  
    private static byte[] ASCII_To_BCD(byte[] ascii, int asc_len) {  
        byte[] bcd = new byte[asc_len / 2];  
        int j = 0;  
        for (int i = 0; i < (asc_len + 1) / 2; i++) {  
            bcd[i] = asc_to_bcd(ascii[j++]);  
            bcd[i] = (byte) (((j >= asc_len) ? 0x00 : asc_to_bcd(ascii[j++])) + (bcd[i] << 4));  
        }  
        return bcd;  
    }  
  
    public static String bcd2Str(byte[] bytes) {  
        char temp[] = new char[bytes.length * 2], val;  
  
        for (int i = 0; i < bytes.length; i++) {  
            val = (char) (((bytes[i] & 0xf0) >> 4) & 0x0f);  
            temp[i * 2] = (char) (val > 9 ? val + 'A' - 10 : val + '0');  
  
            val = (char) (bytes[i] & 0x0f);  
            temp[i * 2 + 1] = (char) (val > 9 ? val + 'A' - 10 : val + '0');  
        }  
        return new String(temp);  
    }
```