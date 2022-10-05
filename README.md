# hse22_hw1

## Заходим на сервер 
```
ssh negubanov@92.242.58.92 -p 32222 -i mykey
```

## Создание папки в ДЗ

```
mkdir hw1
cd hw1
```

## Создаем симлинки на файлы:
```
ln -s /usr/share/data-minor-bioinf/assembly/oil_R1.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oil_R2.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq
ln -s /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq
```

## Случайно выбираем 5 миллионов чтений типа paired-end и 1.5 миллиона чтений типа mate-pairs:

```
seqtk sample -s946 /usr/share/data-minor-bioinf/assembly/oil_R1.fastq 5000000 > R1_paired_end.fastq  
seqtk sample -s946 /usr/share/data-minor-bioinf/assembly/oil_R2.fastq 5000000 > R2_paired_end.fastq  
seqtk sample -s946 /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R1_001.fastq 1500000 > R1_mate_end.fastq   
seqtk sample -s946 /usr/share/data-minor-bioinf/assembly/oilMP_S4_L001_R2_001.fastq 1500000 > R2_mate_end.fastq   
```

## С помощью программ fastQC оценим качество исходных чтений

```
mkdir fastqc      
ls *_end.fastq | xargs -P 4 -tI{} fastqc -o fastqc {}  
```

## С помощью программ multiQC получаем статистику 

```
mkdir multiqc      
multiqc -o multiqc fastqc
```

```
|           multiqc | Search path : /home/negubanov/hw1/fastqc
|         searching | ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 8/8  
|            fastqc | Found 4 reports
|           multiqc | Compressing plot data
|           multiqc | Report      : multiqc/multiqc_report.html
|           multiqc | Data        : multiqc/multiqc_data
|           multiqc | MultiQC complete
```


## Подрежем чтения по качеству и удаляем адаптеры:

```
platanus_trim R1_paired_end.fastq R2_paired_end.fastq
```

```
Checking files: 
  R1_paired_end.fastq R2_paired_end.fastq

Number of trimmed read with adapter: 
NUM_OF_TRIMMED_READ(FORWARD) = 209217
NUM_OF_TRIMMED_BASE(FORWARD) = 208644
NUM_OF_TRIMMED_READ(REVERSE) = 209273
NUM_OF_TRIMMED_BASE(REVERSE) = 357696
NUM_OF_TRIMMED_PAIR(OR) = 209295
NUM_OF_TRIMMED_PAIR(AND) = 209195


Number of trimmed read because of low quality or too short (< 11bp): 
NUM_OF_TRIMMED_READ(FORWARD) = 908261
NUM_OF_TRIMMED_BASE(FORWARD) = 18116765
NUM_OF_TRIMMED_READ(REVERSE) = 1179166
NUM_OF_TRIMMED_BASE(REVERSE) = 35906728
NUM_OF_TRIMMED_PAIR(OR) = 1630835
NUM_OF_TRIMMED_PAIR(AND) = 456592


#### PROCESS INFORMATION ####
User Time:           1.11 min
System Time:         0.03 min
VmPeak:           0.116 GByte
VmHWM:            0.109 GByte
Execution time:      1.13 min
```

```
platanus_internal_trim R1_mate_end.fastq R2_mate_end.fastq
```

```
platanus_internal_trim R1_mate_end.fastq R2_mate_end.fastq
Running with trim internal adapter mode
Checking files: 
  R1_mate_end.fastq R2_mate_end.fastq  (100%)

Number of trimmed read with internal adapter: 
NUM_OF_TRIMMED_READ(FORWARD) = 971573
NUM_OF_TRIMMED_BASE(FORWARD) = 169546671
NUM_OF_TRIMMED_READ(REVERSE) = 955911
NUM_OF_TRIMMED_BASE(REVERSE) = 171281537
NUM_OF_TRIMMED_PAIR(OR) = 1187488
NUM_OF_TRIMMED_PAIR(AND) = 739996


Number of trimmed read with adapter: 
NUM_OF_TRIMMED_READ(FORWARD) = 11195
NUM_OF_TRIMMED_BASE(FORWARD) = 365200
NUM_OF_TRIMMED_READ(REVERSE) = 11216
NUM_OF_TRIMMED_BASE(REVERSE) = 389138
NUM_OF_TRIMMED_PAIR(OR) = 11220
NUM_OF_TRIMMED_PAIR(AND) = 11191


Number of trimmed read because of low quality or too short (< 11bp): 
NUM_OF_TRIMMED_READ(FORWARD) = 360981
NUM_OF_TRIMMED_BASE(FORWARD) = 11700232
NUM_OF_TRIMMED_READ(REVERSE) = 468946
NUM_OF_TRIMMED_BASE(REVERSE) = 23931864
NUM_OF_TRIMMED_PAIR(OR) = 724716
NUM_OF_TRIMMED_PAIR(AND) = 105211


#### PROCESS INFORMATION ####
User Time:           1.13 min
System Time:         0.01 min
VmPeak:           0.167 GByte
VmHWM:            0.161 GByte
Execution time:      1.13 min
```

