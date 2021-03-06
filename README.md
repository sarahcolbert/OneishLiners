# Helpful One-ish Liners in the Shell


### How to delete all files in a directory that contain a string

```
### use awk to print "rm filename" to remove.sh so that you can first check you are only removing the files you want
find . | xargs grep -l STRING | awk '{print "rm "$1}' > remove.sh
bash remove.sh
```


### Check outerr directory for jobs that may have been cancelled due to time limit and get a list of those jobs

```
grep -H -r "TIME LIMIT" | awk '{print $1}' | sed 's/set.//g' | sed 's/.err:slurmstepd://g' | awk '{print}' ORS=',' > timed_out_list.txt
```

### If you have results files named by job number, chromosome, etc. list all of those in a text file in descending order

```
### in directory containing results files
ls -1v foo*.txt > numbered_files.txt
```

Other things you may want to do with this file:

* Get a list of numbers

```
sed -i 's/.txt//g' numbered_files.txt
sed -i 's/foo//g' numbered_files.txt
```

* Get a list of numbers that are missing in the sequence

```
awk 'NR != $1 { for (i = prev + 1; i < $1; i++) {print i} } { prev = $1 + 1 }' ORS=',' numbered_files.txt > missing_numbered_files.txt
```

### Check squeue for how many jobs are running and pending

```
squeue -u saco3854 | awk '
BEGIN {
    abbrev["R"]="(Running)"
    abbrev["PD"]="(Pending)"
    abbrev["CG"]="(Completing)"
    abbrev["F"]="(Failed)"
}
NR>1 {a[$5]++}
END {
    for (i in a) {
        printf "%-2s %-12s %d\n", i, abbrev[i], a[i]
    }
}'
```


## ONE LINERS FOR WORKING WITH SUMMARY STATISTICS

### How to add an N, or sample size column to summary statistics

In this example, we want to create a column with the same sample size (80000) for each SNP and name that column N.

```
awk '{print $1,$2,$3,$4,$5,80000}' file.txt > file_with_N.txt
sed -i '1s/83566/N/' file_with_N.txt
```

### How to swap allele columns that are in the wrong order

Say for example you have a file which looks like:
```
SNP CHROM POS A1  A2
rs10875231  1 100000012 T G
rs186077422 1 10000006  A G
rs114947036 1 100000135 T A
rs6678176 1 100000827 T C
```

And A2 is actually the effect/reference allele and A1 is the non-reference allele, so you want to switch these. You could use:

```
head -1 input_file.txt > output_file.txt; awk 'FNR > 1 { t = $4; $4 = $5; $5 = t; print; }' input_file.txt >> output_file.txt

```

### How to remove NAs or rows with empty fields from munged summary statistics

This code is useful for input files for GNOVA and SUPERGNOVA, since they are not compatible with files with empty fields.

```
zcat munged.sumstats.gz | awk 'NF==5' > complete.sumstats.txt
```
