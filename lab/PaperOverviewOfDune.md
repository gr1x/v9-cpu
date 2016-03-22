# Overview of "Dune: Safe User-level Access to Privileged CPU"


从应用层直接访问并操纵内核层的硬件设备信息，一直是提高应用程序性能的有效途径。但通用操作系统从安全性和隔离的角度出发，并不提供此类底层访问接口。因此，为了访问特定底层硬件设备，通常需要修改操作系统内核，以向应用层暴露底层设备接口；或者直接将应用集成到内核中，作为guest os利用虚拟化技术对底层设备进行访问，但性能又受虚拟化的效率制约。

本文作者提出了一种新的方法，即利用硬件虚拟化为上层应用提供一种进程抽象模型，而非传统的计算机抽象模型。同时，在x64 Linux上实现了Dune，分为内核模块和libDune，内核模块用于向用户层提供访问硬件的接口，libDune为应用访问底层Dune接口提供了通用抽象。


Dune的主要原理是利用了硬件提供的VT-x虚拟化机制，Host OS Linux操作系统运行于VMX root模式，通过插入Dune内核模块,向libDune提供底层硬件访问接口；VMX non-root模式用于运行Guest OS，即libDune和应用程序，应用程序可以和libDune集成在一起运行在non-root的ring0，也可以在libDune的上层，运行于non-root的ring3。

Dune的核心优势在于通过VMCALL支持应用访问HostOS的POSIX接口，同时通过libDune访问硬件特权指令，从而在保持应用兼容性时极大提高了数据访问效率。如文中所述，垃圾回收GC可以直接访问页表，读取dirty比特位等提高回收效率，避免了传统GC需要进行Context Switch的开销。

直接访问底层硬件提高了效率，但也引入了安全风险，Duna利用虚拟机的VMCS结构体来进行细粒度的硬件访问控制，利用IOMMU向应用提供IO接口，以便应用直接访问硬件的DMA功能，提高应用IO效率。

文章最后对Duna进行了性能评估，Dune因为引入了VM call/exit的调用开销和EPT引发的TLB miss开销等，对getpid等简单调用相比于原生Linux来说，因为额外增加的开销降低了性能，但对于正常应用如Sandbox、Wedge和GC，Dune提供了较好的性能提升，如GC提高了40.7%。


综合整篇文章来看，对于应用来说，在访问底层硬件设备信息方面，Dune提供的功能类似于ExoKernel，但Dune利用了硬件虚拟化技术将其作为Guest OS，以进程抽象模型提供了通用接口，避免了ExoKernel中需要针对应用进行定制，同时采用虚拟化技术也提高了安全性；在应用兼容性方面，Dune给应用以VM call/exit方式提供正常的POSIX系统接口，保证了对应用的兼容性，降低了移植复杂性。