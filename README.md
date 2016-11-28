----------

Haruo Suzuki (haruo[at]g-language[dot]org)  
Last Update: 2015-11-05  

----------

# SILVA SSU project
Project started 2015-10-13.

## Run environment
Mac OS X 10.9.5  

## Project directories

	silva_ssu/
	  README.md: project documentation 
	  bin/: scripts
	    bp_search2table.pl: turn blast output (-m 0) into tab delimited format (-m 9)
	    hs_blast_parser.pl (blast_parser.pl): Parsing BLAST output
	    run_blast.sh: run blast program
	  data/: sequence data
	  results/: results of analyses

## Data
Data was downloaded on 2015-09-23 from <http://www.arb-silva.de/no_cache/download/archive/current/Exports/> into `data/`, using:

	URL=http://www.arb-silva.de/fileadmin/silva_databases/current/Exports/SILVA_123_SSURef_Nr99_tax_silva_trunc.fasta.gz
	nohup wget $URL.md5 $URL &
	tail -f nohup.out # Total wall clock time: 1m 55s # Wi-Fi: IAB

MD5 Checksum: 

	cat `basename $URL`.md5
	md5 `basename $URL`

	d5deaf307b443194b66bd08a1ee05a66  SILVA_123_SSURef_Nr99_tax_silva_trunc.fasta.gz
	MD5 (SILVA_123_SSURef_Nr99_tax_silva_trunc.fasta.gz) = d5deaf307b443194b66bd08a1ee05a66

Decompress files:

	gunzip -c `basename $URL` > `basename $URL .gz`

## Scripts

`bp_search2table.pl` was downloaded on 2015-09-23 into `bin/`, using:

        wget https://raw.githubusercontent.com/bioperl/bioperl-live/master/scripts/searchio/bp_search2table.pl
        chmod +x bp_search2table.pl

`blast_parser.pl` was downloaded on 2015-10-09 from <http://kirill-kryukov.com/study/tools/blast-parser/> into `bin/`, using:

	wget http://kirill-kryukov.com/study/tools/blast-parser/blast_parser_1.1.5.zip
	unzip blast_parser_1.1.5.zip

`blast_parser.pl` was copied into `hs_blast_parser.pl`, and then 

	205:        print "\n$qname ($qlen)\n";
	216:        print "     $sname ($slen)\n";
	220:    print "          $frame  $length($ident,$positives,$gaps) - $bits($expect)  $qstart..$qend - $sstart..$send\n";

was changed to

	205:        @_ = split(/ /, $qname); $qname = $_[0]; #print "\n$qname ($qlen)\n";
	216:        print "$qname\t$sname\n"; #print "     $sname ($slen)\n";
	220:    #print "          $frame  $length($ident,$positives,$gaps) - $bits($expect)  $qstart..$qend - $sstart..$send\n";

----------

## Steps

	date +%F
	mkdir -p silva_ssu/{bin,data,results}

### Inspecting Data

	cd data/
	grep -A 7 _tax_silva_trunc.fasta README.txt 

	DB=SILVA_123_SSURef_Nr99_tax_silva_trunc.fasta
	grep "^>" $DB | wc -l # 597607
	grep "^>" $DB | head 
	ls -lh $DB

	-rw-r--r--  1 haruo  staff   902M Oct 14 09:16 SILVA_123_SSURef_Nr99_tax_silva_trunc.fasta

#### Count groups

	DB=SILVA_123_SSURef_Nr99_tax_silva_trunc.fasta
	grep "^>" $DB | cut -d" " -f2 | cut -d";" -f1 | sort | uniq -c | sed s/^/$'\t'/g
	grep "^>" $DB | cut -d" " -f2 | grep "Bacteria" | cut -d";" -f2 | sort | uniq -c | sort -nr | head | sed s/^/$'\t'/g
	grep "^>" $DB | grep "[Cc]hloroplast" | wc -l # 12215
	grep "^>" $DB | grep "[Pp]lastid" | wc -l # 8868
	grep "^>" $DB | grep "[Cc]hloroplast\|[Pp]lastid" | wc -l # 13128
	grep "^>" $DB | grep "[Mm]itochondri" | wc -l # 873
	grep "^>" $DB | grep "Cyanobacteria" | wc -l # 11393

