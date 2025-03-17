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
