# Course Assignments

A description of files within this folder:

Maize files:
* 10 files (1 for each chromosome) with SNPs ordered based on increasing position and missing data encoded by "?" : `maize_chromosome(1..10)_increase.txt
* 10 files (1 for each chromosome) with SNPs ordered based on decreasing position and with missing data encoded by "-" : `maize_chromosome(1..10)_ decrease.txt`
* 1 file with all SNPs with unknown positions in the genome : `maize_snp_unknown.txt`
* 1 file with all SNPs with multiple positions in the genome: `maize_multi_snps.txt

Teosinte files:
* 10 files (1 for each chromosome) with SNPs ordered based on increasing position and missing data encoded by "?" : `teo_chromosome(1..10)_ increase.txt`
* 10 files (1 for each chromosome) with SNPs ordered based on decreasing position and with missing data encoded by "-"
: `teo_chromosome(1..10)_ decrease.txt`
* 1 file with all SNPs with unknown positions in the genome : `teo_snp_unknown.txt`
* 1 file with all SNPs with multiple positions in the genome: `teo_multi_snps.txt`



# UNIX Assignment - Kiara Kappelmann

## Data Inspection

### Attributes of `fang_et_al_genotypes`


here is my snippet of code used for data inspection
``` 
    wc -l fang_et_al_genotypes.txt
    du - h fang_et_al_genotypes.txt
    awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt
    tail -n +6 fang_et_al_genotypes.txt | awk -F "\t" '{print NF; exit}'
    head -n +1 fang_et_al_genotypes.txt
    cut -f3 fang_et_al_genotypes.txt |sort|uniq


By inspecting this file I learned that:

* There are 2783 lines in this file
* The file is 6.7M and is considered fairly large
* There are 986 columns in this file
* There are 986 columns at the end of this file as well
* It looks like there is a column for each SNP that was genotyped
* There are 16 groups in the file



### Attributes of `snp_position.txt`


here is my snippet of code used for data inspection
```
    wc -l snp_position.txt
    du -h snp_position.txt
    snp_position.txt
    tail -n +6 snp_position.txt | awk -F "\t" '{print NF; exit}'
    head -n +1 snp_position.txt
    head -n +3 snp_position.txt
    head -n +3 snp_position.txt |column -t
