File size 8.8G<br>

File format like :<br>
@SRR645377.1 A80549ABXX:2:1:4080:1112/1<br>
ACCCTNCACCCATAGTCAGTCCCTCCACCCCCGGCAGAGATCGAGGAGCTTCCCAGGAGGCGGTGTTGCAGGCGTGGGACTCAGATCGTGCTGCTGGGGC<br>
+<br>
bbbbbBbb`bbbbabfffffffffffffffffffffffffeffefefefdfffffffdffeed`d^bddddb^a`bbdRbcc\dYbbYabcbaa^`bbdb<br>
@SRR645377.2 A80549ABXX:2:1:4926:1114/1<br>
AAGAANAAAGGGGACAAAGAAACTAAAACAAAGGAGTCAAACTTACGATTTTCCCGGACATAGTTGTTGCCTGCAGTGTCCACCTGGAGCCCTTAAAAAT<br>
+<br>
_V\_YBWYV]`_`[`ffdfeeeffefeaff_`bddcdeeeffefedeacffefdeedfefefdcfdcddeeeee`dadbde^eeedfbedfffff^^d^d<br>
@SRR645377.3 A80549ABXX:2:1:7678:1113/1<br>
ATGCCNTTCTGGCTTGTATACTGGGTGTGAAACAACTAATTGTTGGTGTTAACAAAATGGATTCCACTGAGCCATCCTACAGCCAGAAGAGATATGAAGA<br>
+<br>
aaaaaBaaRaaaaaa^`acbbbccc\b]b_dddddccdddddcdddadadaddddddd`daad_dccaac_aa_^bcaabbddddaYa``caaaaa^cdT<br>
这是三个连着的相同结构的长字符串:例如<br>
@SRR645377.1 A80549ABXX:2:1:4080:1112/1<br>
ACCCTNCACCCATAGTCAGTCCCTCCACCCCCGGCAGAGATCGAGGAGCTTCCCAGGAGGCGGTGTTGCAGGCGTGGGACTCAGATCGTGCTGCTGGGGC<br>
+<br>
bbbbbBbb`bbbbabfffffffffffffffffffffffffeffefefefdfffffffdffeed`d^bddddb^a`bbdRbcc\dYbbYabcbaa^`bbdb<br>

把这个作为一个单元，将这个单元里的结构按空格与换行再细分为5部分；例如@SRR645377.1；A80549ABXX:2:1:4080:1112/1；
ACCCTNCACCCATAGTCAGTCCCTCCACCCCCGGCAGAGATCGAGGAGCTTCCCAGGAGGCGGTGTTGCAGGCGTGGGACTCAGATCGTGCTGCTGGGGC；+；
bbbbbBbb`bbbbabfffffffffffffffffffffffffeffefefefdfffffffdffeed`d^bddddb^a`bbdRbcc\dYbbYabcbaa^`bbdb<br>
分别存储在数组或vector中，问题来了，文件比较大，元素个数已成百十亿级别，超过数组或vector的最大限制。<br>

我的处理：<br>
#include <sys/types.h><br>
#include <sys/stat.h><br>
#include <fcntl.h><br>
#include <unistd.h><br>
#include <sys/mman.h><br>
#include<stdio.h><br>
#include<stdlib.h><br>
#include<string.h><br>
int main(){<br>
    int fd;<br>
    void *start;<br>
    struct stat sb;<br>
    fd = open("DNA_gynes_1.fq", O_RDONLY); /*打开/etc/passwd */<br>
    fstat(fd, &sb); /* 取得文件大小 */<br>
    start = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);<br>
    if(start == MAP_FAILED) /* 判断是否映射成功 */<br>
        return;<br>
    int i;<br>
    char *pstart=start;<br>
    for (pstart=start;pstart!=start+500;pstart++){<br>
        putchar(*(pstart));}<br>
    printf("\n");<br>
    munmap(start, sb.st_size); /* 解除映射 */<br>
    close(fd);<br>
    return 0;<br>
}<br>

将文件读入到流中，文件流起始位置start，终止位置*(start+sb.st_size)；<br>
类似将文件读取为一条很长的字符串（包括空格、换行）；可以通过指针访问数据，<br>
但是位置很模糊，不能快速定位，不像数组，可以快速准确定位。<br>

我的想法：可以构建快速提取所需字符的索引，按规律，根据索引，快速查找所需字符串。<br>