## Оценим качество подрезанных чтений с помощью программ fastQC:

```
mkdir Trim
ls *trimmed | xargs -tI{} fastqc -o Trim {}
```

```
|           multiqc | Search path : /home/negubanov/hw1/Trim
|         searching | ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 8/8  
|            fastqc | Found 4 reports
|           multiqc | Compressing plot data
|           multiqc | Report      : multiqc2/multiqc_report.html
|           multiqc | Data        : multiqc2/multiqc_data
|           multiqc | MultiQC complete
```


## Получим статистику с помощью multiqc:

```
mkdir multiqc2  
multiqc -o multiqc2 Trim 
```

## Собирем контиги из подрезанных чтений с помощью platanus assemble:

```
mkdir platanus
time platanus assemble -o Poil -t 8 -n 20 -f R1_paired_end.fastq.trimmed R2_paired_end.fastq.trimmed 2> ~/platanus/Final1.log
```


```
real	0m0.008s
user	0m0.000s
sys	0m0.003s
```


## Собираем скаффолды из контигов и подрезанных чтений с помощью platanus scaffold

```
time platanus scaffold -o Poil -t 1 -c Poil_contig.fa -IP1 *.trimmed -OP2 *.int_trimmed 2> ~/platanus/Final_scaffold.log
```

```
real	0m42,740s
user	0m41,191s
sys	0m1,436s
```

## Уменьшаем количество гэпов с помощью platanus gap_close:

```
platanus gap_close -o Poil -t 1 -c Poil_scaffold.fa -IP1 *.trimmed -OP2 *.int_trimmed 2> ~/platanus/gapclose.log
```


# Бонус

```
seqtk sample -s946 oil_R1.fastq 5000000 > R1_bonus.fastq  
seqtk sample -s946 oil_R2.fastq 5000000 > R2_bonus.fastq  
seqtk sample -s946 oilMP_S4_L001_R1_001.fastq 1500000 > R1_bonus_end.fastq  
seqtk sample -s946 oilMP_S4_L001_R1_001.fastq 1500000 > R2_bonus_end.fastq 
mkdir bonus_fastqc_result
fastqc bonus_R1_bonus.fastq bonus_R2_bonus.fastq bonus_R1_bonus_end.fastq bonus_R2_bonus_end.fastq -o bonus_fastqc_result
multiqc bonus_fastqc_result -o bonus_multiqc_result
platanus_trim bonus_R1_bonus.fastq bonus_R2_bonus.fastq
platanus_internal_trim bonus_R1_bonus_end.fastq bonus_R2_bonus_end.fastq
mkdir bonus_fastqc_trimmed_result
fastqc bonus_R1_bonus.fastq.trimmed bonus_R2_bonus.fastq.trimmed bonus_R1_bonus_end.fastq.int_trimmed bonus_R2_bonus_end.fastq.int_trimmed -o bonus_fastqc_trimmed_result
multiqc bonus_fastqc_result -o bonus_multiqc_trimmed_result
platanus assemble -f bonus_R1_bonus.fastq.trimmed bonus_R2_bonus.fastq.trimmed -o bonus_sub
platanus scaffold -c bonus_sub_contig.fa -b bonus_sub_contigBubble.fa -IP1 bonus_R1_bonus.fastq.trimmed bonus_R2_bonus.fastq.trimmed -OP2 bonus_R1_bonus_end.fastq.int_trimmed bonus_R2_bonus_end.fastq.int_trimmed -o bonus_sub
platanus gap_close -c bonus_sub_scaffold.fa -IP1 bonus_R1_bonus.fastq.trimmed bonus_R2_bonus.fastq.trimmed -OP2 bonus_R1_bonus_end.fastq.int_trimmed bonus_R2_bonus_end.fastq.int_trimmed -o bonus_sub
```












