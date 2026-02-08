# datalab 总结




## 解题总结

### bitAnd
```c
int bitAnd(int x, int y) {
    return ~(~x|~y);
}
```
由德摩根率，先取反再取反等于本身，其中一个取反放进括号内，成功将且转变为或

### bitXor
```c
int bitXor(int x, int y) {
    return (~(x&y)&~(~x&~y));
}
```
异或，即相异为真，相同为假，只给了与、非，那么考虑用与来翻译，则“xy相同”包括同为真同为假两种，两种只要有一种成立则为假，当且仅当两种都不成立才为真，得~(~(x&y)&~(~x&~y))

### samesign
```c
int samesign(int x, int y) {
    if(!x){
    return !y;}
 if(!y){
  return 0;}
 return !((x^y)&(1<<31));
}
```
先将xy中有0的情况单独讨论，再通过异或取最高位判断异同

### logtwo
```c
int logtwo(int v) {
    int r=0;
    r|=((v>>16)>0)<<4;
    r|=((v>>(r|8))>0)<<3;
    r|=((v>>(r|4))>0)<<2;
 r|=((v>>(r|2))>0)<<1;
 r|=((v>>(r|1))>0);
    return r;
}
```
先看左半边是否含有1，若有，右移16位并对结果进行加16，否则不进行操作；
此时左半边必然为0，然后再对右半边的左半边进行同样的操作（实际r|8使得上两次右移一次性完成）再根据左边是否有1决定是否对r进行累加以及是否对数进一步右移

### bitswap
```c
int byteSwap(int x, int n, int m) {
    int p,q,c=n<<3,d=m<<3;
    p=(x>>c)&(0xFF);
 q=(x>>d)&(0xFF);
 
    return (((x^(p<<c))|(q<<c))^(q<<d))|(p<<d);
}
```
先将m,n乘8因为题目以一个字节为最小单位，x分别右移c，d位并与0xFF取且，从而取出第n个、第m个字节
再将pq左移使之恢复到x中对应位置，与x取异或从而抹除x第m、n位字节，再分别交换pq的为止与x取或，从而把交换后的mn字节情况'赋值'上去

### reverse
```c
unsigned reverse(unsigned v) {
 v=((v>>1)&0x55555555U)|((v&0x55555555U)<<1);
 v=((v>>2)&0x33333333U)|((v&0x33333333U)<<2);
 v=((v>>4)&0x0F0F0F0FU)|((v&0x0F0F0F0FU)<<4);
 v=((v>>8)&0x00FF00FFU)|((v&0x00FF00FFU)<<8);//注意永0x0F0F0F0FU这个U不能少
 v=(v>>16)|(v<<16);
 
 
 
 return v;
}
```
经典的自交换方法，奇数偶数位多次交换，不断扩大奇数偶数段的长度

### logicalshift
```c
int logicalShift(int x, int n) {
     
    return (x>>n)&(~(((1<<31)>>n)<<1));
}
```
先进行逻辑右移，然后与前n位为0、后32-n位为一的掩码取交。
掩码生成思路：尝试过程中曾使用~(1<<(32-n))首先发现不能用减号，改为~((1<<(32))>>n),注意到不能左移32位，故最后改为~(((1<<31)>>n)<<1)

### leftBitCount
```c
int leftBitCount(int x){
 int c=0;
 int y=~x;
 int s=!(y>>16)<<4;
 c+=s;
 y=y<<s;
 s=!(y>>24)<<3;
 c+=s;
 y=y<<s;
 s=!(y>>28)<<2;
 c+=s;
 y=y<<s;
 s=!(y>>30)<<1;
 c+=s;
 y=y<<s;
 s=!(y>>31);
 c+=s;//很明显最后两个只检查了一个，漏了一个所以全零要补上1
  return c+(!~x);
 
}
```
首先对x取反得到y，此题改为求解最左边的1的位置，先右移16位检查左半边是否有1，若无，说明x左半边全为1，则r加16，y左移16位，后续仅对y右半边继续上述步骤，否则说明x左半边存在0，无需继续研究右半边，故r不增加，后续继续研究左半边的左半边是否有1，操作同上，最终一步时对剩余未检查的两位仅检查了其中一位，导致如果x全1（只有x其他位全是1才能同故宫特定顺序的左移到这一步，而这一步会导致出现误差的前提是没检查的那一位在x中是1，所以总的来说只有全一的x能导致这个误差），则会少计算一个1，所以用“+(!~x)"补齐误差

