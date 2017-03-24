This is README file for dicomdiff (didiff) version BETA
Author: Jiří Keller, M.D., Ph.D. <keller.public+git-didiff@gmail.com>

This SW is a result of the research funded by the project Nr. LO1611 with
a financial support from the MEYS under the NPU I program.

Table of content
================

1. Introduction
2. Installation - Linux
2.1. Installation on strange/obsolete systems
3. Usage and parameters
3.1. Output modes
3.1.1 DEFAULT
3.1.2. CSV output
3.1.3. First file value
3.1.4. Same value
3.2. Additional output options
3.2.1 Print Label
3.2.2 Decimal tags
3.3 Comparison options
3.3.1. ASCII dump
3.3.2. Binary data
3.3.3. XML
3.3.4. Ignore time
3.3.5. Raw data
3.4. Other options
4. Tutorial
4.1. Are two DICOM files identical ?
4.2 what are the differences between sequences ? Are they "identical" ?
4.3 What changes should be done to our protocol to make as nice images as Site B ?
4.4. Is the dataset measured using the correct settings ? 
5. Known bugs or pitfalls
5.1. If you have binary data in your output and pipe them to grep, it will not
5.2. please be aware that CSV output changes commas to underscores in the whole
6. Sample data


-------------------------------------------------------------------------------

1. Introduction
===============

If you are short of time or "too long; did not read" (TL;DR) type of person,
please scroll down to the Tutorial section ;-).

DICOM is a well established medical image communication and storage format.
From the user's perspective, each DICOM file consists of a header and medical
data (typically image such as CT or MRI, but can be anything from spectroscopic
curve to endoscopic screenshots or photos from your vacation). In reality,
there is no difference between header and data in DICOM standard, as it
contains only key-value pairs (image data is just another "value"). In DICOM
world, the "keys" are called tags and they are referenced by (Group,Element)
pair as there is makes handling more than 2000 items in the DICOM 3 standard
easier. Each Group and Element is a hexadecimal two bytes number. The whole
list is listed in part PS3.6 of the DICOM standard (caled Data Dictionary) - see
http://dicom.nema.org/medical/dicom/current/output/html/part06.html .
Standard tags have even Group numbers, odd Group numbers are reserved for
so called "private" tags, which are typically vendor-specific and may contain
a lot of interesting information, unfortunately in most of the cases undocumen-
ted. In Standard DICOM Dictionary, tags such as patient name (0010,0010),
sex (0010,0040), age, weight, medical device which produced image and many
others are defined. And, by the way, your image is stored in PixelData tag
which is (7fe0,0010).

Dicomdiff (or didiff) is tool inspired by GNU diff, which compares two files
line by line. Our tool compares DICOM tags between two DICOM files or a DICOM
file and ASCII dump (see below).

There are three basic output modes:
	1) basic diff-like output (values from each files on separate lines)
	2) "first file value" and "the corresponding value" with tag and one value per line
	3) CSV output with tag and both values on one line

CSV (comma separated values) format can be very handy as it can be easily
processed using any Office package or an automatic script (eg. awk).

2. Installation - Linux
=======================

For compatibility reasons didiff uses python 2.x (it was developed and tested
on versions 2.7.3 and 2.7.9). It requires the following python package:

dicom - dicom support library (python-dicom package on debian)

Please, install it by typing

sudo apt-get install python-dicom

on your Ubuntu/Debian system or use your package manager or pip/easy_install or
any other package-installation solution for your system.

Didiff can be copied anywhere in your $PATH or even in /usr/bin. It does not
need any particular location in the filesystem. 

2.1. Installation on strange/obsolete systems
--------------------------------------------

If your system is VERY strange, download pydicom from the source
https://github.com/darcymason/pydicom eg.:

wget https://github.com/darcymason/pydicom/archive/master.zip
unzip master
cd pydicom-master
python setup.py install

You WILL need root privileges at least for the last command. Note that python
2.6 is required for installation of pydicom (so forget about your Maemo :-().
For manual installation of pydicom python-setuptools is needed as well.
Your mileage may vary, but please do not ask for support in this case,
only report success ;-).


3. Usage and parameters
=======================

Basic usage is didiff  inputfile1 inputfile2 [parameters].
There are output and comparison parameters. By default, didiff outputs diff-like
output (line numbers are replaced by hexadecimal DICOM tag IDs).

3.1. Output modes
-----------------

These parameters are mutually exclusive as they define the output format.

3.1.1 DEFAULT
--------------
By default, the output is diff-like, which means something like:

