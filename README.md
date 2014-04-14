# NAME

Hoh - Hash of hashes datasheet processing and plotting environment

# VERSION

version 0.10

Very roughed out at the moment, yet it is functional.

# SYNOPSIS

    my $hoh = Hoh -> new();
    $hoh -> delimiter($tab);
    $hoh -> load($filename);

# DESCRIPTION

[Hoh](https://metacpan.org/pod/Hoh) (hash of hashes) fills the need to have plain text readable data files that can be opened and 
manipulated as columns of data. Two-dimensional data manipulation is very useful and does lend to
the use of human-readable intermediate file formats for quick inspection. The joke then becomes whether 
you will be a data farmer and use this object in your code.

In this module, column math can be carried out. Columns can be added or deleted. 
The Hoh can be sorted, and the columns can be printed in any predefined order. 
Statistics can be computed on the columns. Columns can be histogrammed or binned linearly, 
logarithmically, or by user set bin dimensions. Bins can be saved as flat files for later processing. 
Statistical summary sheets can be saved. Data may be plotted using the external module [Chart::Clicker](https://metacpan.org/pod/Chart::Clicker).

The module handles comma-separated (CSV) files, tab- or white-space separated or any user-defined ["delimiter"](#delimiter).
The ["delimiter"](#delimiter) choice does require forethought when making the data file in the first place to avoid 
splitting columns improperly. A clever user may choose a special white-space character if they suspect tab 
is being used inside data items.

Large files are supported. I tested several MB. If you want much larger, then other Tied modules
would suit you better. I could add that support here, but at the moments most files are under 10 MB.

Some special file formats are supported, including loading of Axon Text File (ATF) format. Its header
is properly parsed. Chimera attribute output files can also be produced that can be opened in UCSF Chimera.

# FLAT FILE FORMATS SUPPORTED BY HOH

There are some tricks to understanding this module and how different flat file formats are to be
processed by it, since it handles many different file structures. One should read up on what
a hash of hashes is like in Perl. [See the Perl Data Structure Cookbook](http://perldoc.perl.org/perldsc.html).

## Simple numerical flat data file

Example of a very simple plain-text file for [Hoh](https://metacpan.org/pod/Hoh) to parse:

    0,1,3
    3,5,10
    11,44,12
    3,32,0
    -1,33,0.3
    

Here there are no legal column names, and [Hoh](https://metacpan.org/pod/Hoh) will know because the first line is not written with leading 
alphabetical characters in each of the columns. Basically, if a file is just a series of columns of numbers, 
then one would want to ask the ["Hoh"](#hoh) object to add row keys (i.e. attribute ["generate\_keys"](#generate_keys) set to '1') to the data upon 
input. These row keys are the hash keys in the first dimension. Column names or the column keys are 
automatically generated then starting at A, B, C ...

## Numerical flat data file with named columns

The next level of sophistication is this file type:

    X,Y,Z
    0,1,3
    3,5,10
    11,44,12
    3,32,0
    -1,33,0.3

[Hoh](https://metacpan.org/pod/Hoh) slurps it up and considers the X,Y,Z the names of the columns. If the names collided,
then [Hoh](https://metacpan.org/pod/Hoh) has a fancy routine to prevent name collisions. Again it is recommended for this file type to set the
attribute ["generate\_keys"](#generate_keys) to true to avoid converting column X into the row keys column. The result
may not be desired if X values are repeated or you want to use the other column methods and tools, since the row 
keys are not generally used in the data manipulations. (They are index values only.)

## Numerical flat data file with named rows

The row keys may be explicitly given in the first column. For example:

    KEY1,0,1,3
    KEY2,3,5,10
    KEY3,11,44,12
    KEY4,3,32,0
    KEY5,-1,33,0.3

Now, it may be desirable to set ["generate\_keys"](#generate_keys) to false, especially if you would like to merge two datasets as
hohs overwriting missing pieces of data or some activity that requires coordination of the row index keys. 
On the other hand, if you want to append two flat data files then you may want to set ["generate\_keys"](#generate_keys) to true.
This approach is sound because the method [Hoh](https://metacpan.org/pod/Hoh) uses to form the row keys from a file is to concatenate the name 
of the file with the row number (in the original file). This way two files can be stacked without likely having key 
collisions, since the files cannot have the same path/filename.

## ATF flat data file

This file format is support well. It uses a header with meta data and tab-separated columns. For example: 

    ATF  1.0
    7    3     
    "AcquisitionMode=Gap Free"
    "Comment="
    "YTop=10000,1000"
    "YBottom=-10000,-1000"
    "SweepStartTimesMS=0.000"
    "SignalsExported=Im_Scaled,10_mv"
    "Signals="   "Im_Scaled"     "10_mv"
    "Time (s)"   "Trace #1 (pA)" "Trace #1 (mV)"
    0.000        111.694 19.7754
    0.001        112.000 19.7144
    0.002        112.000 19.7754
    0.003        112.305 19.7754

Hoh has an internal file-test and pre-open routine to scan the file for its header and open it properly even if the extension was
removed or mangled at some point. The second line says 7 3, which means 7 more lines of junk, then the column names. The 
3 means 3 columns. Hoh parses these column names in a time, current, voltage trace to rename them as T,I,V. 

By the way, all files are converted to MS-DOS line endings to allow for MAC, PC, POSIX cross-platform opening of text files.
Why couldn't there have been an agreement on these line endings years ago?

# SAVING HOH DATA TO A FLAT FILE

## Using the print\_order attribute

By now the idea of this module should be clear to open flat files into a hoh, use the magic of Perl on the hoh, including these
methods and other methods in other modules, then to save the output results as a flat file that is very human readable (if you like).
To get the columns printed in a specific order, you specify them in the ["print\_order"](#print_order) attribute, which expects a list of valid column
names. You can also specify a file set in the ["print\_order\_file"](#print_order_file) attribute to load this list of names. That file can have a tab-, 
space-, or line-ending-separated format to list the column names in the order to be printed. 

    $hoh -> print_order( qw( A B C X D W ));

## Sorting the rows

The data will be sorted automatically to the order the rows were read from the input file unless you called a ["sort\_hoh"](#sort_hoh) 
method on one of the columns. In that case, the output file will be saved using the sorted row keys.

    $hoh -> sort_hoh('D',1,0); #descending numerical <=> sort
    $hoh -> sort_hoh('C',0,1); #ascending alphalexical cmp sort

## Using the filekeys attribute

Upon printing your ["Hoh"](#hoh) data to the flat file, you may not want to see the internal ugliness of the keys anymore, especially
if you generated them upon loading. If you do not, then set ["filekeys"](#filekeys) to false. 

    $hoh -> filekeys(0);

## Save the file

Saving the file is simple. You could disable the header line with the column names if you wanted to, but you should be sure
you are using a good ["delimiter"](#delimiter) for your dataset.

    $hoh -> filekeys(0); #do not display row keys in the file
    $hoh -> delimiter("\t"); #tab delimited 
    $hoh -> print_order( qw( A B C X D W ));
    $hoh -> sort_hoh('D',0,0); #ascending numerical <=> sort
    $hoh -> header_off(0); #default is false anyway
    $hoh -> save('output-flat-file.txt');

Or you can pass all the arguments in one big line:

    $hoh -> save('output-flat-file.txt', \@print_order_array, $sort_col, $header_bool, "\t", $filekeys_bool);

It is cumbersome at the moment that arguments are not passed as hashes in any order. That is a design flaw to work out in the
future. Or just use the attributes and forget about it.

# BINNING, HISTOGRAMMING AND STATISTICAL ANALYSES

# PLOTTING AND SAVING GRAPHICAL RESULTS

# ATTRIBUTES

## delimiter

Column separator in the flat file to be opened and parsed by [Hoh](https://metacpan.org/pod/Hoh).

## hoh

The hoh data structure returned as a hash of hashes. The basic architecture is:

    my %hash = $hoh -> hoh;
    my $data_point = $hash{$row}{$col};

The row keys are either those issued by you in the loaded flat file as the first
column. Or if you set the generate\_keys attribute to true, then the keys are
auto-generated by [Hoh](https://metacpan.org/pod/Hoh) for you. These row keys can remain invisible to you upon saving.
To turn them off at that point just set the file\_keys attribute to false.

# METHODS

## add\_columns

Add one or more sparkling new columns. Pass it a list of names:

    $hoh -> add_columns(qw( A B C D E ZZ_TOP ));
    $hoh -> add_columns( '2_more' ); #will work but parser will add A in front of name
    $hoh -> add_column( 'SynName' ); #alias works too

As the above example warns, avoid using characters outside \[A-Z\]\[a-z\]\[0-9\] and \[\_\] 
(underscore). Those characters outside of the acceptable set are deleted. I 
recommend using ALL CAPS and leaving the ["case\_sensitive"](#case_sensitive) attribute to false (default). 
This just makes life easier. Also the first character of a column key name should not
be a number. If it is, then [Hoh](https://metacpan.org/pod/Hoh) will say, "I'm sorry, Dave. I'm afraid I can't do that", 
and it will place a letter in front of your column name. 

Why all these rules? Well, Hoh was written to inteface with other software that follows
this naming convention.

## add\_scalar\_column

Adds a new column and sets it to the desired scalar value. Can be used to reset an
existing column to a new value or can clear that column if set to undef.

     $hoh -> add_scalar_column('TheAnswerToTheUniverse', 42);
    

## column

Returns the named column's values. The only argument is the column name itself. 
The hoh data are by their nature pseudorandom, and thus the data can be pre-sorted
by issuing a sort\_hoh method prior to calling the column method. The last sort is remembered.
This method does return the list of values as an @array.

    my @column_data = $hoh -> column('X');
    

In some sense, ["column"](#column) acts like an read-only attribute, but I am calling it a method, since I rule this universe.
Basically, the reason for my distinction is that separate methods may be invoked to set the column values. For example,
["column\_math"](#column_math) and ["add\_scalar\_column"](#add_scalar_column).

## column\_math

Performs column math on the Hoh by using 'col(column\_name)' notation. For example:

    my $hoh = Hoh -> new();
    $hoh -> delimiter(','); #comma-separated file
    $hoh -> load('XYZ-data.txt'); #dataset with X, Y, and Z headed columns
    $hoh -> add_column('A'); #add new empty column called 'A'
    $hoh -> column_math('A', 'col(X) + col(Y) * col(Z) - sin(col(X))');
    

Simple column math routine for Hoh using native Perl interpreter, allowing for
application of any Perl function, except 'col'. First argument is where the column math result goes; 
the second argument is the column math itself written as above. I know that at the moment this is
not something that is safe to do security wise, and I would appreciate comments on how
to have this powerful feature without compromising the security too much.

## extract\_column\_names

Method is run to re-extract the column names after changing them with various methods above.
The method does not return the names. Instead you access the names with the ["column\_names"](#column_names) attribute.

    my @names = $hoh -> extract_column_names -> column_names;  

Because of the behavior of the module you would rarely need to call this method, that is unless you
poked into the hoh data structure manually and wanted to reconfirm the column names.

## file\_to\_hoh

Generally, you would rather use the ["load"](#load) or ["open\_hoh"](#open_hoh) methods over this one, 
but I document it anyway for those that want to type less if you are using default formatting 
for your flat file, etc.

The ["file\_to\_hoh"](#file_to_hoh) method loads a flat file into a [Hoh](https://metacpan.org/pod/Hoh) object, but unlike ["load"](#load) it returns 
the %hash. To use this method properly, it would be good to know the filetype for this method:

    my %hash = $hoh -> file_to_hoh($file, $delimiter, $case_sensitive, $generate_keys, $file_type);

Each of these arguments are attributes in the [Hoh](https://metacpan.org/pod/Hoh) object. The ["file"](#file) attribute is the full path/filename.
The ["delimiter"](#delimiter) has been described above. The ["case\_sensitive"](#case_sensitive) boolean is defaulted to false, meaning the 
file will be uppercased upon loading. Set it to true if you want mixtures of cases. The ["generate\_keys"](#generate_keys) 
attribute was also described above. The last attribute is ["file\_type"](#file_type), and it should be given one of four
possible file types:

    'TEXT_USERCOLUMNS' #First line contains the column names
    'TEXT_AUTOCOLUMNS' #No column names give, asking Hoh to autogenerate them
    'ATF_TIVCOLUMNS'   #The ATF file with T, I, and V columns
    'ATF_COLUMNS'      #The ATF file with unique numbers of columns from TIV format

## generate\_header 

Generates the ["header"](#header) line, which here is the delimiter separated list of column names. The column names
are generated by the magic of Perl's autoincrement feature for strings. It takes two arguments, but really 
you would never want to use them. These are the column number and delimiter. These ought to be known:

    $hoh -> generate_header($col_num, $delimiter);

These ["column\_number"](#column_number) and ["delimiter"](#delimiter) are already attributes in [Hoh](https://metacpan.org/pod/Hoh) and may already be known upon
file loading. This method is used internally for the most part. The
result is stored in the ["header"](#header) attribute.

## generate\_header\_from\_column\_names

Generates the ["header"](#header) line, which here is the delimiter separated list of known column names. 

    $hoh -> generate_header_from_column_names($columns, $delimiter);

The ["column\_names"](#column_names) attribute is used when columns are not given. The
result is stored in the ["header"](#header) attribute.

## load

Method to load a flat file into a [Hoh](https://metacpan.org/pod/Hoh) object. Same method as ["open\_hoh"](#open_hoh). It open the file and return `$self`,
meaning that there can be a series daisy-chained command calls.

    $hoh -> load('flat-file.txt') -> add_columns(qw(D E F G H)) -> column_math('D', 'col(A) + col(B)');
    

The ["load"](#load) method can take arguments beyond the filename, alebit these could be set using the corresponding attributes.
These arguments are:

    $hoh -> load($file, $delimiter, $case_sensitive, $generate_keys);

Each of these arguments are attributes in the [Hoh](https://metacpan.org/pod/Hoh) object. The ["file"](#file) attribute is the full path/filename.
The ["delimiter"](#delimiter) has been described above. The ["case\_sensitive"](#case_sensitive) boolean is defaulted to false, meaning the 
file will be uppercased upon loading. Set it to true if you want mixtures of cases. The ["generate\_keys"](#generate_keys) 
attribute was also described above.

## open\_hoh

See the ["load"](#load) method since it is an alias.

## open\_print\_order\_file

Opens a flat file that lists the names of the columns you want to save from your Hoh back into a flat file. The
file looks like this, for example:

    COL_A
    COL_B
    COL_D
    COL_A2

Method is run like this:

    $hoh -> open_print_order_file('my-po-file.txt');

## save

Save the ["hoh"](#hoh) hash data to a flat text file. The arguments are
$file, $print\_order, $sort\_column, $header\_off, $delimiter, $fileKeys

## sort\_hoh

This method sorts the ["hoh"](#hoh) data structure according to any selected column. The method has three arguments:

    $hoh -> sort_hoh($sort_column, $sort_order, $sort_type);

But you set these attributes in the [Hoh](https://metacpan.org/pod/Hoh) object itself. The ["sort\_column"](#sort_column) is the column used to determine
the sort order of the hoh. The ["sort\_order"](#sort_order) is a boolean where 0 is ascending and 1 is descending. Finally,
["sort\_type"](#sort_type) is a boolean where 0 is numerical `<=>` and 1 is lexicographical `cmp`.

# AUTHOR

Bryan Krantz <bakrantz@gmail.com>

# COPYRIGHT AND LICENSE

Copyright (c) 2014 Bryan Krantz

This software is licensed under the Artistic License 2.0

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.
