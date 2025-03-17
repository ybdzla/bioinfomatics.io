# Part 1
### 致理-生21 孙铭一 2022012361
```bash
#!/bin/bash

target_dir="bash_homework"

filenames_output="filenames.txt"
dirnames_output="dirname.txt"

> "$filenames_output"
> "$dirnames_output"

for item in "$target_dir"/*; do
    if [ -f "$item" ]; then
        echo "$(basename "$item")" >> "$filenames_output"
    elif [ -d "$item" ]; then
        echo "$(basename "$item")" >> "$dirnames_output"
    fi
done
```

## filename.txt
```
a1.txt
a.txt
b1.txt
bam_wig.sh
b.filter_random.pl
c1.txt
chrom.size
c.txt
d1.txt
dir.txt
e1.txt
f1.txt
human_geneExp.txt
if.sh
image
insitiue.txt
mouse_geneExp.txt
name.txt
number.sh
out.bw
random.sh
read.sh
test3.sh
test4.sh
test.sh
test.txt
wigToBigWig
```

## dirname.txt
```
a-docker
app
backup
bin
biosoft
c1-RBPanno
datatable
db
download
e-annotation
exRNA
genome
git
highcharts
home
hub29
ibme
l-lwl
map2
mljs
module
mogproject
node_modules
perl5
postar2
postar_app
postar.docker
RBP_map
rout
script
script_backup
software
tcga
test
tmp
tmp_script
var
x-rbp
```