(0051, 100d) c (0051,100d) 
< SP L82.3
---
> SP L83.2

Which means that theere is a difference in tag 0051,100D with value being "SP L82.3"
in the first and "SP L83.2" in the second file (btw., this is relative slice 
position).

3.1.2. CSV output
-----------------
Switches  -c or --csv_output print comma separated values output without any
header. The contents is DICOM_TAG_major,DICOM_TAG_minor,value1,value2.
Above-mentioned difference outputs as a single line:

0051,100d,SP L82.3,SP L83.2

3.1.3. First file value
-----------------------
Switches -a or --diff_aval print the value in the 1st file (if values differ).
Above-mentioned difference outputs as a single line:

(0051,100d)=SP L82.3

This format is required by -A/--ASCII options (see below), so please use it
whenever you plan to create a "template" for comparison of the files.

3.1.4. Same value
-----------------

Options -s or --same forces didiff to print only DICOM attributes with the same
value in both files - it is basically inverted filter compared to 3.1.3.
The output format is again suitable for -A/--ASCII options (see below).

3.2. Additional output options
------------------------------

There are some additional output options, which can be used in one or more
output modes.

3.2.1 Print Label
-----------------
Options -l or --print_label print label (name) of DICOM tag in output. This may
be helpful, if you are not (yet) a DICOM guru, who knows all the tags by heart.

Default output mode
~~~~~~~~~~~~~~~~~~~
Format is (TagMajor, TagMinor) c (TagMajor,TagMinor) Tagname
< first file value
---
> second file value

Real-life output:

(0028, 1050) c (0028,1050) WindowCenter
< 438
---
> 446

CSV output
~~~~~~~~~~
Format is TagMajor,TagMinor,value_in_1,TagName,value_in_2:

0028,1050,438,WindowCenter,446

-a or -s output
~~~~~~~~~~~~~~~
Format is (TagMajor,TagMinor) TagName =value

(0028,1050) WindowCenter =438

3.3 Comparison options
----------------------

There are several switches which modify the loading and comparison process:

3.3.1. ASCII dump
-----------------
The option -A or --ASCII informs didiff that the first file is an ASCII output
dump (-A or -s output) and not a regular DICOM file.
Only values listed in the dump are compared (see Tutorial for hint how to make
use of this).

3.3.2. Binary data
------------------
Options -b or --bin make didiff to compare binary values as well. This includes
proprietary tags and image/spectroscopic/any data. The output includes raw
data which look as funny characters on your terminal, so you should redirect 
output of your command (see Tutorial). This option works with any output format.

3.3.3. XML
----------

Some Siemens DICOM files include a tag (0029, 1020) which contains a lot of
interesting data in XML format. The field is binary, but contains 
"MrProtocolData". This is an ASCII name=value section. Didiff can compare these
values as well using -x or --xml switch. The real XML data is ignored in the 
current release now. If you do not use this switch, a private tag 0029,1020
is IGNORED as it contains a very long string, which contains even newlines (!).

Note: For easier parsing, with CSV output, lines starts with xxxx. Please note,
that XML support is still limited and under development (should be handled as
public beta version).

3.3.4. Ignore time
------------------

If you compare two DICOMs, it may be useful not to print tags which change with
scanning the other subject the other day. If you use -t or --ignore_time
options, time- and patient-specific data are ignored. The exception is subject
age, sex, height and weight, which are preserved as you may wish to include
them as they influence image data (eg. phantom height etc). If you do not need
this data, you can easily filter them out yourself,

3.3.5. Raw data
---------------

During the early development stage, it was useful to compare not only values of
the DICOM tag, but the other properties (variable type and comment) as well.
As this may be beneficial for some users, their comparison can be enabled
using -r or --raw options.

3.4. Other options
------------------

As expected, -h or --help prints simple list of switches and parameters.

4. Tutorial
===========

Didiff was designed to be as versatile as a Swiss army knife. Following examples
may guide you through typical usage.
For all examples, please change your directory to sample_data (cd sample_data), 
which includes several example DICOM files.


4.1. Are two DICOM files identical ?
------------------------------------

If you export DICOM data from Siemens MRI scanner, they are named based on
the date and time aof export and not of acquisition. Therefore the answer may be not that
trivial. Simple command

didiff MR.T2.sym.1.1.IMA MR.T2.sym.1.1.IMA

outputs nothing (same as with canonical diff), which means that files are
equal (actually, that header info is the same as image data is not compared
by default).

didiff MR.T2.sym.1.1.IMA MR.T2.sym.1.2.IMA

