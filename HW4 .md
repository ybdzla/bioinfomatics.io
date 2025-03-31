# 第四次作业
### 致理-生21 孙铭一 2022012361
# Mapping
```
（1）请阐述bowtie中利用了 BWT 的什么性质提高了运算速度？并通过哪些策略优化了对内存的需求？
```
# Bowtie 中 BWT 的作用及内存优化策略

## 一、Bowtie 如何利用 BWT 提高运算速度

Bowtie 使用 **Burrows-Wheeler Transform (BWT)** 和 **FM-index** 来高效查找序列位置。它利用了 BWT 的以下性质：

### 1. 字符分组性（局部性）
- BWT 会将相似字符集中，有利于压缩和缓存优化。

### 2. 可逆性 + 后缀数组的快速搜索能力
- BWT 是可逆的。
- 利用 FM-index 和 BWT 可以进行 **后向查找（backward search）**。
- 匹配一个长度为 `m` 的序列只需 `O(m)` 时间，无需扫描整个参考基因组。

✅ **优势**：避免传统的字符串比对方法（如 Smith-Waterman），大幅提升速度。

---

## 二、Bowtie 如何优化内存使用

Bowtie 通过以下策略大大降低了内存占用：

### 1. 使用 FM-index 压缩索引
- 只需存储：
  - BWT 字符串
  - C 表（每个字符的累计出现次数）
  - Occ 表（字符在特定位置前的出现次数）

### 2. 2-bit 编码
- DNA 中的 A、C、G、T 可用 2 位表示，节省大量空间。

### 3. 压缩的 Occ 表
- 使用稀疏存储 + rank 操作，降低内存消耗同时保持查询效率。

### 4. 分段加载
- 索引可分段加载，无需一次性载入所有数据，适应大基因组环境。

---

## 总结

| 目标           | 实现方式                                                   |
|----------------|------------------------------------------------------------|
| 提高查找速度   | BWT + FM-index，快速后向搜索，避免全基因组扫描            |
| 降低内存占用   | 压缩索引结构（FM-index）、2-bit 编码、稀疏表存储、分段加载 |

---
```
（2）用bowtie将 THA2.fa mapping 到 BowtieIndex/YeastGenome 上，得到 THA2.sam，统计mapping到不同染色体上的reads数量(即统计每条染色体都map上了多少条reads)。
```
```bash
test@bioinfo_docker:~$ cd /home/test/mapping
test@bioinfo_docker:~/mapping$ bowtie -v 2 -m 10 --best --strata BowtieIndex/YeastGenome -f THA2.fa -S THA2.sam
# reads processed: 1250
# reads with at least one reported alignment: 1158 (92.64%)
# reads that failed to align: 77 (6.16%)
# reads with alignments suppressed due to -m: 15 (1.20%)
Reported 1158 alignments to 1 output stream(s)
test@bioinfo_docker:~/mapping$ awk '$3 != "*" {print $3}' THA2.sam | sort | uniq -c | sort -nr
    194 chrIV
    169 chrXII
    125 chrVII
    101 chrXV
     78 chrXVI
     71 chrX
     68 chrVIII
     67 chrXIII
     58 chrXIV
     56 chrXI
     51 chrII
     33 chrV
     25 chrIX
     18 chrI
     17 chrVI
     15 chrIII
     12 chrmt
      1 VN:1.0.0
      1 SO:unsorted
      1 LN:948066
      1 LN:924431
      1 LN:85779
      1 LN:813184
      1 LN:784333
      1 LN:745751
      1 LN:666816
      1 LN:576874
      1 LN:562643
      1 LN:439888
      1 LN:316620
      1 LN:270161
      1 LN:230218
      1 LN:1531933
      1 LN:1091291
      1 LN:1090940
      1 LN:1078177
```

