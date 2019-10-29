### Application Isolation

Separación de un programa o el stack de una aplicación del resto de procesos en ejecución

^^^^^^

### ¿Cómo lo podemos conseguir?

* Utilizando una máquina diferente &#x1F926; <!-- .element: class="fragment" data-fragment-index="1" -->
* En linux, usando chroot <!-- .element: class="fragment" data-fragment-index="2" -->
* Utilizando una máquina virtual <!-- .element: class="fragment" data-fragment-index="3" -->
* Utilizando contenedores <!-- .element: class="fragment" data-fragment-index="4" -->
* Otras tecnologías <!-- .element: class="fragment" data-fragment-index="5" -->
  * [Jails de BSD](https://www.freebsd.org/doc/handbook/jails.html) <!-- .element: class="fragment" data-fragment-index="5" -->
  * [Solaris Zones](https://docs.oracle.com/cd/E37929_01/html/E36580/zonesoverview.html)<!-- .element: class="fragment" data-fragment-index="6" -->
^^^^^^

![Containers vs Virtual Machines](images/containers_vs_virtual_machines.png)<!-- .element: class="plain" -->

<small>[_Fuente: Blog de docker_](https://blog.docker.com/2018/08/containers-replacing-virtual-machines/)</small>

notes:

[https://devopsbootcamp.osuosl.org/application-isolation.html](https://devopsbootcamp.osuosl.org/application-isolation.html)

Virtual Machines are programs that act like (or emulate) another computer (Also called a “guest”) that’s running on your physical computer (The “host”). This is useful because a VM completely isolates programs running from the host computer.

Here is a demonstration showing the processes inside of a VM versus a host OS

<pre>
[vm] # ps aux
USER PID %CPU %MEM    VSZ   RSS TTY STAT START   TIME COMMAND
root   1  0.0  0.6 110564  3164 ?   Ss    2015  11:17 /lib/systemd/systemd --system --deserialize 15
root   2  0.0  0.0      0     0 ?   S     2015   0:00 [kthreadd]
root   3  0.0  0.0      0     0 ?   S     2015   3:55 [ksoftirqd/0]
root   5  0.0  0.0      0     0 ?   S<    2015   0:00 [kworker/0:0H]
[... 120+ more lines ...]

[host] # ps aux
USER  PID %CPU %MEM    VSZ   RSS TTY STAT START   TIME COMMAND
root    1  0.0  0.1 200328  5208 ?   Ss   Aug25   0:44 /sbin/init
root    2  0.0  0.0      0     0 ?   S    Aug25   0:00 [kthreadd]
root    3  0.0  0.0      0     0 ?   S    Aug25   0:05 [ksoftirqd/0]
root    5  0.0  0.0      0     0 ?   S<   Aug25   0:00 [kworker/0:0H]
[... 240+ more lines ...]

</pre>
 
### OS Emulation

Let’s talk about that emulation problem. Imagine you are running a computer that has an X86_64 CPU on it. You want to emulate a computer with an ARM5 CPU. The emulated operating system makes system calls to the ‘hardware’ it thinks it is running on. The host operating system translates those system calls into real system calls on the actual hardware. This translation is cost intensive and usually slow, but it’s gotten a lot better over the decades. When emulating a separate CPU architecture, optimizations usually have to be made for the emulated OS to be usable.

When you are emulating an X86_64 VM on an X86_64 piece of hardware, an optimization that can be made in which the host OS passes the system calls directly to the hardware without having to translate anything. This is done by a hypervisor which enforces certain security protocols so the two operating systems (host and guest) are still isolated, but things go much faster.


^^^^^^
El comando ps en un sistema virtualizado

<pre>
[vm] # ps aux
USER PID %CPU %MEM    VSZ   RSS TTY STAT START   TIME COMMAND
root   1  0.0  0.6 110564  3164 ?   Ss    2015  11:17 /lib/systemd/systemd --system --deserialize 15
root   2  0.0  0.0      0     0 ?   S     2015   0:00 [kthreadd]
root   3  0.0  0.0      0     0 ?   S     2015   3:55 [ksoftirqd/0]
root   5  0.0  0.0      0     0 ?   S<    2015   0:00 [kworker/0:0H]
[... 120+ more lines ...]

[host] # ps aux
USER  PID %CPU %MEM    VSZ   RSS TTY STAT START   TIME COMMAND
root    1  0.0  0.1 200328  5208 ?   Ss   Aug25   0:44 /sbin/init
root    2  0.0  0.0      0     0 ?   S    Aug25   0:00 [kthreadd]
root    3  0.0  0.0      0     0 ?   S    Aug25   0:05 [ksoftirqd/0]
root    5  0.0  0.0      0     0 ?   S<   Aug25   0:00 [kworker/0:0H]
[... 240+ more lines ...]

</pre>


^^^^^^

El comando ps en un contenedor

<pre>
[container] $ ps aux
PID   USER     TIME   COMMAND
1     root     0:00   sh
6     root     0:00   ps aux
</pre>

^^^^^^

En lugar de emular el "guest SO", un contenerdor usa el mismo kernel que el host pero de 
alguna manera "miente" al proceso que se ejecuta en el contenedor y le hace creer que es la 
única aplicación que se está ejecutando en ese SO.

notes:

[https://devopsbootcamp.osuosl.org/application-isolation.html#containers](https://devopsbootcamp.osuosl.org/application-isolation.html#containers)

Containers approach application isolation from a different angle. Instead of emulating the guest OS, containers use the same kernel as the host but lie to the guest process and tell it that it’s the only application running on that OS. Containers bypass the emulation problem by avoiding emulation altogether. They run on the same hardware as the host OS but with a thin layer of separation.

Containers have very become popular recently, but their underlying technologies aren’t new. Many application developers and system administrators have begun migrating toward using Containers over VMs as they tend to be more performant, but the industry as a whole is waiting for them to get a bit more battle-tested.

Here is the previous demonstration in a container:

<pre>
[container] $ ps aux
PID   USER     TIME   COMMAND
1     root     0:00   sh
6     root     0:00   ps aux
</pre>

As you can see, instead of emulating an entire OS (running 100+ processes), the container is told that it’s processes (sh and ps in this case) are the only one in this environment. In theory this prevents a malicious attack from inside the container from invading the host OS.

^^^^^^

La forma de "mentir" es utilizar Control Group ([cgroups](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/resource_management_guide/index)) del kernel de Linux.

notes:
[https://devopsbootcamp.osuosl.org/application-isolation.html#container-technologies](https://devopsbootcamp.osuosl.org/application-isolation.html#container-technologies)

CGroups
A Linux kernel-level technology that name-spaces processes. It performs many functions, but it’s used by container engines to convince a process that it’s running in its own environment. This is what isolates a process from other processes, if they think they’re the only thing running they can’t tamper with the host OS.
