---
theme: purplin
highlighter: prism
lineNumbers: false
info: |
  ## Linux day 2023
  Processi, Demoni e Zombie!
css: unocss
align: center

canvasWidth: 800


layout: image-x
image: ../assets/linux.jpeg
imageOrder: 2
---

# Processi, demoni e zombie!

Linux day 2023

---

# Who am I?

- Ingegnere e architetto software ([platformatic.dev](https://platformatic.dev/) )
- Imprenditore ([remotamente.it](https://remotamente.it/)  )
- Consulente freelance 
- Maker 
- Linux user dagli anni '90

<div class="flex">
  <img src="/assets/plt.svg" width="100" >
  <img src="/assets/remotamente.svg" width="150" >
</div>

---

# Un po' di (mia) storia

- Commodore 64 (1985)
- Amiga 500 (1988)
- i486 / Pentium (1991/1998)
- Approx 1996: Linux 
- 1999: abbandono Windows definitivamente

---

# Come funziona un sistema linux?

- Al centro di tutto c'è il kernel

<div h-full flex justify-center items-center>
   <img text-center  src="/assets/kernel.png" width="300"> 
</div>

---

# Startup del SO
- l'HW esegue un software (BIOS) che verifica l'hw
- il BIOS fa partire da un dispositivo di boot (disco, usb, rete) un bootloader
- il bootloader carica il kernel in memoria (da `/boot/vmlinuz`) e lo esegue
- Il kernel, una volta caricato, `/sbin/init` e lo esegue con pid 1

```bash
$ ps -aux | grep /sbin/init

```

- Quindi `init` è il primo <b>processo</b> che viene eseguito dal kernel. 
- Ma cos'e' un processo?

---

# Processi

- <b>Un processo è un programma in esecuzione</b>
- Essendo `init` il primo e <b>unico</b> processo creato dal kernel, il suo compito e' quello di fare lo spawn di ogni altro processo nel sistema. 
- In sintesi un sistema unix e’ fatto da un processo, che crea altri processi, che creano altri processi, etc
- i processi in esecuzione sono visibili con il comando `ps`

```bash
$ ps -aux
```

--- 

# Processi

- Ma quindi tutti i processi sono organizzati ad albero?

```bash
$ pstree
```
Il "babbo" di tutti i processi e' il processo con pid 1, che e' `init` (o `systemd`)

```bash
ls -lah /sbin/init
```
Ma questo implica che un processo deve essere in grado di creare altri processi. Come fa?

---

# Fork

```python
import os, time

print("Starting")

# ritorna il child process id nel parent e 0 nel child
pid_child  = os.fork()

if pid_child > 0:
    print("I just created a child with PID:", pid_child)
    time.sleep(5);
    print("Parent done")
else:
    print("I am the child")
    time.sleep(1);
    print("Child done")

```

--- 

# `htop`

- Usiamo `htop` per vedere i processi in esecuzione
- E' comodo perche' puo' mostrare i processi in un albero, e ha una opzione per filtrarli

   <img text-center  src="/assets/htop.png" > 

---

# `kill`

- Nonostante il nome, kill e’ semplicemente il comando che manda segnali ai processi (di solito “termina”).- `SIGTERM` o 15 e’ il segnale di default. 
- Significa “termina” ma puoi effettuare delle operazioni di cleanup (rilasciare risorse).


```bash
$ kill -15 1234

```

- `SIGKILL` / 9 : muori subito

---

# Demoni

- Quando un comando e’ lanciato in una shell, ha un terminale collegato e ricevera’ un `SIGHUP` quando il processo che controlla (la shell) esce.
- Un demone e’ un processo che gira in background cioe' non e’ collegato a un terminale (e quindi non termina se la shell che lo ha lanciato muore). 
- See https://en.wikipedia.org/wiki/Daemon_(computing) 
- Come si crea un demone? In molti modi, ma il piu’ semplice e’ usare `nohup`. Usando `nohup` se `sdtout` e `stderr` sono collegati a un terminale, li redirige su `nohup.out` e `SIGHUP` viene ignorato

---

# Demone in python

```python
import os, time

print("I am a daemon. My PID is %d" % os.getpid())
while True:
  print("All work and no play makes Jack a dull boy")
  time.sleep(2)
```

- Se lo lanciamo “normalmente” pero’ non cambia nulla. 
- Proviamo a lanciarlo in “backgroud”: `python3 daemon.py &`
- Ora proviamo con: `nohup python3 -u daemon.py &`

--- 

# Zombie

- In Unix, un processo non può’ essere rimosso dalla tabella dei processi fino anche la sua morte non e’ riportata al padre. 
- Quando un processo “muore”, rimane in giro in uno stato di “zombie” fino a che l’exit status viene recuperato.
- Per vederli:

```bash
ps aux | grep ZN+
 5113 pts/0    Z+     0:00 [python] <defunct>

# contarli con:
ps aux | grep ZN+ | wc -l
```

--- 

# The Zombie Daemon

```python
import os, sys, time

print("I am a zombie daemon. My PID is %d" % os.getpid())
while True:
    time.sleep(2)
    pid_child  = os.fork()
    if pid_child > 0:
        print("The zombie daemon %d just created the zombie with PID:", pid_child)
    else:
        print("I am a zombie!!!! RUN!!!!!")
        sys.exit(); # This is actually changing the process in a zombie
```

- Da lanciare con: `nohup python3 -u zombie.py &`
- Gli zombie non possono essere uccisi...sono gia' morti!

--- 

# Let's kill the Zombie Daemon!

```bash
✦ ➜ ps aux | grep zombie     
marco    1466015  0.0  0.0  31756  8960 pts/5    SN   08:43   0:00 python3 -u zombie.py

/work/workspaces/workspace-linuxday2023 
✦ ➜ kill -9 1466015
[1]  + 1466015 killed     nohup python3 -u zombie.py
```

---

# Grazie!

- Un po' di riferimenti:
  - [The Art of Unix Programming](http://www.catb.org/~esr/writings/taoup/html/)
  - [Daemons (computing)](https://en.wikipedia.org/wiki/Daemon_(computing))
  - [sli.dev](https://sli.dev/) 