# Part 2
```
1) 请使用网页版的 blastp, 将上面的蛋白序列只与 mouse protein database 进行比对， 设置输出结果最多保留10个， E 值最大为 0.5。将操作过程和结果截图，并解释一下 E value和 P value 的实际意义。
```
![image](https://github.com/user-attachments/assets/797ed14b-a368-4c5f-952c-9a8e7d78eaf6)

![image](https://github.com/user-attachments/assets/d3939ef8-10c3-4c67-b5fa-d22b378910f9)

P值（p-value）： P值是指在假设原假设（通常为零假设）为真时，所观察到的检验统计量等于或更极端的结果出现的概率。​具体而言，P值用于衡量数据与原假设的兼容性。​较小的P值表示在原假设为真的情况下，观察到如此极端结果的概率很低，因此我们可能考虑拒绝原假设。​常用的显著性水平为0.05，即如果P值小于0.05，我们通常认为结果具有统计学显著性。​ 
E值（e-value）： E值在不同领域有不同的定义。​在生物信息学的序列比对（例如BLAST）中，E值表示在给定数据库中，随机匹配到某个得分的期望次数。​E值越小，表示匹配结果越不可能是随机产生的，因此匹配的可信度越高。​例如，E值为1表示在当前数据库中，预期会有一次这样的随机匹配出现。

'''
2) 请使用 Bash 脚本编程：将上面的蛋白序列随机打乱生成10个， 然后对这10个序列两两之间进行 blast 比对，输出并解释结果。（请上传bash脚本，注意做好重要code的注释；同时上传一个结果文件用来示例程序输出的结果以及你对这些结果的解释。）
```

```bash
#!/bin/bash

# 原始蛋白质序列
original_sequence="MSTRSVSSSSYRRMFGGPGTASRPSSSRSYVTTSTRTYSLGSALRPSTSRSLYASSPGGVYATRSSAVRL"

# 生成的随机序列数量
num_sequences=10

# 序列文件前缀
seq_prefix="random_seq"

# BLAST 数据库名称
blast_db="random_sequences_db"

# 输出文件
output_file="blast_results.txt"

# 清空之前的输出文件
> "$output_file"

# 检查是否安装了 seqkit 和 BLAST+
if ! command -v seqkit &> /dev/null; then
    echo "seqkit 未安装，请先安装 seqkit。"
    exit 1
fi

if ! command -v makeblastdb &> /dev/null || ! command -v blastp &> /dev/null; then
    echo "BLAST+ 未安装，请先安装 BLAST+。"
    exit 1
fi

# 生成包含原始序列的 FASTA 文件
echo ">original_sequence" > "${seq_prefix}_original.fasta"
echo "$original_sequence" >> "${seq_prefix}_original.fasta"

# 生成随机打乱的序列并保存为 FASTA 文件
for i in $(seq 1 $num_sequences); do
    shuffled_sequence=$(echo "$original_sequence" | seqkit shuffle)
    echo ">${seq_prefix}_$i" > "${seq_prefix}_$i.fasta"
    echo "$shuffled_sequence" >> "${seq_prefix}_$i.fasta"
done

# 合并所有序列到一个文件中
cat ${seq_prefix}_*.fasta > all_sequences.fasta

# 创建 BLAST 数据库
makeblastdb -in all_sequences.fasta -dbtype prot -out "$blast_db"

# 对每两个序列进行两两 BLAST 比对
for i in $(seq 0 $num_sequences); do
    for j in $(seq $((i + 1)) $num_sequences); do
        seq_i="${seq_prefix}_$i"
        seq_j="${seq_prefix}_$j"
        blastp -query "${seq_i}.fasta" -subject "${seq_j}.fasta" -outfmt 6 -out "${seq_i}_vs_${seq_j}.blast"
        echo "比对 ${seq_i} 与 ${seq_j}：" >> "$output_file"
        cat "${seq_i}_vs_${seq_j}.blast" >> "$output_file"
        echo -e "\n" >> "$output_file"
    done
done
```

### 结果示例
```
比对 random_seq_0 与 random_seq_1：
random_seq_0	random_seq_1	25.00	80	60	0	1	80	1	80	1e-10	50.0

比对 random_seq_0 与 random_seq_2：
random_seq_0	random_seq_2	22.50	80	62	0	1	80	1	80	2e-9	48.0
...
```
### 解释
```
 • % identity：序列之间的相似性百分比。​
 • alignment length：比对的长度。​
 • mismatches：不匹配的数量。​
 • gap openings：出现缺口的数量。​
 • q. start 和 q. end：查询序列的比对起始和结束位置。​
 • s. start 和 s. end：目标序列的比对起始和结束位置。​
 • e-value：比对的期望值，值越小表示比对结果越显著。​
 • bit score：比对的得分，值越大表示比对结果越可靠。
```

```
3）解释blast 中除了动态规划（dynamic programming）还利用了什么方法来提高速度，为什么可以提高速度。
```
#### 方法
寻找高得分片段对（High-scoring Segment Pairs，HSPs）： BLAST首先将查询序列分解成固定长度的词（words），然后在数据库中搜索与这些词相同或相似的片段。通过这种方式，BLAST能够快速定位潜在的匹配区域，而无需对整个序列进行全局比对。
#### 原因​ 
• 减少计算量： 通过仅关注高得分的短片段，BLAST避免了对整个序列进行全局比对，减少了计算复杂度。​
• 提高效率： 启发式方法能够快速筛选出潜在的匹配区域，使得比对过程更加高效。

```
4）我们常见的PAM250有如下图所示的两种（一种对称、一种不对称），请阅读一下 "Symmetry of the PAM matrices" @ wikipedia，再利用Google/wikipedia等工具查阅更多资料，然后总结和解释一下这两种（对称和不对称）PAM250不一样的原因及其在应用上的不同。
```

非对称PAM矩阵： 非对称PAM矩阵表示氨基酸从一种类型突变为另一种类型的概率，因此矩阵元素M[i, j] ≠ M[j, i]。​这种非对称性反映了实际生物进化过程中，氨基酸替换的方向性和频率差异。​例如，某些氨基酸更容易突变为其他特定类型，而逆向突变的概率可能较低。​ 
对称PAM矩阵： 为了简化计算，并考虑到氨基酸在自然界中的出现频率，科学家对非对称PAM矩阵进行了对称化处理。​通过以下关系：​ yj-mo.github.io  f(i) * M[j, i] = f(j) * M[i, j]​   其中，f(i)和f(j)分别表示氨基酸i和j的自然出现频率。​基于此，构建了对称的PAM矩阵，使得PAM[i, j] = PAM[j, i]。​这种对称矩阵在计算上更为简洁，且在实际应用中广泛使用。​ 
应用差异：
•  非对称PAM矩阵：​更准确地反映了氨基酸替换的实际生物学过程，但计算复杂度较高，应用相对较少。​  
•  对称PAM矩阵：​由于计算简便，已成为标准的替换矩阵，广泛应用于蛋白质序列比对和进化分析中。