##### Domain

	22913 Archaea
	513310 Bacteria
	61384 Eukaryota

##### Bacterial phyla

	200072 Proteobacteria
	138552 Firmicutes
	51253 Actinobacteria
	49244 Bacteroidetes
	13175 Acidobacteria
	11393 Cyanobacteria
	8805 Chloroflexi
	8037 Planctomycetes
	3838 Spirochaetae
	3820 Verrucomicrobia

##### Bacterial genus/species

	NAME="Yersinia" # 735
	NAME="Yersinia pestis"
	NAME="Escherichia-Shigella" # 4395
	NAME="Escherichia coli K-12"
	NAME="Bacillus" # 12566
	NAME="Bacillus \(anthracis\|cereus\)"
	NAME="Bacillus anthracis"
	NAME="Shigella dysenteriae"
	NAME=";Shigella" # 199
	NAME="Shigella flexneri"
	DB=data/SILVA_123_SSURef_Nr99_tax_silva_trunc.fasta
	grep "^>" $DB | grep -n "$NAME" | wc -l
	grep "^>" $DB | grep -n "$NAME" | head
	grep "^>" $DB | grep -n "$NAME" | (head -1; tail -1)
	grep "^>" $DB | grep -n "$NAME" | cut -d";" -f7 | cut -d" " -f1,2 | sort | uniq -c
	#
	>X55059.1.1450 Bacteria;Firmicutes;Bacilli;Bacillales;Bacillaceae;Bacillus;Bacillus anthracis
	>JNOD01000017.2059.3595 Bacteria;Firmicutes;Bacilli;Bacillales;Bacillaceae;Bacillus;Bacillus anthracis
	#
	>X75274.1.1444 Bacteria;Proteobacteria;Gammaproteobacteria;Enterobacteriales;Enterobacteriaceae;Yersinia;Yersinia pestis

	# NAME="Escherichia-Shigella"
	>U88548.1.1541 Bacteria;Proteobacteria;Gammaproteobacteria;Enterobacteriales;Enterobacteriaceae;Escherichia-Shigella;Salmonella enterica subsp. enterica serovar Paratyphi C
	>JQ190338.1.1363 Bacteria;Proteobacteria;Gammaproteobacteria;Enterobacteriales;Enterobacteriaceae;Escherichia-Shigella;uncultured bacterium

	# NAME=";Shigella"
	>JQ904751.1.1509 Bacteria;Proteobacteria;Gammaproteobacteria;Enterobacteriales;Enterobacteriaceae;Escherichia-Shigella;Shigella sonnei
	>AKNB01000277.3472.4997 Bacteria;Proteobacteria;Gammaproteobacteria;Enterobacteriales;Enterobacteriaceae;Escherichia-Shigella;Shigella boydii 4444-74

	  33 Shigella boydii
	  27 Shigella dysenteriae
	  91 Shigella flexneri
	  39 Shigella sonnei
	   9 Shigella sp.

----------

### [Building a BLAST database](http://www.ncbi.nlm.nih.gov/books/NBK279688/)

	DB=SILVA_123_SSURef_Nr99_tax_silva_trunc.fasta
	DB=Bacillus.fasta
	DB=Yersinia.fasta
	DB=data/Shigella.fasta
	makeblastdb -in $DB -dbtype nucl -hash_index -parse_seqids > data/makeblastdb.`basename $DB`.log 2>&1

