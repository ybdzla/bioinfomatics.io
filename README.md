# Homework 2.1
## Preparation
PS C:\Users\86130> docker cp "D:/大三下/生物信息学/PART1/test_command.gtf" sunmingyi_linux:/home/test/linux
Successfully copied 2.56kB to sunmingyi_linux:/home/test/linux
PS C:\Users\86130> docker exec -it sunmingyi_linux bash
test@bioinfo_docker:~$ cd /home/test/linux
test@bioinfo_docker:~/linux$ ls
1.gtf.gz  file  test_command.gtf
## task 1
test@bioinfo_docker:~/linux$ wc -l test_command.gtf
8 test_command.gtf
test@bioinfo_docker:~/linux$ wc -c test_command.gtf
636 test_command.gtf
## task 2
test@bioinfo_docker:~/linux$ grep '^chr_' test_command.gtf | grep 'gene_id "YDL248W"'
chr_IV  ensembl gene    1802    2953    .       +       .       gene_id "YDL248W"; gene_version "1";
chr_IV  ensembl transcript      802     2953    .       +       .       gene_id "YDL248W"; gene_version "1";
chr_IV  ensembl start_codon     1802    1804    .       +       0       gene_id "YDL248W"; gene_version "1";
## task 3
test@bioinfo_docker:~/linux$ sed 's/chr_/chromosome_/g' test_command.gtf | cut -f 1,3,4,5
chromosome_IV   gene    1802    2953
chromosome_IV   transcript      802     2953
chromosome_IV   exon    1802    2953
chromosome_IV   CDS     1802    950
chromosome_IV   start_codon     1802    1804
chromosome_IV   stop_codon      2951    2953
chromosome_IV   gene    762     3836
chromosome_IV   transcript      3762    836
## task 4
test@bioinfo_docker:~/linux$ awk '{print $1, $3, $2, $4, $5, $6, $7, $8, $9}' test_command.gtf | sort -k4,4n -k5,5n > result.gtf
test@bioinfo_docker:~/linux$ cat result.gtf
chromosome_IV gene ensembl 762 3836 . + . gene_id
chr_IV transcript ensembl 802 2953 . + . gene_id
chromosome_IV CDS ensembl 1802 950 . + 0 gene_id
chr_IV start_codon ensembl 1802 1804 . + 0 gene_id
chr_IV gene ensembl 1802 2953 . + . gene_id
chromosome_IV exon ensembl 1802 2953 . + . gene_id
chromosome_IV stop_codon ensembl 2951 2953 . + 0 gene_id
chr_IV transcript ensembl 3762 836 . + . gene_id
## task 5
ls -l test_command.gtf 
-rw-r--r-- 1 test test 636 Mar  4 07:07 test_command.gtf 
chmod 754 test_command.gtf
ls -l test_command.gtf 
-rwxr-xr-- 1 test test 636 Mar  4 07:07 test_command.gtf
