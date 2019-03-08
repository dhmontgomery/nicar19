Favorite code snippets: A responsibly sourced, artisanal list from the LCC
========================================================
author: Allie Kanik & David Montgomery
date: 2019-03-08
autosize: true
width: 1440
height: 900

Merging CSVs
========================================================

One of the tasks I do regularly but have to look up almost every time I do it is merging multiple CSVs (or other spreadsheets) into one.

I'm going to walk you through how to do it in R, and then show ways to do it in Python and Bash as well.

### The basics

This works if your CSVs all have the same structure: 


```r
library(tidyverse) # Load the Tidyverse package
filenames <- dir(path = ".", pattern = ".csv", full.names = TRUE) # Store a list of the filepaths in question
combined_data <- map(filenames, read_csv) %>% # Apply `read_csv()` to every item in `filenames` (our CSVs)
	reduce(rbind) # And combine them into one speadsheet
write_csv(combined_data, "combined_data.csv") # Optionally, re-export back to CSV. Otherwise analyze in R.
```

Getting more complicated
========================================================

What if your CSVs aren't neat and tidy, and you need to do stuff to them before you import & merge them? The `map()` and `reduce()` workflow still functions.

For example, if we need to skip the first three lines of each CSV: 


```r
filenames <- dir(path = ".", pattern = ".csv", full.names = TRUE) 
combined_data <- map(filenames, read_csv, skip = 3) %>% # Append arguments at the end, separated by comma
	reduce(rbind) 
write_csv(combined_data, "combined_data.csv") 
```

Alternately: ` map(filenames, ~ read_csv(.x, skip = 3))`

What if they're not CSVs?
========================================================


```r
filenames <- dir(path = ".", pattern = ".csv", full.names = TRUE) 
combined_data <- map(filenames, readxl::read_excel) %>% # Substitute read_excel for read_csv
	reduce(rbind)
write_csv(combined_data, "combined_data.csv") # Output in whatever format you want
```

What if we want to change our CSVs in merging?
========================================================

A common goal I have, when merging CSVs, is to make a note in the merged spreadsheet of which CSV each row of data originally came from. This is especially the case where, say, the names of the CSVs contain valuable data in and of themselves, like a date that the report was generated. 


```r
filenames <- dir(path = ".", pattern = ".csv", full.names = TRUE) 
short_filenames <- dir(path = ".", pattern = ".csv") # Store short versions of our filenames
combined_data <- map2(.x = filenames, .y = short_filenames, # Pass both our vectors to `map()`
					  ~read_csv(.x) %>% mutate(filename = .y)) %>% # Read in from `filenames` and use `short_filenames` to edit.
	reduce(rbind) 
write_csv(combined_data, "combined_data.csv") 
```

This follows the general Tidyverse and R rules. You could apply `str_remove_all()` to `.y`, for example, if you wanted to remove ".csv" from them. 

The Python way
========================================================

```
import os

data_dir = '/Users/akanik/data/csv/'
file1 = 'whatever-data-1.csv'
file1_path = data_dir + file1
#Make sure your combined file does not live within your data_dir.
#This will cause an endless loop of writing
combined_file = '/Users/akanik/data/whatever-data-ALL.csv'

fout = open(combined_file,'a')

# first file:
for line in open(file1_path):
    fout.write(line)
# now the rest:    
for file in os.listdir(data_dir):
    if file != file1 and file.endswith('.csv'):
        f = open(data_dir + file)
        f.next() # skip the header
        for line in f:
            fout.write(line)
        f.close() # not really needed
fout.close()  
```

The csvkit way
========================================================