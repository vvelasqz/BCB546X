#Workflow

*Author: Valeria VelasquezZapata*

For inpecting data we can use commands as less and wc, which can tell us how many lines, words and bites has a file

	$wc fang_et_al_genotypes.txt snp_position.txt
	2782  2744038 11051938 fang_et_al_genotypes.txt
   	984    13198    82763 snp_position.txt
    	3766  2757236 11134701 total

To know the number of columns in a file we can use awk and tail to remove the header 

	$tail -n +4 fang_et_al_genotypes.txt |awk '{print NF; exit}'
	986

To know the numbers of columns in the snp_position.txt we inspect the file direcly using 

	$awk '{print NF; exit}' snp_position.txt
	15

But when we use tail to remove the header we get a different answer which means that probably we have some incomplete data.

	$tail -n 1 snp_position.txt |awk '{print NF; exit}'
	13

To separate the data of teosinte and Tripsacum we can use a grep -E to match multiple patterns and then send the standard out to a new file, named accordingly

	$grep -E 'ZMMIL|ZMMLR|ZMMMR' fang_et_al_genotypes.txt > tripsacum_genotypes.txt
	$grep -E 'ZMPBA|ZMPIL|ZMPJA' fang_et_al_genotypes.txt > teosinte_genotypes.txt

We can confirm that the files are well made by counting the number of lines and matches of the pattern in the new files, and the original must be the sum of both

	$grep -E -c 'ZMPBA|ZMPIL|ZMPJA' teosinte_genotypes.txt
	975
	$grep -E -c 'ZMMIL|ZMMLR|ZMMMR' tripsacum_genotypes.txt                       
	1573

	$wc teosinte_genotypes.txt tripsacum_genotypes.txt
    	975   961350  3873338 teosinte_genotypes.txt
    	1573  1550978  6240114 tripsacum_genotypes.txt
    	2548  2512328 10113452 total

	$grep -E -c 'ZMPBA|ZMPIL|ZMPJA|ZMMIL|ZMMLR|ZMMMR' fang_et_al_genotypes.txt
	2548

As we can see the number of lines in each files is equal to o the number of matchesof the pattern which menas that we have the matches in only one column of the file. Then the sum of matches of both files is the same as in the original, then the files are well made.

To include the header in each file we extract it from the original file and concatenate it with the tripsacum_genotypes.txt and teosinte_genotypes.txt files

	$head -n 1 fang_et_al_genotypes.txt > header.txt
	$cat header.txt tripsacum_genotypes.txt > tripsacum_genotypes_header.txt
	$cat header.txt teosinte_genotypes.txt > teosinte_genotypes_header.txt

After this step we can transpose the files, to get the information of each SNP in one column

	$awk -f transpose.awk teosinte_genotypes_header.txt > transposed_teosinte_genotypes.txt
	$awk -f transpose.awk tripsacum_genotypes_header.txt > transposed_tripsacum_genotypes.txt

After this, we can extract the columns of interest from the snp_position.txt file, which corresponds to SNP id (first column), chromosome location (third column), nucleotide location (fourth column), using cut command

	$cut -f 1,3,4 snp_position.txt > snp_position_id_chr_pos.txt

In order to get a file with all the required columns, we join the transposed files with the snp_position_id_chr_pos.txt file. Before join the files, we sort them alphanumerically

	$sort -k1,1V transposed_teosinte_genotypes.txt > transposed_teosinte_genotypes_sorted.txt
	$sort -k1,1V transposed_tripsacum_genotypes.txt > transposed_tripsacum_genotypes_sorted.txt
	$sort -k1,1V snp_position_id_chr_pos.txt > snp_position_id_chr_pos_sorted.txt

	$join -11 -21 snp_position_id_chr_pos_sorted.txt transposed_teosinte_genotypes_sorted.txt -t $'\t'> teosinte_join.txt

	$join -11 -21 snp_position_id_chr_pos_sorted.txt transposed_tripsacum_genotypes_sorted.txt -t $'\t'> tripsacum_join.txt

At this point we have two files with the required columns of the assigment, but we need to sort and parse them according to the chromosome number and position. After parsing the file to get the information per chromosome, we can sort the file numerically according to the chromosome and the position, and then separate the files accoding to the chromosome (column 2).

	$sort -k2,2n -k3,3n teosinte_join.txt | awk -F '\t' '{print > ("teosinte_"$2".txt")}'
	$sort -k2,2n -k3,3n tripsacum_join.txt | awk -F '\t' '{print > ("tripsacum_"$2".txt")}'

The last command separated the initial files according to chromosome number. At this point we already have the files with SNPs ordered based on increasing position values and with missing
data encoded by this symbol: ?

To generate the files with SNPs ordered based on decreasing position values and with missing data encoded by this symbol: - we copy the join files and replace the ? symbol per -, using sed

	$cp teosinte_join.txt teosinte_join_rev.txt
	$cp tripsacum_join.txt tripsacum_join_rev.txt

	$sed -ie 's/?/-/g' teosinte_join_rev.txt
	$sed -ie 's/?/-/g' tripsacum_join_rev.txt 

Finally we sort the files accroding to the chromosome number and the position in a decreasing order (reverse). Then we use awk to parse the file using the chromosome information

	$sort -k2,2n -k3,3nr teosinte_join_rev.txt | awk -F '\t' '{print > ("teosinte_rev"$2".txt")}'
	$sort -k2,2n -k3,3nr tripsacum_join_rev.txt | awk -F '\t' '{print > ("tripsacum_rev"$2".txt")}'

**The files named as `teosinte_chr#.txt` and `tripsacum_chr#.txt` have the SNPs ordered based on increasing position values and with missing data encoded by this symbol: ?**

**The files named as `teosinte_rev_chr#.txt` and `tripsacum_rev_chr#.txt` have the SNPs ordered based on decreasing position values and with missing data encoded by this symbol: -**