outputs list of differences between two slices of the same acquisition. To see
it better, let's try CSV output:

didiff MR.T2.sym.1.1.IMA MR.T2.sym.1.2.IMA -c


The output is more compact now, but you may not know what all the tags mean.
Let's add tag description and make a nice CSV file:

didiff MR.T2.sym.1.1.IMA MR.T2.sym.1.2.IMA -c -l > test.csv
localc test.csv

Now, if Libre Office is installed in your system, a spreadsheet should open
with the differences. If asked, select a comma as a field separator.
The differences may be easily outputed using awk command:

awk -F "," '{print "("$1","$2") "$NF}' test.csv

The output should be:

(0020,0013) InstanceNumber
(0008,0018) SOPInstanceUID
(0020,0032) ImagePositionPatient
(0008,0033) ContentTime
(0020,1041) SliceLocation
(0051,100d) N/A
(0019,1016) N/A
(0028,0107) LargestImagePixelValue
(0008,0032) AcquisitionTime


Newer scanners include XML data as well, so let's try the same with the data
from Skyra with and without an extra -x option:


didiff MR.T2.sky.1.1.IMA MR.T2.sky.2.1.IMA -c -l > skyra_noXML.csv
didiff MR.T2.sky.1.1.IMA MR.T2.sky.2.1.IMA -c -l -x > skyra_XML.csv
localc skyra_noXML.csv skyra_XML.csv

Or alternatively, you can see the differences using GNU command

diff -a skyra_noXML.csv skyra_XML.csv 

This outputs a lengthy list of xml tags (please ignore "xxxx," on the beginning
of each line). Note, that without -x, data in the tag 0029, 1020 is ignored,
so there is "N/A" value in skyra_XML.csv for this tag and in the file
skyra_noXML.csv there is no info on 0029,1020 (see section 3.3.3 of this manual
for details).

If you want to see if the same coil was used, try the following magic command:

diff -a skyra_noXML.csv skyra_XML.csv|grep "sCoilSelectMeas.aRxCoilSelectData\[.\].asList\[.\].sCoilElementID\.tCoilID"


4.2 what are the differences between sequences ? Are they "identical" ?
------------------------------------------------------------------

Scenario: site A and site B measure 3D T1-weighted volumes on the
same/comparable MRI scanner. Didiff can list the differences and show if there
are any parameters, in which the sequences differ and how.
In our tutorial, we have T2-weighted dataset from Symphony and Avanto. Let's
compare the first slice:

didiff MR.T2.sym.1.1.IMA MR.T2.ava.1.1.IMA -c -l

Differences include:

DeviceSerialNumber
TimeOfLastCalibration
StudyInstanceUID
SeriesInstanceUID
SeriesNumber
SOPInstanceUID
StudyDescription
SeriesTime
ImagePositionPatient
ContentTime
SliceLocation
PerformedProcedureStepStartTime
PatientSize
FrameOfReferenceUID
PerformedProcedureStepID
StationName
InstitutionName
ImagingFrequency
EchoNumbers
ReferringPhysicianName
DateOfLastCalibration
SoftwareVersions
LargestImagePixelValue
SAR
EchoTime
AcquisitionTime
ReferencedImageSequence
ImageOrientationPatient
ManufacturerModelName
StudyTime
WindowWidth
WindowCenter

You probably do not want to see patient and time-related lines, so using the parameter -t and
omitting all the lines containing "SOP" will reduce the output from 47 lines to only 32.

4.3 What changes should be done to our protocol to make as nice images as Site B ?
---------------------------------------------------------------------------------

Site A has scanner from manufacturer 1, site B from 2. As many key tags are
not vendor-specific, basic comparison can be easily done by didiff, showing the
major differences. The subtle differences are now more problematic, as the
recent version of didiff does not "match" non-Siemens proprietary tags.
If needed by users, this can be added to the future versions.

In our tutorial, we have a copy of Symphony and Avanto sequence settings
run on 3T Siemens Skyra and nice fine-resolution T2.

didiff MR.T2.sky.2.1.IMA MR.T2.sky.3.1.IMA -c  -l -t|egrep -a -v "N/A|SOP"

