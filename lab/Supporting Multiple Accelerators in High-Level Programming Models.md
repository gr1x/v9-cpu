#Supporting Multiple Accelerators in High-Level Programming Models
##一、摘要 
可计算加速器，如多核NVIDIA GPU、Intel Xeon Phi和FPGA，在科学和工程计算的工作站、服务器及超级计算机中正变得越来越常见。充分挖掘这些加速器的大规模并行计算能力，需要在程序的编程模型的设计和实现阶段都对并行计算做有针对性的设计。 
<br>
在本文中，作者设计了在较高层次编程模型就能够支持多加速器的模型，并且实现了对OpenMP的扩展支持，支持从计算机向多种加速器设备批量传输数据以进行计算。这种扩展使得数据计算通过接口平均分布在设备列表中的多个计算设备中，包括多维数组计算、多个加速期之间的共享数据等。计算需求的分布通过将循环迭代分布在多个加速器中进行。作者实现了一种机制，能够自动打包/解包数据，并能够将数组中非连续数据移动到加速器的连续数据区域，这些都不需要CPU的参与。同时，作者定义了多个加速期之间减少技术。结合编译技术和运行时支持，通过异步操作和线程机制，该模型能够支持多个GPU。最后，作者实现对NVIDIA GPU的解决方案，证明该解决方案对OpenMP代码改进具有很好的作用。 
<br>
作者认为，本文的主要贡献包括： 
<br>
1.在OpenMP中实现了针对多种计算加速器的扩展，这些扩展包括数据加载、将多维数组平均分布到多个加速器中、设置多个加速器间的共享数据、将循环迭代分布到多个加速器设备中。 
<br>
2.编译器和运行时的支持，使程序能够支持1中的各种需求，包括内存管理、映射数组时的打包/解包、减少数据移动以及多个加速器的管理。 
<br>
3.实现了一个或多个用户线程同步/异步操作，并对这些操作实现性能分析。 
<br>

##二、问题的提出 
异构计算架构能够将通用计算CPU和多种类型的计算加速器结合，如GPU、MIC和FPGA。当前，多计算加速器正在超级计算机和企业计算服务器中普遍使用。高层次的编程模型如OpenACC和OpenMP都提供了并行计算接口，通过如pragma这样的变成接口实现。但是，这些模型有一个很大的缺点，就是一次只能使用一个加速器进行数据加载和计算。也有很多研究是针对充分利用多个加速器的，比如OpenMP 4.0，能够支持多个设备、实现在程序中对每个加速器进行单独的数据加载。但是截至目前，这些编程接口都需要对每一个加速器手工设置数据解压和计算，并且要小心处理边界条件和在多个加速器间的数据传输、数据同步。 
<br>
文中，作者举了一个并行计算的例子，并对其进行了详细的并行分析。 
<br>

##三、问题的解决 
1.设备类型和虚拟拓扑 
<br>
作者使用device关键字来声明多个设备，同时参考MPI和HPF对多个设备生成虚拟拓扑。 
<br>
2.数据分布 
<br>
使用dist_data子句生命对数组每个维度的分配策略 
<br>
3.Halo区域和Halo更新 
<br>
Halo区域是数据分布情况的注释，仅用在块分布中。作者使用halo子句标识halo区域开始，同时使用halo_update语句来控制halo区域更新操作。 
<br>
4.循环迭代器平均分布 
<br>
当在多个加速器中分配并行循环的数据时，理想的状态是将循环平均分布在每个加速器中。作者使用dist_iteration语句来控制循环的分配。 
<br>

##四、系统实现 
文中的应用针对NVIDIA GPU，生成CUDA代码，使用异构OpenMP编译器作为基础进行实现，主要分为两步：（1）将使用本文提供的语句和指令写成的代码翻译成标准OpenMP 4.0代码，使用单设备编程模型；（2）使用异构OpenMP编译器从标准单设备OpenMP代码生成C/CUDA代码，同时增加对多设备并行计算的支持。 
<br>
1.CUDA内核代码生成 
<br>
CUDA内核使用循环算法分配程序中的循环迭代器，在CUDA内核中映射完整的数据内存，同时使用collapse子句将数据内存的一部分映射到单独的循环中。 
<br>

