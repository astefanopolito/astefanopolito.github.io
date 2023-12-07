---
layout: page
title: Awkcel 
permalink: /Awkcel/
---

## Project Description

Awk is a very unique scripting language used in Linux. It's technically
a command, such as "cd" or "ls", but for input it accepts written scripts, like
an interpreter. It is especially useful for data manipulation and analysis,
since it's primary feature is scanning documents line by line, matching and
manipulating them according to how the inputted script is written.

Although its functionality is incredibly useful, it becomes more tedious when
working with tsv, or tab separated, files. There is no built in functionality
for referencing headers, as the program only runs line by line.

Thus, the goal of awkcel is to create a wrapper for awk, where columns in the
file can be referenced by their headers, and data can be collected and analyzed
solely through reference.


~~~ bash
#!/bin/bash

awkstring=$1
file=$2

#Get headers from file and store in bash array

headers=($(awk -F'\t' '
	BEGIN {
	HEADER=0
	}
	substr($1,1,1) !="#" && HEADER==0 {
	HEADER=1;
	for (i=1;i<=NF;i++) {
		print $i
	}
}' $file))

#Skip header line and comments

varstring="BEGIN {HEADER=0;"

for (( i=0 ; i < ${#headers[@]} ; i++))
do
	varstring="${varstring}COLNAMES[$i]=\"${headers[$i]}\";"
done

varstring="$varstring} substr(\$1,1,1)==\"#\"{next} HEADER==0{HEADER=1;next}{"


for (( i=0 ; i < ${#headers[@]} ; i++))
do
	#OPTIMIZATION::: Check if variable is called in the awk command passed by user
	#so unused columns are not loaded into memory needlessly

	#NOTE: This runs the risk of loading unused columns if column names are refrenced
	#not as variables but rather as parts of strings
	if [[ $awkstring =~ ${headers[$i]} ]]; then
		varstring=${varstring}${headers[$i]}=\$$(($i+1))\;
	fi
done
varstring=${varstring}}

#Load variables into user inputted awk command

awkstring="${varstring}${awkstring}"

#Finally, call awk

awk "$awkstring" $file

~~~

Below is a quick example of how the script is used, where column names can be
reference directly in the scripting language. This is an example scenario with
historical stock data, where each column header is the name of a new stock. The
goal here is to determine Amazon's worst single day performance.


SHELL COMMAND:

~~~

./awkcel 'BEGIN {
	count=0;
}

count==0{
	max=AMZN;
	count++;
}

count==1{
	max=max-AMZN;
	prev=AMZN
	prevdate=date;
	worst=date
	count++;
}

{
	if ((prev - AMZN) > max && prev!="_" && AMZN!="_") {
		max=prev-AMZN
		worst=date
	}
	prev=AMZN
	prevdate=date
}

END {
	printf "%s: %d\n", worst, max;
}
' historical.2011.tsv

---------------------------
OUTPUT:

2011-10-26: 29

---------------------------
~~~