```
（3）查阅资料，回答以下问题:

（3.1）什么是sam/bam文件中的"CIGAR string"? 它包含了什么信息?

（3.2）"soft clip"的含义是什么，在CIGAR string中如何表示？

（3.3）什么是reads的mapping quality? 它反映了什么样的信息?

（3.4）仅根据sam/bam文件的信息，能否推断出read mapping到的区域对应的参考基因组序列? (提示:参考https://samtools.github.io/hts-specs/SAMtags.pdf中对于MD tag的介绍)
```

### （3.1）什么是 CIGAR string？它包含了什么信息？

**CIGAR string（Compact Idiosyncratic Gapped Alignment Report）** 是 SAM/BAM 文件中用于描述一个 read 与参考基因组比对情况的字符串。

它由数字 + 字母组成，描述了 read 的每一段如何与参考序列对齐。
常见的 CIGAR 操作符包括：

| 字母 | 含义 |
|------|------|
| M    | 匹配（match 或 mismatch） |
| I    | 插入（inserted in read） |
| D    | 缺失（deleted from read） |
| N    | 跳跃（跳过参考中的区域，例如 intron） |
| S    | soft clipping（read 的一部分未比对，但保留在 read 中） |
| H    | hard clipping（read 的一部分未比对且在文件中移除） |
| =    | 精确匹配 |
| X    | 错配 |

---
### （3.2）什么是 soft clip？在 CIGAR string 中如何表示？

**Soft clip（软剪切）** 表示 read 的某些碱基虽然存在，但未被用于与参考基因组对齐（可能是低质量区域或 adapter）。

- 这些碱基在 read 序列中仍然保留。
- 用字母 `S` 表示，前面的数字是被剪切的碱基数。

### （3.3）什么是 reads 的 mapping quality？它反映了什么信息？

**Mapping Quality（MAPQ）** 是 SAM 文件中第 5 列，表示一个 read 比对到参考序列上的 **置信度**（是否唯一且正确地比对）。

它通常是一个整数，反映出错概率的对数转换：
MAPQ = -10 * log10(P_mapping_wrong)

含义如下：

- 值越大 → 置信度越高
- 值为 0 → 多个位置可比对（非唯一）
- 不同的比对工具对 MAPQ 的定义略有差异

---

### （3.4）仅根据 sam/bam 文件信息，能否推断出 read mapping 到的参考基因组序列？

**可以在一定程度上推断出。**

方法：

- **MD tag**（SAM optional tag，格式如 `MD:Z:10A5^C4`）提供了 read 对应的参考序列信息（匹配/错配/缺失）。
- 结合：
  - `POS`（比对起始位置）
  - `CIGAR` string（描述比对结构）
  - `SEQ`（read 本身的序列）
  - `MD` tag（参考中与 read 比对的碱基状态）

你可以**重构出 read 所比对到的参考基因组上的碱基序列**，但这不等同于直接获取参考序列的原始片段。

> 参考：[SAM Format Specification - Optional Fields (MD tag)](https://samtools.github.io/hts-specs/SAMtags.pdf)


```
4）软件安装和资源文件的下载也是生物信息学实践中的重要步骤。请自行安装教程中未涉及的bwa软件，从UCSC Genome Browser下载Yeast (S. cerevisiae, sacCer3)基因组序列。使用bwa对Yeast基因组sacCer3.fa建立索引，并利用bwa将THA2.fa，mapping到Yeast参考基因组上，并进一步转化输出得到THA2-bwa.sam文件。
```

```bash
conda install -c bioconda bwa
bwa

Invoke-WebRequest -Uri "http://hgdownload.soe.ucsc.edu/goldenPath/sacCer3/bigZips/sacCer3.fa.gz" -OutFile "sacCer3.fa.gz"
gzip -d sacCer3.fa.gz

bwa index sacCer3.fa

bwa mem sacCer3.fa THA2.fa > THA2-bwa.sam
```

---
# Genome Browser
![image](https://github.com/user-attachments/assets/9110a784-5ad0-4d9e-becd-8bd56f661548)