As we filtered out tags with unknown label and SOP-related (if you are not
familiar with SOP, try reading DICOM specification for details - for example:
http://dicom.nema.org/medical/Dicom/2014a/output/chtml/part03/sect_10.3.html),
we can see a lot of parameters which differ - such as slice
thickness, number of averages etc. You can play with this comparison and try
to figure out the differences yourself. When done, open a spreadsheet
"differences_on_skyra.ods" where shorty commented solution is presented.


4.4. Is the dataset measured using the correct settings ? 
---------------------------------------------------------

First, the "correct" settings are exported into an ascii file and the dataset
may be tested against this "norm". This may reveal eg. changed FOV by
technician, slice order etc. Only relevant DICOM tags should be left in the
ASCII file, therefore minor "irrelevant" changes should be ignored.

Let's have a look at our tutorial data:

The high-resolution dataset should be a template. So first, we need to see
which parameters (including Siemens-specific) do not change with slice
position (excluding time- and subject-specific):

didiff MR.T2.sky.3.1.IMA MR.T2.sky.3.2.IMA -x -s -t >template.txt

now, edit template and remove tags containing age,sex,height (called Size in
DICOM headers) and weight. If you are lazy, let egrep do the job for you:

egrep -a -v "0010,00[1-4]" template.txt >template_fixed.txt

Now we have the fixed template. There should be no differences in the third
slice:

didiff template_fixed.txt MR.T2.sky.3.3.IMA -A

... but there are ! But getting rid of anything SOP-related is easy:

grep -a -v SOP template_fixed.txt >template

Now, the output of above-mentioned didiff command is empty, as expected. With
different FOV, there should be something ;-):

didiff template MR.T2.sky.4.1.IMA -A -c -l

You can try the same with Symphony 1 and 2. Fast/lazy way is a oneliner
for teplate generation - shell pipe is your friend:

didiff MR.T2.sym.1.1.IMA MR.T2.sym.1.2.IMA -x -s -t |egrep -a -v "0010,00[1-4]"  >sym.txt

and see if there are some differences in the third slice:

didiff sym.txt MR.T2.sym.1.3.IMA -A -c -l|grep -a -v SOP

Yes. Default window setting differs. Anyone cares ?

And what about the other acquisition on Symphony ?

didiff sym.txt MR.T2.sym.2.1.IMA -A -c -l|grep -a -v SOP

Some proprietary tags are not set in the second acquisition (please do not ask
ME about the reason, ask your local Siemens support, they would be very
happy about that :-) ). And obviously, series number should differ in this case.

Anyway, you probably want to ignore these tags in your comparison. This can be
done manually, or by a devilish one-liner for lazy bones (sorry for breaking
the format, but I wanted to keep it as one-liner):

didiff sym.txt MR.T2.sym.2.1.IMA -A -c |grep -a -v SOP|awk -F "," '{s=s"|^\\("$1","$2"\\)";}END{print "egrep -a -v \""substr(s,2)"|SOP\" sym.txt"}' > grepcmd.sh;sh grepcmd.sh >sym_fixed.txt

And now, what are the differences between the new template and second slice
of second acquisition ?

didiff sym_fixed.txt MR.T2.sym.2.2.IMA -A -c -l

NONE and we are DONE.

5. Known bugs or pitfalls
=========================

There are no known bugs in this release, however there are two caveats:

1. If you have binary data in your output and pipe them to grep, it will not
work without -a switch at grep side and as a "bonus" didiff crashes with
unhandeled exception "IOError: [Errno 32] Broken pipe". It is not really a bug,
you just need to fix your command ;-) or data.

2. please be aware that CSV output changes commas to underscores in the whole
output ! If you blindly copy-paste an array into ASCII template, you HAVE TO
change it back. Eg.:
"[ 1_ 2_ 3 ]" has to be "[ 1, 2, 3 ]" in your ASCII file.

Explanation: The the reason for this behaviour is in keeping comma as an unique
delimiter, so didiff changes all commas in the csv output values into under-
scores, so you can be sure that parsing csv in, say $(awk -F ","), will be OK.

3. private tag 0029, 1020 is ignored by default (can be changed
using -x switch)

6. Sample data
==============

For testing purposes, several DICOM files are provided to test didiff features
in the sample_data folder:

MR.T2.sym.1.*.IMA - Coronal T2 from symphony
MR.T2.sym.2.*.IMA - the same acquired later (second acquisition)
MR.T2.ava.1.*.IMA - The same loaded to Avanto
MR.T2.ava.2.*.IMA - The same loaded to Avanto, repeated
MR.T2.sky.1.*.IMA - The same loaded to Skyra
MR.T2.sky.2.*.IMA - The same loaded to Skyra - one parameter changed ;-)
MR.T2.sky.3.*.IMA - higher resolution coronal from Skyra
MR.T2.sky.4.*.IMA - same as 2, but FOV modified by user