```



By inspecting this file I learned that:


* There are 984 lines in this file
* The file is 49K and is much smaller than the fang_et_al_genotypes.txt file
* There are 15 columns in this file
* There are also 15 columns at the end of the file
* This file is formatted very differently than the fang_et_al_genotypes.txt file, as each SNP doesn't have it's own column
* I have noticed the first SNPs align with some of the column headers in the previous file


## Data Processing

### For Maize and Teosinte
#### Because both Maize and Teosinte need similar processing, I am doing the groups together
## 
#### First I need to sort the groups out of the `fang_et_al_genotypes.txt` file and do preparation of the two files. Including adding header information, renaming files, and preparing the files for a `JOIN` command

#### File preparation
```
grep -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt > maize.txt
grep -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt > teo.txt
```
But, both of these files do not have the header information. the first column needs to be cut out of the file and joined with the maize.txt and teo.txt
```
head -n +1 fang_et_al_genotypes.txt > head.txt
head -n +1 fang_et_al_genotypes.txt > head.teo.txt
```
Now I have two files, one with header information and the other with genotype information. they need to be combined
```
cat maize.txt >> head.txt
cat teo.txt >> head.teo.txt
```
and then i need to rename the head.txt file to maize.txt for further processing. I don't want my complete file to be named head.txt
```
mv head.txt maize.txt
mv head.teo.txt teo.txt
```

Now I have files separated by maize and teosinte groups, but the ordering of the columns is wrong for the join command for the next step of combining my files with the snp_position.
```
cut -f 3-986 maize.txt > maize_group.txt
cut -f 3-986 teo.txt > teo_group.txt
```
OK - these look ok. Now I am read to prepare what I need from the snp_position.txt file
```
cut -f 1,3,4 snp_position.txt > snp_group.txt
```

OK - time to tranpose the maize and teo files so they will be able to join with the snp file

Using the .awk script provided 
```
awk -f transpose.awk maize_group.txt > maize_transpose.txt
awk -f transpose.awk teo_group.txt > teo_transpose.txt
```

Another step before joining is to make sure the files are in the correct order. `SORT` command
```
sort -k1,1 snp_group.txt > sorted_snp.txt
sort -k1,1 maize_transpose.txt > maize_sort.txt
sort -k1,1 teo_transpose.txt > teo_sort.txt
```
Time to `JOIN`
```
join -1 1 -2 1 -t $'\t' sorted_snp.txt maize_sort.txt > maize_join.txt 
join -1 1 -2 1 -t $'\t' sorted_snp.txt teo_sort.txt > teo_join.txt
```
## Creating files that satisfy the assignment
### 10 files, (1 for each chromosome) with SNPs ordered based on increasing position values and with missing data encoded by this symbol: ?

First, I need to order the SNPs based on position. I am going to create both `_increase.txt` and `_decrease.txt` files now. This helps with later steps and because I don't want files with multiple or unknown SNP positions
## 
#`_increase.txt`
```
grep -v "multiple" maize_join.txt |grep -v "unknown"| sort -k3,3n > maize_increase.txt 
grep -v "multiple" teo_join.txt |grep -v "unknown"| sort -k3,3n > teo_increase.txt
```
#`_decrease.txt`
```
grep -v "multiple" maize_join.txt |grep -v "unknown"| sort -k3,3nr > maize_decrease.txt
grep -v "multiple" teo_join.txt |grep -v "unknown"| sort -k3,3nr > teo_decrease.txt
```

Second, I need to make files for each chromosome - trying to use a loop so I don't have to do this 20 times
using [https://www.geeksforgeeks.org/looping-statements-shell-script/] as a reference for building a loop

this will loop through column 2 for which the value is equal to {1...10} and will then sort the lines based on column 3, and create files using the value in column 2
and using the increasing files so they are in increasing order
i
```
for i in {1..10}; do awk '$2=='$i'' maize_increase.txt|sort -k3,3n > maize_chromosome"$i"_increase.txt; done
for i in {1..10}; do awk '$2=='$i'' teo_increase.txt|sort -k3,3n > teo_chromosome"$i"_increase.txt; done
```
The output results in 20 files: 10 for maize, 10 for teosinte


### Now for the multiple SNP positions 
This is creating a single file with all SNPs that have many positions
Searching for "multiple" and sending the lines to a new file
```
grep "multiple" maize_join.txt > maize_multi_snps.txt
grep "multiple" teo_join.txt > teo_multi_snps.txt
```
The output results in 2 files: 1 for maize and 1 for teosinte

### Now for the unknown SNP positions
Searching for "unknown" and sending the lines to a new file
```
grep "unknown" maize_join.txt > maize_snp_unknown.txt
grep "unknown" teo_join.txt > teo_snp_unknown.txt
```
The output results in 2 files: 1 for maize and 1 for teosinte

### Now I need to find and replace the ? by - in the files using `SED` command within the decreasing files we made earlier
Using `SED` to substitute "?" with "-", the "g" option is to substitute globally and the output is sent to a new file
```
sed 's/?/-/g' maize_decrease.txt > maize_decrease_sed.txt   
sed 's/?/-/g' teo_decrease.txt > teo_decrease_sed.txt 
```
Using an updated version of the loop created earlier to make files that are decreasing in position but have "-" instead of "?"
```
for i in {1..10}; do awk '$2=='$i'' maize_decrease_sed.txt > maize_chromosome"$i"_decrease.txt; done
for i in {1..10}; do awk '$2=='$i'' teo_decrease_sed.txt > teo_chromosome"$i"_decrease.txt; done
```
The output results in a total of 20 files: 10 for maize, 10 for teosinte