### float_i2f
```c
unsigned float_i2f(int x) {
 if(x==0) return 0;
int ax=x;
if(x<0) ax=~x+1;
int e=158;
while(!((ax>>31)&1)){
e+=~0;//非法运算符
ax<<=1;}
int f=(ax>>8)&0x7fffff;
int r=ax&0xff;
int add=(r>128)|((r==128)&(f&1));
f+=add;
e+=f>>23;
f&=0x7fffff;
return (x&0x80000000)|(e<<23)|f;
}
```
若为0直接返回0
若小于0由补码还原回原码
首先假设exp（这里是e）为int范围内的最大正数对应阶码值127+30（由于后续符号位为0也会导致e减一，所以这里e会多加1的158）
然后首先只要x的原码（即ax）最左位是0，就将ax左移一位，e减一，直到ax最左边的位置是1
然后取此时ax的除最高位以外所有位，即待处理的尾数，为了考虑舍入问题，r取这右31位中的最右边8位（刚好在23位之外）
此时当且仅当ax右8位过半（超过128），或者刚好为128且ax右数第9位为1时需要舍入
舍入之后考虑进位溢出问题，当且仅当ax中右数第9位到右数第31位全一时会出现舍入，此时只需尾数清零（等价于f&=0x7fffff），阶码加一（等价于e+=f>>23，完成了对f是否进位的检查与反应）
最后左右移动调整好位置即可

### floatScale2
```c
unsigned floatScale2(unsigned uf) {
    unsigned sign=uf&0x80000000u;
    unsigned e=uf&0x7f800000u;//变量名输入错误
unsigned frac=uf&0x007fffffu;
    if(e==0x7f800000u){
     return uf;}
     if(!e){
      frac<<=1;
      if(frac&0x00800000u){
       frac&=0x007fffffu;
       e=0x00800000u;
       
      }
      return sign|e|frac;}
 if(e+0x00800000==0x7f800000){
  return sign|(e+0x00800000);
 }
 return sign|(e+0x00800000)|frac;
      
}
```
先分别取出符号位、阶码、尾数
首先考虑阶码全为一，可能为NaN或无穷，依题意返回原数
其次考虑阶码全0，首先使尾数左移一位模拟乘二，若进位溢出（frac&0x00800000u为true），就转变为规格化数，阶码加一，尾数取后23位，若未溢出无特殊操作，最后拼接符号位、阶码、尾数输出
再次考虑乘二后超出float最大表示范围者，则使其阶码最大，尾数清零，拼接符号位输出
最后是以上情况之外，阶码“加一”后拼接输出

### float64_f2i
```c
int float64_f2i(unsigned lo, unsigned hi) {
    int sgn = (hi >> 31) & 1;  
    

    unsigned efield = (hi >> 20) & 0x7ff;  // 
    unsigned exp_bias = 1023;
    unsigned exp_val = efield - exp_bias;  // 
```
提取符号位,提取11位阶码,计算无符号指数
```c
    if (efield < exp_bias) return 0;  //
    if (exp_val > 31) return 0x80000000;  // 
```
 下溢判断与上溢判断
 ```c
    // 
    unsigned frac_hi20 = hi & 0x000fffff;
    unsigned ph = (1 << 20) | frac_hi20;
    unsigned pl = lo;

    unsigned shift = 52 - exp_val;  // 
    unsigned qh, ql;  // 
 ```
构造含隐式前导1的尾数,计算移位量,存储移位后结果的高、低32位
```c
    // 
    if (!shift) {
        qh = ph;
        ql = pl;
    } else if (shift < 32) {
        qh = ph >> shift;
        ql = (ph << (32 - shift)) | (pl >> shift);
    } else {
        unsigned s2 = shift - 32;
        qh = 0;
        ql = ph >> s2;
    }

    if (qh) return 0x80000000;  // 
```
分场景处理移位（用!判断shift是否为0）,高32位非零则溢出
```c
    // 
    if (!sgn) {
        if (ql > 0x7fffffff) return 0x80000000;
        return ql;
    } else {
        if (ql > 0x80000000u) return 0x80000000;
        // 
        if ((ql & 0x80000000u)) {
            if (! (ql & 0x7fffffff)) {
                return 0x80000000;
            }
        }
        unsigned twc = (~ql) + 1;  // 
        return twc;
    }
}
```
处理符号和数值范围（用!判断sgn是否为0）,拆分逻辑为嵌套if，避免&&,计算补码

### floatPower2

```c

unsigned floatPower2(int x) {
    if(x<-149){
     return 0u;
     
    }
 if(x<=-127){
  return 1u<<(x+149);
 }
 if(x>=128){
  return 0x7f800000u;
 }
 return ((x+127)<<23);
}

```

要考虑本身过大溢出、本身非规格、本身过于接近0的特殊情况；
本身过于接近0，即比能表示的最小的数（2的负149次方）还要小，此时返回0；
本身过小导致需要非规格化才能表示，即比最小规格化数（2的负126次方）小，计算“1”应左移多少位，由于可知2的负149次方的1在最右边，x每增大1，1左移一位，故共左移（149+x）位；
本身过大超出float范围，此时返回正无穷（0x7f800000）；


### 亮点



1. floatPower2
2. float_i2f
3. leftBitCount




## 参考的重要资料

9月17日ics课堂ppt
《深入理解计算机系统》
