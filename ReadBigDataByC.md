File size 8.8G
File format like :
@SRR645377.1 A80549ABXX:2:1:4080:1112/1
ACCCTNCACCCATAGTCAGTCCCTCCACCCCCGGCAGAGATCGAGGAGCTTCCCAGGAGGCGGTGTTGCAGGCGTGGGACTCAGATCGTGCTGCTGGGGC
+
bbbbbBbb`bbbbabfffffffffffffffffffffffffeffefefefdfffffffdffeed`d^bddddb^a`bbdRbcc\dYbbYabcbaa^`bbdb
@SRR645377.2 A80549ABXX:2:1:4926:1114/1
AAGAANAAAGGGGACAAAGAAACTAAAACAAAGGAGTCAAACTTACGATTTTCCCGGACATAGTTGTTGCCTGCAGTGTCCACCTGGAGCCCTTAAAAAT
+
_V\_YBWYV]`_`[`ffdfeeeffefeaff_`bddcdeeeffefedeacffefdeedfefefdcfdcddeeeee`dadbde^eeedfbedfffff^^d^d
@SRR645377.3 A80549ABXX:2:1:7678:1113/1
ATGCCNTTCTGGCTTGTATACTGGGTGTGAAACAACTAATTGTTGGTGTTAACAAAATGGATTCCACTGAGCCATCCTACAGCCAGAAGAGATATGAAGA
+
aaaaaBaaRaaaaaa^`acbbbccc\b]b_dddddccdddddcdddadadaddddddd`daad_dccaac_aa_^bcaabbddddaYa``caaaaa^cdT
这是三个连着的相同结构的长字符串:例如
@SRR645377.1 A80549ABXX:2:1:4080:1112/1
ACCCTNCACCCATAGTCAGTCCCTCCACCCCCGGCAGAGATCGAGGAGCTTCCCAGGAGGCGGTGTTGCAGGCGTGGGACTCAGATCGTGCTGCTGGGGC
+
bbbbbBbb`bbbbabfffffffffffffffffffffffffeffefefefdfffffffdffeed`d^bddddb^a`bbdRbcc\dYbbYabcbaa^`bbdb
把这个作为一个单元，将这个单元里的结构按空格与换行再细分为5部分；例如@SRR645377.1；A80549ABXX:2:1:4080:1112/1；
ACCCTNCACCCATAGTCAGTCCCTCCACCCCCGGCAGAGATCGAGGAGCTTCCCAGGAGGCGGTGTTGCAGGCGTGGGACTCAGATCGTGCTGCTGGGGC；+；
bbbbbBbb`bbbbabfffffffffffffffffffffffffeffefefefdfffffffdffeed`d^bddddb^a`bbdRbcc\dYbbYabcbaa^`bbdb
分别存储在数组或vector中，问题来了，文件比较大，元素个数已成百十亿级别，超过数组或vector的最大限制。
我的处理：
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
#include<stdio.h>
#include<stdlib.h>
#include<string.h>
int main(){
    int fd;
    void *start;
    struct stat sb;
    fd = open("DNA_gynes_1.fq", O_RDONLY); /*打开/etc/passwd */
    fstat(fd, &sb); /* 取得文件大小 */
    start = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
    if(start == MAP_FAILED) /* 判断是否映射成功 */
        return;
    int i;
    char *pstart=start;
    for (pstart=start;pstart!=start+500;pstart++){
        putchar(*(pstart));}
    printf("\n");
    munmap(start, sb.st_size); /* 解除映射 */
    close(fd);
    return 0;
}
将文件读入到流中，文件流起始位置start，终止位置*(start+sb.st_size)；
类似将文件读取为一条很长的字符串（包括空格、换行）；可以通过指针访问数据，
但是位置很模糊，不能快速定位，不像数组，可以快速准确定位。
我的想法：可以构建快速提取所需字符的索引，按规律，根据索引，快速查找所需字符串。