- [Options for the command-line applications. - BLAST](http://www.ncbi.nlm.nih.gov/books/NBK279675/)
- [BLASTデータベースの作り方について](http://bio-info.biz/tips/other_blast_db.html)
- [makeblastdb | blast検索用のデータベースを作成する方法](http://bi.biopapyrus.net/seq/blast/makeblastdb.html)

| オプション | 説明 |
|:-----------|:-----------|
| -dbtype | 核酸なら nucl を指定。タンパク質なら prot を指定。 |
| -in | データベースを作る元となるファイル |
| -parse_seqids | 入力ファイルが fasta ファイルの時、配列の ID のインデックスを作成 |
| -hash_index | 配列のハッシュをインデックス化 |

### [Extracting data from BLAST databases with blastdbcmd](http://www.ncbi.nlm.nih.gov/books/NBK279689/)

	NAME=Bacillus
	NAME=X55059.1.1450 # Bacillus anthracis
	NAME=JNOD01000017.2059.3595 # Bacillus anthracis
	NAME=CP009968.130421.131958 # Bacillus cereus E33L
	NAME=Yersinia
	NAME=X75274.1.1444 # Yersinia pestis
	NAME=CP010067.3596048.3597576 # Yersinia pseudotuberculosis str. PA3606
	NAME=JQ904751.1.1509 # Shigella sonnei
	NAME=AKNB01000277.3472.4997 # Shigella boydii 4444-74
	NAME=AP010960.225071.226612 # Escherichia coli O111:H- str. 11128
	NAME=CP009789.3913462.3915003 # Escherichia coli K-12
	NAME=CP007592.3952169.3953690 # Escherichia coli O157:H16
	NAME=Shigella
	NAME=X96966.1.1487 # Shigella dysenteriae
	NAME=X96963.1.1488 # Shigella flexneri
	DB=data/SILVA_123_SSURef_Nr99_tax_silva_trunc.fasta
	blastdbcmd -db $DB -entry all -outfmt "%i %t" | grep "$NAME" | \
	# grep -v "Chloroplast\|Mitochondria" | \
 	awk '{print $1}' | blastdbcmd -db $DB -entry_batch - > data/$NAME.fasta

- [Blasted Bioinformatics!?: My IDs not good enough for NCBI BLAST+](http://blastedbio.blogspot.com/2012/10/my-ids-not-good-enough-for-ncbi-blast.html)
- [自家製BLAST用DBから必要な配列エントリ取得 | ぼうのブログ](http://bonohu.jp/blog/2014/08/08/yetanother-blastdbcmd/)

### $BLAST search of $QUERY against $DB with $Evalue

	RESULTS_DIR=results/results-$(date +%F)
	echo $RESULTS_DIR
	mkdir $RESULTS_DIR

#### Run BLAST

	#
	QUERY=data/X55059.1.1450.fasta # Bacillus anthracis
	QUERY=data/JNOD01000017.2059.3595.fasta # Bacillus anthracis
	QUERY=data/CP009968.130421.131958.fasta # Bacillus cereus E33L
	DB=data/Bacillus.fasta
	#
	QUERY=data/X75274.1.1444.fasta # Yersinia pestis
	QUERY=data/CP010067.3596048.3597576.fasta # Yersinia pseudotuberculosis str. PA3606
	DB=data/Yersinia.fasta
	#
	QUERY=data/AP010960.225071.226612.fasta # Escherichia coli O111:H- str. 11128
	QUERY=data/CP007592.3952169.3953690.fasta # Escherichia coli O157:H16
	QUERY=data/JQ904751.1.1509.fasta # Shigella sonnei
	QUERY=data/AKNB01000277.3472.4997.fasta # Shigella boydii 4444-74
	QUERY=data/X96966.1.1487.fasta # Shigella dysenteriae
	QUERY=data/CP009789.3913462.3915003.fasta # Escherichia coli K-12
	QUERY=data/X96963.1.1488.fasta # Shigella flexneri
	DB=data/Shigella.fasta
	#
	BLAST=blastn
	EVALUE=1e-20
	OUTPUT=$RESULTS_DIR/$BLAST.`basename $QUERY`.`basename $DB`.$EVALUE.out
	(time bash bin/run_blast.sh $BLAST $QUERY $DB $EVALUE > $OUTPUT &) > stderr.log 2>&1
	#tail -f stderr.log # Monitor Redirected Standard Error

#### [Parsing BLAST output](http://kirill-kryukov.com/study/tools/blast-parser/)

	perl bin/hs_blast_parser.pl < $OUTPUT > $OUTPUT.parsed
	cat $OUTPUT.parsed

#### [Turn Bio::SearchIO reports into a tabular format like blastall's "-m 9" output](http://www.bioperl.org/wiki/Bioperl_scripts)

        bin/bp_search2table.pl -f blast -i $OUTPUT -o $OUTPUT.tab
	cat $OUTPUT.tab | cut -f11

----------

## Results & Discussion
### Check data

	grep ">" SILVA_123_SSURef_Nr99_tax_silva_trunc.fasta | wc -l # 597607
	grep ">" Bacillus.fasta | wc -l # 12566

### BLASTN QUERY

	echo $OUTPUT
	cat $OUTPUT.parsed | cut -f2 | sed s/^/$'\t'/g
	cat $OUTPUT.parsed | cut -f2 | cut -d";" -f7 | sort -u | sed s/^/$'\t'/g


#### QUERY=data/X55059.1.1450.fasta # Bacillus anthracis
top 10 blastn hits contained different species (Bacillus anthracis, Bacillus cereus, Bacillus thuringiensis), 
all of which had E-Value of 0.

	X55059.1.1450 Bacillus;Bacillus anthracis
	CP010792.290057.291612 Bacillus;Bacillus anthracis
	CP009697.3496156.3497695 Bacillus;Bacillus anthracis
	CP009697.3301464.3303003 Bacillus;Bacillus anthracis
	CP009641.4319953.4321490 Bacillus;Bacillus cereus 03BB108
	CP009335.1473383.1474922 Bacillus;Bacillus thuringiensis
	CP009335.1907611.1909149 Bacillus;Bacillus thuringiensis
	CP009335.1460143.1461682 Bacillus;Bacillus thuringiensis
	CP009318.80065.81604 Bacillus;Bacillus cereus 03BB102
	CP009335.1490559.1492098 Bacillus;Bacillus thuringiensis

	# -perc_identity 100 \
	X55059.1.1450 Bacillus;Bacillus anthracis

	# -perc_identity 100 -qcov_hsp_perc 100 \ 
	0 hits

`-qcov_hsp_perc 100`では、同一配列（自分自身）にヒットしない。

#### QUERY=data/JNOD01000017.2059.3595.fasta # Bacillus anthracis
top 10 blastn hits contained different species (Bacillus anthracis, Bacillus cereus, Bacillus thuringiensis), 
all of which had E-Value of 0.

	JNOD01000017.2059.3595 Bacillus;Bacillus anthracis
	CP010088.1699352.1700888 Bacillus;Bacillus thuringiensis
	CP010852.549377.550932 Bacillus;Bacillus anthracis
	CP010852.291431.292986 Bacillus;Bacillus anthracis
	CP010792.9307.10862 Bacillus;Bacillus anthracis
	CP010852.572649.574204 Bacillus;Bacillus anthracis
	CP010088.1910301.1911838 Bacillus;Bacillus thuringiensis
	CP009968.130421.131958 Bacillus;Bacillus cereus E33L
	CP009968.5187335.5188872 Bacillus;Bacillus cereus E33L
	ABCZ02000033.7421.8952 Bacillus;Bacillus cereus W

	# -perc_identity 100 \
	# -perc_identity 100 -qcov_hsp_perc 100 \ 
	JNOD01000017.2059.3595 Bacillus;Bacillus anthracis
	CP010088.1699352.1700888 Bacillus;Bacillus thuringiensis

#### QUERY=data/CP009968.130421.131958.fasta # Bacillus cereus E33L
セレウス菌にベストヒットしたのは炭疽菌

all of top 10 blastn hits had E-Value of 0. 
Top 4 blastn hits were Bacillus anthracis.

	# -perc_identity 100 -qcov_hsp_perc 100 \
	CP010852.549377.550932 Bacillus;Bacillus anthracis
	CP010852.291431.292986 Bacillus;Bacillus anthracis
	CP010792.9307.10862 Bacillus;Bacillus anthracis
	CP010852.572649.574204 Bacillus;Bacillus anthracis
	CP010088.1910301.1911838 Bacillus;Bacillus thuringiensis
	CP009968.130421.131958 Bacillus;Bacillus cereus E33L
	CP009968.5187335.5188872 Bacillus;Bacillus cereus E33L
	ABCZ02000033.7421.8952 Bacillus;Bacillus cereus W
	CP009641.4265630.4267169 Bacillus;Bacillus cereus 03BB108
	CP009720.4507167.4508704 Bacillus;Bacillus thuringiensis

#### QUERY=data/X75274.1.1444.fasta # Yersinia pestis
top 10 blastn hits contained different species (Yersinia pestis, Yersinia pseudotuberculosis), all of which had E-Value of 0.
including:

	X75274.1.1444 Yersinia;Yersinia pestis
	AKTR01000042.1.1517 Yersinia;Yersinia pestis PY-103
	AXDI01000058.39444.40985 Yersinia;Yersinia pestis 24H
	CP010247.498176.499704 Yersinia;Yersinia pestis Pestoides G
	CP009991.1085713.1087241 Yersinia;Yersinia pestis
	CP010067.3596048.3597576 Yersinia;Yersinia pseudotuberculosis str. PA3606
	CP010067.4274742.4276272 Yersinia;Yersinia pseudotuberculosis str. PA3606
	CP010023.4559415.4560945 Yersinia;Yersinia pestis str. Pestoides B
	CP009996.430314.431842 Yersinia;Yersinia pestis
	CP009996.921547.923077 Yersinia;Yersinia pestis

	# -perc_identity 100 -qcov_hsp_perc 100 \ 
	X75274.1.1444 Yersinia;Yersinia pestis

#### QUERY=data/CP010067.3596048.3597576.fasta # Yersinia pseudotuberculosis str. PA3606
仮性結核菌にベストヒットしたのはペスト菌

Top 10 blastn hits contained different species (Yersinia pestis, Yersinia pseudotuberculosis), all of which had E-Value of 0.

	# -perc_identity 100 -qcov_hsp_perc 100 \ 
	AXDI01000058.39444.40985 Yersinia;Yersinia pestis 24H
	CP010247.498176.499704 Yersinia;Yersinia pestis Pestoides G
	CP009991.1085713.1087241 Yersinia;Yersinia pestis
	CP010067.3596048.3597576 Yersinia;Yersinia pseudotuberculosis str. PA3606
	CP010067.4274742.4276272 Yersinia;Yersinia pseudotuberculosis str. PA3606
	CP010023.4559415.4560945 Yersinia;Yersinia pestis str. Pestoides B
	CP009996.430314.431842 Yersinia;Yersinia pestis
	CP009996.921547.923077 Yersinia;Yersinia pestis
	CP009991.1705822.1707350 Yersinia;Yersinia pestis
	CP009973.1608939.1610467 Yersinia;Yersinia pestis CO92

#### QUERY=data/X96963.1.1488.fasta # Shigella flexneri
赤痢菌にベストヒットしたのは大腸菌。

	# -perc_identity 100 -qcov_hsp_perc 100 \
	CP009578.7256.8810 With: Escherichia-Shigella;Escherichia coli FAP1
	X96963.1.1488 With: Escherichia-Shigella;Shigella flexneri

#### QUERY=data/X96966.1.1487.fasta # Shigella dysenteriae

	# -perc_identity 100 -qcov_hsp_perc 100 \
	AR452370.1.1487 Escherichia-Shigella;unidentified
	X96966.1.1487 Escherichia-Shigella;Shigella dysenteriae

#### QUERY=data/JQ904751.1.1509.fasta # Shigella sonnei

	# -perc_identity 100 -qcov_hsp_perc 100 \
	JQ904751.1.1509 Escherichia-Shigella;Shigella sonnei

#### QUERY=data/AKNB01000277.3472.4997.fasta # Shigella boydii 4444-74

	# -perc_identity 100 -qcov_hsp_perc 100 \
	AKNB01000277.3472.4997 Escherichia-Shigella;Shigella boydii 4444-74

----------

## References
- [SILVA rRNA database](http://www.arb-silva.de)
- [Nucleic Acids Res. 2014 Jan;42(Database issue):D643-8. The SILVA and "All-species Living Tree Project (LTP)" taxonomic frameworks.](http://www.ncbi.nlm.nih.gov/pubmed/24293649)
- [Nucleic Acids Res. 2013 Jan;41(Database issue):D590-6. The SILVA ribosomal RNA gene database project: improved data processing and web-based tools.](http://www.ncbi.nlm.nih.gov/pubmed/23193283)

- BLAST
 - [BLAST® Command Line Applications User Manual - NCBI Bookshelf](http://www.ncbi.nlm.nih.gov/books/NBK279690/)
 - [井上 潤：Blast+](http://www.geocities.jp/ancientfishtree/BLASTplus_JI.html)
 - [Local BLAST の使い方〜検索実行・オプション編（MacOSX版）〜 2011 - 統合TV (togotv)(2011-06-08)](http://togotv.dbcls.jp/20110608.html)
 - [Local BLAST (togotv)(2011-02-25)](http://togotv.dbcls.jp/20110225.html)
- []()

----------

