#UNIX Assignment
#####By Leila Fattel

##Data Inspection

###Attributes of fang\_et\_al\_genotypes
First, I wanted to view the contents of the file by using ````cat````:
````	
$ cat fang_et_al_genotypes.txt
````

After seeing that the contents of this file are huge, I inspected the file closely using ````less````:	
````$ less fang_et_al_genotypes.txt````

Since the contents of the file are big, the easier way to check for the number of lines, words, and bytes, respectively, is to use ````wc````:	
````$ wc fang_et_al_genotypes.txt````	

To check for the size of the file we can use ````ls````:	
````$ ls -lh fang_et_al_genotypes.txt````

Next, I wanted to check for the number of columns in this file by using```awk```, but first I used ````head````and ````tail````to make sure that there is no pseudocode that might interfer with the number of columns:		
````$ (head -n 2; tail -n 2) < fang_et_al_genotypes.txt````

``` $ awk -F "\t" '{print NF; exit}' fang_et_al_genotypes.txt ```	

To check that this file is free of non-ASCII characters, I used ````file````	
````$ file fang_et_al_genotypes.txt````	


By inspecting fang\_et\_al\_genotypes, I learned that:
	
1. The number of lines is 2783	
2. The number of words is 2744038	
3. The number of bytes is 11051939	
4. The file size is 11MB	
5. The file does not contain pseudocode	
6. The number of columns is 986		
7. The text in this file is ASCII	, with very long lines

###Attributes of snp_position.txt
By replacing ````fang_et_al_genotypes.txt````with ````snp_position.txt```` in the code found in the previous section, I inspected this file and learned the following:	

1. The number of lines is 984	
2. The number of words is 13198	
3. The number of bytes is 82763	
4. The file size is 81kB		
5. The file does not contain pseudocode
6. The number of columns is 15		
7. The text in this file is ASCII		


## Data Processing

First, I collected the groups related to maize (ZMMIL, ZMMLR, and ZMMMR) and teosinte (ZMPBA, ZMPIL,and ZMPJA), respectively:

````grep -E "(ZMMIL|ZMMLR|ZMMMR)" fang_et_al_genotypes.txt > maize_genotypes.txt````

```grep -E "(ZMPBA|ZMPIL|ZMPJA)" fang_et_al_genotypes.txt > teosinte_genotypes.txt```



Then, I sorted both according to column 1:


```sort -k1,1 teosinte_genotypes.txt > sorted_teosinte_genotypes.txt```
```sort -c sorted_teosinte_genotypes.txt```
```sort -k1,1 maize_genotypes.txt > sorted_maize_genotypes.txt```

Next, I cut the first line in ````fang_et_al_genotypes```` to add it as a header to the ````sorted_maize_genotypes.txt```` and ````sorted_tesonite_genotypes.txt````

````head -n 1 fang_et_al_genotypes.txt > head_genotypes.txt````

and sorted this file:
````sort -k1,1 head_genotypes.txt > sorted_head_genotypes.txt````

Next, I added the header to the sorted maize and tesonite files:

````cat sorted_head_genotypes.txt sorted_maize_genotypes.txt > Maize_genotypes.txt````
````cat sorted_head_genotypes.txt sorted_teosinte_genotypes.txt > Teosinte_genotypes.txt````

I did the following to check that the number of groups in each file is maintained and correct:

````cut -f 3 fang_et_al_genotypes.txt |sort| uniq -c````
````cut -f 3 sorted_maize_genotypes.txt |sort| uniq -c```` 
````cut -f 3 sorted_teosinte_genotypes.txt |sort| uniq -c````
````cut -f 3 Maize_genotypes.txt |sort| uniq -c````
````cut -f 3 Teosinte_genotypes.txt |sort| uniq -c````

Next, I transposed the Maize and Teosinte genotypes files:
````awk -f transpose.awk Maize_genotypes.txt > transposed_maize_genotypes.txt````
````awk -f transpose.awk Teosinte_genotypes.txt > transposed_teosinte_genotypes.txt````

To get the SNP_ID, Chromosome, and Position columns in  ````snp_position.txt````:
````cut -f -1,3,4 snp_position.txt > snp_new.txt````

Next, I wanted to sort ```snp_new.txt``` without sorting its first line (the header) as well:

````tail -n +2 snp_new.txt | sort  -f -k1 > sorted_snp_noheader.txt````

Then I joined the header with the new sorted file by:

````head -n +1 snp_new.txt > head_snp_new.txt````
````cat head_snp_new.txt sorted_snp_noheader.txt > sorted_snp.txt````

Next, I joined the new SNP file with the transposed maize and teosinte files, respectively:

````join --nocheck-order -1 1 -2 1 -a 1 sorted_snp.txt transposed_maize_genotypes.txt > maize_snp_genotypes.txt ````

````join --nocheck-order -1 1 -2 1 -a 1 sorted_snp.txt transposed_teosinte_genotypes.txt > teosinte_snp_genotypes.txt````

* I had to use ````--nocheck-order```` because it kept giving a sorting error in ````sorted_snp.txt````even though I manually checked that the SNP_IDs were sorted correctly.


Then, I created a two folders called ```maize```  and ```teosinte```, each having two folders in them called ````sorted_increasing```` and ````sorted_decreasing```` to be able to collect files with SNPs ordered in increasing (with missing data encoded by ?) and decreasing (with missing data encoded by -) order, respectively, using ```awk```:
 
 ````for i in {1..10}; do awk '$2 =='$i'' maize_snp_genotypes.txt | sort -nk3 > ./sorted_increasing/chr_${i}_increasing_maize.txt; done````
 
 ````for i in {1..10}; do awk '$2 =='$i'' teosinte_snp_genotypes.txt | sort -nk3 > ./teosinte/sorted_increasing/chr_${i}_increasing_teosinte.txt; done````

````for i in {1..10}; do awk '$2 =='$i'' maize_snp_genotypes.txt | sort -nk3r | sed -e 's/\?/\-/g' > ./maize/sorted_decreasing/chr_${i}_decreasing_maize.txt; done````

````for i in {1..10}; do awk '$2 =='$i'' teosinte_snp_genotypes.txt | sort -nk3r | sed -e 's/\?/\-/g' > ./teosinte/sorted_decreasing/chr_${i}_decreasing_teosinte.txt; done````

Finally, I used ````grep```` to collect the SNPs with multiple positions, and another with unknown positions:

 
````grep "multiple" maize_snp_genotypes.txt > ./maize/multiple_maize.txt````

````grep "multiple" teosinte_snp_genotypes.txt > ./teosinte/multiple_teosinte.txt````

````grep "unknown" maize_snp_genotypes.txt > ./maize/unknown_maize.txt````

````grep "unknown" teosinte_snp_genotypes.txt > ./teosinte/unknown_teosinte.txt````

At the end, I ended up with 44 files.


###Bonus
If we want to remove the multiple/unknown "positions" from the files generated in increasing/decreasing order of positions above, we can the following examples to each file:

````grep -v $3 "unknown" chr_1_decreasing_maize.txt > new_chr_1_decreasing_maize.txt #whereas 1 needs to be changed according to chromosome number````

and: 

````grep -v $3 "multiple" chr_1_increasing_teosinte.txt > new_chr_1_increasing_teosinte.txt #whereas 1 needs to be changed according to chromosome number````



