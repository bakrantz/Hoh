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

This file format is supported well. It uses a header with meta data and tab-separated columns. For example: 

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

      $hoh -> delimiter("\t"); #set delimiter to tab.

## hoh

Get/set the hash of hashes (hoh) datastructure. The basic architecture is:

    my $hoh = Hoh -> new();
       $hoh -> delimiter("\t");
       $hoh -> load('some-file.txt');
    my %hash = $hoh -> hoh;
    foreach my $row (keys(%hash)) {
       foreach my $col (keys(%{$hash{$row}})) {
          my $data_point = $hash{$row}{$col};
          print "Row is $row, and Column is $col, and Data point is $data_point\n";
       };
    };

The row keys are either those issued by you in the loaded flat file as the first
column. Or if you set the generate\_keys attribute to true, then the keys are
auto-generated by [Hoh](https://metacpan.org/pod/Hoh) for you. These row keys can remain invisible to you upon saving.
To turn them off by setting the `filekeys(0)` attribute to false.

## column\_equations

Get/set this hash containing the column math equations. A recalculate routine may
be employed in the future to run through these in the order they were created. For now
it just remembers the equations in this hash.

## statistics

Get/set this multidimensional hash is the complete dataset of all statistics computed on the ["hoh"](#hoh) following
a binning method. The hash structure is complex.

    =head2 statistics_data

Get/set this hash is a hash of hohs, containing the datasets of all statistical values computed in a binning
analysis. This hash of hohs is useful for saving these data to a series of infividual flat files, or
they may used in other routines.

## bin\_data

Get/set this hash is a hash of hohs, containing the primary rows from the ["hoh"](#hoh) divided among the various 
bins so that they may be saved as separate flat files or so that they can be processed by other
routines.

## bins

Get/set this array of bins threshholds. This may be used in various contexts. For tolerance binning, bins should
be set as a list of the bin centers only. But for other types of binning, this acts as an array of arrays,
where each row is a bin and the zeroth column is the min value of the bin and the 1st column is the max value 
of that bin.

## print\_order

Get/set this array of column names that will be printed in the order given to the saved output flat file.

## statistics\_print\_order

Get/set this array of statistical parameters that will be printed in the order given to the saved statistical output flat file.

## statistics\_columns\_to\_analyze

Get/set this array of columns in which statistical analyses will be performed when binning the data.

## column\_names

Get this array of the columns in the [Hoh](https://metacpan.org/pod/Hoh).

## sorted\_keys

Get/set this array of sorted row keys. This array is used when saving the data as a flat file in a particular sort order.
The sort order was determined by the ["sort\_hoh"](#sort_hoh) method.

## original\_keys

Get/set this array of the original order of the row keys when the file was opened. These are saved in the module in case
upon saving it is desired to print the data in the original order it was opened in.

## sort\_column

Set/get the column name as a scalar that will be used to sort the ["hoh"](#hoh).

    $hoh -> sort_column('D'); #sort using column D

See also the method ["sort\_hoh"](#sort_hoh).

## sort\_type

Set/get the type of sort, where logical false, or `0`, is a numerical comparison and logical true, or `1`,
is a lexicographical string comparison. See [http://perldoc.perl.org/perlop.html#Equality-Operators](http://perldoc.perl.org/perlop.html#Equality-Operators). See 
also the method ["sort\_hoh"](#sort_hoh).

## unsorted

Set/get this boolean to logical true if you do not want the ["hoh"](#hoh) sorted upon saving.

    $hoh -> unsorted(1); #Do not sort the hoh

See also the method ["sort\_hoh"](#sort_hoh).

## sort\_order

Set/get this boolean to determine whether the sort will be ascending or descending.
Logical false, or `0`, is ascending; and true is descending. See also the method ["sort\_hoh"](#sort_hoh).

## file

Set/get a full path/filename of the file that you will load into the [Hoh](https://metacpan.org/pod/Hoh) object.

    $hoh -> file('a-text-file.txt');
    $hoh -> load;

## file\_out

Set/get a full path/filename of the file that you will save the [Hoh](https://metacpan.org/pod/Hoh) object as a flat file.

    $hoh -> file_out('output-text-file.txt');
    $hoh -> save;

## file\_type

Get/set the file type designation made upon testing and loading a file into a [Hoh](https://metacpan.org/pod/Hoh).

## file\_extension

Get/set the file extension of the loaded file. The extension includes the period.

## file\_out\_extension

Get/set the file extension of the file to be saved. The extension includes the period.

## file\_preextension

Get/set the filename portion before the extension for the file to be loaded.

## file\_out\_preextension

Get/set the filename portion before the extension for the file to be saved.

## generate\_keys

Get/set a boolean that determines whether row keys will be generated upon loading a file.
When boolean is true `1`, then keys are generated by [Hoh](https://metacpan.org/pod/Hoh), but when boolean is false `0`
then the first column of the input file are consider the row keys for the ["hoh"](#hoh). Use this
attribute in conjunction with ["filekeys"](#filekeys), which toggles whether the row keys will be saved
or not.

## key\_names

Get/set column name for the row keys column. This column name is only for file display purposes. 
It would be the first item in the header line, that is if the ["filekeys"](#filekeys) attribute is set to true.

## case\_sensitive

Get/set this boolean that determines whether the data will be treated in a case-sensitive manner.
If set to false `0` (default), then all file data is converted to upper case. When set to true,
the data is not forced to upper case and is, therefore, case sensitive.

## filekeys

Get/set this boolean that determines whether the row keys will be printed when the file is saved.
Logical false `0` (default) means they will not be saved. Logical true means they will.

## tolerance

Get/set this scalar number when doing either a linear or tolerance binning of the data.

## binned\_column

Get/set this scalar string value for the name of the column to be used to determine the binning of the hoh,
when performing statistical analysis.

## bins\_per\_decade

Get/set this scalar number as the number of bins per decade when performing logarithmic binning of the hoh.

## bin\_number

Get/set this scalar integer as the total number of bins used in binning the hoh.

## header\_off

Get/set this boolean that determines whether the header is printed in the saved flat file output.
Logical false `0` (default) means it will be saved. Logical true means it will not be saved.

## header

Get/set this scalar string of the header. Generally you would not want to alter this, since it 
is composed automatically as the delimted list of column names. But you can alter it prior to saving
if desired.

## original\_header

Get/set this scalar string of the original file header from the loaded text file. This is used when
opening the ATF format files.

## original\_header\_length

Get/set this scalar integer as the number of lines in the header of the original loaded text file.

## column\_number

Get/set this scalar integer as number of columns in the ["hoh"](#hoh).

## probable\_column\_number

Get/set this scalar integer as the number of columns the pre-loading routine believes are present in the data
given a particular ["delimiter"](#delimiter).

## first\_line

Get/set this scalar string as the first line of the text file upon loading. This line is used to
test the file type. 

## print\_order\_file

Set/get a full path/filename of the print order file. The file contains the list of columns you want to save.

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

## cs

This is largely an internal method that operates on strings to set them to the appropriate
case-senstivity given the setting of the boolean attribute ["case\_sensitive"](#case_sensitive). I describe it in detail to 
understand how case-sensitivity is dealt with remember the default is for ["case\_sensitive"](#case_sensitive) is `0`, which
means the keys of the hash and text data are converted to ALL CAPS:

    my $text = 'A case-Sensitive phrase.';
    $hoh -> case_sensitive(1); #set to true
    my $cs_text = $hoh -> cs($text);
    print $cs_text; #shows 'A case-Sensitive phrase.'

But

    my $text = 'A case-Insensitive phrase.';
    $hoh -> case_sensitive(0); #set to false
    my $cs_text = $hoh -> cs($text);
    print $cs_text; #shows 'A CASE-INSENSITIVE PHRASE.'

This method is internally evaluated throughout many of the methods, and it can lead to bugs if you are
not in tune with using ALL CAPS by default.

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

See the ["load"](#load) method since ["open\_hoh"](#open_hoh) is an alias.

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

Save the ["hoh"](#hoh) hash data to a flat text file. The arguments are as follows: filename (scalar, required), print order (array
ref, not required), sort column (scalar, not required), boolean to decide if head is not printed (not required, `/0` means header is printed, where default is `0`), delimiter (scalar, not required), and filekeys boolean (not required, for default `/0` the row 
keys are not printed to the output file). Set filekeys to true if you want to see those keys in the output file.

    $hoh -> save($file, $print_order, $sort_column, $header_off, $delimiter, $fileKeys);

These are all attributes in the [Hoh](https://metacpan.org/pod/Hoh) object so they may be set elsewhere in your implementation.

    $hoh -> out_file($file); #for saving set the out_file attribute
    $hoh -> print_order(qw(D F Z X)); #the order in which the columns are printed
    $hoh -> sort_column('F'); #how the hoh is sorted prior to print
    $hoh -> header_off(1); #the header will not be printer when logical true
    $hoh -> filekeys(1); #will print the row keys from the hash table when logical true
    $hoh -> delimiter("\t"); #tab

## sort\_hoh

This method sorts the ["hoh"](#hoh) data structure according to any selected column. The method has three arguments:

    my %sorted_hash = $hoh -> sort_hoh($sort_column, $sort_order, $sort_type) -> hoh;

But you set these attributes in the [Hoh](https://metacpan.org/pod/Hoh) object itself. The ["sort\_column"](#sort_column) is the column used to determine
the sort order of the hoh. The ["sort\_order"](#sort_order) is a boolean where 0 is ascending and 1 is descending. Finally,
["sort\_type"](#sort_type) is a boolean where 0 is numerical `<=>` and 1 is lexicographical `cmp`.

## statistics\_log\_bin

This method creates a series of log-scale-spaced bins across the selected column data range.

    my ($bin_col, $bins_per_decade, $cols_to_avg) = ('E', 30, [ qw(A B C D E) ] );
    my %stats = $hoh -> statistics_log_bin($bin_col, $bins_per_decade, $cols_to_avg) -> statistics;

The method takes three arguments. The scalar column to use for binning (required), the scalar number 
of bins per decade (required), and the array\_ref list of columns to average (defaults to all columns 
if not specified). This statistics method and those that follow create a hash statistical data on each 
of the bins: count, sum, mean, variance, standard deviation, sum of squares, and mean sum of squares. 
These results of the binning (histogramming) process are recorded in the ["statistics"](#statistics) attribute, 
which is a hash that is accessed as shown in the code example above. The keys to the hash are set as 
'BIN'.$index where $index starts at zero and autoincrements.

## statistics\_linear\_bin

This method creates a series of linear-scale-spaced bins across the selected column data range.

    my $tolerance = 2;
    my %stats = $hoh -> statistics_linear_bin($bin_col, $tolerance, $cols_to_avg) -> statistics;

Here the tolerance argument of the method is the linear step size for each bin. For example, if the 
data range in the column used for binning covers 1-100, then a tolerance value of 2 would create about 
50 bins. The columns to average argument is like stated above in ["statistics\_log\_bin"](#statistics_log_bin). Again upon 
execution, the resulting statistics are access in the ["statistics"](#statistics) attribute as shown above.

## statistics\_bin\_tolerance

This method creates a series of bins across the selected column data range given a list of values to bin around
and a tolerance. The method effectively reduces a dataset by binning and averaging data that fit within each
of the bins set by the array ref, $bin, +/- the scalar tolerance $tol.

    my $bins = [ 10, 20, 30, 40, 55 ];
    my $tolerance = 1;
    my %stats = $hoh -> statistics_bin_tolerance($bin_col, $bins, $tolerance, $cols_to_avg) -> statistics;

The arguments are the same as above, where the new argument $bins is added (an array reference to a list of bin centers).
Thus in this method data are binned based on the bin center and the tolerance as plus or minus the bin center. The bins
will be in this example:

    BIN1 9 - 11
    BIN2 19 - 21
    BIN3 29 - 31
    BIN4 29 - 41
    BIN5 54 - 56

## bin\_search

This method allows for statistical analysis of user selected bins. Using the case above, one could manually define the bins
as:

    @bins = (
             [9, 11],
             [19, 21],
             [29, 31],
             [29, 41],
             [54, 56],
            );
    $hoh -> bins(@bins);
    $hoh -> binned_column('X'); #setting the attribute to the binned column (data used to sort into bins)
    my %stats = $hoh -> bin_search -> compute_statistics_on_bins -> statistics;

Or the functional method call for this example is:

    my %stats = $hoh -> bin_search(\@bins, 'X') -> compute_statistics_on_bins -> statistics;

Thus with this method an arbitrary arrangement of bins can be constructed and analyzed. As noted in the above
examples a second method, ["compute\_statistics\_on\_bins"](#compute_statistics_on_bins), was invoked to compute all the statistics. 
The ["bin\_search"](#bin_search) method just dumps the data into the statistics hash initially. To get more advanced analysis
(if you need it) you then invoke the ["compute\_statistics\_on\_bins"](#compute_statistics_on_bins) method in series. The ["statistics"](#statistics) hash
is modified appropriately.

## compute\_statistics\_on\_bins

This method is used in manual bin averaging of the data. It is called typically after a ["bin\_search"](#bin_search) method.
See above code example for ["bin\_search"](#bin_search).

## save\_statistics

Method that is called to save the ["statistics"](#statistics) to a formatted text file that is [Hoh](https://metacpan.org/pod/Hoh) compatible. There are
four arguments:

    $hoh -> save_statistics($file_out, $print_order, $stats_print_order, $delimiter);

The first is the scalar for a `path/to/file-out-name` (required), which can be specified in the <L/file\_out> 
attribute. The second is an array reference to the print order (not required), which as described above is
the <L/print\_order> attribute. The third is a statistics print order array reference (not required), which 
is the order of particular statistical quanities that are to be printed in the flat file. The statistics hash 
is larger than two-dimensions, and some ordering of the output is required to make it two-dimensional. 
The default is `['MEAN', 'STANDARD_DEVIATION']`, which yields the average of the column for that bin and the standard 
deviation (error). Finally, the delimiter (not required) is also found in the ["delimiter"](#delimiter) attribute. As always,
these arguments can be left out if they were pre-set in the corresponding attributes.

## save\_as\_chimera\_attribute

This method is a way to convert a given column in the ["hoh"](#hoh) to a UCSF `CHIMERA` attribute. 
See [http://www.cgl.ucsf.edu/chimera/](http://www.cgl.ucsf.edu/chimera/) for information on the purpose of attributes. Note to use this script, 
the input file must be specified with the residues listed as the keys in the first column. The file is loaded 
with the ["generate\_keys"](#generate_keys) attribute set to `0`. Then a column with the attribute values is selected for export 
to the attribute file. The output filename is generated from the chimera attribute name argument:

    $hoh -> save_as_chimera_attribute($chimera_attribute_name, $col_with_attribute);

# AUTHOR

Bryan Krantz <bakrantz@gmail.com>

# COPYRIGHT AND LICENSE

Copyright (c) 2014 Bryan Krantz

This software is licensed under the Artistic License 2.0

This is free software; you can redistribute it and/or modify it under
the same terms as the Perl 5 programming language system itself.