2.多设备管理 
<br>
对于多设备管理有四种方案： 
<br>
（1）通过同步操作使用单用户线程管理多个设备。这种方式简单方便，缺点是由于线程需要等待每个线程结束后才能进行下一个动作，限制了并行能力。 
<br>
（2）通过多个用户线程或OpenMP线程管理多个设备，每个线程使用同步方式与其他设备通信。这种方式能够使设备具有良好的并行性，但用户态线程可能会因为等待设备完成而阻塞，浪费计算时间； 
<br>
（3）使用单用户线程管理所有设备，但使用异步操作与设备交互。异步操作能够减少等待并使设备效能最大化，但是用户线程仍需要等待设备操作完成，因此这个县城在等待期间也无法做有用的操作； 
<br>
（4）使用一个或多个后台线程管理一个、多个或所有设备。理论上这种方式不会产生用户线程的阻塞，但实际情况是用户线程往往需要等待设备操作完成才能继续进行程序操作，因此这种优势并不是十分明显。 
<br>
文中作者使用了第二种设备管理方式。 
<br>

3.数组和循环迭代的分配 
<br>
传统C和Fortran将多维数组分配在连续的内存空间，然而每个加速器往往只计算数组的一部分元素，因此运行时需要对数组元素划片（需要单独分配空间、计算边界等）和数据传输。 
<br>

4.Halo区域和缩小 
<br>
为了处理Halo区域，运行时需要复制每个子区域的边界元素。由于Halo区域存在于内存的不连续空间，将会在GPU内核的共享内存空间创建一个缓冲区，每个CUDA线程都能够向这个缓冲区中拷贝各自线程Halo区域的数据。如果硬件支持，Halo区域的数据交换可以通过异步CUDA操作完成。如果硬件不支持，运行时将在两个设备间完成完成数据交换。运行时会检查两个对象的硬件可达性，然后决定使用何种交换方式。 
<br>
Halo区域缩小可以分三步方式进行：（1）内部CUDA线程阻止GPU上的缩小；（2）每个设备以异步回调方式执行缩小操作；（3）全局层面的缩小操作在每个设备上通过循环缩小操作完成。 
<br>

##五、系统评估 
实验使用两个4核Intel Xeon 5530 2.4GHz处理器、24GB DDR3 ECC内存。通过使用超线程，每个核能够支持最多16个OpenMP线程。同时试验中使用4个NVIDIA Tesla C2050 Fermi GPU，每个GPU配有448个GPU核和3GB GDDR5内存，配套软件环境使用CUDA v5.5。 
<br>
1.两种管理多个加速器的机制比较 
<br>
两种管理方式分别是：（1）一个OpenMP线程与所有GPU使用异步操作管理；（2）指定多个OpenMP线程管理，每个线程使用同步方式管理一个GPU。实验结果证明，随着GPU数量的增大，性能并没有线性增长。根本的原因在于硬件的限制，因为当有大量数据在内存移动时，共享的数据总线将很快成为瓶颈；GPU数量的增加将导致串行内存操作增加，同时增大了竞争，导致性能下降。 
<br>

2.使用多GPU的性能 
<br>
作者通过向量乘法Y = a*X+Y的性能图证明，大量数据拷贝耗费巨大的计算时间，仅通过增加GPU数量并不能够有效改进计算效率。 
<br>
作者进行了矩阵乘法C(i,j) = sum(k=0 to n)((A(i,k) * B(k,j)) 的运算测试，算法复杂度是O(3)，这里可以采取三种策略：（1）以行为数据块计算；（2）以列为数据块计算；（3）同时以行和列分隔数据块计算。后面两种计算方式的内存并不是连续分布的，因此需要数据打包/解包操作。实验表明，由于硬件结构的限制，多个GPU的数据拷贝是串行的，因此增加GPU并不能带来效率的提高。 
<br>
Jecobi算法用来计算2D视图，作者以行位单位对Jacobi算法进行性能测试。结果显示，该算法具有良好的可扩展性，作者设计的3个实验环境执行时间分别缩小了24%、34%和42%。 